# DaOps 기반 AI 활용 세미나 구성안

## 0. 세미나 개요

### 세미나 제목 후보

- **AI 챗봇을 넘어 운영형 Agent로: DaOps 구축 사례 기반 실무 세미나**
- **운영 가능한 AI Agent 시스템 설계: LLM, LangGraph, Multi-Agent, Observability**
- **DaOps 사례로 보는 AI Agent 설계와 운영 아키텍처**

### 전체 목표

- AI, LLM, Agent의 관계를 정확히 이해한다.
- LangChain, LangGraph, RAG, MCP, A2A, HITL, Harness Engineering의 역할을 큰 그림에서 이해한다.
- DaOps 코드를 사례로 운영 업무를 Multi-Agent Workflow로 설계하는 방법을 이해한다.
- Celery, RabbitMQ, Loki, Grafana를 포함한 운영형 AI Agent 시스템 구조를 이해한다.
- AI Agent를 운영 환경에서 신뢰성 있게 사용하기 위한 평가, 관측, 개선 체계를 이해한다.

### 전체 포지셔닝

> 본 세미나는 단순히 LangChain/LangGraph 사용법을 설명하는 교육이 아니라, 실제 DB 운영 분석 서비스인 DaOps를 사례로 AI Agent를 어떻게 설계하고, 비동기 작업으로 실행하며, 로그와 대시보드로 관측하고, 평가와 개선 체계까지 연결할 수 있는지 설명하는 실무형 세미나이다.

### 전체 구성

| 일차 | 주제 | 핵심 질문 |
|---|---|---|
| 1일차 | AI/LLM/Agent 개념 지도 | AI와 LLM은 무엇이 다르고, Agent는 왜 별도 개념인가? |
| 2일차 | DaOps로 보는 LangGraph Agent 설계 | 운영 업무를 어떻게 Multi-Agent Workflow로 바꾸는가? |
| 3일차 | 운영 가능한 Agent 시스템 | Celery, RabbitMQ, Loki, Grafana는 왜 필요한가? |
| 4일차 | 신뢰성, 평가, 확장 방향 | Agent 결과를 어떻게 믿고, 개선하고, 운영 확장할 것인가? |

---

# 1일차. AI, LLM, Agent 개념 지도

## 1일차 주제

**AI는 LLM이 아니다. LLM은 Agent도 아니다.**

## 1일차 목표

- AI, ML, Deep Learning, Foundation Model, LLM의 포함 관계를 이해한다.
- LLM과 Agent의 차이를 이해한다.
- LangChain, LangGraph, RAG, MCP, A2A, HITL, Harness Engineering의 역할을 개념적으로 이해한다.
- DaOps가 단순 LLM 호출 서비스가 아니라 운영형 Agent 시스템이라는 점을 이해한다.

## 1일차 권장 시간 구성

| 시간 | 목차 | 구성 키워드 |
|---:|---|---|
| 0~5분 | 오프닝 | 세미나 목적, 기존 사내 교육과 차별점, DaOps 사례 중심 |
| 5~15분 | AI와 LLM의 관계 | AI, Machine Learning, Deep Learning, Foundation Model, LLM |
| 15~25분 | LLM의 역할과 한계 | 자연어 생성, 추론 보조, 환각, 최신성 한계, 도구 부재, 상태 부재 |
| 25~35분 | 챗봇/RPA/Agent 차이 | 질의응답, 정해진 자동화, 목표 기반 작업 수행, Tool 사용, State 관리 |
| 35~45분 | LangChain/LangGraph/RAG 개념 | LLM 앱 개발, Chain, Tool, Graph, State, Edge, Retrieval |
| 45~55분 | MCP/A2A/HITL/Harness Engineering 개념 | 외부 도구 표준 연결, Agent 간 통신, 사람 승인, 모델 주변 실행 환경 |
| 55~60분 | DaOps 위치 정리 | DaOps = LLM 호출 + Collector + Graph + Worker + Log + Evaluation |

## 1일차 상세 목차 및 구성 키워드

### 1. 왜 AI Agent를 이야기하는가

- 기존 운영 업무의 반복성
- DBA/SRE 초기 점검 절차
- 장애/성능 저하 분석의 복잡성
- Metric, Wait Event, SQL, OS 정보의 연결
- 단순 챗봇과 운영 자동화의 차이
- AI를 “답변 생성기”가 아니라 “업무 흐름 보조 시스템”으로 보는 관점

### 2. AI는 LLM인가?

