# 2일차 세미나 구성안: DaOps로 보는 LangGraph Agent 설계

## 1. 세미나 주제

**운영 업무를 Multi-Agent Workflow로 바꾸는 방법**

## 2. 2일차 목표

- DaOps가 해결하려는 운영 문제를 이해한다.
- LangGraph의 Node, Edge, State, Conditional Routing 구조를 이해한다.
- DaOps의 Classifier, Global Health, Domain Sub Agent, Aggregator, Diagnosis Summary, Quality Gate 흐름을 이해한다.
- Fan-out/Fan-in, Join, 중복 실행 방지 같은 실전 설계 포인트를 이해한다.
- 운영 분석 업무를 Agent workflow로 분해하는 관점을 익힌다.

## 3. 대상자

- 1일차에서 AI Agent의 개념을 이해한 구성원
- DaOps의 내부 Agent 흐름을 알고 싶은 DBA/SRE/개발자
- LangGraph를 단순 문법이 아니라 실무 workflow 설계 관점에서 이해하고 싶은 구성원
- 운영 업무를 여러 전문 Agent로 나누는 기준을 알고 싶은 구성원

## 4. 사전 지식

- 1일차의 LLM, Agent, Tool, State, Routing 개념
- Python 함수와 데이터 구조에 대한 기초 이해
- DB 운영 분석에서 Metric, Wait Event, SQL, OS 지표가 함께 사용된다는 감각
- Graph, Node, Edge 같은 기본적인 흐름 제어 개념

## 5. 2일차 핵심 메시지

- 운영 분석 업무는 단일 LLM 호출이 아니라 여러 판단 단계와 도구 호출로 구성된다.
- LangGraph는 Agent 실행 흐름을 상태 기반 workflow로 설계하기 위한 도구다.
- DaOps는 Classifier, Global Health, Domain Sub Agent, Aggregator, Summary, Quality Gate로 분석 흐름을 나눈다.
- Multi-Agent 구조에서는 병렬 실행보다 병합, 중복 실행 방지, 상태 관리가 더 어렵다.
- 좋은 Agent 설계는 "어떤 Agent를 만들까"보다 "언제 어떤 Agent를 호출하고 결과를 어떻게 합칠까"에서 결정된다.

## 6. 권장 시간 구성

| 시간 | 목차 | 핵심 질문 |
|---:|---|---|
| 0~5분 | 1일차 복습 | LLM 호출과 Agent workflow는 무엇이 다른가? |
| 5~13분 | DaOps 문제 정의 | 어떤 운영 문제를 Agent workflow로 바꾸려는가? |
| 13~23분 | DaOps 전체 Agent Workflow | 요청은 어떤 단계로 분석되는가? |
| 23~35분 | LangGraph 핵심 구조 | State, Node, Edge, Routing은 어떤 역할을 하는가? |
| 35~45분 | DaOps MainState와 주요 Node | DaOps는 상태를 어떻게 들고 흐름을 제어하는가? |
| 45~55분 | Fan-out/Fan-in과 Join | 여러 Agent 결과를 어떻게 병합하고 중복을 막는가? |
| 55~60분 | 구현 이슈와 설계 판단 | 실제 구현에서 무엇이 어려웠고 어떻게 해결했는가? |

---

# 7. 상세 목차 및 구성 키워드

## 7.1 1일차 복습: LLM 호출과 Agent Workflow 차이

### 구성 의도

- 1일차의 개념을 2일차 구현 구조로 자연스럽게 연결한다.
- "Agent는 모델이 아니라 시스템"이라는 메시지를 다시 상기시킨다.

### 주요 키워드

- LLM
- Tool
- State
- Routing
- Verification
- Observability
- Agent는 실행 흐름을 가진 시스템
- DaOps는 단순 답변 생성기가 아님
- "DB가 느립니다" 반복 예시

### 강조 메시지

- 2일차부터는 Agent를 개념이 아니라 workflow로 본다.
- 핵심 질문은 "LLM에게 무엇을 물어볼까"가 아니라 "분석 흐름을 어떻게 나눌까"다.

## 7.2 DaOps 문제 정의

### 구성 의도

- DaOps가 왜 Multi-Agent 구조를 필요로 하는지 설명한다.
- 운영 분석 업무의 복잡성을 먼저 보여준 뒤 LangGraph 구조로 넘어간다.

### 주요 키워드

- Oracle DB 운영 분석
- 성능 저하 원인 분석
- 초기 점검 자동화
- 반복 업무 표준화
- DBA/SRE 분석 보조
- Metric 기반 진단
- Wait Event 분석
- SQL 성능 분석
- OS/Host 상태 분석
- 근거 중심 리포트

