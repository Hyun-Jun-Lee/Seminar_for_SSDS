# 1일차 세미나 구성안: AI, LLM, Agent 개념 지도

## 1. 세미나 주제

**AI는 LLM이 아니다. LLM은 Agent도 아니다.**

## 2. 1일차 목표

- AI, Machine Learning, Deep Learning, Foundation Model, LLM의 포함 관계를 이해한다.
- LLM이 잘하는 일과 못하는 일을 구분한다.
- 챗봇, RPA, AI Agent의 차이를 운영 업무 관점에서 이해한다.
- LangChain, LangGraph, RAG, MCP, A2A, HITL, Harness Engineering의 역할을 큰 그림으로 이해한다.
- DaOps가 단순 LLM 호출 서비스가 아니라 운영형 AI Agent 시스템이라는 점을 이해한다.

## 3. 대상자

- AI Agent 개념을 실무 시스템 관점에서 이해하고 싶은 DBA/SRE/운영 담당자
- LLM API 호출과 Agent 시스템의 차이를 알고 싶은 개발자
- DaOps가 어떤 구조로 AI를 활용하는지 큰 그림을 이해하려는 구성원
- LangChain, LangGraph, RAG, MCP, A2A 같은 용어를 한 번에 정리하고 싶은 구성원

## 4. 사전 지식

- 기본적인 IT 운영 업무 흐름에 대한 이해
- API, 로그, 데이터 조회, 배치/비동기 작업에 대한 기초 감각
- AI/LLM에 대한 깊은 지식은 없어도 무방
- Python 코드 경험이 있으면 2일차 이후 이해에 도움이 됨

## 5. 1일차 핵심 메시지

- AI는 LLM보다 넓은 개념이다.
- LLM은 강력한 언어 모델이지만, 혼자서는 운영 시스템이 될 수 없다.
- Agent는 모델 하나가 아니라 Model, Tool, State, Routing, Verification, Observability를 포함한 시스템 설계 방식이다.
- DaOps는 챗봇이 아니라 운영 데이터를 수집하고, 분석 흐름을 제어하고, 근거 기반 진단을 생성하는 Agent 시스템이다.
- 좋은 Agent는 좋은 모델만으로 만들어지지 않고, 모델을 둘러싼 실행 환경과 검증 체계로 완성된다.

## 6. 권장 시간 구성

| 시간 | 목차 | 핵심 질문 |
|---:|---|---|
| 0~5분 | 오프닝 | 왜 지금 AI Agent를 이야기하는가? |
| 5~15분 | AI와 LLM의 관계 | AI와 LLM은 같은 말인가? |
| 15~25분 | LLM의 역할과 한계 | LLM은 무엇을 잘하고, 무엇을 못하는가? |
| 25~35분 | 챗봇/RPA/Agent 차이 | Agent는 챗봇이나 RPA와 무엇이 다른가? |
| 35~43분 | Agent 구성요소 | Agent 시스템은 어떤 부품으로 구성되는가? |
| 43~50분 | LangChain/LangGraph/RAG | 각 기술은 Agent 시스템에서 어떤 역할을 하는가? |
| 50~56분 | MCP/A2A/HITL/Harness Engineering | 확장성과 운영 안정성을 위해 무엇이 필요한가? |
| 56~60분 | DaOps 위치 정리 | DaOps는 이 개념 지도에서 어디에 있는가? |

---

# 7. 상세 목차 및 구성 키워드

## 7.1 오프닝: 왜 AI Agent를 이야기하는가

### 구성 의도

- 세미나가 단순 AI 유행어 소개가 아니라 실제 운영 업무 개선을 위한 이야기임을 명확히 한다.
- DaOps 사례를 통해 추상적인 AI Agent 개념을 실무 맥락에 붙인다.
- 4일 전체 세미나의 방향을 간단히 예고한다.

### 주요 키워드

- 운영 업무 반복성
- DBA/SRE 초기 점검 절차
- 장애/성능 저하 분석
- Metric, Wait Event, SQL, OS 정보 연결
- 단순 답변 생성과 업무 흐름 보조의 차이
- AI를 "답변 생성기"가 아니라 "운영 분석 보조 시스템"으로 보는 관점
- DaOps 사례 중심 세미나
- 기존 사내 AI 교육과 차별점