- AI: 넓은 개념
- Machine Learning: 데이터 기반 학습
- Deep Learning: 신경망 기반 학습
- Foundation Model: 범용 대형 모델
- LLM: 언어 중심 Foundation Model
- Multimodal Model: 텍스트, 이미지, 음성 등
- LLM은 AI의 한 종류일 뿐
- “AI = LLM” 오해 정리

### 3. LLM은 무엇을 잘하고 못하는가

- 잘하는 것
  - 자연어 이해
  - 요약
  - 분류
  - 설명
  - 코드 생성 보조
  - 패턴 해석
  - 근거 기반 리포트 생성
- 못하는 것
  - 최신 데이터 자체 조회
  - DB 직접 조회
  - 시스템 상태 직접 관측
  - 장기 작업 상태 관리
  - 권한 통제
  - 결과 검증
  - 운영 장애 추적
- 핵심 메시지
  - LLM은 “두뇌” 역할을 할 수 있지만, 단독으로 운영 시스템이 될 수는 없음

### 4. 챗봇, RPA, AI Agent 차이

- 챗봇
  - 사용자 질문에 답변
  - 단일 턴/멀티 턴 대화
  - 일반 Q&A 중심
- RPA
  - 정해진 절차 자동화
  - 규칙 기반
  - 예외 대응 한계
- AI Agent
  - 목표 기반 실행
  - Tool 사용
  - State 관리
  - 실행 경로 선택
  - 결과 검증
  - 실패 처리
  - 반복 개선
- DaOps 예시
  - “DB가 느립니다” 질문
  - 일반 챗봇: 일반 원인 목록 답변
  - DaOps Agent: Global Health 분석 → 필요한 Sub Agent 호출 → 진단 요약 → 품질 검증

### 5. Agent의 구성요소

- Model
  - LLM
  - 사내 DS LLM API
  - gpt-oss-120b 등
- Tool
  - Oracle metric 조회
  - Collector
  - SQL 실행
  - Log 조회
  - 외부 API
- State
  - 요청 정보
  - target_nodes
  - visited_nodes
  - reason_data
  - diagnosis_summary
- Memory
  - 단기 대화 컨텍스트
  - 장기 사용자/업무 지식
  - 과거 장애 사례
- Planning/Routing
  - Classifier
  - Global Health 판단
  - Sub Agent 선택
- Evaluation
  - Quality Gate
  - LLM-as-a-judge
  - 정답셋 비교
- Observability
  - run_id
  - 로그
  - Dashboard
  - 오류 추적

### 6. LangChain의 역할

- LLM 호출 추상화
- PromptTemplate
- Output Parser
- Tool 정의
- Runnable
- Chain 구성
- with_structured_output
- Pydantic Schema 연동
- LangChain은 LLM 앱 개발 편의 도구
- LangChain 자체가 Agent 품질을 보장하지는 않음

### 7. LangGraph의 역할

- 상태 기반 Workflow
- Node
- Edge
- Conditional Edge
- StateGraph
- Fan-out
- Fan-in
- Join
- Checkpoint
- Long-running Agent
- Multi-Agent Orchestration
- DaOps에서 LangGraph가 필요한 이유
  - Classifier 이후 동적 분기
  - Global Health 이후 Sub Agent 선택
  - 여러 Agent 결과 병합
  - Summary/Quality Gate 단계 제어

### 8. RAG의 역할

- Retrieval-Augmented Generation
- 외부 문서 검색
- 사내 매뉴얼 검색
- 장애 대응 Runbook 검색
- 과거 장애 사례 검색
- RAG가 적합한 영역
  - 비정형 문서
  - 매뉴얼
  - 가이드
  - 과거 사례
- RAG가 덜 적합한 영역
  - 최신 Metric 직접 분석
  - 시계열 수치 계산
  - DB 상태 조회
- DaOps 관점
  - Metric 분석은 SQL/Collector 기반
  - Runbook/장애사례는 RAG 후보

### 9. MCP 개념

- Model Context Protocol
- AI 애플리케이션과 외부 도구/데이터 연결 표준
- Tool Server
- Resource
- Prompt
- File system
- Database
- API
- DaOps 확장 후보
  - Oracle Metric MCP Server
  - Loki Log Search MCP Server
  - Runbook MCP Server
  - Grafana Query MCP Server

### 10. A2A 개념

- Agent-to-Agent Protocol
- 서로 다른 Agent 간 통신
- Agent Discovery
- Task 위임
- Message 교환
- 독립 Agent 서비스 간 협업
- DaOps 확장 후보
  - DB Agent
  - OS Agent
  - Log Agent
  - Report Agent
  - Alarm Agent

### 11. HITL 개념

