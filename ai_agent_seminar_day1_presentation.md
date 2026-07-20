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

# Part 2. LLM의 약점을 시스템으로 보완하기

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

# Part 3. AI 서비스 설계와 개선 루프

---

# AI 서비스 설계란 무엇인가

AI 서비스 설계란,

LLM의 강점은 활용하고 약점은 소프트웨어 구조로 보완해
사용자의 요청이 신뢰 가능한 결과와 행동으로 이어지게 만드는 일입니다.

- LLM이 판단에 필요한 정보를 어떻게 받을 것인가
- 어떤 외부 시스템을 조회하거나 실행하게 할 것인가
- 중간 결과와 사용자 맥락을 어떻게 유지할 것인가
- 틀린 답, 실패, 불확실성을 어떻게 감지하고 복구할 것인가

---

# AI 서비스 설계의 핵심 질문

- LLM이 반드시 지켜야 할 입력·출력 계약은 무엇인가
- 무엇을 성공·실패로 기록하고 어떻게 회귀 테스트할 것인가
- 어떤 행동은 자동 실행하고 어떤 행동은 사람 승인 뒤 실행할 것인가
- 사용자 수정과 실패 사례를 어떻게 평가셋·Prompt·Rule 개선으로 연결할 것인가

따라서 AI 서비스 개발에는 프롬프트 작성 능력뿐 아니라
데이터, Tool, State, 검증, 운영 피드백을 설계하는 소프트웨어 역량이 필요합니다.

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
- Prompt / Version
- Agent 실행 경로
- Tool 호출 결과
- 중간 분석 결과
- 최종 답변
- 검증 / 승인 결과
- 사후 정답 / 피드백

---

# 축적한 데이터는 어디에 쓰는가

- Prompt / Schema 개선
- Tool / Routing 개선
- 평가셋 구축
- 회귀 테스트
- Runbook / RAG 개선
- 도메인 기준 정교화
- Agent 품질 대시보드 구성
- 운영 자동화 범위 조정

AI 서비스는 데이터를 축적할수록 더 정확한 답을 하는 것을 넘어,
더 잘 검증되고 더 일관되게 운영되는 시스템으로 개선됩니다.

---

# 실패 사례를 품질 개선 재료로 바꾸기

축적 데이터는 단순 로그가 아니라,
Agent가 어떤 신호를 과대 해석했는지 찾아내는 근거가 됩니다.

예: 장애 원인 분석 Agent의 오답

- 사용자 질문: "결제 API 장애 원인 찾아줘."
- 초기 답변: "최근 배포 이후 오류가 증가했으므로 배포가 원인으로 보입니다."
- 문제: 배포 시점과 오류 증가만 보고 원인을 단정했습니다.
- 사후 확인: 실제 원인은 DB connection pool 고갈이었습니다.
- 배포는 같은 시간대에 있었지만 직접 원인은 아니었습니다.

---

# 실패 패턴을 설계 변경으로 반영

오답을 모아보면 Prompt 문구만의 문제가 아니라,
어떤 Tool을 언제 조회하고 어떤 형식으로 판단할지의 문제로 드러납니다.

- 패턴 분석: 배포 시점과 오류 증가가 겹치면 배포 원인으로 과대 추론
- Prompt 개선: 확정 사실과 원인 후보를 반드시 분리하도록 지시
- Tool 경로 보강: DB pool, connection timeout metric 조회를 필수 단계로 추가
- Schema 보강: evidence, hypothesis, missing_check 필드 추가
- 판단 기준 추가: 상관관계만으로 원인을 확정하지 않도록 규칙화
- 답변 톤 조정: 근거가 부족하면 단정 대신 추가 확인 항목을 제시

---

# 같은 오답이 다시 나오지 않게 검사

이전에 틀렸던 상황을 다시 입력하고,
같은 단정이 반복되면 실패로 처리합니다.

예전 오답을 테스트 케이스로 저장하는 3단계

1. 상황 재현

- 결제 API 5xx 증가
- 같은 시간대에 배포 기록도 존재
- DB connection pool 고갈 metric도 존재

2. 통과 기준 정의

- 배포를 확정 원인으로 단정하지 않음
- DB pool 고갈을 더 강한 근거로 제시
- 확정 사실, 원인 후보, 추가 확인을 분리

3. 실패 조건 설정

- "배포가 원인입니다"처럼 단정하면 실패
- DB metric 근거가 빠지면 실패
- Prompt/model 변경 때마다 이 케이스 재실행

---

# 실패 사례를 Runbook/RAG 지식으로 바꾸기

사후 원인이 확인된 장애 사례는
다음 장애 대응에 재사용할 수 있는 운영 지식으로 정리합니다.

DB pool 고갈 장애를 Runbook/RAG 지식으로 만드는 3단계

1. 사후 원인 확인

- 결제 API 5xx와 latency 증가
- DB connection pool saturation 확인
- 배포는 같은 시간대였지만 직접 원인은 아님

2. Runbook 규칙 추가

- 배포 단정 전 DB pool usage와 connection timeout 확인
- rollback 판단 전 DB access pattern 변경 여부 확인
- owner, updated_at, 문서 version 기록

3. RAG 검색 문서화