### 설명 흐름 후보

- "DB가 느립니다"라는 질문으로 시작
- 일반 챗봇은 가능한 원인 목록을 답변
- 운영형 Agent는 실제 Metric/Wait Event/SQL/Host 상태를 확인
- 필요한 세부 Agent를 호출하고 결과를 병합
- 최종적으로 근거 기반 진단과 추천 액션을 생성

## 7.2 AI와 LLM의 관계

### 구성 의도

- "AI = LLM"이라는 흔한 오해를 먼저 정리한다.
- LLM이 전체 AI 기술 지형에서 어디에 위치하는지 개념 지도를 제공한다.
- 이후 Agent 설명으로 넘어가기 위한 기반을 만든다.

### 주요 키워드

- AI: 사람의 지능적 작업을 기계가 수행하도록 만드는 넓은 분야
- Machine Learning: 데이터 기반 학습
- Deep Learning: 신경망 기반 학습
- Foundation Model: 범용 대형 모델
- LLM: 언어 중심 Foundation Model
- Multimodal Model: 텍스트, 이미지, 음성, 영상 처리
- Generative AI
- Predictive AI
- AI > ML > Deep Learning > Foundation Model > LLM
- LLM은 AI의 한 종류

### 강조 메시지

- AI는 LLM보다 넓다.
- LLM은 최근 AI 활용을 폭발시킨 핵심 기술이지만, AI 전체와 동일하지 않다.
- 운영 시스템에서 중요한 것은 LLM 자체보다 LLM을 어떤 데이터, 도구, 프로세스와 연결하느냐다.

## 7.3 LLM의 역할과 한계

### 구성 의도

- LLM을 과소평가하지도, 과대평가하지도 않는 균형 잡힌 관점을 제공한다.
- "LLM은 두뇌 역할을 할 수 있지만 혼자서는 운영 시스템이 될 수 없다"는 메시지를 만든다.

### LLM이 잘하는 것

- 자연어 이해
- 요약
- 분류
- 설명
- 코드 생성 보조
- 패턴 해석
- 텍스트 변환
- 보고서 초안 작성
- 근거 기반 리포트 생성 보조
- 복잡한 정보를 사람이 읽기 쉬운 형태로 재구성

### LLM이 못하거나 약한 것

- 최신 시스템 상태 직접 조회
- DB 직접 조회
- OS Metric 직접 관측
- 장기 작업 상태 관리
- 권한 통제
- 실행 이력 추적
- 결과 검증
- 정량 계산의 일관성 보장
- 운영 장애 원인 확정
- 위험 작업 승인
- 외부 도구 사용 없이 실제 조치 수행

### 운영 업무 관점의 한계

- 데이터가 없으면 추측할 수밖에 없음
- 근거가 없으면 그럴듯한 오답이 될 수 있음
- Tool이 없으면 실제 시스템을 볼 수 없음
- State가 없으면 장기 흐름을 관리할 수 없음
- Observability가 없으면 장애 추적이 어려움
- Evaluation이 없으면 개선 여부를 판단하기 어려움

### 강조 메시지

- LLM은 운영자의 판단을 돕는 강력한 언어/추론 엔진이다.
- 하지만 운영 서비스가 되려면 Tool, State, 권한, 로그, 검증, 평가가 함께 필요하다.

## 7.4 챗봇, RPA, AI Agent 차이

### 구성 의도

- Agent를 단순 챗봇이나 자동화 스크립트와 구분한다.
- 청중이 익숙한 챗봇/RPA와 비교하여 Agent의 필요성을 이해하게 한다.

### 챗봇 키워드

- 사용자 질문에 답변
- 대화 인터페이스
- 단일 턴/멀티 턴 Q&A
- 일반 지식 응답
- 문서 기반 질의응답
- 실행보다 응답 중심
- 상태와 도구 사용이 제한적일 수 있음

### RPA 키워드

- 정해진 절차 자동화
- 규칙 기반 실행
- 화면/업무 프로세스 자동화
- 반복 작업 처리
- 예외 대응 한계
- 동적 판단 한계
- 사전에 정의된 플로우 중심

