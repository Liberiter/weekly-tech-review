# NVIDIA @ Computex 2026: "에이전트 AI에서 피지컬 AI로"

> 작성일: 2026-06-08
> 주제: NVIDIA Computex 2026 (GTC Taipei) 키노트 리뷰 — Vera Rubin · RTX Spark · 피지컬 AI + 시장 동향
> 행사: COMPUTEX 2026 / GTC Taipei (2026-06-01 ~ 06-05, 타이베이) · 젠슨 황 기조연설 2026-06-01

---

## TL;DR

젠슨 황(Jensen Huang)이 2026-06-01 타이베이에서 약 2시간 키노트를 했다. 한 줄 메시지는 — **"이제 AI는 데이터센터(AI 팩토리)부터 PC, 로봇·자동차까지 모든 컴퓨팅을 다시 짠다"**.

올해 키노트의 무게중심은 작년과 다르다. 작년이 *"더 똑똑한 모델/칩"*이었다면, 올해는 ① **AI 팩토리(Vera Rubin)를 양산 단계로** 끌어올리고 ② **에이전트를 PC 안으로(RTX Spark)** 내리며 ③ **AI를 현실 세계로(피지컬 AI — 로봇·자율주행)** 확장하는 세 갈래다. Computex 전체 테마도 그래서 **"AI Together"**, 키워드는 **"Year of the Agent(에이전트의 해)"**였다.

핵심 키워드 4개: **Vera Rubin · 피지컬 AI(Physical AI) · 온디바이스 에이전트 · AI 팩토리**

읽는 법: 1장은 큰 그림, 2장은 **발표별 상세(무엇/원리/의미/실무 + 구체 수치)**, 3장은 시장 동향·트렌드, 4장은 종합 인사이트, 5장은 역할별 액션, 마지막은 FAQ와 출처다.

> ⚠️ **사실관계 주의(중요)**: Vera Rubin **플랫폼 자체는 2026년 1월 CES에서 처음 공개**됐고, 이번 Computex에서 **"풀 프로덕션(full production) 돌입"**으로 확정·구체화됐다. 본 문서는 어떤 수치가 *CES 공개분*이고 어떤 게 *Computex 확정·신규분*인지 구분해 표기한다. 모든 제품 수치는 6장 1차 출처(NVIDIA 공식)에서 확인 가능하다.

---

## 1. 큰 방향성: "세 개의 컴퓨팅을 동시에 다시 짠다"

젠슨 황의 키노트를 관통하는 그림은 **AI가 세 곳에서 동시에 일어난다**는 것이다.

- **① 데이터센터 = "AI 팩토리"**: 모델을 *돌리는 곳*을 공장처럼 본다. 투입은 전기, 산출은 **토큰(token)**. 그래서 승부는 "와트당·달러당 얼마나 많은 토큰을 뽑느냐"로 수렴한다. Vera Rubin이 이 공장의 새 표준.
- **② PC = "개인 AI 에이전트의 집"**: 클라우드에 묻지 않고 **로컬에서 에이전트를 상시 구동**하도록 PC를 재설계. RTX Spark가 그 진입점.
- **③ 현실 세계 = "피지컬 AI"**: 화면 속 답변을 넘어 **로봇·자율주행차**가 물리 법칙을 이해하고 행동하게 한다. Cosmos 3 · Alpamayo · Isaac GR00T가 이 축.

> 한 줄 정리: 작년이 *"모델 자랑"*이었다면 올해는 **"그 지능을 공장에서 양산하고(데이터센터), 손안에 넣고(PC), 몸을 입힌다(로봇)"**.

이 그림을 떠받치는 경쟁력은 작년과 같다 — **풀스택 수직 통합**(칩 → 시스템 → 네트워크 → 소프트웨어 → 모델). 다만 올해는 그게 *전시*가 아니라 **양산·공급망**의 언어로 내려왔다(아래 Vera Rubin의 "조립 시간 2시간 → 5분"이 상징).

---

## 2. 영역별 상세 리뷰

각 항목은 **① 무엇인가 → ② 어떻게 작동/구성되나 → ③ 왜 중요한가 → ④ 실무 시사점** 순서다.

---

### 🏭 2-1. Vera Rubin — 차세대 AI 팩토리 ("풀 프로덕션" 선언)