- Human-in-the-loop
- 사람 승인
- 사람 검토
- 사람 수정
- 조치 실행 전 승인
- 위험 작업 통제
- DaOps 확장 후보
  - SQL Kill 전 승인
  - 파라미터 변경 전 승인
  - 장애 보고서 발송 전 승인
  - 자동 조치 추천 후 승인

### 12. Harness Engineering 개념

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
- 핵심 메시지
  - 좋은 Agent는 좋은 모델만으로 만들어지지 않음
  - 모델을 둘러싼 구조가 품질과 안정성을 결정함

### 13. DaOps를 개념 지도에 배치

- DaOps의 LLM
  - 진단 요약
  - 근거 기반 분석
  - 최종 답변 생성
- DaOps의 Tool
  - Collector
  - Oracle SQL 조회
  - Chart 생성
  - Log 조회
- DaOps의 State
  - MainState
  - target_nodes
  - visited_nodes
  - reason_data
- DaOps의 Orchestration
  - LangGraph
- DaOps의 Operation
  - FastAPI
  - Celery
  - RabbitMQ
- DaOps의 Observability
  - Loki
  - Grafana
- DaOps의 Evaluation
  - Quality Gate
  - 정답셋
  - LLM-as-a-judge

## 1일차 발표자료 후보 슬라이드

1. 세미나 전체 로드맵
2. AI, ML, Deep Learning, LLM 관계
3. LLM은 무엇을 잘하고 못하는가
4. 챗봇/RPA/Agent 비교
5. Agent 구성요소
6. LangChain vs LangGraph
7. RAG가 필요한 경우와 아닌 경우
8. MCP/A2A/HITL/Harness Engineering 개념 지도
9. DaOps를 개념 지도 위에 배치
10. 1일차 요약

---

# 2일차. DaOps로 보는 LangGraph Agent 설계

## 2일차 주제

**운영 업무를 Multi-Agent Workflow로 바꾸는 방법**

## 2일차 목표

- DaOps의 문제 정의를 이해한다.
- LangGraph의 Node, Edge, State, Conditional Routing 구조를 이해한다.
- DaOps의 Classifier, Global Health, Sub Agent, Aggregator, Summary, Quality Gate 구조를 이해한다.
- 실제 구현 중 발생한 Fan-out/Fan-in, Join, 중복 실행 문제를 통해 실전 설계 포인트를 이해한다.

## 2일차 권장 시간 구성

| 시간 | 목차 | 구성 키워드 |
|---:|---|---|
| 0~5분 | 1일차 복습 | LLM vs Agent, State, Tool, Graph |
| 5~15분 | DaOps 문제 정의 | Oracle DB 운영 분석, 반복 점검, 초기 진단 자동화 |
| 15~25분 | DaOps 전체 Agent Workflow | Classifier, Global Health, Sub Agent, Summary, Quality Gate |
| 25~40분 | LangGraph 핵심 구조 | StateGraph, Node, Edge, Conditional Edge, Send, Join |
| 40~50분 | DaOps 코드 예시 | MainState, build_graph, route 함수, node 함수 |
| 50~60분 | 구현 이슈와 설계 판단 | 중복 실행, Fan-in, target_nodes, visited_nodes, END 처리 |

## 2일차 상세 목차 및 구성 키워드

### 1. DaOps 문제 정의

- Oracle DB 운영 분석
- 성능 저하 원인 분석
- 초기 점검 자동화
- 반복 업무 표준화
- DBA/SRE 분석 보조
- Metric 기반 진단
- 근거 중심 리포트
- Multi-Agent가 필요한 이유
  - 단일 Agent가 모든 도메인을 분석하기 어려움
  - CPU, Memory, Temp, Undo, Lock 등 도메인 분리 필요
  - Global Health로 전체 상태 판단 후 세부 Agent 호출

### 2. DaOps 전체 구조

- 사용자 질문 입력
- 요청 파라미터
  - db_info
  - module
  - svr_catg
  - start_time
  - end_time
  - metric_type
  - metric_option
- Classifier
- Global Health Agent
- Domain Sub Agents
  - Temp Agent
  - Undo Agent
  - Lock Agent
  - OS/CPU Agent
  - Memory Agent
  - PGA Agent
- Aggregator / Join
- Diagnosis Summary
- Quality Gate
- Final Response
- Report 저장
- Messenger 전송

### 3. LangGraph 핵심 개념 복습

- StateGraph
- TypedDict State
- Node
- Edge
- Conditional Edge
- Router
- Send
- START
- END
- State Reducer
- operator.add
- 병렬 실행
- 상태 병합
- Checkpoint
- Compile