### AI Agent 키워드

- 목표 기반 작업 수행
- Tool 사용
- State 관리
- 실행 경로 선택
- 조건부 분기
- 외부 데이터 조회
- 중간 결과 축적
- 결과 검증
- 실패 처리
- 반복 개선
- Multi-Agent 협업

### DaOps 예시

- 사용자 질문: "DB가 느립니다"
- 일반 챗봇: CPU, Lock, I/O, SQL 문제일 수 있다고 일반론 답변
- RPA: 정해진 SQL 몇 개를 순서대로 실행
- DaOps Agent: 질문 분류 → Global Health 분석 → 필요한 Sub Agent 선택 → Metric/Wait Event/SQL 근거 수집 → 진단 요약 → 품질 검증

### 강조 메시지

- 챗봇은 말한다.
- RPA는 정해진 절차를 실행한다.
- Agent는 목표를 기준으로 도구를 사용하고 상태를 관리하며 실행 흐름을 결정한다.

## 7.5 Agent의 구성요소

### 구성 의도

- Agent를 모델 하나가 아니라 여러 구성요소가 결합된 시스템으로 설명한다.
- 이후 DaOps 구조를 이해하기 위한 기본 프레임을 제공한다.

### Model

- LLM
- 사내 DS LLM API
- gpt-oss-120b 등
- 자연어 이해
- 분석 보조
- 요약/리포트 생성
- 판단/분류 보조

### Tool

- Oracle metric 조회
- Collector
- SQL 실행
- Chart 생성
- Log 조회
- 외부 API
- Runbook 검색
- Grafana/Loki Query

### State

- 요청 정보
- 사용자 질문
- 분석 대상 DB/Host
- start_time/end_time
- target_nodes
- visited_nodes
- reason_data
- diagnosis_summary
- quality_gate_result
- final_answer

### Memory

- 단기 대화 컨텍스트
- 사용자별 선호/업무 맥락
- 장기 업무 지식
- 과거 장애 사례
- 반복되는 패턴
- 단, 운영 시스템에서는 Memory와 감사/보안 이슈를 함께 고려해야 함

### Planning/Routing

- Classifier
- request_type 판단
- Global Health 판단
- Sub Agent 선택
- 조건부 분기
- Fan-out/Fan-in
- 실패 시 fallback

### Verification

- Pydantic Schema
- structured output
- Quality Gate
- 근거 포함 여부 확인
- 질문 응답 여부 확인
- 과도한 추측 방지
- 운영 액션 안전성 확인

### Observability

- run_id
- node_name
- execution time
- structured logging
- Loki
- Grafana
- error trace
- Dashboard
- alert

### Evaluation

- 정답셋
- 고정 질문
- LLM-as-a-judge
- rule-based check
- regression test
- prompt version
- model version
- 실패 사례 수집
- 개선 전후 비교

### 강조 메시지

- Agent는 Model + Tool + State + Routing + Verification + Observability + Evaluation의 조합이다.
- 모델 성능만 좋아서는 운영 품질이 보장되지 않는다.

## 7.6 LangChain의 역할

### 구성 의도

- LangChain을 "Agent를 자동으로 잘 만들어주는 마법"이 아니라 LLM 애플리케이션 개발 편의 도구로 설명한다.
- LangGraph와 역할을 구분하기 위한 전 단계로 사용한다.

### 주요 키워드

- LLM 호출 추상화
- Chat model wrapper
- PromptTemplate
- Output Parser
- Tool 정의
- Runnable
- Chain 구성
- with_structured_output
- Pydantic Schema 연동
- provider 교체 편의성
- prompt/model/tool 연결

### DaOps 연결 키워드

- 사내 LLM API 호출 래핑
- Prompt 구성
- structured output 사용
- Classifier 결과를 Schema로 받기
- Agent 결과를 정해진 형식으로 받기

### 강조 메시지

- LangChain은 LLM 앱 개발을 편하게 해준다.
- 하지만 LangChain 자체가 Agent 품질, 운영 안정성, 평가 체계를 보장하지는 않는다.

## 7.7 LangGraph의 역할

