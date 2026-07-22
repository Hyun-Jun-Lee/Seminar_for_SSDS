# 3일차: 운영 가능한 Agent 시스템

## 오늘의 주제

**잘 설계한 Agent workflow를 실제 서비스로 운영하는 방법**

2일차에서는 DaOps를 예시로 `Classifier`, `Global Health`, `Domain Sub Agent`, `Join`, `Diagnosis Summary`, `Quality Gate`가 연결된 LangGraph workflow를 보았습니다.

3일차에서는 이 workflow를 사용자가 안정적으로 호출하고, 운영자가 추적하고, 장애가 났을 때 복구할 수 있는 서비스 구조로 확장합니다.

핵심 질문은 다음과 같습니다.

> "DB가 느립니다"라는 요청이 들어왔을 때, Agent 실행이 오래 걸리거나 실패하거나 중복 실행되어도 운영자가 전체 흐름을 추적할 수 있게 만들려면 어떤 구조가 필요한가?

---

# 1. 2일차 복습: Agent 내부 Workflow

2일차의 DaOps Agent workflow는 다음 흐름이었습니다.

```text
User Question
  -> Classifier
  -> Global Health
  -> Domain Sub Agent
  -> Join
  -> Diagnosis Summary
  -> Quality Gate
  -> Final Response
```

이 구조의 관심사는 Agent 내부 판단입니다.

- 질문을 어떻게 분류할 것인가?
- 전체 상태를 먼저 볼 것인가?
- 어떤 Domain Agent를 호출할 것인가?
- 여러 Agent 결과를 어떻게 합칠 것인가?
- 최종 답변을 어떻게 검증할 것인가?

하지만 실제 서비스에서는 여기서 끝나지 않습니다.

사용자는 요청을 보낸 뒤 상태를 알고 싶어 하고, 운영자는 실행 로그를 보고 싶어 하며, 장애가 나면 어디서 실패했는지 찾아야 합니다.

3일차부터는 Agent를 **workflow 코드**가 아니라 **운영 대상 서비스**로 봅니다.

---

# 2. 운영 서비스에서 새로 생기는 질문

Agent workflow가 잘 작동해도 운영 서비스가 되려면 다른 질문에 답해야 합니다.

| 질문 | 운영에서 필요한 답 |
|---|---|
| 요청을 받으면 바로 끝나는가? | 오래 걸릴 수 있으므로 비동기 실행이 필요하다 |
| 사용자는 무엇을 받는가? | 최종 답변이 아니라 먼저 `run_id`를 받는다 |
| 결과는 어디에 저장되는가? | DB에 요청, 상태, 결과, 오류를 저장한다 |
| 실패하면 어떻게 아는가? | 상태와 로그를 `FAILED`로 남긴다 |
| Worker가 죽으면 어떻게 되는가? | Queue 재전달과 중복 실행 가능성을 고려한다 |
| 운영자는 무엇으로 추적하는가? | `run_id`로 DB, 로그, Dashboard를 연결한다 |

운영형 Agent 시스템에서 중요한 것은 답변 생성만이 아닙니다.

**요청 접수, 비동기 실행, 상태 저장, 결과 조회, 실패 추적, 로그 관측**이 함께 설계되어야 합니다.

---

# 3. 왜 API 요청 안에서 Agent를 끝내면 안 되는가

가장 단순한 구조는 API가 요청을 받고 Agent를 바로 실행한 뒤 응답하는 방식입니다.

```text
User
  -> FastAPI
  -> LangGraph Agent 실행
  -> LLM 호출
  -> Tool / DB / Metric 조회
  -> 최종 답변 반환
```

이 구조는 짧은 작업에는 단순합니다.

하지만 DaOps 같은 운영 분석 Agent에는 문제가 생깁니다.

- 여러 번의 LLM 호출이 필요할 수 있다.
- Metric, SQL, OS 지표 조회가 느릴 수 있다.
- Domain Sub Agent가 여러 개 실행될 수 있다.
- 외부 API나 DB 연결이 일시적으로 실패할 수 있다.
- HTTP timeout이 먼저 발생할 수 있다.
- 사용자는 브라우저나 메신저에서 오래 기다려야 한다.
- 중간에 실패하면 어디까지 실행됐는지 알기 어렵다.