### Multi-Agent가 필요한 이유

- 단일 Agent가 모든 도메인을 깊게 분석하기 어려움
- CPU, Memory, Temp, Undo, Lock, PGA 등 도메인이 다름
- 전체 상태를 먼저 보고 세부 분석 대상을 정해야 함
- 요청 유형에 따라 필요한 분석 경로가 달라짐
- 여러 분석 결과를 하나의 진단으로 병합해야 함

### 강조 메시지

- 운영 분석은 "한 번 물어보고 답하기"가 아니라 "상태를 보고 다음 분석을 선택하는 과정"이다.

## 7.3 DaOps 전체 Agent Workflow

### 구성 의도

- 전체 구조를 먼저 보여줘서 이후 각 Node 설명이 길을 잃지 않게 한다.
- 요청부터 최종 답변까지의 큰 흐름을 하나의 그림으로 설명할 수 있게 한다.

### 주요 단계

- 사용자 질문 입력
- 요청 파라미터 수집
- Classifier
- Global Health Agent
- Domain Sub Agents
- Aggregator/Join
- Diagnosis Summary
- Quality Gate
- Final Response
- Report 저장
- Messenger 전송

### 요청 파라미터 키워드

- db_info
- module
- svr_catg
- start_time
- end_time
- metric_type
- metric_option
- user_question

### Domain Sub Agent 후보

- Temp Agent
- Undo Agent
- Lock Agent
- OS/CPU Agent
- Memory Agent
- PGA Agent

### 강조 메시지

- DaOps의 Agent workflow는 "분류 → 전체 상태 확인 → 세부 분석 → 결과 병합 → 요약 → 검증" 흐름이다.

## 7.4 LangGraph 핵심 구조

### 구성 의도

- LangGraph 문법을 깊게 파기보다 DaOps workflow를 이해하는 데 필요한 개념만 정리한다.
- 각 개념을 DaOps의 실제 역할과 연결한다.

### 주요 키워드

- StateGraph
- State
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
- Compile
- Checkpoint

### 설명 포인트

- Node는 하나의 처리 단계
- Edge는 다음 단계로 가는 연결
- Conditional Edge는 상태를 보고 다음 경로를 선택
- State는 workflow 전체가 공유하는 실행 문맥
- Reducer는 병렬 실행 결과를 합치는 방식
- Send는 동적으로 여러 Node를 호출할 때 사용

### 강조 메시지

- LangGraph의 핵심은 "상태를 들고 조건에 따라 실행 흐름을 바꾸는 것"이다.

## 7.5 DaOps MainState

### 구성 의도

- Agent workflow에서 State가 왜 중요한지 구체적으로 보여준다.
- 단순 함수 호출과 상태 기반 실행의 차이를 설명한다.

### 주요 필드 키워드

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

### 필드별 설명 방향

- user_question: 사용자의 원본 요청
- request_type: 분석 요청 유형
- target_nodes: 실행해야 할 분석 Agent 목록
- global_health_target_nodes: Global Health가 추가로 선택한 Agent 목록
- visited_nodes: 이미 실행된 Agent 목록
- reason_data: 각 Agent가 만든 근거 데이터
- diagnosis_summary: 최종 진단 요약
- quality_gate_result: 품질 검증 결과

### 강조 메시지

- State는 Agent 실행의 기억 장치다.
- Multi-Agent workflow에서는 어떤 Agent를 실행했고, 어떤 결과가 쌓였고, 다음에 무엇을 해야 하는지를 State로 관리해야 한다.

## 7.6 Classifier Node

### 구성 의도

- 사용자 질문을 바로 답변하지 않고 먼저 분류하는 이유를 설명한다.
- 어떤 분석 경로를 탈지 결정하는 첫 번째 관문으로 Classifier를 소개한다.

### 주요 키워드

- 사용자 질문 분류
- request_type 판단
- 직접 Sub Agent 호출 여부 판단
- Global Health 우선 판단 여부
- target_nodes 결정
- Prompt 설계
- Pydantic Schema
- structured_output
- 사내 LLM 제약
- fallback

### 설명 포인트

- 질문이 구체적인지, 전체 점검이 필요한지 판단
- "Temp가 부족한가요?"처럼 특정 영역 질문이면 직접 Agent 호출 가능
- "DB가 느립니다"처럼 넓은 질문이면 Global Health를 먼저 수행
- LLM 분류 결과를 Schema로 받아 후속 흐름에 사용

### 강조 메시지

- Classifier는 Agent workflow의 입구이자 라우터다.

