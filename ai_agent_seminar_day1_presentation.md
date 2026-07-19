# AI Agent 서비스 설계의 기본 구성요소

---

# AI = LLM 이라는 오해

AI는 사람의 지능적 작업을 기계가 수행하도록 만드는 넓은 분야입니다.

```text
AI
└─ Machine Learning
   └─ Deep Learning
      └─ Foundation Model
         ├─ LLM
         └─ Multimodal Model
```

- 예측 모델 (Random Forest, XGBoost, LightGBM)
- 추천 시스템 (Matrix Factorization, Wide & Deep, DeepFM)
- 이상 탐지 (Isolation Forest, One-Class SVM, Autoencoder)
- 컴퓨터 비전 (ResNet, YOLO, EfficientNet)
- 음성 인식 (Whisper, wav2vec 2.0, DeepSpeech)
- 강화학습 (DQN, PPO, AlphaZero)
- 언어 모델 (BERT, T5, GPT)

LLM은 그중 언어 중심 Foundation Model입니다.

---

# LLM이 잘하는 것

LLM은 언어와 패턴을 다루는 데 강합니다.

- 자연어 의도 파악
- 비정형 입력을 구조화된 형태로 변환
- 요약
- 분류
- 설명
- 코드 생성 보조
- 텍스트 변환
- 패턴 해석

---

# LLM이 약한 것

- 학습 데이터에 없는 최신 사실을 스스로 알 수 없음
  - LLM의 기본 지식은 학습 시점과 입력 컨텍스트에 갇혀 있기 때문
- 입력에 없는 시스템 상태를 추측할 수 있음
  - 관측한 데이터와 추론한 내용을 명확히 구분하지 못할 수 있기 때문
- 그럴듯하지만 틀린 답변을 생성할 수 있음
  - 사실 검증보다 자연스럽고 가능성 높은 문장 생성에 최적화되어 있기 때문
- 긴 문맥에서 세부 조건을 누락할 수 있음
  - 긴 입력에서 모든 제약을 같은 중요도로 유지하기 어렵기 때문
- 복잡한 수치 계산이나 단위 변환에서 흔들릴 수 있음
  - LLM은 계산 과정을 계산기처럼 동작하지 않고 다음 토큰을 확률적으로 생성하기 때문
- 원인 후보와 확정 사실을 구분하지 못할 수 있음
  - 가능성 높은 설명을 확정적인 결론처럼 표현할 수 있기 때문

---

# 최신성·환각·문맥 누락 보완

LLM이 약한 부분은 시스템 설계로 보완해야 합니다.

1. **최신성/외부 상태 한계 보완**
   - DB, Metric, Log 조회 Tool 연결
   - 조회 시점과 시간 범위 명시
   - Tool 호출 결과를 LLM 입력에 포함

2. **환각 보완**
   - 근거 중심 Prompt 작성
   - 원인 후보와 확정 사실 구분
   - 최종 답변 검증 레이어 추가

3. **문맥 누락 보완**
   - 입력 데이터를 구조화
   - State로 중간 결과 관리
   - Prompt를 짧고 명확한 역할별 단계로 분리

---

# 계산 오류와 모호한 질문 보완

4. **수치 계산/단위 오류 보완**
   - 계산은 LLM 전달 전 단계에서 수행
   - rule-based check 추가
   - 수치 결과와 자연어 해석을 분리(역할 분리)

5. **모호한 질문 보완**
   - 필수 파라미터 정의
   - 정보 부족 시 추가 질문 생성
   - 기본값을 사용한 경우 답변에 명시

---

# 도메인 기준과 답변 일관성 보완

6. **도메인 기준 부족 보완**
   - 사내 운영 기준과 threshold 정리
   - Runbook/RAG 연결
   - 도메인별 Agent Prompt에 판단 기준 포함

7. **답변 일관성 보완**
   - structured output과 Schema 사용
   - Prompt/model version 기록
   - 평가셋 기반 회귀 테스트 수행


---

# AI 서비스 설계란 무엇인가

AI 서비스 개발에서 중요한 설계는 단순히 모델을 고르는 일이 아닙니다.

LLM을 소프트웨어 시스템 안에 어떻게 배치할지 결정하는 일입니다.

- 어떤 데이터를 LLM에 줄 것인가
- 어떤 Tool을 붙일 것인가
- State를 어떻게 관리할 것인가
- 실패, 재시도, 검증을 어떻게 설계할 것인가

---

# AI 서비스 설계의 추가 질문

- Prompt와 Schema를 어떻게 구조화할 것인가
- 로그와 평가를 어떻게 남길 것인가
- 권한과 승인 흐름을 어떻게 둘 것인가
- 사용자의 피드백을 어떻게 다시 개선에 사용할 것인가

따라서 AI 서비스 개발에는 프롬프트 작성 능력뿐 아니라 소프트웨어 설계와 구현 역량이 함께 필요합니다.

---