즉, API 요청 하나 안에서 모든 일을 끝내는 구조는 운영 분석 Agent와 잘 맞지 않습니다.

운영형 Agent는 다음처럼 나누어 생각합니다.

```text
API 요청
  -> 작업 접수
  -> run_id 반환

Worker 실행
  -> Agent workflow 실행
  -> 상태와 결과 저장
  -> 로그 기록
```

핵심은 **요청 접수와 Agent 실행을 분리하는 것**입니다.

---

# 4. DaOps 운영 아키텍처 전체 흐름

DaOps 운영 구조는 다음처럼 볼 수 있습니다.

```text
Frontend / Messenger
  |
  v
FastAPI
  - 요청 검증
  - run_id 생성
  - 요청 정보 DB 저장
  - Celery task 발행
  - run_id 즉시 반환
  |
  v
RabbitMQ Queue
  - 작업 메시지 보관
  - Worker에게 전달
  |
  v
Celery Worker
  - LangGraph Agent 실행
  - LLM / Tool / Metric 조회
  - 중간 로그 기록
  - 결과 DB 저장
  |
  v
DB / Log / Dashboard
  - 상태 조회
  - 결과 조회
  - Loki 로그 검색
  - Grafana Dashboard 확인
```

사용자 관점에서는 요청을 보내고 `run_id`를 받은 뒤 상태와 결과를 조회합니다.

운영자 관점에서는 같은 `run_id`로 DB 상태, Worker 로그, Node 실행 로그, Grafana Dashboard를 추적합니다.

운영형 Agent는 LangGraph만으로 완성되지 않습니다.

**FastAPI, Queue, Worker, DB, Log, Dashboard가 함께 있어야 서비스가 됩니다.**

---

# 5. ReAct 전환 시 운영에서 달라지는 것

현재 DaOps workflow는 큰 흐름이 비교적 명확한 Node 구조입니다.

```text
Classifier
  -> Global Health
  -> Domain Sub Agent
  -> Join
  -> Diagnosis Summary
  -> Quality Gate
```

각 Node는 정해진 책임을 수행하고 다음 Node로 결과를 넘깁니다.

예를 들어 SQL Agent라면 "Top SQL을 조회하고 병목 후보를 정리한다"처럼 책임이 비교적 고정됩니다.

ReAct 형태로 전환하면 Node 내부 실행 방식이 달라집니다.

```text
SQL ReAct Agent
  -> 현재 문제를 정리한다
  -> 필요한 Tool을 선택한다
  -> Tool을 실행한다
  -> Observation을 확인한다
  -> 다음 Tool이 필요한지 판단한다
  -> 충분하면 분석 결과를 만든다
```

즉, LangGraph의 큰 흐름은 유지하더라도 Domain Agent 내부는 더 동적으로 바뀝니다.

| 구분 | 기존 Node Agent | ReAct Agent |
|---|---|---|
| 실행 흐름 | 미리 정한 처리 단계 중심 | 상황에 따라 Tool 선택과 반복 |
| Tool 호출 | Node 구현 안에 고정되기 쉬움 | Agent가 다음 Tool을 선택 |
| 로그 단위 | Node 시작/종료 중심 | Node 안의 step/action/observation 중심 |
| 실패 지점 | 어느 Node에서 실패했는지 추적 | 어느 step, 어떤 Tool에서 실패했는지 추적 |
| 운영 위험 | Node 중복 실행, timeout | 반복 실행, 과도한 Tool 호출, Tool 오용 |

ReAct 전환에서 중요한 것은 "더 똑똑한 Agent"를 만드는 것만이 아닙니다.

운영 관점에서는 **한 Node 안에서 실행 횟수와 Tool 호출 순서가 동적으로 변한다는 점**이 더 중요합니다.

그래서 기존의 `run_id + node_name` 추적만으로는 부족해질 수 있습니다.

ReAct Agent 로그에는 `step_id`, `action`, `tool_name`, `observation_status`, `duration_ms`가 함께 남아야 합니다.