**① 무엇인가**
Grace Blackwell의 뒤를 잇는 **차세대 풀스택 AI 슈퍼컴퓨터 플랫폼**. NVIDIA는 이를 *"여섯 개의 신규 칩 + 다섯 개의 랙스케일 시스템 = 하나의 에이전트 AI 슈퍼컴퓨터"*로 표현한다. **CES 2026(1월)에서 공개**됐고, 이번 Computex에서 **"풀 프로덕션 돌입"**과 **2026년 하반기 파트너 제품 출시**를 확정했다.

**② 어떻게 작동/구성되나 — 구체 수치**

여섯 개 신규 칩:

| 칩 | 역할 |
|----|------|
| **Vera CPU** | 에이전트 추론용 CPU (§2-2) |
| **Rubin GPU** | 추론·학습 가속기 |
| **NVLink 6 Switch** | GPU 간 초고속 연결 |
| **ConnectX-9 SuperNIC** | 노드 간 네트워킹 |
| **BlueField-4 DPU** | 데이터·KV 캐시·보안 처리 |
| **Spectrum-6 Ethernet Switch** | AI 팩토리용 이더넷 |

랙스케일 시스템 — **Vera Rubin NVL72** (1개 랙):
- **GPU 72개 + Vera CPU 36개**를 하나의 랙으로 묶음.
- **Rubin GPU**: AI 추론용 **NVFP4 연산 50 페타플롭(PFLOPS)**, 3세대 Transformer Engine(하드웨어 가속 적응형 압축 내장).
- **NVLink 6**: GPU당 **3.6 TB/s**, NVL72 랙 전체 **260 TB/s** — NVIDIA 표현으로 *"인터넷 전체보다 많은 대역폭"*. 인네트워크 컴퓨트로 collective 연산 가속.
- **vs Blackwell 성능 주장**: 추론 **토큰당 비용 최대 10배↓**, MoE 학습에 필요한 **GPU 수 4배↓**, **조립·서비스 최대 18배 빠름**.

생산성/공급망(이번 Computex의 핵심 메시지):
- 케이블 없는(cable-free) 설계로 **트레이 1개 조립 시간이 2시간 → 5분**으로 단축.
- **Spectrum-X 이더넷 포토닉스(co-packaged optics)** 양산 — **전력 효율 5배·AI 가동시간 5배·배포 1.3배 빠름**. 초기 도입사: **CoreWeave, Lambda, Oracle Cloud Infrastructure**.

> 💡 **AI 팩토리를 한 장의 그림으로**
> ```
>   [전기]  ──►  ┌─────────────────────────┐  ──►  [토큰 = 매출]
>               │   Vera Rubin NVL72 랙     │
>               │  GPU 72 + Vera CPU 36     │
>               │  NVLink 6: 260 TB/s/랙    │
>               └─────────────────────────┘
>   승부처:  "1와트·1달러당 토큰을 얼마나 뽑나"
>   Rubin의 답:  토큰당 비용 최대 1/10 (vs Blackwell)
> ```

**③ 왜 중요한가**
키노트의 진짜 뉴스는 *새 칩*이 아니라 **"양산 가능해졌다"**는 점이다. "2시간 → 5분"은 단순 자랑이 아니라 **공급망·생산 캐파(capacity)가 경쟁력**임을 보여준다 — 칩이 아무리 좋아도 못 만들면 못 판다. "토큰당 비용 1/10"은 AI 서비스의 **단위경제성**을 직접 바꾼다(에이전트가 24/7 돌수록 추론 비용이 곧 마진).

**④ 실무 시사점**
- 클라우드/인프라를 쓰는 입장이라면, 2026 하반기부터 **Rubin 기반 인스턴스의 토큰당 단가 인하**를 가격 협상·로드맵에 반영할 여지.
- "성능"보다 **"공급 가능 시점·물량"**이 실제 도입의 병목이 되는 시대 — 벤더 선택 시 *납기·캐파*를 스펙만큼 따질 것.

---

### 🧠 2-2. Vera CPU — "사람이 아니라 에이전트를 위한 CPU"

**① 무엇인가**
Rubin 플랫폼의 두뇌 격 CPU. 슬로건은 *"에이전트 추론(agentic reasoning)을 위한 CPU"* — 기존 CPU가 *사람 중심 워크로드*에 맞춰졌다면, Vera는 **AI 에이전트가 도구를 호출하고 오케스트레이션하는 패턴**에 최적화됐다.