### 4. DaOps MainState 설명 키워드

- user_question
- request_type
- target_nodes
- global_health_target_nodes
- visited_nodes
- reason_data
- diagnosis_summary
- diagnosis_summary_started
- reason_join_ready
- quality_gate_result
- final_answer
- 상태는 Agent 실행의 기억 장치
- 단순 함수 호출과 상태 기반 실행의 차이

### 5. Classifier Node

- 사용자 질문 분류
- request_type 판단
- 직접 Sub Agent 호출 여부 판단
- Global Health 우선 판단 여부
- target_nodes 결정
- Prompt 설계
- Pydantic Schema
- structured_output
- 사내 LLM 제약
- 분류 실패 시 fallback

### 6. Global Health Agent

- 전체 DB/Host 상태 선분석
- Host CPU Util %
- DB Wait Time Ratio
- DB CPU Time Ratio
- DB Time/sec
- AAS
- Top Event
- 세부 분석 Agent 선택
- global_health_target_nodes
- Global → Sub Agent 확장 구조

### 7. Domain Sub Agent

- Temp Agent
  - TEMP Tablespace
  - TEMP Workarea
  - TEMP Segment
  - spill
  - time_to_full
- Undo Agent
  - CR Undo
  - Rollback
  - Undo Segment
- Lock Agent
  - TX
  - TM
  - library cache
  - buffer busy
- OS/CPU Agent
  - Host CPU
  - DB CPU
  - Run Queue
  - Memory
  - Swap
- Memory/PGA Agent
  - PGA 사용량
  - Top PGA Process
- 각 Agent는 도메인 책임 분리

### 8. Aggregator / Join 구조

- Fan-out
- Fan-in
- 여러 Agent 결과 병합
- visited_nodes 누적
- reason_data 누적
- target_nodes와 visited_nodes 비교
- reason_join_ready
- diagnosis_summary_started
- 중복 실행 방지
- END 처리 주의
- Join Node가 필요한 이유

### 9. Diagnosis Summary Node

- 여러 Agent 분석 결과 통합
- 전체 진단 요약
- 주요 근거
- suspected_root_causes
- recommended_actions
- severity_score
- pattern_classification
- 근거 중심 답변
- 결론보다 근거 우선

### 10. Quality Gate

- 최종 답변 품질 검증
- 근거 포함 여부
- 질문 응답 여부
- 과도한 추측 여부
- 심각도 일관성
- 누락된 분석 여부
- hallucination 방지
- 평가 결과에 따른 재생성/수정 후보

### 11. 실제 구현 중 발생한 문제

- global_health_reason 중복 실행
- diagnosis_summary_node 중복 실행
- target_nodes와 global_health_target_nodes 분리 필요
- route_after_reason_join에서 END 반환 문제
- agent_join_node 실행 타이밍
- visited_nodes 중복
- State reducer 설계
- sorted(set()) dedup
- Graph 흐름 디버깅 어려움
- 로그 기반 추적 필요

### 12. 코드 설명 후보

- build_graph()
- MainState
- classify_node
- classifier_route
- global_health_reason_node
- direct_dispatch_route
- register_analysis_node
- register_analysis_node_edge
- agent_join_node
- route_after_reason_join
- diagnosis_summary_node
- quality_gate_node

## 2일차 발표자료 후보 슬라이드

1. 1일차 복습: LLM 호출과 Agent Workflow 차이
2. DaOps 문제 정의
3. DaOps 전체 Agent Workflow
4. LangGraph 기본 구성요소
5. MainState 구조
6. Classifier Node
7. Global Health Agent
8. Domain Sub Agents
9. Fan-out/Fan-in과 Join
10. Diagnosis Summary와 Quality Gate
11. 실제 구현 이슈
12. 2일차 요약

---

# 3일차. 운영 가능한 Agent 시스템

## 3일차 주제

**Agent는 잘 만들어도 운영되지 않으면 서비스가 아니다.**

## 3일차 목표

- Streamlit 데모와 운영 서비스의 차이를 이해한다.
- LLM Agent에서 비동기 처리가 필요한 이유를 이해한다.
- FastAPI, Celery, RabbitMQ 기반 실행 구조를 이해한다.
- run_id 기반 상태 추적과 중복 실행 대응을 이해한다.
- Loki, Grafana 기반 Observability 구조를 이해한다.

## 3일차 권장 시간 구성