```text
run_id=20260722_153012_abcd
node_name=sql_agent
step_id=1
action=tool_call
tool_name=query_top_sql
observation_status=success
duration_ms=820
```

```text
run_id=20260722_153012_abcd
node_name=sql_agent
step_id=2
action=tool_call
tool_name=query_execution_plan
observation_status=timeout
duration_ms=10000
```

이렇게 남겨야 운영자가 다음 질문에 답할 수 있습니다.

- ReAct Agent가 어떤 Tool을 어떤 순서로 호출했는가?
- 같은 Tool을 반복 호출했는가?
- 어느 Observation 이후 실패했는가?
- timeout은 LLM에서 났는가, Tool에서 났는가?
- 충분한 근거 없이 답변을 만들었는가?

ReAct Agent에는 실행 예산도 필요합니다.

Tool을 선택할 수 있다는 것은 필요한 만큼 반복할 수 있다는 뜻이지만, 운영 서비스에서는 무제한 반복을 허용할 수 없습니다.

예를 들어 다음과 같은 제한이 필요합니다.

```python
react_config = {
    "max_steps": 6,
    "max_tool_calls": 5,
    "tool_timeout_sec": 10,
    "total_timeout_sec": 60,
    "allowed_tools": [
        "query_metric",
        "query_top_sql",
        "query_lock",
    ],
}
```

| 제한 | 이유 |
|---|---|
| `max_steps` | 추론과 Tool 호출 반복을 제한 |
| `max_tool_calls` | 외부 시스템 조회 비용과 부하 제한 |
| `tool_timeout_sec` | 느린 Tool 하나가 전체 실행을 막지 않게 함 |
| `total_timeout_sec` | Worker 점유 시간 제한 |
| `allowed_tools` | Agent가 호출 가능한 Tool 범위 제한 |

Tool 정책도 명확해야 합니다.

DaOps처럼 운영 시스템을 분석하는 Agent에서는 조회 Tool과 조치 Tool을 구분해야 합니다.

| Tool 유형 | 예시 | 운영 정책 |
|---|---|---|
| Read Tool | metric 조회, Top SQL 조회, Lock 조회 | ReAct Agent가 직접 호출 가능 |
| Diagnostic Tool | 실행 계획 조회, 세션 상세 조회 | timeout과 argument validation 필요 |
| Action Tool | session kill, parameter 변경, restart | 기본적으로 직접 실행 금지 |

ReAct Agent가 Tool을 선택할 수 있게 하더라도, 어떤 Tool을 호출할 수 있는지는 운영 시스템이 통제해야 합니다.

또 하나 중요한 점은 로그에 모델의 내부 추론을 그대로 남기지 않는 것입니다.

ReAct라고 해서 모든 reasoning을 원문으로 저장하면 민감 정보, 불필요한 추론, 잘못된 중간 판단이 로그에 남을 수 있습니다.

대신 운영 로그에는 다음 정도를 남기는 것이 적절합니다.

```text
reason_summary="SQL 병목 가능성이 높아 Top SQL 조회가 필요함"
action="tool_call"
tool_name="query_top_sql"
observation_summary="elapsed time 상위 SQL 3건 확인"
```

핵심은 다음과 같습니다.

> ReAct 전환은 Node 구현 방식을 바꾸는 일이면서, 동시에 step 단위 추적, 실행 예산, Tool 정책, 로그 마스킹을 함께 설계해야 하는 운영 변경입니다.

---

# 6. FastAPI의 역할

FastAPI는 Agent를 오래 실행하는 곳이 아닙니다.

FastAPI의 핵심 역할은 요청을 받고, 검증하고, 작업을 Worker에게 넘기는 것입니다.

```text
FastAPI가 하는 일
  1. 요청 파라미터 검증
  2. run_id 생성
  3. 요청 정보를 DB에 저장
  4. Celery task 발행
  5. 사용자에게 run_id 반환
  6. 상태 조회 API 제공
  7. 결과 조회 API 제공
```

예시 코드는 다음과 같습니다.

```python
@router.post("/agent/runs")
def create_agent_run(request: AgentRunRequest):
    run_id = uuid4().hex

    save_run(
        run_id=run_id,
        question=request.question,
        status="PENDING",
    )

    execute_agent_task.delay(run_id)

    return {
        "run_id": run_id,
        "status": "PENDING",
    }
```