**② 어떻게 작동/구성되나 — 구체 수치**
- **88개 커스텀 "Olympus" 코어**, Armv9.2 호환.
- 메모리 대역폭 **1.2 TB/s LPDDR5X**, 칩렛 경계 없는 **온칩 패브릭 3.6 TB/s**, 단일 스레드 성능을 위한 **클록당 10 명령(IPC)**.
- NVLink-C2C로 Rubin GPU와 직결.
- **성능 주장**: x86 CPU 대비 **에이전트 작업 완료 약 1.8배 빠름**(키노트 발표). NVIDIA는 이를 *"대규모 AI 팩토리에서 가장 전력 효율적인 CPU"*로 포지셔닝.

**③ 왜 중요한가**
주목점은 **CPU의 목적 함수가 바뀌었다**는 것. 에이전트는 "한 번 크게 계산"이 아니라 **"작은 도구 호출을 수없이, 낮은 지연으로"** 반복한다. Vera는 이 패턴(오케스트레이션·저지연)을 겨냥했다. 즉 §2-1의 GPU(연산)와 §2-2의 CPU(조율)가 *에이전트*라는 한 방향으로 같이 설계됐다. (구글이 추론 전용 TPU를 따로 만든 흐름과 동일한 시그널 — [[2026_06_01_Google_IO_2026]] §2-9 참고.)

**④ 실무 시사점**
- 에이전트 단위경제성은 GPU 토큰 비용뿐 아니라 **오케스트레이션(도구 호출·라우팅) 오버헤드**에도 달려 있음을 상기. 워크로드가 "추론 한 방"인지 "도구 다발"인지에 따라 최적 하드웨어가 달라진다.

---

### 💻 2-3. RTX Spark — 개인 AI 에이전트를 위한 PC ("Year of the Agent"의 하드웨어)

**① 무엇인가**
*"개인 AI 에이전트 시대를 위해 Windows PC를 다시 발명한다"*를 표방한 **슈퍼칩 기반 노트북/소형 데스크톱**. NVIDIA 30년 기술(CUDA·RTX·DLSS·FP4·TensorRT·OptiX·Reflex·G-SYNC)을 한 칩에 모았다고 강조. **MediaTek과 공동 개발**.

> 참고: 2025년의 **DGX Spark**(개발자용 미니 AI 박스)와 이름이 비슷하지만, **RTX Spark는 일반 Windows PC(노트북/데스크톱)용**으로 별개 라인이다.

**② 어떻게 작동/구성되나 — 구체 수치**
- **GPU**: Blackwell RTX, **CUDA 코어 6,144개**, **5세대 텐서 코어(FP4 정밀도)**.
- **메모리**: **통합 메모리 128GB**.
- **AI 성능**: 최대 **1 페타플롭(PFLOP)**.
- **CPU**: **20코어 Grace CPU**, GPU와 **NVLink-C2C**로 연결.
- **폼팩터**: 두께 14mm·무게 3파운드(약 1.36kg)급 14~16인치 노트북, 소형 데스크톱. 색재현 정확한 탠덤 OLED + G-SYNC.
- **출시/파트너**: **2026년 가을**, **ASUS·Dell·HP·Lenovo·Microsoft Surface·MSI**(이후 Acer·GIGABYTE).
- **앱 효과 예시**: Adobe Photoshop/Premiere AI·그래픽 **2배 빠름**.

(관련 발표) **DGX Station(GB300)**: **코히어런트 메모리 748GB, 최대 FP4 20 페타플롭, 최대 1조 파라미터 모델 구동**, Windows 버전 발표 — 기업 데스크에 "AI 슈퍼컴"을 놓는 라인.

**③ 왜 중요한가**
이게 Computex 전체를 관통한 **"Year of the Agent"**의 물리적 실체다. 핵심은 **"클라우드에 묻지 않고 로컬에서 에이전트를 상시 구동"** — 128GB 통합 메모리는 *큰 모델을 PC에서 직접 돌리기 위한* 스펙이다. 프라이버시(데이터가 기기 밖으로 안 나감)·지연시간·오프라인·비용 측면에서 클라우드 에이전트와 다른 자리를 노린다.

**④ 실무 시사점**
- **온디바이스 vs 클라우드 에이전트**의 분기점이 하드웨어로 구체화. 민감 데이터·저지연·오프라인 요건이 있는 워크로드는 로컬 에이전트가 선택지가 된다.
- 단, 1차 출처에 **가격 미공개** — 도입 판단은 가을 출시·가격 공개 후로. (소프트웨어 관점의 에이전트 흐름은 [[2026_05_26_Agentic_AI]] 참고; 본 절은 그 **하드웨어/온디바이스 보완판**.)