- service: payment-api
- symptom: 5xx 증가, latency 증가
- metric: db.pool.usage, connection_timeout
- action: pool 설정 확인, DB 부하 확인, scale-out

---

# Runbook/RAG를 근거로 답변하기

RAG는 LLM이 운영 기준을 외워서 답하게 하는 것이 아니라,
필요한 시점에 근거 문서를 찾아 답변 입력에 포함하는 구조입니다.

질문: "결제 API 5xx가 늘었는데 바로 rollback해야 하나요?"

검색된 근거

- payment-api 장애 대응 Runbook v3.2
- DB connection pool 고갈 대응 절차
- 배포 후 장애 판단 기준

답변 기준

- 5xx와 DB pool 사용률이 함께 증가하면 DB pool 먼저 확인
- 배포 시점만으로 rollback을 확정하지 않음
- 근거가 배포 변경과 직접 연결될 때 rollback 후보로 판단

Agent 답변

"현재 정보만으로 rollback을 확정하기는 어렵습니다.
Runbook 기준상 DB pool usage와 connection timeout을 먼저 확인한 뒤 rollback 여부를 판단하겠습니다."

---

# Part 4. 반복 가능한 작업 방식: Skill

---

# 여기서 말하는 Skill이란?

Skill은 Agent에게 "어떻게 일해야 하는지"를 알려주는 작업 매뉴얼입니다.

특정 제품의 기능명이라기보다,
반복 가능한 절차와 판단 기준을 구조화한 개념입니다.

- 개념
  - Agent가 반복 작업을 수행할 때 따르는 절차, 기준, 금지 사항
- 구현 예시
  - Claude Code나 Codex의 Skills는 이런 작업 방식을 파일과 규칙으로 패키징한 형태
- 포함되는 것
  - 분석 순서, 리뷰 기준, 테스트 기준, 배포 절차, 문서화 방식
- 구분할 것
  - Tool, MCP, RAG처럼 외부 능력이나 지식 검색 자체를 뜻하는 말은 아님

---

# Skill로 만들 수 있는 것들

- 코드 리뷰 Skill
  - 변경 파일을 읽고, 버그·회귀·테스트 누락을 우선 점검
- 장애 분석 Skill
  - Metric, Log, Deploy 기록을 정해진 순서로 조회하고 원인 후보를 분리
- 문서 작성 Skill
  - 대상 독자, 구조, 용어 기준에 맞춰 문서 초안을 작성하고 검토
- 테스트 작성 Skill
  - 요구사항과 변경 코드를 기준으로 happy path, edge case, failure path 생성
- PR 작성 Skill
  - 변경 요약, 검증 결과, 리스크, 배포 후 확인 항목을 일정한 형식으로 정리
- 발표 자료 작성 Skill
  - 원본 문서에서 핵심 메시지를 뽑고, 슬라이드 구조와 시각 계층을 맞춤

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

# Compound Engineering 핵심 흐름

- 요구사항 정리
  - /ce-brainstorm → /ce-plan
  - 아이디어를 바로 구현하지 않고, 질문과 계획을 거쳐 요구사항과 구현 단위를 정리합니다.
- 구현·리뷰 루프
  - /ce-work → /ce-simplify-code → /ce-code-review
  - 계획을 기준으로 구현하고, 코드를 단순화한 뒤, 리뷰로 품질 저하를 점검합니다.
- 지식 축적
  - /ce-compound
  - 해결한 문제와 배운 패턴을 문서화해 다음 작업의 맥락으로 재사용합니다.

---

# Part 5. Agent 생태계와 연결 기술

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

# Harness Engineering이 설계하는 것

- Context
  - LLM이 판단에 필요한 정보를 어떻게 받을 것인가
- Tool
  - 외부 시스템을 어떻게 조회하고 실행할 것인가
- State
  - 중간 결과와 작업 맥락을 어떻게 유지할 것인가
- Guardrail
  - 권한, 승인, 실패 처리를 어디에 둘 것인가
- Evaluation
  - 답변 품질을 어떻게 측정하고 회귀 테스트할 것인가
- Memory
  - 실패 사례와 운영 지식을 어떻게 축적할 것인가

---

# 모델 중심 사고 vs Harness 중심 사고

모델 중심 사고

- 좋은 모델을 고르면 품질이 좋아진다
- 프롬프트를 잘 쓰면 해결된다
- 답변이 틀리면 모델을 바꾼다

Harness 중심 사고

- 모델이 볼 데이터와 Tool을 설계한다
- 실패·검증·승인 흐름을 시스템에 넣는다
- 실패 사례를 평가셋, Runbook, Skill로 축적한다

---

# 왜 Harness Engineering이 중요해졌는가

- 모델 성능 상향 평준화
  - 좋은 모델을 쓰는 것만으로는 차별화가 어려워졌습니다.
- Agentic application 확산
  - 모델이 답변을 넘어 Tool을 사용하고 실제 작업을 수행합니다.
- 운영 신뢰성 요구 증가
  - 실패 처리, 권한, 승인, 추적, 평가가 제품 품질을 좌우합니다.
- 개선 루프의 중요성
  - 실패 사례를 평가셋, Runbook, Skill로 축적해야 품질이 유지됩니다.

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