중요한 점은 `create_agent_run()` 안에서 LangGraph를 직접 오래 실행하지 않는다는 것입니다.

API는 빠르게 응답하고, 실제 Agent 실행은 Worker가 담당합니다.

---

# 7. Web Process와 Worker Process는 메모리를 공유하지 않는다

운영 구조에서 자주 헷갈리는 부분이 있습니다.

FastAPI process와 Celery Worker process는 보통 서로 다른 프로세스입니다.

```text
FastAPI process
  - HTTP 요청 처리
  - run_id 생성
  - task 발행

Celery Worker process
  - Queue에서 task 소비
  - LangGraph 실행
  - 결과 저장
```

따라서 FastAPI의 메모리에 저장한 값이 Worker에서 그대로 보인다고 생각하면 안 됩니다.

예를 들어 `app.state.current_run` 같은 방식으로 실행 상태를 저장하면, Web process 안에서는 보일 수 있지만 Worker process에서는 보장되지 않습니다.

운영 상태는 메모리가 아니라 DB 같은 외부 저장소에 둬야 합니다.

| 저장할 것 | 위치 |
|---|---|
| 요청 원문 | DB |
| 실행 상태 | DB |
| 최종 결과 | DB |
| 오류 메시지 | DB / Log |
| 상세 실행 흐름 | Log / Loki |

운영형 Agent에서 상태 저장은 선택 사항이 아니라 기본 구조입니다.

---

# 8. run_id는 운영 추적 번호다

`run_id`는 단순한 UUID가 아닙니다.

운영형 Agent에서 `run_id`는 요청, 실행, 결과, 로그, 사용자 문의를 연결하는 기준입니다.

```text
run_id = "20260722_153012_abcd"

DB
  -> runs.run_id
  -> runs.status
  -> runs.result
  -> runs.error

Worker Log
  -> run_id=20260722_153012_abcd
  -> node_name=global_health
  -> status=started

Grafana
  -> run_id 변수로 로그 검색
```

`run_id`가 없으면 운영자는 다음 질문에 답하기 어렵습니다.

- 사용자가 문의한 요청이 어떤 실행인가?
- 어느 Worker가 실행했는가?
- 어느 Node에서 실패했는가?
- 결과가 저장됐는가?
- 같은 작업이 중복 실행됐는가?

그래서 모든 주요 로그와 DB row에는 `run_id`가 포함되어야 합니다.

> run_id는 운영형 Agent의 추적 번호이자 장애 대응의 시작점입니다.

---

# 9. 작업 상태 모델

운영 서비스는 작업 상태를 명확히 정의해야 합니다.

예를 들어 Agent 실행 상태는 다음처럼 둘 수 있습니다.

| 상태 | 의미 |
|---|---|
| `PENDING` | 요청은 접수됐지만 Worker가 아직 실행하지 않음 |
| `RUNNING` | Worker가 Agent workflow를 실행 중 |
| `COMPLETE` | 최종 결과 저장 완료 |
| `FAILED` | 실행 중 오류 발생 |
| `CANCELLED` | 사용자 또는 운영자가 취소 |

상태 전이는 아무렇게나 바뀌면 안 됩니다.

```text
PENDING
  -> RUNNING
  -> COMPLETE

PENDING
  -> RUNNING
  -> FAILED

PENDING
  -> CANCELLED
```

특히 중요한 것은 이미 끝난 작업입니다.

```text
COMPLETE -> RUNNING   허용하지 않음
COMPLETE -> FAILED    허용하지 않음
FAILED   -> COMPLETE  재처리 정책 없이는 허용하지 않음
```

상태 전이를 제어하지 않으면 Worker 재실행이나 중복 실행 상황에서 결과가 덮어써질 수 있습니다.

운영형 Agent에서는 상태 모델이 곧 실행 제어 장치입니다.

---

# 10. Celery의 역할

Celery는 긴 작업을 Worker에서 실행하게 해주는 Background Task 시스템입니다.