---

### 🌍 2-4. Cosmos 3 — 피지컬 AI를 위한 월드 파운데이션 모델

**① 무엇인가**
**물리 세계를 이해하는 오픈 파운데이션 모델(omnimodel)**. 텍스트·이미지·비디오·소리를 받아 **현실 세계의 행동/물리 예측**으로 연결한다. 로봇·시뮬레이션이 *적은 데이터로도 현실에 일반화*하도록 돕는 게 목적.

**② 어떻게 작동/구성되나**
- **mixture-of-transformers** 아키텍처. 1인칭/3인칭 시점 모두 지원.
- 추론(reasoning)과 생성(generation) 트랜스포머를 결합해 **물리 예측**을 수행.
- NVIDIA는 피지컬 AI 개발 사이클(평가)을 *"몇 달 → 며칠"*로 단축한다고 주장.

**③ 왜 중요한가**
LLM이 "언어의 월드 모델"이라면, Cosmos는 **"물리의 월드 모델"**을 지향한다. 로봇이 현실에서 학습하려면 데이터 수집이 느리고 비싸다 → **시뮬레이션 안에서 물리를 이해/생성**하는 모델이 그 병목을 푼다. 구글 Gemini Omni의 "월드 모델" 지향([[2026_06_01_Google_IO_2026]] §2-2)과 같은 큰 흐름.

**④ 실무 시사점**
- 로보틱스·제조·물류 R&D: 실물 시험 전 **시뮬레이션 기반 데이터 생성/검증** 비중이 커진다. "현실 데이터 부족"을 합성 데이터·월드 모델로 메우는 전략을 검토할 시점.

---

### 🚗 2-5. Alpamayo 2 Super + DRIVE Hyperion — 추론하는 자율주행

**① 무엇인가**
- **Alpamayo 2 Super**: 자율주행(AV)용 **오픈 추론 모델**. *"세계 최초의 추론하는 자율주행"*을 표방하며, **320억(32B) 파라미터 reasoning VLA(Vision-Language-Action) 모델**로 보도됨.
- **DRIVE Hyperion**: 자율주행 차량용 표준 플랫폼(+ 안전 OS **Halos**).

**② 어떻게 작동/구성되나**
- Alpamayo는 *인지→추론→행동*을 사람처럼 연결하는 것을 목표로, **closed-loop 강화학습** 및 포토리얼리스틱 시나리오 생성(**OmniDreams**)과 함께 제공.
- **DRIVE Hyperion**은 NVIDIA 공식 표현으로 **세계 모빌리티 서비스의 약 97%를 커버**하는 글로벌 표준을 지향. (보도 기준 주요 완성차 파트너로 닛산·현대·메르세데스-벤츠 등이 언급됨 — 2차 출처.)

**③ 왜 중요한가**
자율주행의 패러다임이 *"규칙·인지 모델의 묶음"*에서 **"추론하는 단일 AI(VLA)"**로 이동하는 신호. 모델이 **오픈**이라는 점은 NVIDIA가 *플랫폼(칩+OS+모델)*의 표준을 쥐려는 전략과 맞물린다.

**④ 실무 시사점**
- 모빌리티·로보틱스 업계: VLA(시각-언어-행동) 단일 모델로의 전환과, NVIDIA 표준(Hyperion/Halos) 채택 여부가 공급망 선택의 변수.

---

### 🤖 2-6. Isaac GR00T — 휴머노이드 로봇 레퍼런스 디자인

**① 무엇인가**
**오픈 휴머노이드 로봇 레퍼런스 디자인**(피지컬 AI 연구의 진입장벽을 낮추는 표준 설계). 컴퓨팅 모듈로 **Jetson Thor + Isaac GR00T 소프트웨어**를 쓴다.

**② 어떻게 작동/구성되나**
- 레퍼런스 바디: **Unitree** 휴머노이드 섀시 + 촉각 5지 핸드(Sharpa Wave) + Jetson Thor.
- 생태계 지표: GR00T 모델 누적 **27.4만 다운로드**, GR00T X Embodiment 데이터셋 **Hugging Face 1,000만 다운로드**, GR00T 1.7은 **2만 시간 분량의 사람 1인칭(egocentric) 데이터**로 학습.
- 시뮬레이션 스택 갱신: Isaac Sim 6.0(GA, 그래스핑 가능 에셋 1,000+), Isaac Lab 3.0(Newton 물리, 멀티GPU), Isaac Teleop(GA).