# 처음부터 완벽한 AI 서비스는 없다

AI 서비스는 처음부터 완벽한 답변을 내는 제품이라기보다,
실제 사용 데이터와 실패 사례를 축적하면서 개선되도록 설계해야 하는 시스템입니다.

중요한 것은 한 번 좋은 답을 내는 것이 아니라,
실패를 다음 개선으로 연결하는 구조입니다.

---

# 무엇을 축적해야 하는가

- 사용자 질문
- 입력 파라미터
- Agent 실행 경로
- Tool 호출 결과
- 중간 분석 결과
- 최종 답변
- Quality Gate 결과
- 사용자 피드백
- 사후 확인된 실제 원인

---

# 축적한 데이터는 어디에 쓰는가

- Prompt 개선
- Routing 개선
- 평가 데이터셋 구축
- 회귀 테스트
- Runbook/RAG 자료화
- 자주 나오는 장애 패턴 정리
- Agent 품질 대시보드 구성

AI 서비스는 데이터를 축적할수록 도메인에 맞게 똑똑해질 수 있습니다.

---

# 반복 가능한 작업 방식도 설계해야 한다

Tool, State, Logging만으로 Agent 품질이 자동으로 안정화되지는 않습니다.

실제 운영에서는 Agent가 매번 같은 기준과 절차로 일하도록 작업 방식을 명시해야 합니다.

- 어떤 순서로 문제를 분석할 것인가
- 어떤 기준으로 코드를 리뷰할 것인가
- 어떤 테스트를 통과해야 완료로 볼 것인가
- 어떤 상황에서 사용자 승인이나 추가 질문이 필요한가
- 프로젝트별 금지 사항과 컨벤션은 무엇인가

이처럼 Agent의 실행 절차와 판단 기준을 재사용 가능한 형태로 정리한 것이 Skill입니다.

---

# Skill은 Agent의 작업 방식이다

Skill은 Agent에게 "어떻게 일해야 하는지"를 알려주는 운영 매뉴얼에 가깝습니다.

- 반복 작업 절차
- 코드 리뷰 기준
- 테스트 기준
- 배포 절차
- 문서화 방식
- 프로젝트 컨벤션
- 금지 사항

---

# Skill 생태계 예시

- Compound Engineering
  - brainstorm, plan, work, review, commit/push/PR 같은 개발 workflow를 구조화
  - 예: 기능 아이디어를 요구사항으로 정리하고, 구현 계획을 만든 뒤, 코드 리뷰와 PR 생성까지 일관된 절차로 수행
- addyosmani/agent-skills
  - AI coding agent를 위한 production-grade engineering skills 모음
  - 예: 접근성 점검, 성능 최적화, 코드 품질 리뷰, 테스트 작성 같은 반복 엔지니어링 작업을 skill로 수행
- nexu-io/open-design
  - coding agent를 design engine처럼 사용해 prototype, landing page, dashboard, slide 등 산출물 생성
  - 예: 자연어 요구사항을 기반으로 대시보드 시안, 랜딩 페이지, 발표용 화면을 빠르게 생성하고 반복 수정

---

# LangChain

LangChain 공식 문서는 LangChain을 Agent와 LLM 애플리케이션을 빠르게 만들기 위한 프레임워크로 설명합니다.

1. **Model/Tool 통합**
   - Provider마다 API, 파라미터, message format이 다릅니다.
   - LangChain은 표준 모델 인터페이스와 다양한 통합을 제공해 모델과 Tool을 일관된 방식으로 연결하게 합니다.

2. **Agent 기본 구조 제공**
   - Agent는 모델과 Tool을 결합해 어떤 Tool을 사용할지 판단하고 반복적으로 작업을 수행합니다.
   - LangChain의 Agent는 LangGraph 위에서 동작하며 durable execution, streaming, human-in-the-loop, persistence 같은 기능을 활용할 수 있습니다.

즉, LangChain은 단순 모델 호출 wrapper라기보다  
모델, Tool, Agent loop를 빠르게 구성하기 위한 상위 프레임워크에 가깝습니다.

출처: LangChain 공식 문서 Overview, Agents

---

# LangGraph

LangGraph 공식 문서는 LangGraph를 장기 실행되고 상태를 유지해야 하는 Agent workflow를 만들고 운영하기 위한 orchestration framework/runtime으로 설명합니다.

- State: Agent가 작업 중 유지해야 하는 현재 상태
- Node: 모델 호출, Tool 실행, 검증 같은 작업 단계
- Edge: 다음에 어떤 단계로 갈지 결정하는 흐름
- Persistence: 실패하거나 중단되어도 이어서 실행할 수 있는 상태 저장
- Human-in-the-loop: 중요한 단계에서 사람의 검토나 승인을 끼워 넣는 구조

즉, LangChain이 Agent를 빠르게 만들기 위한 상위 프레임워크라면,  
LangGraph는 복잡한 Agent 실행 흐름을 명시적으로 제어하기 위한 기반입니다.