FastAPI가 task를 발행하면 Celery Worker가 Queue에서 가져와 실행합니다.

```text
FastAPI
  -> execute_agent_task.delay(run_id)
  -> RabbitMQ
  -> Celery Worker
  -> LangGraph 실행
```

Celery task는 보통 다음 책임을 갖습니다.

```python
@celery_app.task(bind=True, acks_late=True)
def execute_agent_task(self, run_id: str):
    mark_running(run_id)

    try:
        request = load_run_request(run_id)
        result = graph.invoke({
            "user_question": request.question,
            "run_id": run_id,
        })
        save_result(run_id, result)
        mark_complete(run_id)

    except Exception as exc:
        save_error(run_id, str(exc))
        mark_failed(run_id)
        raise
```

이 코드에서 중요한 것은 세 가지입니다.

1. Worker가 실행 시작 시 `RUNNING`으로 상태를 바꾼다.
2. 성공하면 결과를 저장하고 `COMPLETE`로 바꾼다.
3. 실패하면 오류를 저장하고 `FAILED`로 바꾼다.

Celery는 비동기 실행을 가능하게 하지만, 동시에 중복 실행과 실패 대응 설계도 필요하게 만듭니다.

---

# 11. Celery 설정에서 봐야 할 것

Celery는 설정에 따라 운영 동작이 달라집니다.

| 설정 | 의미 | 운영 영향 |
|---|---|---|
| `task_acks_late=True` | 작업이 끝난 뒤 ack | Worker 장애 시 재전달 가능 |
| `worker_prefetch_multiplier=1` | Worker가 미리 많이 가져가지 않음 | 긴 작업에서 작업 쏠림 감소 |
| `task_ignore_result=True` | Celery result backend에 결과 저장 안 함 | 결과는 별도 DB에 저장 |
| `task_time_limit` | task 최대 실행 시간 | 무한 실행 방지 |
| `task_soft_time_limit` | 정리 가능한 timeout | 실패 처리 기회 제공 |

예를 들어 `acks_late=True`는 안정성을 높입니다.

Worker가 작업을 가져간 뒤 죽으면 메시지를 잃지 않고 다시 전달할 수 있기 때문입니다.

하지만 그 말은 같은 `run_id` 작업이 다시 실행될 수 있다는 뜻이기도 합니다.

```text
Worker A가 task 수신
  -> RUNNING 처리
  -> LangGraph 실행 중 Worker 종료
  -> RabbitMQ가 메시지 재전달
  -> Worker B가 같은 run_id 실행
```

따라서 Celery 설정은 Queue만의 문제가 아니라 DB 상태 전이와 idempotency 설계까지 연결됩니다.

---

# 12. RabbitMQ의 역할

RabbitMQ는 FastAPI와 Worker 사이에서 작업 메시지를 보관하고 전달하는 Message Broker입니다.

```text
Producer
  FastAPI

Broker
  RabbitMQ Exchange / Queue

Consumer
  Celery Worker
```

RabbitMQ가 하는 일은 단순 전달보다 넓습니다.

- Worker가 바쁠 때 작업을 Queue에 보관한다.
- Worker가 새로 뜨면 쌓인 작업을 전달한다.
- Ack를 받기 전 Worker가 죽으면 메시지를 다시 전달할 수 있다.
- 관리 UI를 통해 Queue 적재량과 Consumer 상태를 확인할 수 있다.

운영에서 봐야 할 RabbitMQ 지표는 다음과 같습니다.

| 지표 | 의미 |
|---|---|
| Queue backlog | 아직 처리되지 않은 작업 수 |
| Consumer count | Queue를 소비하는 Worker 수 |
| Redelivered messages | 재전달된 메시지 수 |
| Ack rate | 처리 완료 속도 |
| Connection state | Worker와 Broker 연결 상태 |

RabbitMQ는 단순한 통로가 아니라 작업 안정성과 재전달 동작에 영향을 주는 핵심 구성요소입니다.

---

# 13. 중복 실행은 버그가 아니라 운영 위험이다

Queue와 Worker 구조에서는 같은 작업이 두 번 실행될 수 있습니다.

대표 상황은 다음과 같습니다.