**③ 왜 중요한가**
Computex에 **사상 첫 "AI Robotics Zone"**이 생긴 것과 같은 맥락 — 휴머노이드가 **R&D → 실배포** 문턱을 넘는 중. NVIDIA는 *칩(Jetson Thor) + 시뮬(Isaac/Cosmos) + 데이터(GR00T)*를 묶어 **로봇계의 "안드로이드" 같은 공통 플랫폼**을 노린다.

**④ 실무 시사점**
- 로봇 도입을 검토하는 제조/물류: 자체 풀스택 구축 대신 **레퍼런스 디자인 + 시뮬레이션 학습**으로 PoC 비용·기간을 줄이는 경로가 열린다.

---

### ⚙️ 2-7. Nemotron 3 — 오픈 추론 모델 & 에이전트 인프라

**① 무엇인가**
NVIDIA의 **오픈 추론·멀티모달 모델 패밀리**(에이전트 인프라용). 자사 칩 위에서 에이전트를 잘 돌리기 위한 *모델 레이어*.

**② 어떻게 작동/구성되나 — 구체 수치**
- **Nemotron 3 Ultra**: **5,500억(550B) 파라미터 MoE**, **추론 최대 5배 빠름·운영비 30%↓**, **2026-06-04 제공**. 채택사: Perplexity·Palantir·ServiceNow·CrowdStrike.
- **Nemotron 3.5**: 음성인식 40개 로케일, 콘텐츠 안전 모델(23개 안전 카테고리).
- 에이전트 런타임/툴킷(NemoClaw 등) 및 LangChain·Factory AI 등과의 연동 발표.

**③ 왜 중요한가**
NVIDIA가 **칩만이 아니라 "오픈 모델 + 에이전트 런타임"까지** 직접 제공하며 풀스택을 위로 확장. "30% 싸게, 5배 빠르게"는 §2-1·2-2와 같은 **추론 단가 인하** 메시지의 모델 버전.

**④ 실무 시사점**
- 오픈 가중치 모델을 자가 호스팅하려는 팀에게 선택지 추가. 단, *벤더 종속(NVIDIA 스택 최적화)* 정도를 함께 따질 것.

---

### 🎮 2-8. DLSS 4.5 Ray Reconstruction — 게이밍/그래픽스

**① 무엇인가**
DLSS의 다음 단계. **Ray Reconstruction**(레이 트레이싱에 신경망 렌더링 적용) 기능을 확장. 게이머/크리에이터용 발표.

**② 어떻게 작동/구성되나**
- **2026년 8월 출시 예정**, **모든 GeForce RTX GPU 지원**.
- 입자 효과 선명화·고스팅 감소·공간 인지 향상. 신규 적용 게임 11종(Marvel Rivals, NARAKA: BLADEPOINT, Gothic 1 Remake 등). Blender 5.3에도 올가을 탑재.

**③ 왜 중요한가**
NVIDIA의 핵심 캐시카우(게이밍)가 여전히 **"신경망으로 렌더링을 대체"**하는 방향으로 진화 중임을 확인. AI 기술(텐서 코어)이 게임 품질로 직접 환원되는 선순환.

**④ 실무 시사점**
- 게임·실시간 3D·시각화 개발자: 신경망 렌더링이 표준이 되며, 전통 셰이더 파이프라인 대비 *AI 가속 경로*의 비중이 커진다.

---

## 3. 시장 동향 & 트렌드 (요청 주제)

키노트 밖, **업계 전체의 신호**를 정리한다.

### 3-1. Computex 2026 = "AI Together" / "Year of the Agent"
- 공식 통계(주최 측): 테마 **"AI Together"**, **방문객 111,312명·152개국**, 사상 첫 **AI Robotics Zone** 신설. (포커스: AI 컴퓨팅 · 로보틱스/지능형 모빌리티 · 차세대 기술)
- 기조 화자: **Qualcomm 크리스티아노 아몬, Intel 립부 탄, Marvell, NXP** 등 — 즉 NVIDIA만의 잔치가 아니라 **반도체 업계 전체가 "에이전트"로 수렴**.
- 관통 메시지: **"PC를 클라우드 허락 없이 로컬에서 에이전트를 상시 구동하도록 재설계"** — Qualcomm CEO는 **"2026년은 에이전트의 해"**라고 선언(2차 출처).