| 시간 | 목차 | 구성 키워드 |
|---:|---|---|
| 0~5분 | 2일차 복습 | Graph, Node, Agent, Summary |
| 5~15분 | 데모와 운영 서비스의 차이 | Streamlit 한계, 장기 작업, 장애 추적 |
| 15~30분 | FastAPI + Celery + RabbitMQ 구조 | 비동기 작업, Queue, Worker, Task |
| 30~40분 | run_id와 상태 관리 | 요청 추적, DB 저장, 중복 실행, idempotency |
| 40~52분 | Loki + Grafana 관측 구조 | 로그 수집, Explore, Dashboard, run_id 검색 |
| 52~60분 | 운영 체크리스트 | timeout, retry, alert, dashboard, 장애 대응 |

## 3일차 상세 목차 및 구성 키워드

### 1. 왜 운영 아키텍처가 필요한가

- LLM 호출 지연
- Multi-Agent 장기 실행
- HTTP timeout
- 사용자 대기 문제
- 작업 상태 조회 필요
- 실패 추적 필요
- 재시도 필요
- 중복 실행 가능성
- 운영 로그 필요
- 서비스와 Worker 분리 필요

### 2. Streamlit 데모와 운영 서비스 차이

- Streamlit
  - 빠른 데모
  - 단일 사용자 실습
  - 운영 제어 한계
  - 비동기 처리 한계
- 운영 서비스
  - API 서버
  - Queue
  - Worker
  - DB 저장
  - 로그 수집
  - 모니터링
  - 알림
  - 권한
  - 배포/재시작 관리

### 3. DaOps 운영 흐름

- Frontend / Messenger
- FastAPI
- run_id 생성
- 요청 정보 저장
- 기본 차트 생성
- Celery task 발행
- RabbitMQ Queue 적재
- Celery Worker 실행
- LangGraph 실행
- 결과 저장
- 상태 COMPLETE/FAILED 갱신
- Messenger 전송
- Report 조회
- Loki 로그 수집
- Grafana 대시보드 확인

### 4. FastAPI 역할

- API endpoint
- 요청 검증
- Pydantic request model
- run_id 발급
- DB 저장
- Celery task dispatch
- 즉시 응답
- 상태 조회 API
- 결과 조회 API
- app.state 사용 한계
- Worker와 Web process 분리

### 5. Celery 역할

- Background task
- 긴 작업 처리
- Worker process
- Queue 소비
- task_acks_late
- worker_prefetch_multiplier
- task_ignore_result
- timeout
- retry
- worker_process_init
- Graph/LLM client 초기화
- 중복 실행 가능성
- idempotency 필요

### 6. RabbitMQ 역할

- Message Broker
- Queue
- Exchange
- Routing Key
- Durable Queue
- Message Ack
- Redelivery
- Connection loss
- 관리 UI
- 방화벽/포트 이슈
- Nginx proxy 후보

### 7. run_id 기반 추적

- 요청 단위 식별자
- 로그 상관관계
- DB 저장 키
- Grafana 검색 키
- Node 실행 추적
- 오류 추적
- 사용자 문의 대응
- 중복 실행 감지
- idempotent update
- ContextVar 기반 로그 주입 후보

### 8. 중복 실행과 실패 대응

- task_acks_late=True
- Worker 중단 시 redelivery
- connection loss
- RabbitMQ message 재전달
- retry 없는 경우에도 재실행 가능
- run_id unique 처리
- 상태 전이 제어
- RUNNING → COMPLETE
- RUNNING → FAILED
- 이미 COMPLETE인 작업 재실행 방지
- DB lock 또는 optimistic check
- partial result 저장

### 9. Loki 로그 수집

- Log file
- Alloy
- loki.source.file
- labels
  - service
  - component
  - run_id
  - node_name
- structured logging
- log level
- INFO/ERROR 분리
- JSON log 후보
- 검색 쿼리
  - `{service="da-dai"} |= "run_id=..."`
- 로그 보존 정책
- 민감 정보 마스킹

### 10. Grafana Dashboard

- Explore
- Dashboard variable
- run_id 변수
- service 변수
- component 변수
- node_name 변수
- Panel
- Table
- Logs
- Time series
- Error count
- Latency
- Queue backlog
- LLM timeout
- Node duration

### 11. Dashboard 후보

- Agent Job Overview
  - 요청 수
  - 성공 수
  - 실패 수
  - 평균 실행 시간
  - P95 실행 시간
- Node Execution
  - Node별 실행 횟수
  - Node별 평균 latency
  - Node별 실패율
- LLM Error
  - timeout
  - validation error
  - parsing error
  - schema error
- Celery/RabbitMQ
  - queue backlog
  - worker alive
  - task failure
  - redelivery