```text
1. FastAPI가 run_id=R1 작업을 발행한다.
2. Worker A가 R1을 가져간다.
3. Worker A가 LangGraph 실행 중 죽는다.
4. RabbitMQ는 ack를 받지 못한다.
5. RabbitMQ가 R1 메시지를 다시 전달한다.
6. Worker B가 R1을 다시 실행한다.
```

retry 설정을 명시적으로 하지 않아도 이런 재실행은 발생할 수 있습니다.

따라서 운영형 Agent는 같은 `run_id`가 다시 들어와도 안전해야 합니다.

예를 들어 Worker 시작 시점에 다음을 확인할 수 있습니다.

```python
def mark_running_if_allowed(run_id: str) -> bool:
    run = get_run(run_id)

    if run.status in ("COMPLETE", "CANCELLED"):
        return False

    if run.status == "RUNNING":
        return False

    update_status(run_id, from_status="PENDING", to_status="RUNNING")
    return True
```

핵심은 같은 작업이 다시 실행되어도 이미 완료된 결과를 덮어쓰지 않게 하는 것입니다.

> 중복 실행은 예외 상황이 아니라 분산 작업 시스템에서 고려해야 할 정상적인 위험입니다.

---

# 14. 실패를 어떻게 남길 것인가

Agent 실행은 실패할 수 있습니다.

실패 원인은 다양합니다.

- LLM timeout
- API rate limit
- DB connection error
- Metric 조회 실패
- 특정 Tool 응답 오류
- LangGraph Node 내부 예외
- Worker process 종료

운영에서 중요한 것은 실패를 숨기지 않는 것입니다.

실패 시 최소한 다음 정보를 남겨야 합니다.

| 항목 | 이유 |
|---|---|
| `run_id` | 요청 단위 추적 |
| `status=FAILED` | 상태 조회에서 실패 확인 |
| `failed_node` | 어느 단계에서 실패했는지 확인 |
| `error_type` | timeout, connection, validation 등 분류 |
| `error_message` | 원인 분석의 시작점 |
| `partial_result` | 실패 전까지의 분석 결과 보존 |
| `created_at`, `updated_at` | 장애 시간대 확인 |

부분 결과도 중요합니다.

예를 들어 OS/CPU Agent와 Lock Agent는 성공했지만 SQL Agent에서 실패했다면, 전체를 버리는 것보다 어디까지 성공했는지 남기는 것이 장애 분석에 도움이 됩니다.

운영형 Agent에서 실패 처리는 `except`로 끝나는 코드가 아니라, 나중에 원인을 찾을 수 있게 남기는 설계입니다.

---

# 15. 로그는 남기는 것보다 찾을 수 있어야 한다

로그를 많이 남긴다고 운영이 쉬워지지는 않습니다.

운영자가 찾을 수 있는 형태로 남겨야 합니다.

좋은 Agent 로그는 다음 정보를 포함합니다.

```text
timestamp=2026-07-22T15:30:12
level=INFO
service=daops-agent
component=worker
run_id=20260722_153012_abcd
node_name=global_health
event=node_started
```

Node 실행 로그는 최소한 다음 이벤트를 남기는 것이 좋습니다.

```text
run_started
node_started
node_completed
node_failed
llm_call_started
llm_call_completed
tool_call_started
tool_call_failed
run_completed
run_failed
```

운영자가 궁금해하는 질문은 보통 다음과 같습니다.

- 이 `run_id`는 어디까지 실행됐는가?
- 어떤 Node가 가장 오래 걸렸는가?
- 실패가 LLM 문제인가, Tool 문제인가?
- 특정 시간대에 실패가 늘었는가?
- Worker별 오류가 다른가?

이 질문에 답하려면 로그에 `run_id`, `component`, `node_name`, `event`, `duration_ms`, `error_type` 같은 구조화된 필드가 있어야 합니다.

---

# 16. Loki로 로그를 모으는 이유

로컬 파일에만 로그를 남기면 운영자가 검색하기 어렵습니다.

Loki는 로그를 중앙에서 수집하고, Grafana에서 검색할 수 있게 해줍니다.

DaOps 로그 수집 흐름은 다음처럼 볼 수 있습니다.

