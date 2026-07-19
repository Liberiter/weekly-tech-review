# 2026-07-06. Delta Lake — "그냥 파일 더미"가 어떻게 '테이블'이 되는가

> 레이크하우스의 맨 아래 **저장 포맷** 층을 확대한다. Databricks의 심장인 Delta Lake는, 사실 놀랍도록 단순한 아이디어 하나로 시작한다.
>
> **Parquet 파일 더미 + 트랜잭션 로그 한 폴더.** 이 조합이 왜 "데이터 레이크 위의 데이터베이스"를 가능하게 하는지 밑바닥부터 뜯어본다.

---

## 목차

1. [도입: 왜 "그냥 Parquet"으로는 안 됐나](#1-도입-왜-그냥-parquet으로는-안-됐나)
2. [한 문장 정의 — Delta Lake = Parquet + 트랜잭션 로그](#2-한-문장-정의--delta-lake--parquet--트랜잭션-로그)
3. [`_delta_log` 폴더를 열어보다](#3-_delta_log-폴더를-열어보다)
4. [ACID는 실제로 어떻게 보장되나 — 낙관적 동시성 제어](#4-acid는-실제로-어떻게-보장되나--낙관적-동시성-제어)
5. [킬러 기능 3종 — 타임트래블·스키마·MERGE](#5-킬러-기능-3종--타임트래블스키마merge)
6. [성능의 비밀 — OPTIMIZE·Z-Order·Liquid Clustering·Deletion Vectors](#6-성능의-비밀--optimizez-orderliquid-clusteringdeletion-vectors)
7. [UniForm — Iceberg와의 화해](#7-uniform--iceberg와의-화해)
8. [정리 & SI 관점의 시사점](#8-정리--si-관점의-시사점)
9. [더 살펴볼 자료](#9-더-살펴볼-자료)

---

## 1. 도입: 왜 "그냥 Parquet"으로는 안 됐나

데이터 레이크의 원래 모습은 소박했다. **S3(또는 HDFS) 같은 값싼 저장소에 Parquet 파일을 잔뜩 쌓는다.** 컬럼 지향 포맷이라 압축도 잘 되고 분석 쿼리도 빠르다. 여기까지는 좋다.

문제는 그 파일 더미를 **"테이블"처럼 다루려는 순간** 터진다.

- **동시성**: A가 파일을 읽는 중에 B가 새 파일을 쓰고 옛 파일을 지우면? A는 반쯤 갱신된, 깨진 상태를 본다.
- **원자성**: 100개 파일을 새로 쓰다가 50번째에서 잡이 죽으면? 반만 반영된 쓰레기 상태가 남는다. "전부 되든가, 전부 안 되든가"가 안 된다.
- **수정·삭제**: GDPR 때문에 특정 사용자 레코드 하나를 지워야 한다. Parquet은 불변(immutable)이라 파일 전체를 다시 써야 한다. UPDATE/DELETE 개념 자체가 없다.
- **"지금 이 테이블이 뭐냐"의 정의 부재**: 폴더 안 파일이 진짜 데이터인지, 쓰다 만 임시 파일인지, 지웠어야 할 파일인지 아무도 모른다. `ls`가 곧 테이블 정의인 셈.

> 한마디로: **파일 시스템은 "폴더 안의 파일 목록"은 알아도, "이 테이블의 올바른 상태"는 모른다.**

데이터 웨어하우스(Oracle, Redshift…)는 이 문제를 수십 년 전에 풀었다. 트랜잭션, 커밋, 롤백. 하지만 그건 비싸고 닫힌 시스템이었다. **"레이크의 개방성·저렴함은 그대로 두고, 웨어하우스의 신뢰성만 얹을 수 없을까?"** — 이 질문의 답이 Delta Lake다.

---

## 2. 한 문장 정의 — Delta Lake = Parquet + 트랜잭션 로그

Delta Lake의 정의는 정말 이게 전부다.

> **데이터는 여전히 평범한 Parquet 파일로 저장한다. 단, 그 옆에 `_delta_log`라는 폴더를 두고 "무슨 일이 언제 일어났는지"를 순서대로 기록한다.**

핵심 발상의 전환은 이렇다.

- **파일 목록(`ls`)이 테이블을 정의하지 않는다.**
- **트랜잭션 로그가 테이블을 정의한다.**

즉, "이 테이블의 현재 상태 = 로그를 처음부터 끝까지 재생(replay)한 결과"다. 폴더에 파일이 1,000개 있어도, 로그가 "그중 진짜 유효한 건 이 300개"라고 말하면 테이블은 300개짜리다. 나머지 700개는 그냥 아직 안 지워진 과거의 흔적일 뿐.

이 단순한 규칙 하나에서 ACID·타임트래블·스키마 관리·동시성이 전부 파생된다. 이게 Delta Lake를 이해하는 **단 하나의 열쇠**다.

---

## 3. `_delta_log` 폴더를 열어보다

Delta 테이블 폴더를 실제로 열면 이렇게 생겼다.

```
/mytable/
├── _delta_log/
│   ├── 00000000000000000000.json   ← 커밋 0 (테이블 생성)
│   ├── 00000000000000000001.json   ← 커밋 1 (데이터 추가)
│   ├── 00000000000000000002.json   ← 커밋 2 (일부 행 삭제)
│   ├── ...
│   └── 00000000000000000010.checkpoint.parquet  ← 10개마다 요약
├── part-0000-....snappy.parquet     ← 실제 데이터
├── part-0001-....snappy.parquet
└── part-0002-....snappy.parquet
```

### 3.1 커밋 파일(JSON) — "이번에 뭘 했나"

각 커밋은 번호가 매겨진 JSON 한 개다. 안에는 **"이 트랜잭션이 어떤 파일을 추가(`add`)하고 어떤 파일을 제거(`remove`)했는지"**가 액션 목록으로 들어있다. 개념적으로:

```json
// 00000000000000000002.json  (일부 행을 지운 커밋)
{"remove": {"path": "part-0000-....parquet", "dataChange": true}}
{"add":    {"path": "part-0003-....parquet", "size": 812, "stats": "{\"numRecords\":40,...}"}}
{"commitInfo": {"operation": "DELETE", "timestamp": 1751000000000}}
```

포인트: **삭제조차 "옛 파일 remove + 새 파일 add"라는 기록의 추가**로 표현된다. 데이터 파일 자체는 불변이고, 변하는 건 로그뿐이다. (이걸 로그 구조 저장, *log-structured*이라 부른다.)

### 3.2 통계(stats) — 쿼리를 건너뛰게 해주는 힌트

`add` 액션 안의 `stats`를 주목하자. 각 파일의 **행 개수, 컬럼별 min/max 값**이 로그에 함께 적힌다. 그래서 `WHERE date = '2026-07-01'` 쿼리가 오면, Delta는 파일을 열어보지도 않고 로그의 min/max만 보고 **관련 없는 파일을 통째로 건너뛴다**(data skipping). 로그가 곧 인덱스 역할을 한다.

### 3.3 체크포인트 — 로그가 길어지는 걸 막는 장치

커밋이 수천 개 쌓이면 매번 처음부터 재생하는 게 느려진다. 그래서 Delta는 **10커밋마다 그때까지의 상태를 하나의 Parquet(checkpoint)로 압축 요약**한다. 읽을 때는 "가장 최근 체크포인트 + 그 이후 커밋 몇 개"만 재생하면 된다.

> **핵심 통찰 — "테이블은 명사가 아니라 동사다"**
>
> 전통 DB에서 테이블은 *고정된 무언가*였다. Delta에서 테이블은 **"지금까지 일어난 모든 변경의 합"**, 즉 이벤트의 누적이다. 이 관점이 타임트래블(과거 시점 재생), 감사(누가 언제 뭘 바꿨나), CDC(변경분만 추출)를 전부 *공짜로* 만든다. 로그를 남겼더니 부수 효과로 강력한 기능들이 딸려온 것이다.

---

## 4. ACID는 실제로 어떻게 보장되나 — 낙관적 동시성 제어

"트랜잭션 로그가 있다"는 건 알겠는데, **여러 명이 동시에 쓰면** 어떻게 충돌을 막을까? Delta는 락(lock)을 거의 안 쓴다. 대신 **낙관적 동시성 제어(Optimistic Concurrency Control)**를 쓴다.

과정은 이렇다.

1. **읽기**: 작업 시작 시점의 최신 버전(예: 커밋 5)을 읽는다.
2. **작업**: 자기 할 일을 로컬에서 계산한다. (새 Parquet 파일 쓰기 등)
3. **커밋 시도**: "다음 번호인 커밋 6 파일을 만들겠다"고 시도한다.
4. **충돌 검사**:
   - 아무도 안 건드렸으면 → 커밋 6 생성 성공. 끝.
   - 그 사이 누가 먼저 커밋 6을 만들었으면 → **내 커밋은 실패**. 나는 커밋 6을 다시 읽어 "내 변경과 겹치는가"를 검사하고, 안 겹치면 커밋 7로 재시도한다.

여기서 **"커밋 N번 파일을 만드는 행위"가 원자적**이어야 이 모든 게 성립한다. 파일 하나를 "이미 있으면 실패"로 만드는 것 — 이게 원자성의 마지막 못이다.

> ⚠️ **SI에서 반드시 확인할 지점**: S3는 오랫동안 이 "동일 이름 파일의 원자적 생성"을 보장하지 못했다(그래서 과거엔 별도 커밋 서비스가 필요했다). 지금은 조건부 쓰기(conditional put)로 대부분 해결됐지만, **어떤 스토리지(S3/ADLS/GCS/온프렘)에 올리느냐에 따라 동시 쓰기 보장 방식이 달라진다.** 멀티 라이터(writer) 구성 시 반드시 점검할 것.

---

## 5. 킬러 기능 3종 — 타임트래블·스키마·MERGE

로그 구조 하나에서 파생되는 대표 기능들이다.

### 5.1 Time Travel — 과거로 쿼리하기

로그가 버전별로 다 남아있으니, "커밋 3 시점의 테이블"을 그대로 재생할 수 있다.

```sql
-- 버전으로
SELECT * FROM mytable VERSION AS OF 3;
-- 시각으로
SELECT * FROM mytable TIMESTAMP AS OF '2026-07-01 09:00:00';
```

용도: **실수 복구**(잘못 덮어쓴 테이블 되돌리기), **재현 가능한 ML**(그때 그 데이터로 학습), **감사·디버깅**. 실무에서 "어제까진 맞았는데?"를 검증하는 최강 도구다.

### 5.2 스키마 강제 & 진화 (Schema Enforcement & Evolution)

- **강제(Enforcement)**: 테이블 스키마와 안 맞는 데이터를 쓰려 하면 **거부**한다. 레이크가 "쓰레기장(data swamp)"이 되는 걸 막는 최소한의 방벽.
- **진화(Evolution)**: 그런데 진짜 새 컬럼이 필요하면 `mergeSchema` 옵션으로 **의도적으로** 스키마를 넓힐 수 있다. 엄격함과 유연함을 옵션 하나로 오간다.

### 5.3 MERGE — Upsert와 CDC의 심장

Delta의 진짜 실무 무기. "있으면 업데이트, 없으면 삽입"을 SQL 한 방에.

```sql
MERGE INTO target t
USING updates u  ON t.id = u.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

이 한 구문이 **CDC(변경 데이터 캡처) 파이프라인의 핵심**이다. 소스 DB의 변경분을 받아 레이크하우스에 반영하는 일 — ETL 파이프라인 설계의 심장이 바로 이 `MERGE`다.

---

## 6. 성능의 비밀 — OPTIMIZE·Z-Order·Liquid Clustering·Deletion Vectors

로그 구조는 편리하지만, 그냥 두면 **작은 파일이 수없이 쌓이는 문제(small file problem)**가 생긴다. 파일이 잘게 쪼개질수록 여는 오버헤드가 커져 쿼리가 느려진다. Delta의 정비 도구들:

| 도구 | 하는 일 | 비유 |
|------|---------|------|
| **OPTIMIZE** | 작은 파일 여러 개를 적당한 크기로 합침(compaction) | 서랍 속 잡동사니 정리 |
| **Z-Ordering** | 자주 같이 필터되는 컬럼 기준으로 데이터를 물리적으로 재배치 → data skipping 극대화 | 관련 물건끼리 같은 칸에 |
| **Liquid Clustering** | Z-Order·파티셔닝을 대체하는 신형. 사전에 파티션 키를 고정하지 않고 유연하게 클러스터링. 편향(skew)에도 강함 | 알아서 재배치되는 스마트 수납 |
| **Deletion Vectors** | 삭제 시 파일을 다시 쓰지 않고 "이 행은 지워짐" 표식만 별도 기록 → 삭제/업데이트가 빨라짐 | 지우개 대신 취소선 |
| **VACUUM** | 로그가 더는 참조하지 않는 오래된 파일을 실제로 물리 삭제(기본 7일 보존) | 재활용 쓰레기 배출 |

> ⚠️ **VACUUM 주의**: 물리 삭제라서, 보존 기간을 너무 짧게 잡으면 **타임트래블 가능 범위가 사라진다.** "며칠 전으로 되돌리기"와 "저장 비용 절감"은 트레이드오프 — SI 표준 정책으로 미리 합의해 둘 항목이다.

> **핵심 통찰 — "느슨하게 쓰고, 나중에 정리한다"**
>
> Delta의 성능 철학은 **쓰기는 빠르고 단순하게(append 위주), 무거운 최적화는 나중에 배치로** 몰아서 한다는 것이다. 쓸 때마다 완벽히 정렬하려 들면 파이프라인이 느려진다. 대신 로그가 "무엇이 흩어져 있는지" 알기 때문에, OPTIMIZE를 나중에 한 방에 돌릴 수 있다. 로그가 있어서 가능한 게으름이다.

---

## 7. UniForm — Iceberg와의 화해

여기서 한 가지 흥미로운 질문: **"Databricks가 왜 경쟁 포맷 Iceberg까지 껴안았나."** 그 기술적 접점이 바로 **UniForm(Universal Format)**이다.

- Delta로 데이터를 쓰면, UniForm이 **Iceberg용 메타데이터를 자동으로 함께 생성**한다.
- 데이터 파일(Parquet)은 어차피 공통이다. 차이는 **로그·메타데이터 계층뿐**이다. 그래서 "같은 Parquet에 Iceberg 메타데이터도 하나 더 얹어주면", Snowflake·Trino 같은 외부 엔진이 **같은 데이터를 Iceberg 테이블로 읽는다.**

> 한 번 쓰고, Delta로도 Iceberg로도 읽힌다.
> 포맷 전쟁을 "둘 다 지원"으로 우회한 실용적 한 수.

이게 Databricks의 전략 — **"우리는 포맷이 아니라 그 위의 지능을 판다"** — 의 실제 구현체다. "열린 포맷 전쟁"을 정면 대결 대신 "둘 다 지원"으로 우회한 셈이다.

---

## 8. 정리 & SI 관점의 시사점

**정리:**

- Delta Lake = **평범한 Parquet + `_delta_log` 트랜잭션 로그.** "파일 목록이 아니라 로그가 테이블을 정의한다"가 전부의 열쇠.
- 로그 구조 하나에서 **ACID(낙관적 동시성) · 타임트래블 · 스키마 관리 · MERGE/CDC**가 전부 파생된다.
- 로그에 담긴 **통계(min/max)**가 data skipping을, **OPTIMIZE·Z-Order·Liquid Clustering·Deletion Vectors**가 성능을 책임진다.
- **UniForm**으로 Iceberg와도 호환 — "포맷 종속 해제"의 실체.

**우리 SI 프로젝트에 주는 시사점 (토론 포인트):**

- **표준 운영 정책부터 합의**: VACUUM 보존 기간(타임트래블 범위 ↔ 비용), OPTIMIZE/클러스터링 주기, 멀티 라이터 시 스토리지별 동시성 보장 방식 — 이 세 가지는 프로젝트 초기에 팀 표준으로 못 박아야 나중에 사고가 안 난다.
- **마이그레이션 논리**: 고객이 이미 Parquet 레이크를 쓰고 있다면 `CONVERT TO DELTA`로 **데이터 복제 없이** 로그만 얹어 전환할 수 있다. "갈아엎지 않는다"는 영업 메시지의 기술적 근거.
- **왜 Delta인가를 한 문장으로**: 고객에게 "레이크의 저렴함·개방성은 그대로, 웨어하우스의 신뢰성(트랜잭션·복구·품질)만 얹는다"로 설명하면 통한다.

> **토론거리:** 우리 첫 고객 환경(클라우드/온프렘, S3/ADLS/GCS)에서 **동시 쓰기 보장**과 **VACUUM·타임트래블 정책**을 각각 어떻게 표준화할 것인가? 기존 Parquet 자산은 `CONVERT TO DELTA`로 흡수할까, 재적재할까?

---

## 9. 더 살펴볼 자료

- [Delta Lake 공식 사이트](https://delta.io/)
- [Delta Lake Documentation — Concurrency control](https://docs.delta.io/latest/concurrency-control.html)
- [Delta Transaction Log Protocol (GitHub)](https://github.com/delta-io/delta/blob/master/PROTOCOL.md)
- [Diving Into Delta Lake: Unpacking The Transaction Log (Databricks Blog)](https://www.databricks.com/blog/2019/08/21/diving-into-delta-lake-unpacking-the-transaction-log.html)
- [Delta Lake Time Travel (Databricks Docs)](https://docs.databricks.com/delta/history.html)
- [Liquid Clustering (Databricks Docs)](https://docs.databricks.com/delta/clustering.html)
- [Delta Lake UniForm — one table, many formats (Databricks Docs)](https://docs.databricks.com/delta/uniform.html)
- [OPTIMIZE & Z-Ordering (Databricks Docs)](https://docs.databricks.com/delta/optimize.html)