### 구성 의도

- LangGraph를 상태 기반 Agent workflow를 설계하는 도구로 설명한다.
- DaOps의 Multi-Agent 구조와 직접 연결한다.

### 주요 키워드

- 상태 기반 Workflow
- StateGraph
- Node
- Edge
- Conditional Edge
- START/END
- Router
- Send
- Fan-out
- Fan-in
- Join
- State Reducer
- Checkpoint
- Long-running Agent
- Multi-Agent Orchestration

### DaOps에서 필요한 이유

- Classifier 이후 동적 분기
- Global Health 이후 Sub Agent 선택
- Temp/Undo/Lock/OS/Memory/PGA Agent 병렬 호출
- 여러 Agent 결과 병합
- Diagnosis Summary 단계 제어
- Quality Gate 단계 제어
- 중복 실행 방지
- visited_nodes/target_nodes 기반 흐름 추적

### 강조 메시지

- 단순 Chain은 순차 처리에 가깝다.
- 운영 분석은 조건부 분기, 병렬 실행, 상태 병합, 실패 처리가 필요하다.
- DaOps에서 LangGraph는 Agent 실행 흐름을 관리하는 Orchestrator 역할을 한다.

## 7.8 RAG의 역할

### 구성 의도

- RAG를 만능 해결책으로 설명하지 않고, 적합한 영역과 부적합한 영역을 구분한다.
- DaOps에서 Metric 분석과 문서 검색의 차이를 분명히 한다.

### 주요 키워드

- Retrieval-Augmented Generation
- 외부 문서 검색
- Vector DB
- Embedding
- 유사도 검색
- 사내 매뉴얼 검색
- 장애 대응 Runbook 검색
- 과거 장애 사례 검색
- 지식 기반 답변

### RAG가 적합한 영역

- 비정형 문서
- 매뉴얼
- 운영 가이드
- 장애 대응 Runbook
- 과거 장애 사례
- 정책 문서
- FAQ

### RAG가 덜 적합한 영역

- 최신 Metric 직접 분석
- 시계열 수치 계산
- DB 현재 상태 조회
- SQL 결과 기반 판단
- threshold 기반 이상 탐지
- 운영 시스템 실시간 상태 확인

### DaOps 관점

- Metric 분석은 SQL/Collector 기반이 적합
- Runbook/장애사례/운영 매뉴얼은 RAG 후보
- RAG는 Agent의 Tool 중 하나일 수 있음
- 모든 문제를 RAG로 풀려고 하면 구조가 흐려짐

### 강조 메시지

- RAG는 문서 지식을 연결하는 좋은 방법이다.
- 하지만 운영 Metric 분석에서는 구조화 데이터 조회와 도메인 로직이 더 중요할 수 있다.

## 7.9 MCP 개념

### 구성 의도

- MCP를 깊게 구현 설명하기보다, "LLM 애플리케이션과 외부 도구/데이터를 연결하는 표준화된 방식"으로 소개한다.
- 1일차에서는 확장 후보 수준으로만 설명하고, 4일차에서 더 자세히 다룰 수 있도록 여지를 둔다.

### 주요 키워드

- Model Context Protocol
- Tool Server
- Resource
- Prompt
- 외부 도구 연결 표준
- 파일 시스템 연결
- Database 연결
- API 연결
- Tool 호출 인터페이스 표준화
- Agent와 Tool의 분리

### DaOps 확장 후보

- Oracle Metric MCP Server
- Loki Log Search MCP Server
- Grafana Query MCP Server
- Runbook MCP Server
- File System MCP Server
- Ticket System MCP Server

### 주의 키워드

- 권한
- 인증/인가
- 감사 로그
- 민감 정보 마스킹
- 네트워크 접근 통제
- Tool 호출 제한

### 강조 메시지

- 현재 DaOps에서는 Python 함수/Collector가 Tool 역할을 한다.
- 향후 MCP를 적용하면 Tool을 더 표준화하고 재사용 가능한 형태로 분리할 수 있다.

## 7.10 A2A 개념

### 구성 의도

- A2A를 현재 필수 기술이 아니라 향후 Agent 간 협업/분리 배포를 위한 확장 방향으로 소개한다.
- LangGraph 내부 Sub Agent와 독립 서비스 Agent의 차이를 구분한다.