```text
FastAPI log file
Celery Worker log file
  |
  v
Alloy / Promtail
  |
  v
Loki
  |
  v
Grafana Explore / Dashboard
```

Loki에서는 label을 잘 설계해야 합니다.

| Label | 예시 |
|---|---|
| `service` | `daops-agent` |
| `component` | `api`, `worker` |
| `env` | `dev`, `prod` |
| `host` | `agent-server-01` |

`run_id`는 label로 둘 수도 있지만, cardinality가 커질 수 있으므로 로그 본문 필드로 검색하는 방식도 고려해야 합니다.

예시 LogQL은 다음과 같습니다.

```logql
{service="daops-agent"} |= "run_id=20260722_153012_abcd"
```

```logql
{service="daops-agent", component="worker"} |= "ERROR"
```

```logql
{service="daops-agent"} |= "node_name=sql_agent" |= "node_failed"
```

핵심은 로그를 파일로 남기는 것이 아니라, 장애 시 빠르게 검색할 수 있는 운영 데이터로 만드는 것입니다.

---

# 17. Grafana Dashboard에서 봐야 할 것

Grafana는 Agent 실행 상태를 한눈에 보기 위한 화면입니다.

운영자는 단순히 성공 여부만 보고 싶어 하지 않습니다.

어디서 오래 걸리고, 어디서 실패하고, Queue가 밀리는지 보고 싶어 합니다.

Dashboard는 다음처럼 나눠 볼 수 있습니다.

| Dashboard | 보는 것 |
|---|---|
| Agent Job Overview | 요청 수, 성공 수, 실패 수, 평균 실행 시간 |
| Node Execution | Node별 latency, 실패율, 실행 횟수 |
| LLM Error | timeout, rate limit, provider error |
| Celery / RabbitMQ | queue backlog, worker alive, redelivery |
| Operation View | severity_score 분포, 주요 장애 유형 |

대표 지표는 다음과 같습니다.

- 요청 수
- 성공 수
- 실패 수
- 평균 실행 시간
- P95 실행 시간
- Node별 실행 시간
- Node별 실패율
- Queue backlog
- Worker alive
- Redelivery count
- LLM timeout count

Grafana의 목적은 예쁜 화면이 아니라 운영 판단입니다.

> 운영자는 "성공했는가"뿐 아니라 "어디서 오래 걸리고 어디서 실패하는가"를 볼 수 있어야 합니다.

---

# 18. 프로세스 운영: 실행보다 재시작이 중요하다

운영 환경에서는 FastAPI, Celery Worker, RabbitMQ, Loki 수집기가 각각 별도 프로세스로 실행됩니다.

```text
FastAPI
Celery Worker
RabbitMQ
Alloy / Promtail
Grafana / Loki
```

서비스를 운영하다 보면 "어떻게 실행하는가"보다 다음 질문이 더 중요해집니다.

- 지금 프로세스가 살아 있는가?
- 배포 후 어떤 순서로 재시작하는가?
- Worker만 재시작해도 되는가?
- 로그는 어디서 확인하는가?
- RabbitMQ connection 문제는 어떻게 확인하는가?
- 장애 시 어떤 순서로 점검하는가?

간단한 운영 환경에서는 `tmux`로 프로세스를 나눠 실행할 수 있습니다.

조금 더 운영에 가깝게 가면 `supervisor` 같은 프로세스 관리 도구를 사용할 수 있습니다.

```text
supervisorctl status
supervisorctl restart daops-api
supervisorctl restart daops-worker
```

운영 시스템은 실행 방법보다 재시작, 상태 확인, 로그 확인 절차가 더 중요해질 때가 많습니다.

---

# 19. 반복 예시: "DB가 느립니다"

오늘도 같은 질문으로 전체 흐름을 봅니다.

> DB가 느립니다. 원인을 분석해 주세요.

운영 처리 관점에서는 다음 순서로 진행됩니다.