출처: LangGraph 공식 문서 Overview, Graph API overview

---

# LangChain과 LangGraph의 역할


| 구분 | LangChain | LangGraph |
|---|---|---|
| 중심 역할 | Agent framework와 Model/Tool 통합 | 상태 기반 Agent orchestration runtime |
| 주 관심사 | Model, Tool, Agent loop, Middleware | State, Node, Edge, Routing, Persistence |
| 흐름 구조 | 빠르게 시작할 수 있는 사전 구성 Agent 구조 | 조건부 분기, 반복, 병렬, 재시도, 장기 실행 |
| 적합한 작업 | LLM 앱, Tool calling Agent, 구조화 출력 | Multi-Agent, human-in-the-loop, 장기 실행 workflow |


---

# RAG 활용 방안

RAG는 LLM이 모든 지식을 내부에 기억하도록 만드는 방식이 아니라,
필요한 시점에 외부 지식과 근거 문서를 검색해 답변에 반영하는 구조입니다.

앞에서 본 최신성 한계, 도메인 기준 부족, 근거 부족 문제를 줄이는 데 특히 유용합니다.

- 장애 대응 Runbook 검색
- 사내 DB 운영 매뉴얼 검색
- 과거 장애 사례 검색
- SQL 튜닝 가이드 검색
- 정기 점검 절차 검색
- 장애 보고서 템플릿 검색
- 서비스별 운영 정책 검색
- FAQ와 문의 이력 검색

---

# MCP 개념

MCP = Model Context Protocol


MCP Server가 제공하는 것:
- Tool: 모델이 호출할 수 있는 기능
- Resource: 모델이 참고할 수 있는 데이터
- Prompt: 재사용 가능한 지시 템플릿

MCP 연결 구조:
- Server: Tool, Resource, Prompt를 제공하는 쪽
- Client: MCP Server에 연결해 기능을 모델/Agent에게 노출하는 쪽

핵심은 Tool을 애플리케이션 내부 함수가 아니라 표준 인터페이스를 가진 외부 능력으로 분리하는 것입니다.

---

# 일반 Tool과 MCP의 차이

| 구분 | 일반 Tool | MCP |
|---|---|---|
| 연결 방식 | 앱 내부 함수로 직접 연결 | 표준 프로토콜로 Server 연결 |
| 재사용성 | 해당 앱에 종속되기 쉬움 | 여러 앱/Agent에서 재사용 가능 |
| 확장 방식 | 코드 수정으로 Tool 추가 | MCP Server를 연결해 확장 |
| 책임 분리 | Agent 코드와 Tool 구현이 가까움 | Agent와 Tool 제공 계층을 분리 |

---

# A2A 개념

A2A는 Agent-to-Agent입니다.

서로 다른 Agent가 작업을 위임하고, 메시지를 주고받고, 결과를 돌려주는 협업 구조를 뜻합니다.

- Agent Discovery
- Task Delegation
- Message Exchange
- 결과 반환
- 독립 Agent 서비스 간 협업
- 중앙 Orchestrator와 전문 Agent 분리

단일 애플리케이션 안의 함수 호출보다, 독립적으로 운영되는 Agent 서비스 간 협업에 가깝습니다.

---

# A2A는 언제 필요한가


- Agent를 팀/도메인별로 독립 배포해야 할 때
- DB Agent, OS Agent, Log Agent처럼 책임 경계가 명확할 때
- 서로 다른 프레임워크나 시스템의 Agent를 연결해야 할 때
- 중앙 Orchestrator가 전문 Agent에게 작업을 위임해야 할 때
- Agent 간 인증, 메시지 표준, 장애 전파 관리가 필요할 때


---

# Harness Engineering

Harness Engineering은 모델 주변 실행 환경을 설계하는 관점입니다.

1. 모델 성능이 상향 평준화
   - 좋은 모델을 쓰는 것만으로는 차별화가 어렵고, 이제는 모델을 어떻게 제품/업무 흐름에 연결하느냐가 중요

2. AI 서비스의 병목이 모델 밖에서 발생
   - 데이터 연결, Tool 설계, Prompt/Schema, Logging, Evaluation, 권한 관리가 실제 품질과 안정성을 결정함.

3. Agentic application이 늘어나고 있기 때문
   - 모델이 단순히 답변하는 것이 아니라 Tool을 쓰고 작업을 수행하면서, State, Workflow, Verification, Observability가 필수가 됨.

4. Production 수준의 신뢰성이 요구되기 때문
   - 데모는 쉽게 만들 수 있지만, 운영 서비스는 실패 처리, 재시도, 추적, 평가, 보안, 승인 흐름이 필요함.

5. 개선 루프가 중요해졌기 때문
   - AI 서비스는 처음부터 완벽하지 않기 때문에 질문, 실행 경로, Tool 결과, 답변, 피드백, 실패 사례를 축적하고 다시 개선하는 구조가 필요함.

---