- Operation/Business View
  - DB별 요청 수
  - 이상 유형 분포
  - severity_score 분포
  - 자주 호출된 Agent

### 12. Supervisor/tmux 운영 관리 후보

- Supervisor
- supervisord.conf
- supervisorctl status
- restart
- stop/start
- log 확인
- tmux session
- 배포 후 재시작
- git pull 후 restart
- FastAPI와 Celery 분리 실행
- 장애 발생 시 확인 순서

### 13. 운영 전환 체크리스트

- API timeout 설정
- LLM timeout 설정
- Celery worker 설정
- RabbitMQ queue durable 여부
- run_id 로그 포함 여부
- ERROR 로그 분리
- Grafana dashboard 구성
- 알림 기준 정의
- 중복 실행 방지
- DB 상태 전이 제어
- 장애 대응 runbook
- 배포/rollback 절차
- 민감 정보 마스킹
- 권한 관리
- 성능 테스트

## 3일차 발표자료 후보 슬라이드

1. 2일차 복습: Agent Workflow
2. 데모와 운영 서비스의 차이
3. DaOps 운영 아키텍처 전체 그림
4. FastAPI 역할
5. Celery/RabbitMQ 역할
6. run_id 기반 추적
7. 중복 실행과 idempotency
8. Loki 로그 수집 구조
9. Grafana Dashboard 구성
10. 운영 체크리스트
11. 실제 운영 중 겪은 문제
12. 3일차 요약

---

# 4일차. 신뢰성, 평가, Harness Engineering, 확장 방향

## 4일차 주제

**AI Agent를 믿고 운영하려면 무엇이 필요한가**

## 4일차 목표

- LLM 답변을 그대로 신뢰하면 안 되는 이유를 이해한다.
- structured output, Pydantic schema, Quality Gate의 필요성을 이해한다.
- LLM-as-a-judge와 평가 데이터셋 기반 개선 루프를 이해한다.
- Harness Engineering 관점에서 DaOps 구조를 재해석한다.
- HITL, MCP, A2A 기반 향후 확장 방향을 이해한다.

## 4일차 권장 시간 구성

| 시간 | 목차 | 구성 키워드 |
|---:|---|---|
| 0~5분 | 3일차 복습 | 운영 아키텍처, 로그, Dashboard |
| 5~15분 | 왜 신뢰성 확보가 필요한가 | 환각, 근거 부족, 잘못된 조치, 운영 리스크 |
| 15~28분 | 구조화 출력과 Quality Gate | Pydantic, Schema, evidence-first, 검증 |
| 28~38분 | Evaluation 체계 | 정답셋, LLM-as-a-judge, 회귀 테스트, Prompt 개선 |
| 38~48분 | Harness Engineering | Model + Tool + State + Verification + Observability |
| 48~56분 | HITL/MCP/A2A 확장 | 사람 승인, Tool 표준화, Agent 분리 |
| 56~60분 | 전체 요약 및 Q&A | 4일 핵심 메시지 정리 |

## 4일차 상세 목차 및 구성 키워드

### 1. 왜 LLM 답변을 그대로 믿으면 안 되는가

- Hallucination
- 그럴듯한 오답
- 근거 누락
- 최신 데이터 부재
- 도메인 해석 오류
- Metric 단위 오류
- 시간 범위 오류
- DB/Host 혼동
- 심각도 과대/과소 판단
- 운영 조치 리스크
- DBA/SRE 검토 필요

### 2. 근거 중심 응답 설계

- 결론보다 근거 우선
- evidence-first prompting
- 수치 기반 판단
- 시계열 변화
- threshold
- abnormal pattern
- 원인 후보와 확정 원인 분리
- recommended_actions
- confidence
- uncertainty 표현
- “확인 필요” 표현
- 과도한 단정 방지

### 3. structured output

- Pydantic Schema
- with_structured_output
- json_schema
- function_calling
- json_mode
- Literal validation
- validation error
- timeout trade-off
- 사내 LLM 제약
- schema 단순화
- output parser fallback
- parsing retry
- schema 강제의 장단점

### 4. Quality Gate

- 최종 답변 검증
- 질문에 답했는가
- 핵심 근거가 포함됐는가
- Agent 결과를 누락하지 않았는가
- 심각도 판단이 일관적인가
- 과도한 추측이 있는가
- 운영 액션이 위험하지 않은가
- 재생성 여부 판단
- 품질 점수
- Reject/Revise/Accept

### 5. Evaluation 체계