```text
1. 사용자가 요청을 보낸다.

2. FastAPI가 요청을 수신한다.
   - question 저장
   - run_id 생성
   - status=PENDING 저장

3. FastAPI가 Celery task를 발행한다.
   - execute_agent_task.delay(run_id)

4. RabbitMQ가 작업 메시지를 Queue에 보관한다.

5. Celery Worker가 작업을 가져간다.
   - status=RUNNING 변경
   - LangGraph Agent 실행

6. LangGraph 내부 workflow가 실행된다.
   - Classifier
   - Global Health
   - Domain Sub Agent
   - Join
   - Diagnosis Summary
   - Quality Gate

7. 각 Node와 ReAct step은 로그를 남긴다.
   - run_id
   - node_name
   - step_id
   - tool_name
   - event
   - duration_ms

8. Worker가 결과를 저장한다.
   - status=COMPLETE
   - final_answer 저장
   - diagnosis_summary 저장

9. 실패하면 오류를 저장한다.
   - status=FAILED
   - failed_node
   - error_message
   - partial_result

10. 운영자는 Grafana에서 run_id로 전체 실행 흐름을 추적한다.
```

이 예시에서 중요한 것은 Agent가 좋은 답을 만드는 것만이 아닙니다.

사용자와 운영자가 **요청 이후의 실행 상태를 추적할 수 있어야 한다는 것**입니다.

---

# 20. 운영 전환 체크리스트

운영 전환은 배포 버튼을 누르는 일이 아닙니다.

추적, 복구, 검증, 개선이 가능한 상태를 만드는 일입니다.

운영 전환 전에는 최소한 다음을 확인해야 합니다.

| 영역 | 확인할 것 |
|---|---|
| API | timeout, 요청 검증, 상태 조회 API |
| Worker | Celery 설정, 동시 실행 수, time limit |
| Queue | RabbitMQ durable queue, backlog 확인 |
| 상태 관리 | run_id, 상태 전이, 중복 실행 방지 |
| ReAct 실행 | max_steps, tool timeout, allowed_tools |
| 결과 저장 | final result, partial result, error 저장 |
| 로그 | run_id, node_name, step_id, tool_name, ERROR 분리 |
| 관측성 | Loki 수집, Grafana Dashboard |
| 장애 대응 | 재시작 절차, runbook, 알림 기준 |
| 보안 | 민감 정보 마스킹, 권한 관리 |
| 성능 | 부하 테스트, P95 실행 시간, queue backlog |

이 체크리스트는 "완벽한 운영"을 의미하지 않습니다.

하지만 최소한 문제가 생겼을 때 어디를 봐야 하는지 알 수 있는 상태를 만듭니다.

---

# 21. 3일차 요약

3일차의 핵심은 다음 여섯 가지입니다.

1. Agent workflow가 잘 만들어져도 운영 구조가 없으면 서비스가 아닙니다.
2. LLM Agent는 오래 걸리고 실패할 수 있으므로 API 요청과 Worker 실행을 분리해야 합니다.
3. ReAct 전환은 step 단위 추적, 실행 예산, Tool 정책을 함께 요구합니다.
4. FastAPI, Celery, RabbitMQ는 요청 접수, 비동기 실행, 작업 전달을 나누는 구조입니다.
5. `run_id`, 상태 저장, idempotency는 중복 실행과 장애 대응의 핵심입니다.
6. Loki와 Grafana는 Agent 실행을 검색 가능하고 관측 가능한 운영 대상으로 만듭니다.

운영형 Agent 시스템은 LLM을 잘 호출하는 코드가 아니라, **실행을 맡기고, 상태를 남기고, 실패를 추적하고, 다시 확인할 수 있는 구조**입니다.

---

# 22. 4일차 예고

3일차는 Agent workflow를 운영 가능한 서비스 구조로 만드는 방법을 다뤘습니다.

4일차에서는 그 다음 질문을 다룹니다.

> Agent가 만든 결과를 어떻게 믿을 수 있게 만들 것인가?

4일차에서는 다음 내용을 봅니다.

- structured output
- Quality Gate 고도화
- LLM-as-a-judge
- 평가 데이터셋
- HITL
- MCP
- A2A

3일차가 "Agent를 어떻게 운영할 것인가"였다면, 4일차는 "Agent 결과를 어떻게 검증하고 신뢰할 것인가"로 이어집니다.