### 3-2. 무게중심 이동: "클라우드 모델" → "온디바이스 + 피지컬 AI"
- **온디바이스 에이전트**: NVIDIA RTX Spark, Intel·Qualcomm의 AI PC 발표가 한 방향. *에이전트가 폰·PC·로봇 등 엣지에서 상시 동작*하는 그림.
- **피지컬 AI**: PwC Strategy& 추정 — **피지컬 AI 글로벌 시장가치 2030년 약 €4,300억(€430B)**, 본격 상업화는 3~5년 내(2차 출처). Computex의 첫 로보틱스 존이 이를 뒷받침.

### 3-3. "버블인가, 산업 인프라인가" — AI CapEx 대투자
- AI 투자가 *투기적 IT 지출*이 아니라 **산업 인프라 건설**에 가깝다는 프레이밍이 확산. **Morgan Stanley는 2028년까지 글로벌 데이터센터 건설비만 약 $2.9조**로 추정(2차 출처).
- 같은 맥락에서 NVIDIA의 "토큰당 비용 1/10"·"가장 전력 효율적"이라는 메시지는, **천문학적 CapEx를 정당화할 단위경제성(전력·단가)**으로 경쟁축이 옮겨갔음을 보여준다. (구글 연 CapEx $180B+ 흐름과 동일 — [[2026_06_01_Google_IO_2026]] §1·§3.)

> 📌 **표로 보는 큰 그림**
>
> | 축 | 2025까지 | 2026(Computex) |
> |----|----------|----------------|
> | 자랑거리 | "더 똑똑한 모델/칩" | **"양산·공급망·단가"** (2h→5min, 토큰 1/10) |
> | AI가 도는 곳 | 주로 클라우드 | 클라우드 **+ PC(온디바이스) + 로봇/차(피지컬)** |
> | 경쟁 키워드 | 지능(intelligence) | **에이전트 × 와트당·달러당 토큰** |
> | 업계 분위기 | 모델 경쟁 | **"Year of the Agent" + 인프라 건설 붐** |

---

## 4. 종합 인사이트 — 읽어내야 할 시그널

1. **메시지가 "성능 자랑"에서 "양산·공급망"으로 내려왔다.**
   "조립 2시간 → 5분", "풀 프로덕션"이 키노트의 진짜 헤드라인. 칩 성능이 아니라 **만들어 낼 수 있는 능력(캐파)**이 해자(moat)가 됐다.

2. **경쟁축이 '지능'에서 '와트당·달러당 토큰'으로 완전히 이동.**
   Rubin의 "토큰당 비용 1/10", Vera의 "가장 전력 효율적", Nemotron의 "30% 싸게"가 모두 같은 곡을 부른다 — **AI 팩토리의 단위경제성**. (구글 I/O의 Flash·추론 TPU와 정확히 같은 방향.)

3. **"에이전트"가 소프트웨어를 넘어 하드웨어로 내려왔다.**
   작년의 에이전트가 *모델·프레임워크* 얘기였다면([[2026_05_26_Agentic_AI]]), 올해는 **에이전트 전용 CPU(Vera)·에이전트 PC(RTX Spark)**로 실리콘에 새겨졌다. "Year of the Agent"는 칩 레벨의 사건.

4. **피지컬 AI가 다음 전장으로 공식화.**
   Cosmos 3(월드 모델) · Alpamayo(자율주행 VLA) · Isaac GR00T(휴머노이드)가 한 묶음. Computex 첫 로보틱스 존, PwC €430B 전망이 시장 신호. **"화면 속 답"에서 "현실의 행동"으로.**

5. **NVIDIA의 전략은 여전히 "플랫폼을 깐다".**
   GR00T(로봇 레퍼런스), DRIVE Hyperion(자율주행 표준), 오픈 Cosmos/Alpamayo/Nemotron — *오픈을 무기로 표준의 길목*을 쥔다. 구글이 프로토콜(UCP/AP2)로 길목을 쥐려는 것과 같은 게임, 다른 레이어(실리콘·로봇).

6. **남는 질문**: ① 천문학적 CapEx(데이터센터 $2.9조 추정)를 받쳐줄 **실수요·수익화**가 따라오는가(버블 논쟁). ② 온디바이스 에이전트의 **가격(RTX Spark 미공개)**과 실사용성. ③ 피지컬 AI의 **안전·책임**(로봇·차가 "추론"으로 행동할 때의 신뢰).

---

## 5. 역할별 "그래서 뭘 해야 하나"

