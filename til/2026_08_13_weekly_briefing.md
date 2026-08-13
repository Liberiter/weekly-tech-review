# 주간 정리 (2026-08-04 ~ 08-13)

---

# 1. 아키텍처 이해

## 1.1 전체 흐름

```text
Oracle DR
    │
    ▼
datalake_airflow
    ├ Airflow / MWAA
    ├ AWS Glue
    ├ Amazon S3
    └ Redshift ODS / Mart
    ▼
Redshift Federation
    │
    ▼
bithumb-ai-etl
    ├ Databricks Jobs
    ├ Unity Catalog Delta tables
    ├ Metric Views
    ├ Genie
    └ Download Gateway
```

## 1.2 `datalake_airflow` - 원천에서 Mart까지

### 1.2.1 원천 수집

```text
Oracle DR
    │ Oracle JDBC
    │ Glue Connection
    ▼
AWS Glue Job
    │ Oracle 테이블 조회
    │ 일별/시간별 증분 처리
    │ 필요한 변환
    │ S3 저장
    ▼
S3 ODS Landing
```

### 1.2.2 S3 → Redshift ODS

```text
S3 ODS Landing
    │
    ▼
Redshift ODS (원천 구조에 가까운 테이블)
```

### 1.2.3 Redshift ODS → Mart

```text
Redshift ODS
    ├ 회원 ODS
    ├ 거래 ODS
    ├ 자산 ODS
    ├ 등급 ODS
    └ 이벤트 ODS
    ▼
Mart DAG
    ├ JOIN
    ├ 필터
    ├ 업무 규칙 적용
    ├ 집계
    └ 기준일별 데이터 생성
    ▼
Mart
```

## 1.3 `bithumb-ai-etl` - Mart에서 소비까지

### 1.3.1 Redshift 데이터 읽기

```text
Mart
    │
    ▼
ext_redshift.mart.<table>
    │ Databricks Federation
    ▼
Databricks Spark / SQL
```

### 1.3.2 Databricks Daily Batch

```text
daily_batch_1
    │ 실행할 테이블 목록 확인
    │ (메타 테이블)
    ▼
daily_batch_2
    ├ 테이블별 병렬 처리
    │ batch_process
    ├ Federation으로 Redshift 조회
    ├ 날짜 범위 필터
    ├ full / incremental 처리
    ├ timestamp 변환
    └ Unity Catalog 저장
    ▼
target_catalog.mart
target_catalog.mart_enc
```

### 1.3.3 집계 및 view 생성

```text
Unity Catalog Delta tables
    ├ Fact 집계
    ├ 회원/거래 데이터 조합
    ├ Materialized View 갱신
    └ Metric View 생성
    ▼
mart_view
```

### 1.3.4 최종 데이터 소비

```text
Metric View / Materialized View
    ├ Genie 분석 질의
    ├ Download Gateway
    ├ BI / SQL 분석
    └ 기타 Databricks 분석 애플리케이션
```

## 1.4 작업 실행 주체 구분

```text
Airflow
    └ 전체 AWS 파이프라인 실행 순서 관리

AWS Glue
    └ Oracle에서 읽고 Amazon S3로 저장

Redshift
    ├ ODS 저장
    └ Mart SQL 실행

Databricks Jobs
    ├ Redshift Federation 읽기
    ├ Delta table 적재
    ├ batch 처리
    └ View / 집계 갱신

Unity Catalog
    └ Databricks 테이블・스키마・권한・데이터 관리
```

---

# 2. 배포 (DAB)

- DAB의 본질은 배포 자동화가 아니라 설정의 코드화다. 워크스페이스에만 있던 잡/스케줄/클러스터 사양을 YAML로 Git에 넣는 것이고, 리뷰와 롤백은 그 결과다.
- `resources`는 REST API 스키마 그대로라서 API 문서가 곧 DAB 문서다. UI로 만든 리소스는 `bundle generate`로 YAML로 뽑을 수 있다.
- 리소스 정의는 한 벌만 두고 환경차는 `variables` + `targets` 오버라이드로 흡수한다.
- `mode: development` / `production`은 편의 기능이 아니라 사고 방지 장치다. dev는 이름 접두사와 스케줄 자동 일시정지, prod는 브랜치 검증과 `run_as` 요구.
- 배포 엔진이 Terraform에서 direct로 넘어갔다(2026-06 GA, 신규 번들 기본값). 폐쇄망에 유리하고 더 빠르다. 단 direct는 YAML에서 지운 필드를 기본값으로 되돌리므로 유지할 값은 명시해야 한다.
- `deploy`와 `run`은 다르다. 배포 성공은 리소스가 존재한다는 뜻이지 코드가 맞다는 뜻이 아니다.
- 번들 경계는 함께 배포되고 함께 롤백되는 범위로 정한다. 지금 규모에서는 번들 분할 없이 파일 분할(`include:`)로 충분하다.
- `root_path` 기본값이 배포자 개인 홈이라 운영에서는 서비스 프린시펄 + 공용 경로를 명시해야 한다.

## 2.1 YAML

