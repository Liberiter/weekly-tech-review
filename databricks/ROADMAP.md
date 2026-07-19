# Databricks 주간 테크리뷰 로드맵

> 회사 Databricks SI 프로젝트를 앞두고, 약 1~2개월간 **Databricks 플랫폼 + 데이터 엔지니어링**을 큰 주제로 잡고 매주 작은 소주제 하나씩 다룬다.
>
> 초반엔 기초 개념으로 팀 눈높이를 맞추고 → 중반에 Databricks 고유 기능 → 후반에 체험·트렌드로 가볍게 마무리하는 흐름.

---

## 📅 8주 로드맵 (주 1개 기준)

| 주차 | 날짜 | 주제 | 축 | 한 줄 | 상태 |
|----|------|------|----|------|------|
| 1 | 2026-06-29 | Data Intelligence Platform 조감도 | Databricks | 5층 구조로 읽는 플랫폼의 야망 | ✅ 완료 |
| 2 | 2026-07-06 | **Delta Lake 뜯어보기** | DE 기초 | Parquet + 트랜잭션 로그 = 왜 '테이블'이 되나 | ✅ 완료 |
| 3 | 2026-07-13 | Medallion Architecture | DE 기초 | Bronze→Silver→Gold, 파이프라인 설계 국룰 | ✅ 완료 (정리본 보유) |
| 4 | 2026-07-20 | **데이터 모델링** | DE 기초 | Star Schema·차원 모델링, Grain과 SCD | 🔜 이번 주 |
| 5 | | Unity Catalog | Databricks | catalog.schema.table 3단 거버넌스 | ⬜ |
| 6 | | Lakeflow / DLT | Databricks | 선언형 ETL, 코드를 어떻게 줄이나 | ⬜ |
| 7 | | DBSQL & Photon | Databricks | 레이크 위에서 웨어하우스 성능 내는 법 | ⬜ |
| 8 | | Databricks Assistant / Genie | 체험형 | 자연어로 데이터 다루기, 실제 써보니 | ⬜ |
| 9 | | Delta Lake vs Iceberg | 트렌드 | 요즘 뜨거운 '열린 포맷 전쟁' | ⬜ |

---

## 🧩 여유분 (순서 교체·대체용)

### Databricks 제품 계열
- **Agent Bricks** — 1주차의 '새 주인공' 심화 (도메인 에이전트 자동 최적화)
- **Databricks Apps** — 데이터 위에 웹앱 바로 배포, Streamlit 호스팅과 비교
- **Lakebase (Postgres)** — 분석DB(OLAP)와 운영DB(OLTP) 통합 떡밥
- **MLflow 3.0** — 실험 관리 & LLM 옵스

### 데이터 엔지니어링 기초 계열 (플랫폼 무관, 팀 기본기)
- **배치 vs 스트리밍** — Structured Streaming 맛보기
- **파티셔닝 & 파일 최적화** — small file problem, Z-ordering
- ~~**데이터 모델링** — Star Schema, dimensional modeling~~ → 3주차로 편성 완료
- **데이터 품질** — Great Expectations / DLT expectations

### 시야 넓히기
- **Databricks vs Snowflake** — 라이벌 철학 비교
- **Lakehouse는 어떻게 탄생했나** — 창업·인수(MosaicML·Tabular·Neon) 히스토리

---

## 운영 메모
- 파일명 규칙: `databricks/YYYY_MM_DD_주제.md`
- 각 글 구성: 도입(문제 제기) → 개념 뜯어보기 → **핵심 통찰** 박스 → **SI 관점 시사점** → 더 살펴볼 자료
- 날짜/상태는 진행하면서 이 표에 업데이트