## 7.7 Global Health Agent

### 구성 의도

- 전체 상태 선분석이 왜 필요한지 설명한다.
- Global Health가 Domain Sub Agent 선택의 근거가 된다는 점을 강조한다.

### 주요 키워드

- 전체 DB/Host 상태 선분석
- Host CPU Util %
- DB Wait Time Ratio
- DB CPU Time Ratio
- DB Time/sec
- AAS
- Top Event
- 세부 분석 Agent 선택
- global_health_target_nodes
- Global to Sub Agent 확장

### 설명 포인트

- 전체 상태를 먼저 본 뒤 병목 후보를 좁힘
- CPU 문제가 의심되면 OS/CPU Agent 호출
- Lock이 두드러지면 Lock Agent 호출
- Temp 사용량이 비정상적이면 Temp Agent 호출
- Global Health 결과가 Sub Agent fan-out의 출발점이 됨

### 강조 메시지

- Global Health는 전체 진단의 탐색 단계다.
- 모든 Agent를 무조건 호출하는 대신 필요한 Agent를 선택하게 만든다.

## 7.8 Domain Sub Agent

### 구성 의도

- 도메인별 전문 Agent가 어떤 책임을 가지는지 설명한다.
- Multi-Agent 구조가 단순히 Agent 수를 늘리는 것이 아니라 책임 분리라는 점을 보여준다.

### Temp Agent 키워드

- TEMP Tablespace
- TEMP Workarea
- TEMP Segment
- spill
- time_to_full
- temp usage trend

### Undo Agent 키워드

- CR Undo
- Rollback
- Undo Segment
- undo retention
- long transaction

### Lock Agent 키워드

- TX
- TM
- library cache
- buffer busy
- blocking session
- wait chain

### OS/CPU Agent 키워드

- Host CPU
- DB CPU
- Run Queue
- Load Average
- Memory
- Swap

### Memory/PGA Agent 키워드

- PGA 사용량
- Top PGA Process
- memory pressure
- process별 사용량

### 강조 메시지

- 각 Agent는 도메인 책임을 가진다.
- 좋은 Multi-Agent 설계는 Agent 이름을 많이 만드는 것이 아니라 책임 경계를 명확히 나누는 것이다.

## 7.9 Aggregator / Join 구조

### 구성 의도

- Multi-Agent workflow에서 가장 실전적인 난점인 병합과 중복 실행 문제를 설명한다.
- Fan-out보다 Fan-in이 어렵다는 포인트를 전달한다.

### 주요 키워드

- Fan-out
- Fan-in
- Join
- 여러 Agent 결과 병합
- visited_nodes 누적
- reason_data 누적
- target_nodes와 visited_nodes 비교
- reason_join_ready
- diagnosis_summary_started
- 중복 실행 방지
- END 처리 주의

### 설명 포인트

- 여러 Sub Agent가 병렬로 실행될 수 있음
- 각 Agent 결과는 reason_data에 누적
- visited_nodes로 완료 여부를 확인
- target_nodes와 visited_nodes를 비교해 Summary 실행 시점 판단
- Join Node가 없으면 Summary가 너무 빨리 실행되거나 여러 번 실행될 수 있음

### 강조 메시지

- Multi-Agent에서 중요한 것은 여러 Agent를 부르는 것이 아니라 "모두 끝났는지 알고 한 번만 합치는 것"이다.

## 7.10 Diagnosis Summary Node

### 구성 의도

- 여러 Agent 결과를 사용자에게 읽히는 진단으로 바꾸는 단계를 설명한다.
- 근거 중심 답변의 중요성을 연결한다.

### 주요 키워드

- 여러 Agent 분석 결과 통합
- 전체 진단 요약
- 주요 근거
- suspected_root_causes
- recommended_actions
- severity_score
- pattern_classification
- evidence-first response
- 결론보다 근거 우선

### 설명 포인트

- 각 Agent의 개별 분석을 하나의 진단 문맥으로 통합
- 원인 후보와 확정 원인을 구분
- 수치/시계열/Wait Event/SQL 근거를 포함
- 운영자가 다음에 무엇을 확인해야 하는지 제안

### 강조 메시지

- Summary는 예쁜 문장 생성 단계가 아니라 근거를 구조화해 운영 판단을 돕는 단계다.

## 7.11 Quality Gate

### 구성 의도

- 최종 답변을 바로 사용자에게 내보내지 않고 검증하는 이유를 설명한다.
- 4일차 신뢰성/평가 파트로 이어지는 연결고리를 만든다.

### 주요 키워드