- 테스트 데이터셋
- 고정 질문
- 고정 입력 데이터
- 기대 답변
- 핵심 근거 체크
- LLM-as-a-judge
- rule-based check
- regression test
- prompt version 관리
- model version 관리
- 평가 로그 저장
- 실패 사례 수집
- 개선 전후 비교
- 운영 품질 지표

### 6. LLM-as-a-judge

- 평가자 LLM
- 채점 기준
- rubric
- factuality
- completeness
- evidence coverage
- actionability
- safety
- consistency
- severity alignment
- false positive
- false negative
- judge bias
- judge prompt 관리
- 사람이 검토한 golden set 필요

### 7. Prompt 개선 루프

- 실패 사례 수집
- 원인 분류
  - 데이터 부족
  - Prompt 불명확
  - Schema 과도
  - LLM 추론 실패
  - Tool 결과 오류
  - Join 누락
- Prompt 수정
- 평가 재실행
- 결과 비교
- 배포 여부 결정
- metadata DB 저장
- 코드 배포 없이 튜닝 가능한 구조 후보

### 8. Harness Engineering으로 보는 DaOps

- Model
  - DS LLM API
  - gpt-oss-120b
- Prompt
  - system prompt
  - user prompt
  - evidence-first prompt
- Tool
  - Collector
  - SQL query
  - Chart generator
  - Log query
- State
  - MainState
  - target_nodes
  - visited_nodes
- Orchestration
  - LangGraph
  - conditional routing
  - fan-out/fan-in
- Verification
  - Pydantic
  - Quality Gate
- Observability
  - Loki
  - Grafana
  - run_id
- Permission
  - 향후 HITL
  - Action 승인
- Failure Handling
  - timeout
  - retry
  - fallback
  - partial result
- Evaluation
  - LLM-as-a-judge
  - regression test

### 9. HITL 확장 방향

- 운영 조치 전 승인
- SQL 실행 승인
- Kill session 승인
- 알림 발송 승인
- 보고서 발송 승인
- 장애 티켓 생성 승인
- 사람 수정 후 재실행
- Approval Node
- Interrupt
- Checkpoint
- Audit log
- 권한 관리

### 10. MCP 확장 방향

- 현재 구조
  - Python 함수/Collector가 Tool 역할
- MCP 적용 후보
  - Oracle Metric MCP Server
  - Loki Search MCP Server
  - Grafana Query MCP Server
  - Runbook MCP Server
  - File System MCP Server
  - Ticket System MCP Server
- 장점
  - Tool 표준화
  - 시스템 간 연결성
  - 재사용성
  - Agent와 Tool 분리
- 주의점
  - 권한
  - 보안
  - 네트워크
  - 감사 로그
  - Tool 호출 제한

### 11. A2A 확장 방향

- 현재 구조
  - 하나의 LangGraph 내부에서 Sub Agent 호출
- A2A 적용 후보
  - DB Agent 독립 서비스
  - OS Agent 독립 서비스
  - Log Agent 독립 서비스
  - Report Agent 독립 서비스
  - Alarm Agent 독립 서비스
- 장점
  - Agent 독립 배포
  - 팀 간 Agent 연동
  - 이기종 프레임워크 연동
  - 중앙 Orchestrator와 전문 Agent 분리
- 주의점
  - 통신 비용
  - 장애 전파
  - 인증/인가
  - 메시지 표준
  - 상태 일관성

### 12. 최종 발전 방향

- 현재 DaOps
  - 분석 중심 Agent
  - 근거 기반 진단
  - 리포트 생성
  - 로그 관측
- 다음 단계
  - HITL 기반 조치 추천
  - Runbook RAG
  - 장애 사례 검색
  - MCP Tool Server
  - Agent Evaluation Dashboard
  - Action Agent
  - 자동 티켓 생성
  - 알림 정책 연동
- 최종 목표
  - 반복 점검 자동화
  - 장애 초기 분석 표준화
  - 운영자 의사결정 보조
  - 신뢰 가능한 AI 운영 시스템

## 4일차 발표자료 후보 슬라이드

1. 3일차 복습: 운영 가능한 Agent 시스템
2. 왜 LLM 답변을 그대로 믿으면 안 되는가
3. 근거 중심 응답 설계
4. Structured Output과 Pydantic
5. Quality Gate
6. Evaluation과 LLM-as-a-judge
7. Prompt 개선 루프
8. Harness Engineering으로 보는 DaOps
9. HITL 확장 방향
10. MCP 확장 방향
11. A2A 확장 방향
12. 4일 전체 요약

---

# 전체 세미나에서 반복할 핵심 메시지

