# 2일차: DaOps로 보는 LangGraph Agent 설계

## 오늘의 주제

**운영 업무를 Multi-Agent Workflow로 바꾸는 방법**

1일차에서는 AI Agent를 모델 하나가 아니라 `Model`, `Tool`, `State`, `Routing`, `Verification`, `Observability`가 결합된 시스템으로 보았습니다.

2일차에서는 이 개념을 DaOps 사례에 붙여서 봅니다. 핵심 질문은 다음과 같습니다.

> "DB가 느립니다"라는 질문을 받았을 때, Agent는 어떤 순서로 판단하고 어떤 전문 Agent를 호출하며, 여러 분석 결과를 어떻게 하나의 진단으로 합칠 것인가?

---

# 1. 1일차 복습: LLM 호출과 Agent Workflow의 차이

LLM 호출은 보통 입력과 출력이 단순합니다.

```text
User Question
  -> LLM
  -> Answer
```

하지만 운영 분석은 이렇게 끝나지 않습니다.

```text
User Question
  -> 요청 분류
  -> 전체 상태 확인
  -> 필요한 도메인 분석 선택
  -> Metric / SQL / OS 지표 조회
  -> 여러 분석 결과 병합
  -> 근거 중심 요약
  -> 품질 검증
  -> 최종 답변
```

Agent Workflow는 LLM에게 한 번 질문하는 구조가 아니라, **상태를 들고 여러 단계를 거치며 다음 행동을 선택하는 실행 흐름**입니다.

2일차부터는 Agent를 "답변 생성기"가 아니라 **운영 분석 workflow**로 봅니다.

---

# 2. DaOps가 다루는 운영 문제

DaOps가 다루려는 대표 질문은 단순합니다.

> DB가 느립니다. 원인을 분석해 주세요.

하지만 실제 분석은 단순하지 않습니다.

- DB 자체가 느린 것인가?
- Host CPU가 높은 것인가?
- Lock 대기가 발생했는가?
- 특정 SQL이 병목인가?
- TEMP, UNDO, PGA, Memory 이슈가 있는가?
- 배포나 특정 시간대 이벤트와 연결되는가?
- 지금 당장 조치해야 하는가, 추가 확인이 필요한가?

이 질문에 답하려면 한 Agent가 모든 것을 한 번에 판단하기보다, **전체 상태를 먼저 보고 필요한 전문 분석으로 나누는 구조**가 더 적합합니다.

---

# 3. 왜 Multi-Agent 구조가 필요한가

운영 분석에는 여러 도메인이 섞입니다.

| 도메인 | 확인할 것 |
|---|---|
| CPU / OS | Host CPU, Load Average, Run Queue |
| DB Wait | Top Wait Event, DB Time, AAS |
| SQL | Top SQL, 실행 계획, elapsed time |
| Lock | Blocking session, wait chain |
| TEMP | TEMP usage, spill, time_to_full |
| UNDO | long transaction, undo retention |
| Memory / PGA | memory pressure, process별 PGA |

모든 Agent가 모든 도메인을 깊게 분석하면 흐름이 복잡해지고 비용도 커집니다.

그래서 DaOps는 다음 관점을 사용합니다.

1. 먼저 요청을 분류한다.
2. 전체 상태를 빠르게 확인한다.
3. 필요한 도메인 Agent만 실행한다.
4. 결과를 병합한다.
5. 근거 중심으로 요약한다.
6. 최종 답변을 검증한다.

핵심은 **Agent를 많이 만드는 것**이 아니라 **언제 어떤 Agent를 호출하고 결과를 어떻게 합칠지 설계하는 것**입니다.

---

# 4. DaOps 전체 Agent Workflow

DaOps의 큰 흐름은 다음과 같습니다.

```text
START
  |
  v
사용자 질문 / 요청 파라미터 수집
  |
  v
Classifier
  |
  +--> 특정 도메인 질문이면 Domain Sub Agent 직접 호출
  |
  +--> 넓은 성능 저하 질문이면 Global Health 먼저 실행
                                  |
                                  v
                          필요한 Domain Sub Agent 선택
                                  |
                                  v
                         Fan-out: 여러 Agent 병렬 실행
                                  |
                                  v
                         Join: 결과 병합 / 완료 여부 확인
                                  |
                                  v
                         Diagnosis Summary
                                  |
                                  v
                         Quality Gate
                                  |
                                  v
                         Final Response
```

이 흐름을 한 문장으로 정리하면:

> DaOps는 "분류 -> 전체 상태 확인 -> 세부 분석 -> 결과 병합 -> 진단 요약 -> 품질 검증"으로 운영 분석을 진행합니다.

---

# 5. LangGraph가 필요한 이유