- 최종 답변 품질 검증
- 근거 포함 여부
- 질문 응답 여부
- 과도한 추측 여부
- 심각도 일관성
- 누락된 분석 여부
- hallucination 방지
- 운영 액션 안전성
- 재생성/수정 후보

### 설명 포인트

- 답변이 질문에 실제로 답했는지 확인
- Agent 결과를 누락하지 않았는지 확인
- 근거 없이 단정하지 않았는지 확인
- 위험한 조치를 무책임하게 제안하지 않았는지 확인

### 강조 메시지

- 운영형 Agent는 답변 생성보다 답변 검증이 더 중요할 수 있다.

## 7.12 실제 구현 이슈와 설계 판단

### 구성 의도

- 실무 세미나의 신뢰도를 높이기 위해 실제로 겪은 문제를 숨기지 않는다.
- 구현 난점을 설계 판단으로 연결한다.

### 주요 이슈 키워드

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

### 설명 포인트

- 병렬 실행 결과가 여러 경로로 합쳐지며 중복 실행 가능
- Summary는 모든 분석이 끝난 뒤 한 번만 실행되어야 함
- END 처리와 Join 처리의 경계가 중요
- 로그와 run_id가 없으면 Graph 흐름 추적이 어려움

### 강조 메시지

- LangGraph를 쓰면 workflow를 만들 수 있지만, 올바른 상태 설계와 디버깅 전략이 없으면 운영하기 어렵다.

## 7.13 코드 설명 후보

### 구성 의도

- 발표에서 실제 코드 예시로 보여줄 후보를 정리한다.
- 코드는 문법 설명보다 설계 의도를 보여주는 용도로 사용한다.

### 코드 후보

- MainState
- build_graph()
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

### 코드 설명 방향

- MainState: 무엇을 기억해야 하는가
- build_graph: 전체 workflow가 어떻게 연결되는가
- route 함수: 어떤 조건에서 어디로 가는가
- analysis node: 도메인 Agent는 어떤 책임을 가지는가
- join node: 언제 병합하는가
- quality gate: 최종 답변을 어떻게 검증하는가

---

# 8. 발표자료 후보 슬라이드

1. 2일차 제목: 운영 업무를 Multi-Agent Workflow로 바꾸는 방법
2. 1일차 복습: LLM 호출과 Agent Workflow 차이
3. DaOps 문제 정의
4. DaOps 전체 Agent Workflow
5. LangGraph 기본 구성요소
6. DaOps MainState 구조
7. Classifier Node
8. Global Health Agent
9. Domain Sub Agents
10. Fan-out/Fan-in과 Join
11. Diagnosis Summary와 Quality Gate
12. 실제 구현 이슈
13. 코드 예시 후보
14. 2일차 요약 및 3일차 예고

# 9. 2일차에서 사용할 반복 예시

## 예시 질문

> DB가 느립니다. 원인을 분석해 주세요.

## Workflow 관점 처리

- Classifier가 넓은 성능 저하 질문으로 분류
- Global Health가 전체 DB/Host 상태를 선분석
- CPU, Lock, Temp 등 이상 징후에 따라 Sub Agent 선택
- 선택된 Domain Sub Agent들이 각자 분석 수행
- Aggregator/Join이 결과를 병합
- Diagnosis Summary가 근거 중심 진단 생성
- Quality Gate가 답변 품질 검증

## 2일차 활용 방식

- 전체 Workflow 그림에서 사용
- Classifier 설명에서 request_type 예시로 사용
- Global Health 설명에서 target_nodes 선택 예시로 사용
- Join 설명에서 여러 Sub Agent 결과 병합 예시로 사용

# 10. 2일차 마무리 요약

- DaOps의 핵심은 운영 분석 업무를 Agent workflow로 분해하는 것이다.
- LangGraph는 State, Node, Edge, Conditional Routing으로 이 흐름을 구성한다.
- Classifier는 요청을 분류하고, Global Health는 전체 상태를 보고, Domain Sub Agent는 도메인별 분석을 수행한다.
- Aggregator/Join은 여러 Agent 결과를 병합하고 중복 실행을 막는 핵심 설계 지점이다.
- Quality Gate는 최종 답변을 운영자가 신뢰할 수 있는 형태로 검증하는 단계다.

# 11. 3일차 예고

- 3일차에서는 잘 설계한 Agent workflow를 실제 서비스로 운영하려면 무엇이 필요한지 다룬다.
- FastAPI, Celery, RabbitMQ를 통한 비동기 실행 구조를 설명한다.
- run_id, Loki, Grafana를 활용한 운영 추적과 Observability를 살펴본다.