| 당신이 …라면 | 주목할 항목 | 액션 |
|--------------|------------|------|
| **인프라/플랫폼 엔지니어** | Vera Rubin, Spectrum-X 포토닉스 | 2026 하반기 Rubin 기반 인스턴스의 **토큰당 단가·납기**를 로드맵/계약에 반영 |
| **AI 제품/서비스 기획** | 토큰당 비용 1/10, Nemotron 30%↓ | AI 기능의 **단위경제성(토큰비×호출수)** 재계산, 가격·마진 재설계 |
| **에이전트 개발자** | Vera CPU, RTX Spark, NemoClaw | **온디바이스 vs 클라우드** 에이전트 분기 검토(민감데이터·저지연·오프라인 요건) |
| **로보틱스/제조 R&D** | Cosmos 3, Isaac GR00T | 실물 시험 전 **시뮬레이션 학습/합성데이터** 비중 확대, 레퍼런스 디자인으로 PoC |
| **모빌리티/자동차** | Alpamayo 2, DRIVE Hyperion/Halos | **VLA 단일모델 전환**과 NVIDIA 표준 채택 여부를 공급망 전략에 반영 |
| **경영/전략** | CapEx $2.9조, 피지컬 AI €430B | AI 투자를 *산업 인프라*로 보되 **수익화·실수요**로 ROI 검증, 피지컬 AI 시장 진입 타이밍 점검 |
| **게임/실시간 3D** | DLSS 4.5 Ray Reconstruction | 신경망 렌더링(8월) 대응, AI 가속 렌더 경로 비중 확대 |

---

## 6. FAQ (왕초보용)

**Q1. Computex랑 GTC Taipei가 뭐가 달라요?**
Computex는 타이베이에서 열리는 **세계 최대급 컴퓨터/하드웨어 전시회**(매년 6월). 올해 NVIDIA는 여기에 자사 행사 **GTC를 얹어("GTC Taipei at Computex")** 젠슨 황 키노트를 진행했다. 즉 *장소·시점은 Computex, 발표 주체는 NVIDIA*.

**Q2. "AI 팩토리"가 비유인가요, 진짜 공장인가요?**
비유다. **데이터센터를 공장에 빗댄 표현** — 투입은 전기, 산출은 토큰(AI 응답). "토큰을 얼마나 싸게·많이 찍어내느냐"가 핵심이라 *공장*이라 부른다.

**Q3. RTX Spark랑 작년 DGX Spark랑 같은 거예요?**
아니다. **DGX Spark(2025)**는 개발자용 미니 AI 박스, **RTX Spark(2026)**는 **일반 Windows 노트북/데스크톱용** 슈퍼칩이다. 이름만 비슷한 별개 라인.

**Q4. NVFP4 / 페타플롭(PFLOP)이 뭔가요?**
- **PFLOP** = 초당 1,000조 번 연산. AI 칩 성능 단위.
- **NVFP4** = NVIDIA의 **4비트 부동소수점** 포맷. 숫자를 더 적은 비트로 표현해 **같은 전력으로 더 많은 추론**을 돌리는 게 목적(정밀도↓ 대신 처리량·효율↑). "토큰당 비용 1/10"의 기술적 토대 중 하나.

**Q5. "피지컬 AI(Physical AI)"가 뭔가요?**
화면 속 텍스트/이미지를 다루는 AI를 넘어, **로봇·자율주행차처럼 물리적 몸을 가지고 현실에서 행동**하는 AI. 물리 법칙(중력·마찰·충돌)을 이해해야 해서, 그걸 학습시키는 월드 모델(Cosmos)·시뮬레이터(Isaac)가 함께 등장한다.

**Q6. VLA 모델이 뭔가요?**
**Vision-Language-Action** — *본다(영상) → 언어로 추론한다 → 행동한다*를 하나의 모델로 묶은 것. 자율주행/로봇이 규칙 묶음 대신 **단일 추론 모델**로 운전·조작하게 하는 접근(Alpamayo가 예).

---

## 7. 출처 (Sources)