이 workflow는 단순 함수 호출 체인으로도 어느 정도 만들 수 있습니다.

하지만 다음 요구사항이 생기면 그래프 구조가 유리해집니다.

- 상태를 여러 단계에서 공유해야 한다.
- 조건에 따라 다음 실행 경로가 달라진다.
- 여러 Agent를 병렬로 실행해야 한다.
- 병렬 결과를 안전하게 합쳐야 한다.
- 특정 Node가 중복 실행되지 않게 막아야 한다.
- 실행 흐름을 로그로 추적해야 한다.

LangGraph는 이런 흐름을 `State`, `Node`, `Edge`, `Conditional Edge`로 명시하게 해줍니다.

---

# 6. LangGraph 핵심 구조

LangGraph의 기본 구성요소는 네 가지로 볼 수 있습니다.

| 구성요소 | 역할 | DaOps 예시 |
|---|---|---|
| State | workflow 전체가 공유하는 실행 문맥 | user_question, target_nodes, reason_data |
| Node | 하나의 처리 단계 | Classifier, Global Health, Temp Agent |
| Edge | 다음 단계로 이동하는 연결 | Classifier -> Global Health |
| Conditional Edge | 상태를 보고 다음 경로 선택 | request_type에 따라 Agent 선택 |

가장 단순한 형태는 다음과 같습니다.

```python
from langgraph.graph import StateGraph, START, END

graph = StateGraph(dict)
graph.add_node("classifier", classifier_node)
graph.add_node("global_health", global_health_node)

graph.add_edge(START, "classifier")
graph.add_conditional_edges("classifier", route_after_classifier)
graph.add_edge("global_health", END)

app = graph.compile()
```

중요한 것은 문법이 아니라 관점입니다.

> LangGraph는 "상태를 들고 조건에 따라 실행 흐름을 바꾸는 구조"를 코드로 표현합니다.

---

# 7. DaOps MainState

Agent workflow에서 State는 실행의 기억 장치입니다.

DaOps는 다음과 같은 정보를 State에 담습니다.

```python
class MainState(TypedDict):
    user_question: str
    request_type: str
    target_nodes: list[str]
    global_health_target_nodes: list[str]
    visited_nodes: list[str]
    reason_data: list[dict]
    diagnosis_summary: dict | None
    diagnosis_summary_started: bool
    reason_join_ready: bool
    quality_gate_result: dict | None
    final_answer: str | None
```

각 필드는 단순한 데이터가 아니라 workflow 제어에 사용됩니다.

| 필드 | 의미 |
|---|---|
| `user_question` | 사용자의 원본 질문 |
| `request_type` | 질문 분류 결과 |
| `target_nodes` | 실행해야 할 도메인 Agent 목록 |
| `global_health_target_nodes` | Global Health가 추가로 선택한 Agent 목록 |
| `visited_nodes` | 이미 실행된 Agent 목록 |
| `reason_data` | 각 Agent가 만든 근거 데이터 |
| `diagnosis_summary_started` | Summary 중복 실행 방지 |
| `quality_gate_result` | 최종 답변 검증 결과 |

Multi-Agent workflow에서는 "무엇을 실행했는지", "무엇이 끝났는지", "무엇을 합쳐야 하는지"를 State로 관리해야 합니다.

---

# 8. Classifier Node

Classifier는 사용자 질문을 바로 답하지 않습니다.

먼저 질문의 성격을 분류합니다.

```text
"TEMP가 부족한가요?"
  -> 특정 도메인 질문
  -> Temp Agent 직접 호출 가능

"DB가 느립니다"
  -> 넓은 성능 저하 질문
  -> Global Health 먼저 실행

"최근 10분 CPU 병목인지 확인해 주세요"
  -> CPU/OS 중심 질문
  -> OS/CPU Agent 우선 호출
```

Classifier가 결정해야 하는 것은 보통 다음과 같습니다.

- request_type은 무엇인가?
- 전체 상태 점검이 필요한가?
- 바로 호출할 Domain Agent가 있는가?
- target_nodes는 무엇인가?
- 분류 결과를 신뢰할 수 있는가?

Classifier는 Agent workflow의 입구이자 라우터입니다.

---

# 9. Global Health Agent

Global Health Agent는 전체 상태를 먼저 봅니다.

목적은 정답을 바로 내는 것이 아니라, **어떤 도메인 분석이 필요한지 좁히는 것**입니다.

확인 항목:

- Host CPU Util %
- DB Wait Time Ratio
- DB CPU Time Ratio
- DB Time/sec
- AAS
- Top Event
- 주요 Metric 변화

예시:

```text
Global Health 결과
  - Host CPU Util: 92%
  - DB CPU Time Ratio: 높음
  - Top Wait Event: CPU + db file sequential read

선택할 Sub Agent
  - OS/CPU Agent
  - SQL Agent
```