### 주요 키워드

- Agent-to-Agent Protocol
- Agent Discovery
- Task 위임
- Message 교환
- 독립 Agent 서비스 간 협업
- 중앙 Orchestrator
- 전문 Agent
- 이기종 프레임워크 연동
- Agent 간 계약

### DaOps 확장 후보

- DB Agent 독립 서비스
- OS Agent 독립 서비스
- Log Agent 독립 서비스
- Report Agent 독립 서비스
- Alarm Agent 독립 서비스
- Ticket Agent

### 주의 키워드

- 통신 비용
- 장애 전파
- 인증/인가
- 메시지 표준
- 상태 일관성
- 관측성
- 배포 복잡도

### 강조 메시지

- 현재 DaOps는 하나의 LangGraph 안에서 Sub Agent를 호출하는 구조에 가깝다.
- A2A는 Agent를 독립 서비스로 분리하고 협업시키려는 미래 확장 방향으로 볼 수 있다.

## 7.11 HITL 개념

### 구성 의도

- 운영형 Agent에서 사람 승인과 검토가 왜 중요한지 설명한다.
- 특히 위험한 운영 조치에는 자동 실행보다 승인 흐름이 필요하다는 메시지를 준다.

### 주요 키워드

- Human-in-the-loop
- 사람 승인
- 사람 검토
- 사람 수정
- 조치 실행 전 승인
- 위험 작업 통제
- Approval Node
- Interrupt
- Checkpoint
- Audit log
- 권한 관리

### DaOps 확장 후보

- SQL Kill 전 승인
- 파라미터 변경 전 승인
- 장애 보고서 발송 전 승인
- 자동 조치 추천 후 승인
- 티켓 생성 전 검토
- 알림 발송 전 검토

### 강조 메시지

- 운영 업무에서 모든 것을 자동화하는 것이 목표는 아니다.
- 위험한 조치는 사람이 승인하고, Agent는 근거와 추천안을 제공하는 구조가 현실적이다.

## 7.12 Harness Engineering 개념

### 구성 의도

- 좋은 Agent는 모델 하나가 아니라 모델 주변 실행 환경으로 만들어진다는 프레임을 제공한다.
- 1일차의 여러 개념을 하나의 시스템 설계 관점으로 묶는다.

### 주요 키워드

- 모델 주변 실행 환경 설계
- Prompt
- Tool
- State
- Memory
- Permission
- Verification
- Logging
- Evaluation
- Failure Handling
- Observability
- Deployment
- Runtime
- Feedback loop

### DaOps 연결 키워드

- Prompt: evidence-first prompt, classifier prompt
- Tool: Collector, Oracle SQL, Chart, Log query
- State: MainState, target_nodes, visited_nodes, reason_data
- Verification: Pydantic, Quality Gate
- Logging: run_id, node_name, Loki
- Evaluation: 정답셋, LLM-as-a-judge
- Failure Handling: timeout, fallback, retry, partial result

### 강조 메시지

- 좋은 Agent는 좋은 모델만으로 만들어지지 않는다.
- 모델을 둘러싼 구조가 품질과 안정성을 결정한다.

## 7.13 DaOps를 개념 지도에 배치

### 구성 의도

- 앞에서 설명한 모든 개념을 DaOps라는 실제 시스템에 다시 연결한다.
- 2일차 LangGraph 설계, 3일차 운영 아키텍처, 4일차 신뢰성/평가 파트로 자연스럽게 이어지게 한다.

### DaOps의 LLM

- 진단 요약
- 근거 기반 분석
- 최종 답변 생성
- 질문 분류 보조
- 분석 결과 설명

### DaOps의 Tool

- Collector
- Oracle SQL 조회
- Metric 조회
- Chart 생성
- Log 조회
- 향후 Runbook 검색

### DaOps의 State

- MainState
- user_question
- request_type
- target_nodes
- global_health_target_nodes
- visited_nodes
- reason_data
- diagnosis_summary
- quality_gate_result

### DaOps의 Orchestration