### 1차 출처 (NVIDIA 공식)
- [NVIDIA GTC Taipei at COMPUTEX: Live Updates on What's Next in AI — blogs.nvidia.com](https://blogs.nvidia.com/blog/nvidia-gtc-taipei-computex-2026-news/) ← 키노트 종합(Vera Rubin NVL72, Vera CPU 88코어, RTX Spark, Cosmos 3, Alpamayo 2, Isaac GR00T, Nemotron 3 Ultra, DRIVE Hyperion 등)
- [NVIDIA Kicks Off the Next Generation of AI With Rubin — Six New Chips, One Incredible AI Supercomputer — nvidianews.nvidia.com](https://nvidianews.nvidia.com/news/rubin-platform-ai-supercomputer) ← Rubin 6칩, Rubin GPU 50 PFLOPS NVFP4, NVLink 6(3.6TB/s·랙 260TB/s), 토큰비 10배↓, 풀 프로덕션 (※ 플랫폼 최초 공개는 CES 2026/1월)
- [NVIDIA at COMPUTEX 2026: RTX Spark, DLSS 4.5, RTX Updates — nvidia.com/geforce](https://www.nvidia.com/en-us/geforce/news/computex-2026-nvidia-geforce-rtx-announcements/) ← RTX Spark(6,144 CUDA·128GB·1 PFLOP·20코어 Grace·가을 출시·파트너), DLSS 4.5 Ray Reconstruction(8월·전 RTX)
- [NVIDIA GTC Taipei 2026 Keynote (CEO Jensen Huang) — nvidia.com/en-tw/gtc/taipei](https://www.nvidia.com/en-tw/gtc/taipei/keynote/) ← 키노트 일시(6/1, 타이베이)

### 2차 출처 (언론·정리)
- [Three key takeaways from NVIDIA at Computex 2026 — CRN Asia](https://www.crnasia.com/news/2026/components-and-peripherals/three-key-takeaways-from-nvidia-at-computex-2026) ← Vera CPU "x86 대비 1.8배", 조립 2h→5min, RTX Spark 스펙
- [NVIDIA Computex 2026 Keynote Live Coverage — ServeTheHome](https://www.servethehome.com/nvidia-computex-2026-keynote-live-coverage/) ← Vera CPU 1.2TB/s·IPC, Nemotron, Isaac GR00T(Unitree/Jetson Thor)
- [5 Of The Coolest Reveals From Nvidia's 2026 Computex Event — AOL](https://www.aol.com/articles/5-coolest-reveals-nvidias-2026-194700000.html) ← Alpamayo 2 Super(32B VLA), DLSS 4.5 게임 목록, Isaac GR00T 구성, RTX Spark 128GB/1PFLOP
- [Nvidia Computex 2026 keynote as it happened — TechRadar](https://www.techradar.com/news/live/nvidia-computex-2026)
- [COMPUTEX 2026 Concludes Successfully as Global Innovation Shapes a New AI Ecosystem — PR Newswire](https://www.prnewswire.com/news-releases/computex-2026-concludes-successfully-as-global-innovation-shapes-a-new-ai-ecosystem-302792536.html) ← 공식 통계(테마 "AI Together", 방문객 111,312명/152개국, AI Robotics Zone), PwC 피지컬 AI €430B(2030)
- [COMPUTEX 2026 Opens Amid Surging Global Demand for AI Infrastructure — PR Newswire](https://www.prnewswire.com/news-releases/computex-2026-opens-amid-surging-global-demand-for-ai-infrastructure-302788497.html)
- [Highlights From Computex 2026: The Hardware Industry Just Bet Everything on AI Agents — Medium](https://medium.com/@noahbean3396/highlights-from-computex-2026-the-hardware-industry-just-bet-everything-on-ai-agents-f1400d478538) ← Qualcomm "Year of the Agent"
- [AI Market Trends 2026: Global Investment, Risks, and Buildout — Morgan Stanley](https://www.morganstanley.com/insights/articles/ai-market-trends-institute-2026) ← 데이터센터 건설비 ~$2.9조(2028까지) 추정 *(원문 직접 확인 권장 — 작성 시 페이지 응답 지연)*

> ⚠️ **디스클레이머**: 본 문서는 위 출처를 바탕으로 방향성·큰 그림 중심으로 요약·해설한 것이다.
> ① **CES vs Computex 구분**: Vera Rubin 플랫폼의 상세 스펙(50 PFLOPS·NVLink 260TB/s·88코어 등)은 **2026년 1월 CES 공개분**이며, 이번 Computex는 **"풀 프로덕션 돌입 + 하반기 출시"**를 확정한 것이다.
> ② **검증 수준**: NVIDIA 제품 수치는 1차(공식) 출처로 교차확인했다. **시장 수치(Morgan Stanley $2.9조, PwC €430B)와 "1.8배·80% 완성차" 등 일부 값은 2차 출처 기준**이며, RTX Spark **가격은 미공개**다. 정확한 값·최신 수치는 1차 출처를 직접 확인하길 권장한다.
