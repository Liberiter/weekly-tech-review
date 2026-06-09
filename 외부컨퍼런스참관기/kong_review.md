# Kong Agentic Era


## 환영사
- Networking하는 시간이 되었으면 좋겠다.


## Kong과 함께 준비하는 AI 에이전트 시대

- "자동차로 레스토랑에 가는 모든 여정을 계획하고 수행해줘."
    => 에이전트가 모두 해줄 수 있는 시대가 온다. 그러나 그 안에는 각 에이전트를 연결하는 LLM과 API가 필요하다.

- Agent Failures won't be
    - model failure
    - reasoning failure
    - prompting failure
    : 그러나 연결에서 실패한다.

- It's all about AI connectivity

- 5 Connectivity redblocks
    1. Agents spend money.
    2. Sovereignty already matters.
    3. Trends, hype, and more hype.
    4. Safety & Security
    5. 63% of breached orgs didn't have AI governance in place
    => 가트너의 해결 방법
        => Kong의 방법으로 변경 => Kong AI Connectivity

- Control Tower가 되겠다.


## Context Is King

- From UI to API
    => 사용자가 직접 누르는 게 아니라 에이전트가 API로 해결하는 시대

- Where is AI going?
    1. LLM Cost Increase
    2. Local LLM Capabilities
    3. Managed Agents Eco System
    4. Hybrid Agentic Infra

- LLM Connectivity
    => 다양한 LLM을 한 번에 연결 (AI Gateway)
    => 다양한 MCP를 한 번에 연결 (MCP Gateway)

- 그러나 Production에서 성공하기는 쉽지 않다.
    => Context가 없기 때문에.

- Context: What an agent needs to actually be useful

- Challenges
    1. Raw sources aren't agent-ready
    2. Identity & auth don't translate throughout
    3. Everyone reinvent their own wheel
    => Context가 Mediation Layer가 되어야 한다.
    => Kong의 Context Mesh가 해결할 수 있다.

- Deterministic to non-deterministic


## CJ제일제당의 AI 직원 만들기 프로젝트를 위한 거버넌스 전략

- Dx와 AX 그리고 AI 직원

- 진화의 3단계
    1. LLM: 사고 엔진
    2. AI 에이전트: 자율성
    3. AI 직원: 사람과 동일한 역할
    이후에는 AI 시스템이 된다.

- Why Kong?
    => AI Gateway는 API Gateway와 유사하다.

- AI 직원에게 필요한 것
    1. 비용 거버넌스: Token Rate Limit
    2. 지능 라우팅 및 확장: AI Proxy Advanced
    3. 행동 권한 통제: AI MCP Proxy


## 다양한 에이전트 프로토콜 이해하기

- API -> AI의 API

- 표준화
    - Context (MCP)
    - Execution (Function Calling)

- 또 다시 분산 시스템 => MSA
    => 연결을 위한 프로토콜에 '관리' 더하기 (Observability, Governance, Control)

- Control Plane => 다중 에이전트 워크플로우
    1. MCP Gateway
    2. Agent Gateway
    3. Observability


## Beyond API to AI - AI 중심의 세상을 연결하다

- AI 경쟁력
    : 기존의 레거시 데이터와 서비스를 얼마나 잘 연결하고 활용할 수 있는가?
    => API

- Before: 레거시 -> API
    - Legacy System -> API Transformation -> Digital Transformation
    => API가 곧 디지털 자산
    => 그 API를 단일 Gateway로 관리하는 것이 Kong

- Now: AI와 에이전트의 등장
    => AI를 제어하는 것은 곧 API를 제어하는 것.
    => AI 전환 조건: 중앙 통제(Central Governance) => Kong이 하는 일

- AI Gateway


## MCP 워크플로우: MCP 설계, 테스트 및 디버깅

- Agents act now. MCP is how they reach your tools. Then, how do we run this everywhere safely?
    => Context가 문제

- AI Agent가 각 MCP Server를 호출하기 전에 MCP Gateway를 거치게 한다.