- LangGraph
- Classifier
- Global Health
- Domain Sub Agents
- Aggregator/Join
- Diagnosis Summary
- Quality Gate

### DaOps의 Operation

- FastAPI
- Celery
- RabbitMQ
- Worker
- run_id
- 상태 저장
- 결과 조회
- Messenger 전송

### DaOps의 Observability

- Loki
- Grafana
- structured logging
- run_id 검색
- node별 실행 추적
- error log
- latency
- queue backlog

### DaOps의 Evaluation

- Quality Gate
- 정답셋
- LLM-as-a-judge
- rule-based check
- prompt 개선 루프
- 실패 사례 수집

### 강조 메시지

- DaOps는 LLM 호출 하나로 구성된 서비스가 아니다.
- DaOps는 운영 데이터 수집, Agent workflow, 비동기 실행, 로그 관측, 품질 검증을 결합한 운영형 AI Agent 시스템이다.

---

# 8. 발표자료 후보 슬라이드

1. 1일차 제목: AI는 LLM이 아니다. LLM은 Agent도 아니다.
2. 세미나 전체 로드맵
3. 왜 AI Agent를 이야기하는가
4. "DB가 느립니다" 질문으로 보는 챗봇과 Agent 차이
5. AI, ML, Deep Learning, Foundation Model, LLM 관계
6. LLM이 잘하는 것과 못하는 것
7. 챗봇/RPA/AI Agent 비교
8. Agent 구성요소: Model, Tool, State, Routing, Verification, Observability
9. LangChain의 역할
10. LangGraph의 역할
11. RAG가 필요한 경우와 아닌 경우
12. MCP/A2A/HITL 개념 지도
13. Harness Engineering으로 보는 Agent 시스템
14. DaOps를 개념 지도 위에 배치
15. 1일차 요약 및 2일차 예고

# 9. 1일차에서 사용할 반복 예시

## 예시 질문

> DB가 느립니다. 원인을 분석해 주세요.

## 챗봇식 응답 관점

- 일반적인 원인 후보 나열
- CPU, I/O, Lock, SQL 성능 문제 가능성 설명
- 실제 시스템 상태는 확인하지 못함
- 근거 수치가 부족함

## Agent식 처리 관점

- 사용자 질문 분류
- 분석 대상 DB/시간 범위 확인
- Global Health 분석
- CPU/Memory/Lock/Temp/Undo 등 필요한 Sub Agent 선택
- Metric, Wait Event, SQL, OS 정보 수집
- 여러 Agent 결과 병합
- 근거 중심 진단 요약
- Quality Gate로 최종 답변 검증

## 1일차 활용 방식

- 오프닝에서 문제 제시
- 챗봇/RPA/Agent 비교에서 재사용
- Agent 구성요소 설명에서 Model/Tool/State/Routing으로 분해
- DaOps 위치 정리에서 전체 구조로 다시 조립

# 10. 1일차 마무리 요약

- AI는 LLM보다 넓고, LLM은 AI의 한 종류다.
- LLM은 자연어 이해와 생성에 강하지만, 운영 시스템을 직접 관측하거나 관리하지는 못한다.
- Agent는 LLM에 Tool, State, Routing, Verification, Observability를 결합한 시스템 설계 방식이다.
- LangChain은 LLM 앱 개발 편의 도구이고, LangGraph는 상태 기반 Agent workflow를 구성하는 데 유용하다.
- RAG는 문서 지식 연결에 적합하지만, 실시간 Metric 분석에는 구조화 데이터 조회가 더 중요할 수 있다.
- MCP, A2A, HITL은 DaOps의 향후 확장 방향으로 이해하면 된다.
- DaOps는 단순 챗봇이 아니라 운영 업무를 Multi-Agent Workflow로 설계한 사례다.

# 11. 2일차 예고

- 2일차에서는 DaOps의 LangGraph 구조를 중심으로 실제 Agent workflow를 살펴본다.
- Classifier, Global Health, Domain Sub Agent, Aggregator, Diagnosis Summary, Quality Gate의 흐름을 설명한다.
- Fan-out/Fan-in, Join, 중복 실행 방지, visited_nodes/target_nodes 같은 실전 설계 포인트를 다룬다.