Global Health는 모든 Agent를 무조건 호출하지 않게 해줍니다.

---

# 10. Domain Sub Agent

Domain Sub Agent는 각자 책임이 명확해야 합니다.

| Agent | 책임 |
|---|---|
| Temp Agent | TEMP 사용량, spill, time_to_full 분석 |
| Undo Agent | long transaction, undo retention, rollback 이슈 분석 |
| Lock Agent | blocking session, wait chain, TX/TM lock 분석 |
| OS/CPU Agent | Host CPU, DB CPU, run queue, load average 분석 |
| Memory/PGA Agent | PGA 사용량, memory pressure, process별 사용량 분석 |

좋은 Domain Agent는 다음을 명확히 합니다.

- 어떤 입력을 받는가?
- 어떤 Tool과 지표를 조회하는가?
- 어떤 근거 데이터를 출력하는가?
- 어떤 경우에 "확정 원인"이 아니라 "후보"로 표현해야 하는가?

Multi-Agent 설계의 핵심은 책임 경계입니다.

---

# 11. Fan-out / Fan-in

Global Health가 여러 Agent를 선택하면 workflow는 fan-out 됩니다.

```text
Global Health
  ├─ OS/CPU Agent
  ├─ Lock Agent
  └─ SQL Agent
```

각 Agent는 독립적으로 분석 결과를 만듭니다.

```text
OS/CPU Agent -> CPU 사용률과 run queue 근거
Lock Agent   -> blocking session 근거
SQL Agent    -> Top SQL과 실행 시간 근거
```

문제는 fan-out이 아니라 fan-in입니다.

```text
OS/CPU Agent \
Lock Agent    -> Join -> Diagnosis Summary
SQL Agent    /
```

여러 결과가 모두 도착했는지 확인하고, Summary를 한 번만 실행해야 합니다.

---

# 12. Join에서 생기는 문제

Multi-Agent workflow에서 자주 생기는 문제는 다음과 같습니다.

1. Summary가 너무 빨리 실행된다.
2. Summary가 여러 번 실행된다.
3. 일부 Agent 결과가 누락된다.
4. 이미 실행한 Agent가 다시 실행된다.
5. END 처리와 Join 처리가 충돌한다.

DaOps는 이런 문제를 막기 위해 State에 완료 정보를 남깁니다.

```python
visited_nodes = ["os_cpu", "lock"]
target_nodes = ["os_cpu", "lock", "sql"]

reason_join_ready = set(visited_nodes) >= set(target_nodes)
```

그리고 Summary 중복 실행을 막기 위해 별도 flag를 둡니다.

```python
if state["diagnosis_summary_started"]:
    return END
```

핵심은 간단합니다.

> 여러 Agent를 부르는 것보다, 모두 끝났는지 알고 한 번만 합치는 것이 더 어렵습니다.

---

# 13. Diagnosis Summary Node

Diagnosis Summary는 여러 Agent의 결과를 사람이 읽을 수 있는 진단으로 바꿉니다.

하지만 단순히 문장을 예쁘게 만드는 단계가 아닙니다.

Summary가 해야 할 일:

- 각 Agent의 핵심 근거를 통합한다.
- 원인 후보와 확정 원인을 구분한다.
- 수치, 시계열, Wait Event, SQL 근거를 포함한다.
- 추가 확인이 필요한 항목을 분리한다.
- 운영자가 다음에 취할 조치를 제안한다.

좋은 Summary 예시는 다음 구조를 가집니다.

```text
1. 요약 결론
2. 핵심 근거
3. 원인 후보
4. 추가 확인 필요 항목
5. 권장 조치
6. 신뢰도 / 불확실성
```

운영형 Agent 답변은 "그럴듯한 설명"보다 **근거와 판단 기준**이 중요합니다.

---

# 14. Quality Gate

Quality Gate는 최종 답변을 사용자에게 내보내기 전 검증하는 단계입니다.

검증 기준:

| 기준 | 질문 |
|---|---|
| Relevance | 사용자 질문에 직접 답했는가? |
| Evidence Coverage | 핵심 근거가 포함됐는가? |
| Consistency | Agent 결과와 최종 답변이 모순되지 않는가? |
| Safety | 위험한 조치를 단정적으로 권하지 않는가? |
| Completeness | 중요한 분석 결과를 누락하지 않았는가? |
| Uncertainty | 데이터 부족 시 확인 필요를 표현했는가? |

Quality Gate 결과는 보통 세 가지 중 하나입니다.

```text
ACCEPT  -> 최종 답변 사용
REVISE  -> 답변 수정 또는 재생성
REJECT  -> 근거 부족 / 위험 답변으로 차단
```