## 메시지 1. AI는 LLM보다 넓다

- LLM은 AI의 한 종류
- Agent는 LLM을 활용한 시스템 설계 방식
- 운영 서비스는 LLM API 호출만으로 완성되지 않음

## 메시지 2. Agent는 모델이 아니라 시스템이다

- Model
- Tool
- State
- Memory
- Routing
- Verification
- Observability
- Operation

## 메시지 3. DaOps는 단순 챗봇이 아니다

- 사용자 질문 분류
- 운영 데이터 수집
- 도메인별 Agent 분석
- 결과 병합
- 근거 기반 진단
- 품질 검증
- 로그/대시보드 관측
- 비동기 Worker 실행

## 메시지 4. RAG는 만능이 아니다

- 문서/매뉴얼/과거 사례에는 적합
- 최신 Metric 분석에는 구조화 데이터 조회가 더 중요
- RAG보다 중요한 것은 어떤 데이터를 언제 어떤 Agent에게 줄 것인가

## 메시지 5. 운영형 AI Agent에는 Observability가 필수다

- run_id
- node_name
- execution time
- error log
- queue backlog
- worker status
- dashboard
- alert

## 메시지 6. 신뢰성은 평가와 개선 루프로 만든다

- 정답셋
- LLM-as-a-judge
- Prompt version
- Model version
- Quality Gate
- 회귀 테스트
- 실패 사례 수집

---

# 발표 준비 우선순위

## 1순위: 전체 그림

- AI/LLM/Agent 개념 지도
- DaOps 전체 아키텍처 그림
- LangGraph Workflow 그림
- 운영 아키텍처 그림
- Observability 구조 그림
- Evaluation Loop 그림

## 2순위: DaOps 코드 예시

- MainState
- build_graph
- classifier_route
- global_health_route
- agent_join_node
- diagnosis_summary_node
- celery task
- logging formatter
- Grafana query 예시

## 3순위: 실제 사례

- DB 성능 저하 질문 예시
- Global Health 결과 예시
- Sub Agent 호출 예시
- 최종 진단 리포트 예시
- LLM timeout 사례
- structured_output validation error 사례
- Celery 중복 실행 가능성 사례
- Grafana run_id 검색 사례

## 4순위: 체크리스트

- Agent 설계 체크리스트
- 운영 전환 체크리스트
- 평가 체계 체크리스트
- HITL 적용 체크리스트
- MCP/A2A 확장 검토 체크리스트

---

# 세미나 준비 시 주의점

## 1. 1일차는 코드보다 그림 중심

- 대상자가 AI를 LLM으로만 이해하고 있다면 코드부터 들어가면 안 됨
- 개념 지도와 업무 예시 중심
- 용어를 설명한 뒤 바로 DaOps에 연결

## 2. 2일차부터 코드 비중 증가

- LangGraph 문법 설명은 최소화
- 왜 이 구조가 필요한지 중심
- 실제 구현 이슈를 반드시 포함

## 3. 3일차는 가장 차별화되는 파트

- 기존 사내 교육은 보통 운영 아키텍처가 약함
- Celery/RabbitMQ/Loki/Grafana는 강하게 가져갈 것
- “데모와 서비스의 차이”를 강조

## 4. 4일차는 과장하지 않기

- MCP/A2A는 현재 적용이 아니라 확장 방향으로 설명
- HITL도 향후 Action Agent의 안전장치로 설명
- Harness Engineering은 DaOps 구조를 해석하는 프레임으로 사용

## 5. 실패 경험을 숨기지 않기

- timeout
- schema validation error
- node 중복 실행
- join 설계 문제
- celery redelivery
- log 검색 불편함
- 이런 사례가 오히려 실무 세미나의 신뢰도를 높임

---

# 최종 세미나 소개 문구

> 본 세미나는 AI와 LLM의 기본 개념부터 시작해, LangGraph 기반 Multi-Agent 구조가 실제 운영 업무에 어떻게 적용되는지 DaOps 사례로 설명합니다. 또한 단순 챗봇 구현을 넘어 Celery, RabbitMQ, Loki, Grafana를 활용한 비동기 실행, 관측, 평가, 운영 전환 구조까지 함께 다룹니다.

---

# 이후 준비할 산출물 후보

- 4일차별 발표자료 초안
- DaOps 전체 아키텍처 다이어그램
- LangGraph Workflow 다이어그램
- 운영 아키텍처 다이어그램
- Grafana Dashboard 예시 화면
- Agent Evaluation 체크리스트
- MCP/A2A 확장 설계안
- 발표 대본
- Q&A 예상 질문 목록
