# 2026-06-09. Kong, API Gateway, 그리고 AI Gateway — Agentic Era의 연결 인프라

> **"예전엔 앱끼리 API로 연결했다. 이제는 AI 에이전트끼리·AI 모델끼리 연결해야 한다.
> 그 연결을 안전하게 통제·관측·과금하는 새로운 인프라 계층이 필요하다."** — Kong이 행사 내내 던진 한 문장

> 📌 이 문서는 *The Kong Agentic Era World Tour Seoul (2026-06-09)* 현장 참석 노트 + 어제 정리한 사전 리서치([[kong-agentic-era-research]]) + 행사 후 웹 검증을 통합한 **공부용 정리 자료**입니다. 행사장에서 다 소화하지 못한 내용을 다시 복습하는 게 목적입니다.

---

## 목차

1. [30초 요약](#1-30초-요약)
2. [기초 — API와 API Gateway 다시 보기](#2-기초--api와-api-gateway-다시-보기)
3. [전환 — Kong은 왜 "API Gateway 회사"이길 거부하는가](#3-전환--kong은-왜-api-gateway-회사이길-거부하는가)
4. [핵심 1 — AI Gateway란 무엇인가](#4-핵심-1--ai-gateway란-무엇인가)
5. [핵심 2 — 세 개의 게이트웨이 (LLM / MCP / Agent)](#5-핵심-2--세-개의-게이트웨이-llm--mcp--agent)
6. [핵심 3 — Context Is King과 Context Mesh](#6-핵심-3--context-is-king과-context-mesh)
7. [심화 — 에이전트 프로토콜 (MCP / Function Calling / A2A)](#7-심화--에이전트-프로토콜-mcp--function-calling--a2a)
8. [현장 사례 — CJ제일제당의 'AI 직원' 거버넌스 전략](#8-현장-사례--cj제일제당의-ai-직원-거버넌스-전략)
9. [Kong이 말하는 5가지 Connectivity Roadblock](#9-kong이-말하는-5가지-connectivity-roadblock)
10. [코드로 보는 AI Gateway](#10-코드로-보는-ai-gateway)
11. [정리 — 한 줄 요약 · 사내 활용 · FAQ](#11-정리--한-줄-요약--사내-활용--faq)
12. [용어 사전 · 더 공부할 자료](#12-용어-사전--더-공부할-자료)

---

## 1. 30초 요약

- **Kong**은 원래 **API Gateway**(모든 API 요청이 지나는 단일 관문)로 유명해진 실리콘밸리 회사다.
- 그런데 오늘 행사에서 Kong은 **"우리는 더 이상 단순 API Gateway 회사가 아니다"**라고 선언했다. 자신을 **"AI Connectivity 회사 / AI 플랫폼 회사"**로 재정의했다.
- 논리는 단순하다: **AI Gateway는 API Gateway와 구조적으로 똑같은 일**(인증·트래픽 제어·관측·과금)을 한다. 단지 지나가는 트래픽이 *API 호출* → *AI 토큰/에이전트 호출*로 바뀌었을 뿐이다.
- 그래서 Kong은 **API · LLM · MCP · 에이전트(A2A) 트래픽을 하나의 통제탑(Control Tower)에서 관리하는 플랫폼**을 만들겠다고 한다.
- 오늘 행사의 새 키워드 3개: **AI Connectivity**(3계층 통합), **Context Mesh**(에이전트에게 사내 데이터를 먹이는 계층), **Agent Gateway**(에이전트끼리의 통신을 통제).

> 💡 어제까지의 이해: "Kong = API 게이트웨이 회사". 오늘 이후의 이해: **"Kong = AI 시대의 모든 연결을 통제하는 플랫폼이 되려는 회사"**. 이 한 줄의 변화가 행사 전체의 메시지다.

---

## 2. 기초 — API와 API Gateway 다시 보기

> 어제 [[kong-agentic-era-research]]에서 다뤘지만, 오늘 내용을 이해하려면 이 토대가 단단해야 한다. 한 단계 더 깊게 복습한다.

### API = 프로그램끼리의 약속/창구
배달앱이 카드사에 "이 결제 승인해줘"라고 요청하는 통로가 **API**다. 회사가 커지면 이런 API가 수백~수천 개로 늘어난다(벤츠는 1,500개 이상). 그러면 공통 문제가 생긴다:

| 문제 | 질문 |
|---|---|
| 인증/보안 | 누가 이 API를 쓸 수 있지? |
| 트래픽 제어 (Rate Limiting) | 한 명이 1초에 1만 번 때리면? |
| 분석/과금 | 누가 얼마나 썼지? |
| 부하 분산 (Load Balancing) | 서버 하나가 죽으면? |
| 관측 (Observability) | 어디서 느려지고 어디서 터지나? |

### API Gateway = 모든 요청이 지나는 단일 관문

이걸 **API마다 따로 만들면 지옥**이다. 그래서 모든 API 앞에 **"공항 보안검색대 + 안내데스크"** 같은 단일 관문을 둔다. 그게 **API Gateway**다.

```
        [클라이언트들]
   웹 ·  모바일 · 외부 파트너
            │
            ▼
   ┌───────────────────────┐
   │     API Gateway       │  ← 인증 · Rate Limit · 로깅 · 라우팅 · 캐싱
   │   (Kong Gateway)      │     "모든 요청이 여기서 검사받고 분배됨"
   └───────────────────────┘
        │      │      │
        ▼      ▼      ▼
    [결제]  [주문]  [회원]   ← 뒤에 숨은 수백 개의 백엔드 API/서비스
```

**Kong Gateway**가 바로 이 관문 제품이다. NGINX 엔진 기반으로 노드당 초당 5만 건 이상 처리하고, **오픈소스(Apache 2.0)**라서 폭발적으로 퍼졌으며, 플러그인 60개+ 로 기능을 붙인다.

> 💡 **핵심 통찰**: API Gateway가 하는 일은 본질적으로 **"중앙에서 가로채서(intercept) → 검사하고 → 정책을 적용하고 → 기록하고 → 보낸다"**이다. 이 패턴을 기억하라. **AI Gateway도 정확히 같은 패턴**을 LLM/에이전트 트래픽에 적용한다.

---

## 3. 전환 — Kong은 왜 "API Gateway 회사"이길 거부하는가

오늘 행사의 한 세션 제목이 **"Beyond API to AI"**였다. Kong의 서사는 이렇게 흐른다.

```
[과거]  레거시 시스템  ──API화──▶  디지털 자산(API)  ──단일 게이트웨이로 관리──▶  Kong의 영역
          (Digital Transformation: API가 곧 회사의 디지털 자산이 된다)

[현재]  AI·에이전트 등장  ──▶  "AI를 제어하는 것 = 그 AI가 호출하는 API를 제어하는 것"
          (AI Transformation의 조건 = 중앙 통제(Central Governance) = Kong이 하던 바로 그 일)
```

핵심 논리는 두 단계다:

1. **"AI 경쟁력 = 레거시 데이터·서비스를 얼마나 잘 연결·활용하느냐"** → 그 연결 수단이 결국 **API**다.
2. **"AI를 제어한다 = AI가 호출하는 API/도구/모델을 제어한다"** → 그건 Kong이 10여 년간 해온 **게이트웨이·거버넌스**와 똑같은 문제다.

그래서 Kong의 결론: **"AI Gateway는 API Gateway와 유사하다. 우리는 이미 그 일을 잘한다. 따라서 우리는 AI 플랫폼 회사가 되겠다."**

> ⚠️ **세일즈 서사임을 인지하자**: 이 논리는 깔끔하지만 Kong에게 유리하게 짜인 프레이밍이다. 실제로는 AI 트래픽엔 API 트래픽에 없던 새 문제들(토큰 비용, 프롬프트 주입, 비결정성, 컨텍스트 관리)이 많아서 "똑같다"고 단정할 수는 없다. 그럼에도 **"중앙 관문에서 통제한다"는 아키텍처 패턴이 재사용된다**는 건 사실이다.

### "Control Tower(통제탑)"가 되겠다
Kong이 반복한 비유가 **Control Tower(관제탑)**이다. 공항 관제탑이 모든 비행기의 이착륙을 한곳에서 통제하듯, **API·LLM·MCP·에이전트 트래픽을 하나의 통제탑에서 본다**는 것. 이것이 Kong이 미는 **"AI Connectivity"** 플랫폼의 핵심 그림이다.

---

## 4. 핵심 1 — AI Gateway란 무엇인가

**AI Gateway = LLM·AI 트래픽 버전의 API Gateway**다. 앱이 LLM을 부를 때(그리고 에이전트가 도구를 부를 때) 그 사이에 끼어서 통제하는 관문이다.

### 왜 필요한가? — AI 트래픽의 새로운 골칫거리

| API Gateway가 풀던 문제 | AI Gateway가 추가로 풀어야 하는 문제 |
|---|---|
| 누가 API를 쓰나 (인증) | 누가 **어떤 모델**을 쓰나 |
| 초당 호출 수 제한 | **토큰 단위** 예산/한도 (= 곧 돈) |
| API 키 유출 방지 | **PII(개인정보)·프롬프트 주입** 차단 |
| 어느 서버로 보낼까 (라우팅) | **어느 모델로 보낼까** (비용/품질/지연 기반) |
| 같은 응답 캐싱 | **의미가 비슷한 질문** 캐싱 (시맨틱 캐싱) |

### Kong AI Gateway가 실제로 하는 일

- **멀티 LLM 라우팅**: OpenAI, Anthropic(Claude), Google Gemini, AWS Bedrock, Azure, Mistral 등 **수천 개 모델을 하나의 통일된 API로**. 코드 수정 없이 모델 교체.
- **지능형 라우팅**: 비용 기반·지연시간 기반·**시맨틱(의미) 라우팅** — 질문 성격에 따라 싼 모델/비싼 모델로 분배.
- **시맨틱 캐싱**: "의미가 같은 질문"은 캐시로 답해 토큰 비용 절감.
- **토큰 거버넌스**: 사용자·모델·시간대별 토큰 예산/쿼터, 컨텍스트 윈도우 관리.
- **보안**: PII 자동 차단(20개+ 카테고리, 12개 언어), 프롬프트 가드, Vault 연동 비밀관리.
- **관측/과금**: 어느 팀이 어떤 모델에 얼마 썼는지 **showback/chargeback**(2025년 OpenMeter 인수로 강화).

> 💡 **"API call 경제 → Token 경제"**: 예전엔 *API 호출 횟수*가 비용·가치의 단위였다. 이제는 *AI 토큰*이 단위다. Kong은 자신을 **"토큰의 흐름을 안전하게 연결·통제·수익화하는 회사"**로 포지셔닝한다.

---

## 5. 핵심 2 — 세 개의 게이트웨이 (LLM / MCP / Agent)

오늘 행사에서 가장 중요한 그림. Kong은 2026년 4월 **AI Gateway 3.14 + Agent Gateway(GA)** 발표로, AI Gateway가 **세 종류의 트래픽**을 하나의 control plane에서 통제한다고 정리했다.

```
                        ┌───────────────────────────────┐
                        │   Kong AI Gateway (통제탑)    │
                        │  통합 관측 · 거버넌스 · 보안  │
                        └───────────────────────────────┘
                                        │
              ┌─────────────────────────┼─────────────────────────┐
              ▼                         ▼                         ▼
   ┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
   │     LLM Gateway     │   │     MCP Gateway     │   │    Agent Gateway    │
   │      앱 ↔ 모델      │   │   에이전트 ↔ 도구   │   │ 에이전트 ↔ 에이전트 │
   │     (LLM 호출)      │   │    (MCP 툴 접근)    │   │     (A2A 통신)      │
   └─────────────────────┘   └─────────────────────┘   └─────────────────────┘
```

| 게이트웨이 | 통제하는 트래픽 | 한 줄 설명 |
|---|---|---|
| **LLM Gateway** | 앱 → LLM | 여러 모델 제공사를 단일 API로 통합, 비용/품질 라우팅 |
| **MCP Gateway** | 에이전트 → 도구/데이터 | MCP 프로토콜로 도구 접근을 표준화·인증·검사. 기존 REST API를 코드 없이 MCP 서버로 변환 + OAuth 2.1 중앙 인증 |
| **Agent Gateway** | 에이전트 → 에이전트 (A2A) | 에이전트별 신원·인증, A2A 메시지 실시간 검사(프롬프트 주입·이상행동), 에이전트별 비용 배분·감사 로그 |

이 세 개를 합쳐 Kong은 **"AI Data Path(AI 데이터 경로) 전체를 통제하는 유일한 플랫폼"**이라고 홍보한다.

> ⚠️ **"유일한(only)"은 마케팅 문구**: Kong은 "LLM+MCP+A2A를 함께 지원하는 시장 유일의 게이트웨이"라고 말하지만, **Solo.io, Gravitee, Apigee, MuleSoft** 등도 비슷한 통합 기능을 같은 시기에 내놨다. 현장에서 "유일하다"가 나오면 *"경쟁사도 비슷한 걸 한다"*고 필터링하자. (단, A2A 거버넌스에선 Kong이 앞선다는 평가는 있다.)

---

## 6. 핵심 3 — Context Is King과 Context Mesh

행사의 또 다른 세션 **"Context Is King"**의 메시지: **모델·추론·프롬프트가 좋아도, 에이전트는 결국 'Context(맥락)'가 없으면 프로덕션에서 실패한다.**

### "From UI to API" — 사용자가 누르는 게 아니라 에이전트가 호출한다
예전엔 사람이 화면(UI)을 눌러서 일했다. 이제는 **에이전트가 직접 API를 호출**해서 일을 끝낸다. 그런데 에이전트가 쓸모 있으려면 회사의 진짜 데이터·도구에 **실시간으로, 안전하게** 닿아야 한다. 여기서 문제가 터진다.

### 3가지 Context Challenge (행사에서 강조)

1. **Raw sources aren't agent-ready** — 원천 데이터가 에이전트가 바로 먹을 수 있는 형태가 아니다. (레거시는 *하루 지난 배치 데이터*, 에이전트는 *실시간*이 필요)
2. **Identity & auth don't translate** — 기존의 정적 인증(API 키, 서비스 계정)은 에이전트의 **위임된 신원(delegated identity)**을 표현 못 한다. → OAuth 2.1 기반 토큰 교환이 필요.
3. **Everyone reinvents the wheel** — 모든 팀이 에이전트마다 통합을 처음부터 다시 만든다. → 느리고 위험하다.

→ 결론: **Context가 별도의 'Mediation Layer(중재 계층)'가 되어야 한다.**

### Context Mesh — Gartner의 개념 → Kong의 제품 (2026년 2월 출시)

**Gartner가 정의한 "Context Mesh"**: *"에이전트가 시스템 전반의 상태를 안전하게 발견하고, 추론하고, 액션을 트리거하게 해주는 실시간 컨텍스트 메시"*. Gartner는 **"이 전환 없이는 2027년까지 에이전틱 AI 과제의 40%가 취소 위기"**라고 경고했다.

**Kong Context Mesh**는 이걸 제품으로 구현한 것:
- Kong Konnect가 이미 알고 있는 정보(엔드포인트·스키마·인증요건·정책)를 활용해
- **사내 API를 자동으로 발견(discover) → 에이전트용 도구로 변환(generate) → 런타임 거버넌스와 함께 배포(deploy)**
- 즉 **"흩어진 사내 데이터를 에이전트가 바로 먹을 수 있는 도구로 자동 변환해주는 계층"**.

```
    흩어진 사내 자산                 Context Mesh                    에이전트     
  ┌──────────────────┐        ┌────────────────────────┐        ┌────────────────┐
  │  REST API        │        │      Context Mesh      │        │    AI Agent    │
  │  이벤트 스트림   │  ───▶  │  1. 자동 발견          │  ───▶  │  (실시간 사용) │
  │  애플리케이션    │        │  2. 도구로 변환        │        │                │
  │  (제각각 스키마) │        │  3. 거버넌스 적용·배포 │        │                │
  └──────────────────┘        └────────────────────────┘        └────────────────┘
```

> 💡 노트에 적은 **"Deterministic → Non-deterministic"**의 의미: API 시대는 *결정론적*(같은 입력 = 같은 출력)이었지만, AI 에이전트는 *비결정론적*이다. 그래서 단순 라우팅을 넘어 **관측·검사·거버넌스가 훨씬 더 중요**해진다 — 이게 Kong이 "새 인프라 계층이 필요하다"고 주장하는 근거다.

### AI Connectivity = 3계층 통합

Kong이 미는 큰 그림은 **세 계층을 하나의 플랫폼에서 통제**하는 것:

| 계층 | 무엇 | Kong 제품 |
|---|---|---|
| **Application Layer** (애플리케이션) | 기존 API·서비스 | Kong Gateway / Konnect |
| **Intelligence Layer** (지능) | LLM·에이전트 | Kong AI Gateway (LLM/MCP/Agent) |
| **Context Layer** (맥락) | 에이전트에게 줄 사내 데이터·도구 | **Context Mesh** + MCP Registry |

---

## 7. 심화 — 에이전트 프로토콜 (MCP / Function Calling / A2A)

행사의 **"다양한 에이전트 프로토콜 이해하기"** 세션 정리. 핵심 메시지: **"API의 시대에서 → AI의 API 시대로"**, 그리고 그걸 위해 **표준(프로토콜)**이 등장하고 있다.

### 표준화의 두 축

| 축 | 표준 | 역할 |
|---|---|---|
| **Context (맥락 주입)** | **MCP** (Model Context Protocol) | 에이전트가 외부 도구·데이터에 *연결*되는 표준 규약 |
| **Execution (실행)** | **Function Calling** | LLM이 실제로 함수/도구를 *호출*하는 방식 |
| **Collaboration (협업)** | **A2A** (Agent-to-Agent) | 에이전트끼리 서로 대화·협업하는 프로토콜 |

### 또 다시 분산 시스템 = MSA의 데자뷔

행사에서 인상적이었던 프레이밍: **"에이전트 세상은 결국 또 하나의 분산 시스템(MSA)"**이다. 마이크로서비스가 많아지자 서비스 메시(관측·거버넌스·제어)가 필요했듯, **에이전트가 많아지면 똑같이 '관리'가 필요**하다.

```
   [MSA 시대]   서비스가 많아짐  →  Service Mesh (관측·거버넌스·제어)
       │
       ▼ (같은 패턴 반복)
   [Agent 시대]  에이전트가 많아짐 →  MCP Gateway + Agent Gateway + Observability
                                      = Kong이 말하는 "Control Plane"
```

→ Kong의 주장: **연결 프로토콜(MCP/A2A)에 '관리(Observability·Governance·Control)'를 더한 Control Plane**이 다중 에이전트 워크플로우의 핵심이고, 그게 자기들 자리라는 것.

---

## 8. 현장 사례 — CJ제일제당의 'AI 직원' 거버넌스 전략

행사에서 가장 실무적이었던 고객 발표. CJ제일제당이 **"AI 직원"**을 만들면서 어떻게 거버넌스를 잡았는지.

### 진화의 3단계 (+ 그 이후)

```
   1단계   LLM         = 사고 엔진 (생각만 함)
   2단계   AI 에이전트 = 자율성 (스스로 도구를 씀)
   3단계   AI 직원     = 사람과 동일한 역할 (책임·권한을 가짐)
   그 이후 AI 시스템   = 여러 AI 직원이 모인 조직
```

DX(디지털 전환) → AX(AI 전환) → **AI 직원**으로 가는 여정.

### Why Kong? — "AI Gateway는 API Gateway와 유사하다"
CJ제일제당도 같은 논리로 Kong을 택했다: 이미 검증된 게이트웨이 거버넌스를 AI에 그대로 적용할 수 있다는 것.

### AI 직원에게 필요한 3가지 통제 (→ Kong 기능 매핑)

| 필요한 것 | 의미 | Kong 기능 |
|---|---|---|
| **비용 거버넌스** | AI 직원이 돈(토큰)을 쓴다 → 한도가 필요 | **Token Rate Limit** (토큰 단위 속도 제한) |
| **지능 라우팅·확장** | 상황에 맞는 모델로 분배, 확장성 | **AI Proxy Advanced** |
| **행동 권한 통제** | AI 직원이 *할 수 있는 행동*을 제한 | **AI MCP Proxy** (MCP 도구 접근 통제) |

> 💡 **실무 포인트**: "AI 직원"이라는 개념이 멋있어 보이지만, CJ 발표의 진짜 교훈은 **"자율적인 AI에게 일을 맡기려면 *비용·라우팅·행동권한* 3가지를 반드시 게이트웨이에서 통제해야 한다"**는 점이다. 자율성과 통제는 짝이다.

---

## 9. Kong이 말하는 5가지 Connectivity Roadblock

오프닝 키노트("Kong과 함께 준비하는 AI 에이전트 시대")의 핵심 주장: **에이전트의 실패는 모델·추론·프롬프트의 실패가 아니라 '연결(connectivity)'의 실패다.**

> *"Agent failures won't be model failures, reasoning failures, or prompting failures — they'll be connectivity failures."*

그러면서 제시한 **5가지 걸림돌(Roadblock)**:

| # | Roadblock | 의미 |
|---|---|---|
| 1 | **Agents spend money** | 에이전트는 토큰을 태우며 *실제 돈*을 쓴다 → 통제 없으면 비용 폭발 |
| 2 | **Sovereignty already matters** | 데이터 주권·국가별 규제가 *이미* 현실 문제 |
| 3 | **Trends, hype, and more hype** | 과대광고 속에서 진짜 운영 가능한 걸 골라내야 함 |
| 4 | **Safety & Security** | 프롬프트 주입·PII 유출·이상행동 |
| 5 | **거버넌스 부재** | **침해당한 조직의 63%가 AI 거버넌스가 없었다** (보안 사고의 근본 원인) |

→ Kong의 결론: 이 다섯을 푸는 답이 **"Kong AI Connectivity"**, 즉 **하나의 통제탑에서 모든 AI 트래픽을 거버넌스하는 것**.

> ⚠️ 이건 전형적인 **"문제 제기 → 우리가 답"** 키노트 구조다. 걸림돌 자체(특히 1·2·4·5)는 실제로 타당하다. 다만 "그 답은 반드시 Kong"이라는 결론은 세일즈다. 걸림돌은 **업계 공통 과제**로, 솔루션은 **여러 선택지 중 하나**로 받아들이면 균형 잡힌 시각이다.

---

## 10. 코드로 보는 AI Gateway

말로만 들으면 추상적이니, AI Gateway가 코드에서 어떻게 보이는지 감을 잡자.

### Before — 앱이 모델 제공사에 직접 붙는 경우 (게이트웨이 없음)

```python
# OpenAI에 직접 호출 → 모델 바꾸려면 코드·인증·SDK를 다 갈아야 함
from openai import OpenAI
client = OpenAI(api_key="sk-...")           # 키가 앱 코드에 노출
resp = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "안녕"}],
)
# 비용 추적? Rate limit? PII 차단? → 전부 앱마다 직접 구현해야 함
```

### After — Kong AI Gateway를 거치는 경우

```python
# base_url만 Kong AI Gateway로 바꾸면 끝.
# 모델 교체·인증·비용통제·PII차단·캐싱은 게이트웨이가 처리한다.
from openai import OpenAI
client = OpenAI(
    api_key="kong-managed-token",            # 실제 모델 키는 Kong이 Vault에 보관
    base_url="https://ai-gw.mycompany.com/v1"  # ← 여기만 바뀜
)
resp = client.chat.completions.create(
    model="my-team/default",                 # 논리적 이름. 실제 모델은 Kong이 라우팅
    messages=[{"role": "user", "content": "안녕"}],
)
# Kong이 자동으로: 토큰 한도 체크 → PII 스캔 → 시맨틱 캐시 확인
#                → 비용/지연 기준으로 OpenAI/Claude/Gemini 중 라우팅 → 사용량 기록
```

### 게이트웨이 정책 예시 (선언적 설정)

```yaml
# Kong에 "AI Proxy" + "토큰 한도" + "PII 차단" 플러그인을 붙이는 모습 (개념 예시)
plugins:
  - name: ai-proxy-advanced          # 멀티 LLM 라우팅
    config:
      targets:
        - model: openai/gpt-4o
          weight: 70                  # 70%는 GPT-4o
        - model: anthropic/claude-opus
          weight: 30                  # 30%는 Claude로 (A/B·비용분산)
  - name: ai-rate-limiting-advanced  # 토큰 단위 속도 제한
    config:
      limit_by: consumer
      window_size: 60
      tokens_per_window: 100000       # 분당 10만 토큰까지
  - name: ai-sanitizer               # PII 자동 차단
    config:
      categories: [email, phone, ssn, credit_card]
```

> 💡 **여기서 "AI Gateway ≈ API Gateway"가 체감된다**: 앱은 `base_url` 한 줄만 바꾸고, 나머지 정책(인증·한도·보안·라우팅)은 전부 **게이트웨이에 선언적으로** 붙인다. 정확히 API Gateway가 하던 방식 그대로다.

---

## 11. 정리 — 한 줄 요약 · 사내 활용 · FAQ

### 한 줄 요약
> **Kong은 "API Gateway가 API 트래픽에 해주던 통제(인증·한도·관측·과금)를, AI 토큰·MCP·에이전트 트래픽에도 똑같이 해주는 단일 통제탑"이 되어 'AI 플랫폼 회사'로 도약하려 한다.**

### 사내에서 그대로 쓸 만한 한 줄들
- "AI Gateway는 새로운 게 아니라, **API Gateway 패턴을 AI 트래픽에 재적용**한 것이다."
- "에이전트 도입의 진짜 리스크는 모델 성능이 아니라 **연결·비용·거버넌스**다. (침해 조직의 63%가 AI 거버넌스 부재)"
- "AI 시대의 비용 단위는 호출 횟수가 아니라 **토큰**이다 → **토큰 거버넌스**가 필수."
- "에이전트에게 사내 데이터를 먹이는 게 진짜 난제다 → **Context(맥락) 계층**이 다음 전장이다." (Gartner: 2027년까지 에이전틱 AI의 40%가 취소 위기)
- "에이전트 세상 = **또 하나의 분산 시스템(MSA)** → 관측·거버넌스·제어가 반복해서 필요해진다."

### FAQ

**Q1. AI Gateway랑 API Gateway, 결국 같은 거야 다른 거야?**
구조(중앙 관문에서 가로채 정책 적용)는 같다. 하지만 다루는 문제가 다르다: API Gateway는 *호출 횟수·인증·라우팅*, AI Gateway는 거기에 더해 *토큰 비용·프롬프트 주입·PII·시맨틱 캐싱·모델 라우팅*. **"같은 아키텍처, 다른 관심사"**가 정답.

**Q2. MCP, A2A, Function Calling 차이가 헷갈려.**
- **MCP** = 에이전트가 도구·데이터에 *연결*되는 표준 (맥락 주입)
- **Function Calling** = LLM이 함수를 *실제로 호출*하는 방식 (실행)
- **A2A** = 에이전트끼리 *협업·대화*하는 프로토콜 (협업)
역할이 겹치지 않는 **3개의 다른 층**이라고 보면 된다.

**Q3. Context Mesh가 그래서 뭐야?**
흩어진 사내 API·데이터를 **자동으로 발견해서 → 에이전트가 바로 쓸 수 있는 도구로 변환 → 거버넌스 걸어서 배포**해주는 계층. Gartner가 개념을 만들었고 Kong이 2026년 2월 제품화했다. "에이전트에게 회사 데이터를 안전하게 떠먹여주는 층"으로 기억하면 된다.

**Q4. 우리가 당장 Kong을 써야 하나?**
이 문서의 목적은 *판단 재료*지 *구매 권유*가 아니다. 핵심 질문은 **"우리 조직이 LLM·에이전트를 여러 팀에서 쓰기 시작했고, 비용·보안·거버넌스를 중앙에서 통제할 필요가 생겼는가?"**다. Yes라면 AI Gateway 범주(Kong, Solo.io, Gravitee, LiteLLM, Portkey 등)를 비교 검토할 시점이다.

**Q5. "유일하다", "31배 빠르다" 같은 수치는 믿어도 돼?**
대부분 **Kong 자체 벤치마크/마케팅**이다. 방향성(통합·성능 강점)은 참고하되, 절대 수치와 "유일/최초"는 경쟁사 자료와 교차검증하기 전엔 그대로 인용하지 말 것.

---

## 12. 용어 사전 · 더 공부할 자료

### 약어·용어 사전

| 용어 | 뜻 |
|---|---|
| **API Gateway** | 모든 API 요청이 지나는 단일 관문 (인증·트래픽·관측) |
| **AI Gateway** | LLM·AI 트래픽 버전의 게이트웨이 |
| **AI Connectivity** | Kong이 미는 "API+LLM+MCP+에이전트를 하나로 연결·통제"하는 플랫폼 비전 |
| **LLM** | ChatGPT·Claude 같은 거대언어모델 |
| **MCP** | Model Context Protocol. 에이전트가 도구·데이터에 붙는 표준 |
| **Function Calling** | LLM이 함수/도구를 실제로 호출하는 방식 |
| **A2A** | Agent-to-Agent. 에이전트 간 통신 프로토콜 |
| **Context Mesh** | 사내 데이터를 에이전트용 도구로 자동 변환·거버넌스하는 계층 (Gartner 개념 / Kong 제품) |
| **Token (토큰)** | AI가 처리하는 텍스트 단위 = AI 시대의 비용·과금 단위 |
| **거버넌스** | 누가/무엇을/얼마나 쓰는지 통제·관측·감사하는 것 |
| **Control Plane / Control Tower** | 모든 트래픽을 중앙에서 통제하는 제어 평면(관제탑) |
| **Showback / Chargeback** | 팀·부서별 사용량을 보여주거나(showback) 실제 청구(chargeback)하는 것 |
| **Konnect** | Kong의 통합 SaaS 플랫폼(API+AI 관리·과금) |

### 더 공부할 자료
- **Kong 공식 — AI Gateway 제품 페이지**: https://konghq.com/products/kong-ai-gateway
- **Kong 공식 — Agent Gateway 출시 블로그**: https://konghq.com/blog/product-releases/kong-agent-gateway
- **Kong 공식 — Context Mesh 소개**: https://konghq.com/blog/product-releases/introducing-kong-context-mesh
- **Kong 공식 — Gartner의 Context Mesh 해설**: https://konghq.com/blog/enterprise/gartners-context-mesh
- **Kong Docs — AI Gateway**: https://developer.konghq.com/ai-gateway/
- **MCP 기초 복습**: 사내 문서 `2026_05_18_MCP.md` 참고
- **에이전트 기초 복습**: 사내 문서 `2026_05_26_Agentic_AI.md` 참고
- **학습 로드맵 제안**: ① API Gateway 개념(이 문서 §2) → ② MCP 깊이 파기(`2026_05_18_MCP.md`) → ③ AI Gateway 실습(Kong Docs 핸즈온) → ④ A2A/멀티에이전트 거버넌스(이 문서 §5·§7)

---

*작성: 2026-06-09 | 출처: 현장 노트(`kong_review.md`) + 사전 리서치(`kong-agentic-era-research.md`) + 행사 후 웹 검증(Kong 공식 발표 자료). ⚠️ 표시는 Kong 자체 마케팅이거나 교차검증이 필요한 항목.*