운영형 Agent에서는 답변 생성보다 답변 검증이 더 중요할 수 있습니다.

---

# 15. 실제 구현에서 어려웠던 지점

LangGraph를 쓰면 workflow를 명시할 수 있지만, 자동으로 좋은 설계가 되는 것은 아닙니다.

DaOps 구현에서 특히 어려운 부분은 다음이었습니다.

- Global Health 이후 Sub Agent가 중복 실행될 수 있음
- Diagnosis Summary가 여러 경로에서 중복 실행될 수 있음
- `target_nodes`와 `global_health_target_nodes`를 분리해야 함
- Join Node가 너무 빨리 실행되면 일부 결과가 누락됨
- `visited_nodes`가 중복 누적될 수 있음
- 병렬 실행 결과를 합치려면 reducer 설계가 필요함
- Graph 흐름은 로그가 없으면 디버깅하기 어려움

설계 판단:

```text
target_nodes
  -> 처음부터 실행하기로 한 Agent 목록

global_health_target_nodes
  -> Global Health가 추가로 선택한 Agent 목록

visited_nodes
  -> 실제 완료된 Agent 목록

reason_data
  -> 각 Agent가 만든 근거 데이터
```

상태 필드를 분리하면 흐름은 조금 복잡해지지만, 중복 실행과 누락을 줄일 수 있습니다.

---

# 16. 코드로 볼 때 집중할 부분

실제 코드를 볼 때 모든 줄을 문법적으로 해석할 필요는 없습니다.

다음 질문에 집중하면 됩니다.

| 코드 영역 | 볼 질문 |
|---|---|
| `MainState` | 무엇을 기억해야 하는가? |
| `build_graph()` | 전체 workflow가 어떻게 연결되는가? |
| `classifier_route()` | 어떤 조건에서 어디로 가는가? |
| `global_health_reason_node()` | 전체 상태가 어떻게 Sub Agent 선택으로 이어지는가? |
| `agent_join_node()` | 언제 병합하는가? |
| `route_after_reason_join()` | Summary 실행 조건은 무엇인가? |
| `quality_gate_node()` | 최종 답변을 어떻게 검증하는가? |

코드는 LangGraph 문법을 외우기 위한 자료가 아니라 **workflow 설계 의도를 확인하는 자료**입니다.

---

# 17. 반복 예시: "DB가 느립니다"

오늘 강의 전체에서 같은 예시를 반복해서 봅니다.

> DB가 느립니다. 원인을 분석해 주세요.

Workflow 관점에서는 이렇게 처리됩니다.

```text
1. Classifier
   - 넓은 성능 저하 질문으로 분류

2. Global Health
   - 전체 DB/Host 상태 선분석
   - CPU, Lock, Temp 등 이상 후보 확인

3. Domain Sub Agent
   - 필요한 Agent만 선택해 분석

4. Join
   - 각 Agent의 reason_data 병합
   - 모든 target node 완료 여부 확인

5. Diagnosis Summary
   - 근거 중심 진단 생성

6. Quality Gate
   - 답변 누락, 과도한 추측, 위험 조치 여부 검증
```

---

# 18. 2일차 요약

2일차의 핵심은 다음 다섯 가지입니다.

1. 운영 분석은 단일 LLM 호출이 아니라 여러 판단 단계와 Tool 호출로 구성됩니다.
2. LangGraph는 State, Node, Edge, Conditional Routing으로 Agent workflow를 설계하게 해줍니다.
3. DaOps는 Classifier, Global Health, Domain Sub Agent, Join, Summary, Quality Gate로 분석 흐름을 나눕니다.
4. Multi-Agent 구조에서 어려운 부분은 병렬 실행보다 결과 병합, 중복 실행 방지, 상태 관리입니다.
5. 좋은 Agent 설계는 "어떤 Agent를 만들까"보다 "언제 어떤 Agent를 호출하고 결과를 어떻게 합칠까"에서 결정됩니다.

---

# 19. 3일차 예고

2일차는 Agent 내부 workflow를 설계하는 방법을 다뤘습니다.

3일차에서는 이 workflow를 실제 운영 서비스로 만들기 위한 구조를 봅니다.

- FastAPI는 요청을 받고 run_id를 발급합니다.
- Celery는 긴 Agent 실행을 Worker에서 처리합니다.
- RabbitMQ는 API와 Worker 사이의 Queue 역할을 합니다.
- Loki와 Grafana는 실행 로그와 상태를 관측하게 해줍니다.

다음 질문으로 이어집니다.

> 잘 설계한 Agent workflow를 사용자가 안정적으로 호출하고, 운영자가 추적할 수 있는 서비스로 만들려면 무엇이 필요한가?