- 함정의 근원은 plain 스칼라의 자동 타입 추론이다. `NO`가 false가 되고, `1.10`이 1.1이 되고, 앞자리 0이 사라지고, 중복 키는 조용히 마지막 값을 채택한다.
- 계좌번호처럼 앞자리 0이 의미를 갖는 값에서 실제로 사고가 난다. 조인이 조용히 실패하고 발견은 한참 뒤다.
- 앵커/별칭은 파일 경계를 못 넘는다. 파일을 넘는 재사용은 복합 변수(`type: complex`)로 한다.
- YAML 버그는 코드 리뷰로 못 잡으므로 `yamllint` + `bundle validate`를 CI에 넣는 게 맞다.

---

# 3. ETL

- 지금 코드는 Redshift gold를 그대로 적재한다. 적재하는 중에 데이터 변환이 약간은 일어난다.
- `write_mode`의 진짜 축은 성능이 아니라 재실행 안전성이다. `append`만 멱등하지 않은데 가장 단순해서 가장 많이 쓰인다. `append` 태스크에서 repair는 복구가 아니라 데이터 오염이다.
- fact + dim 조인은 측정 전에 모델 구조에서 예측되는 튜닝 지점이다. 설계 시점에 후보 목록을 만들 수 있고 그게 곧 모니터링 대상이다.
- Metric View에 `materialization`을 걸면 물리 MV가 생기고 옵티마이저가 자동으로 매칭한다(안 걸리면 원본으로 fallback). 정의는 한 곳이고 MV는 캐시라 진실이 갈라지지 않는다. 대가는 신선도.
- metric view의 `comment`는 문서가 아니라 런타임 입력이다. LLM에게 지식을 주는 게 아니라 LLM이 결정할 것의 개수를 줄인다.
- Federation은 관리형 JDBC다. 편의보다 거버넌스(권한/감사/리니지)가 더 큰 이득이고, 성능 변수는 푸시다운인데 겉으로 구별이 안 되므로 실행 계획을 봐야 한다.
- 상태 테이블은 최종 상태만 필요해 이력을 안 남긴다(SCD Type 1). "특정 시간대만 읽는다"에는 상류가 그 밖에서는 안 건드린다는 무언의 계약이 깔려 있다. 고수위 방식이 구조적으로 안전하다.
- UTC 변환은 "이 값은 KST다"라는 데이터 밖의 지식을 데이터 안에 새기는 작업이다. 방향을 틀려도 에러가 안 나고 날짜 경계에서만 표가 난다.
- `on_success` 알림은 침묵이 정상인지 이상인지 구별하게 해준다. 알림 채널 자체의 자가 진단이기도 하다.

---

# 4. 플랫폼

- Control Plane(UI/스케줄러/메타데이터/Unity Catalog)은 벤더 계정, 데이터는 언제나 우리 계정 S3에 있다.
- classic과 serverless의 선택은 성능 옵션이 아니라 네트워크 경계의 위치를 바꾸는 결정이다.
- 서버리스가 우리 계정에 없는 이유는 멀티테넌트 풀링 없이 즉시 기동이 성립하지 않기 때문이다. 원천 DB 연결은 불가능한 게 아니라 NCC나 PrivateLink 설정이 필요한 것이다.
- Workspace : Metastore = N:1, Catalog : Workspace = M:N. 워크스페이스를 지워도 카탈로그가 남는 건 소유주가 계정 레벨 메타스토어이기 때문이다.
- Unity Catalog는 Tables/Volumes/Models/Shares를 하나의 권한 평면 아래 두는 게 목적이다.
- DBU는 Databricks Unit이고 성능 단위가 아니라 과금 단위다.
- 공용 wheel(artifacts)의 진짜 이유는 재사용이 아니라 테스트 가능성이다. 노트북 코드에는 테스트를 붙일 수 없다.

---

# 5. 보안 / 개인정보

- 개인정보 판단은 컬럼 단독이 아니라 조인 가능성으로 한다.
- 원본을 저장하지 않는 정적 마스킹이고 `mem_id`는 SHA-256 + salt로 가명화한다. 안전성이 salt 보관 한 곳에 걸려 있다.
- 마스킹 대상은 코드가 아니라 Unity Catalog 태그로 지정한다. 규칙과 대상 목록이 분리돼 컬럼이 추가돼도 배포가 필요 없다. 대신 태그 권한이 곧 개인정보 통제 권한이 된다.
- 정적 마스킹을 고른 것과 metric view를 materialize하는 것은 서로를 가능하게 하는 한 쌍의 결정이다.
- 시크릿과 파라미터를 가르는 기준은 민감도가 아니라 실행 이력/로그에 남아도 되는가다. 위젯으로 받으면 그대로 남는다.

---

# 6. 오케스트레이션 경계 (확인 중)

- 플랫폼 밖에서 안으로 들어오는 구간은 Airflow(MWAA), 안에서 안으로 도는 구간은 번들 잡으로 갈려 있다.
- 두 구간이 신호가 아니라 cron 간격으로 이어져 있어서, 상류가 늦으면 하류는 성공한 채로 어제 데이터를 읽는다.
- 백업 배치 lookback이 28시간인 건 감사 로그 도착 지연에 대한 여유이고 anti join이 중복을 흡수한다.
- 한 리포 안에 Airflow 버전이 섞여 있고 Airflow 2는 이미 지원이 끝났다. 우리 업무 범위는 아니지만 기록은 필요하다.
- dbt는 템플릿 기본이 기존 엔진이다. 실제 설치 버전은 우리 리포의 버전 사양 파일에 박혀 있어서 자동으로 올라가지 않는다.
