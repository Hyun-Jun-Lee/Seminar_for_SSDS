# 3일차 세미나 구성안: 운영 가능한 Agent 시스템

## 1. 세미나 주제

**Agent는 잘 만들어도 운영되지 않으면 서비스가 아니다.**

## 2. 3일차 목표

- Streamlit 데모와 운영 서비스의 차이를 이해한다.
- LLM Agent에서 비동기 처리가 필요한 이유를 이해한다.
- FastAPI, Celery, RabbitMQ 기반 실행 구조를 이해한다.
- run_id 기반 상태 추적과 중복 실행 대응을 이해한다.
- Loki, Grafana 기반 Observability 구조를 이해한다.
- 운영 전환을 위한 체크리스트를 정리한다.

## 3. 대상자

- 2일차에서 DaOps Agent workflow를 이해한 구성원
- Agent를 실제 운영 서비스로 배포하고 관리해야 하는 개발자
- LLM 호출 지연, Worker, Queue, 로그 추적, 대시보드에 관심 있는 운영 담당자
- 데모 수준의 AI 앱과 운영형 AI 시스템의 차이를 알고 싶은 구성원

## 4. 사전 지식

- API 서버와 Worker 프로세스의 차이
- Queue, Message Broker, Background Task에 대한 기초 이해
- 로그와 모니터링의 기본 개념
- 2일차의 LangGraph workflow 구조

## 5. 3일차 핵심 메시지

- Agent workflow가 잘 만들어져도 운영 구조가 없으면 서비스가 아니다.
- LLM/Multi-Agent 실행은 오래 걸릴 수 있으므로 API 요청과 Worker 실행을 분리해야 한다.
- Queue, Worker, run_id, 상태 저장, 로그 추적은 운영형 Agent의 필수 구성요소다.
- 중복 실행과 실패 대응은 예외 상황이 아니라 운영 설계의 일부다.
- Observability가 없으면 Agent 결과를 믿기 어렵고 장애 대응도 어렵다.

## 6. 권장 시간 구성

| 시간 | 목차 | 핵심 질문 |
|---:|---|---|
| 0~5분 | 2일차 복습 | Agent workflow는 어떤 흐름으로 실행되는가? |
| 5~13분 | 데모와 운영 서비스의 차이 | 왜 Streamlit 데모만으로는 부족한가? |
| 13~25분 | FastAPI + Celery + RabbitMQ 구조 | 긴 Agent 실행을 어떻게 비동기로 처리하는가? |
| 25~35분 | run_id와 상태 관리 | 요청과 실행 결과를 어떻게 추적하는가? |
| 35~45분 | 중복 실행과 실패 대응 | Worker/Queue 환경에서 어떤 문제가 생기는가? |
| 45~55분 | Loki + Grafana 관측 구조 | 로그와 대시보드로 무엇을 봐야 하는가? |
| 55~60분 | 운영 체크리스트 | 운영 전환 전에 무엇을 확인해야 하는가? |

---

# 7. 상세 목차 및 구성 키워드

## 7.1 2일차 복습: Agent Workflow에서 운영 서비스로

### 구성 의도

- LangGraph workflow 설계와 운영 아키텍처를 연결한다.
- "잘 작동하는 코드"와 "운영 가능한 서비스"는 다르다는 전환점을 만든다.

### 주요 키워드

- Classifier
- Global Health
- Domain Sub Agent
- Aggregator/Join
- Diagnosis Summary
- Quality Gate
- Multi-Agent 장기 실행
- 여러 단계의 LLM/Tool 호출
- 실패 가능성
- 실행 추적 필요

### 강조 메시지

- 2일차가 Agent 내부 흐름이었다면, 3일차는 그 흐름을 서비스로 운영하는 방법이다.

## 7.2 왜 운영 아키텍처가 필요한가

### 구성 의도

- 단순 데모와 운영 서비스의 차이를 분명히 한다.
- 비동기 실행, 상태 조회, 실패 추적이 왜 필수인지 설명한다.

### 주요 키워드

- LLM 호출 지연
- Multi-Agent 장기 실행
- HTTP timeout
- 사용자 대기 문제
- 작업 상태 조회 필요
- 실패 추적 필요
- 재시도 필요
- 중복 실행 가능성
- 운영 로그 필요
- 서비스와 Worker 분리

### 설명 포인트

- Agent 실행은 여러 LLM 호출과 DB/Metric 조회를 포함할 수 있음
- HTTP 요청 하나 안에서 모든 처리를 끝내면 timeout과 사용자 경험 문제가 발생
- 작업이 실패했는지, 어디까지 진행됐는지 추적할 수 있어야 함
- 운영자는 장애 발생 시 로그와 상태를 확인할 수 있어야 함

### 강조 메시지

- Agent는 잘 만들어도 운영되지 않으면 서비스가 아니다.

## 7.3 Streamlit 데모와 운영 서비스 차이

### 구성 의도

- 빠른 데모 도구와 운영 서비스 구조의 역할을 구분한다.
- 세미나 청중이 흔히 보는 AI 데모와 DaOps 운영 구조의 차별점을 보여준다.

### Streamlit 키워드

- 빠른 데모
- 단일 사용자 실습
- UI 확인 용도
- 간단한 PoC
- 운영 제어 한계
- 비동기 처리 한계
- 장애 추적 한계
- 권한/배포 관리 한계

### 운영 서비스 키워드

- API 서버
- Queue
- Worker
- DB 저장
- 상태 조회
- 결과 조회
- 로그 수집
- 모니터링
- 알림
- 권한
- 배포/재시작 관리
- 장애 대응 runbook

### 강조 메시지

- Streamlit은 빠른 검증에 좋고, 운영 서비스는 지속적인 실행과 장애 대응을 위해 필요하다.

## 7.4 DaOps 운영 흐름

### 구성 의도

- 실제 DaOps의 운영 실행 경로를 요청부터 결과 조회까지 하나의 흐름으로 설명한다.
- 3일차 전체 구조의 중심 그림으로 사용한다.

### 주요 흐름

- Frontend / Messenger
- FastAPI 요청 수신
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

### 강조 메시지

- 운영형 Agent는 요청 처리, 비동기 실행, 결과 저장, 알림, 로그 관측이 하나의 흐름으로 이어져야 한다.

## 7.5 FastAPI 역할

### 구성 의도

- API 서버가 Agent를 직접 오래 실행하는 것이 아니라 요청을 받고 작업을 위임하는 역할임을 설명한다.

### 주요 키워드

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

### 설명 포인트

- FastAPI는 사용자 요청을 받고 유효성을 검증
- 장기 실행 작업은 Celery에 넘김
- API는 run_id를 반환해 사용자가 상태를 조회할 수 있게 함
- Web process와 Worker process는 메모리를 공유하지 않음

### 강조 메시지

- FastAPI는 Agent 실행 그 자체보다 요청 접수, 검증, 위임, 조회 인터페이스 역할이 중요하다.

## 7.6 Celery 역할

### 구성 의도

- 긴 Agent 실행을 Worker에서 처리하는 구조를 설명한다.
- Celery 설정이 운영 안정성에 영향을 준다는 점을 보여준다.

### 주요 키워드

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

### 설명 포인트

- Celery Worker가 LangGraph workflow를 실제 실행
- 작업 성공/실패에 따라 상태를 갱신
- Worker가 죽거나 connection loss가 나면 메시지가 재전달될 수 있음
- 재전달은 안정성을 높이지만 중복 실행 가능성도 만든다

### 강조 메시지

- Celery를 쓰면 비동기 실행이 가능하지만, 중복 실행과 상태 전이 설계를 함께 해야 한다.

## 7.7 RabbitMQ 역할

### 구성 의도

- Message Broker가 API와 Worker 사이에서 어떤 역할을 하는지 설명한다.
- Queue 운영에서 고려할 지점을 정리한다.

### 주요 키워드

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

### 설명 포인트

- FastAPI가 작업 메시지를 발행
- RabbitMQ가 메시지를 Queue에 보관
- Celery Worker가 Queue에서 메시지를 소비
- Ack 전 Worker 장애가 나면 메시지가 다시 전달될 수 있음

### 강조 메시지

- RabbitMQ는 단순 전달자가 아니라 작업 안정성과 재전달 동작에 영향을 주는 핵심 운영 구성요소다.

## 7.8 run_id 기반 추적

### 구성 의도

- 운영형 Agent에서 요청 단위 추적이 왜 중요한지 설명한다.
- 로그, DB, Dashboard, 사용자 문의 대응을 하나의 run_id로 연결한다.

### 주요 키워드

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

### 설명 포인트

- 사용자가 문의한 결과를 run_id로 찾음
- 각 Node 실행 로그에 run_id를 포함
- DB 상태와 로그를 같은 키로 연결
- 중복 실행 여부를 run_id로 감지

### 강조 메시지

- run_id는 운영형 Agent의 추적 번호이자 장애 대응의 시작점이다.

## 7.9 중복 실행과 실패 대응

### 구성 의도

- Worker/Queue 환경에서 발생할 수 있는 현실적인 문제를 설명한다.
- 실패 대응과 idempotency를 운영 설계의 핵심으로 다룬다.

### 주요 키워드

- task_acks_late=True
- Worker 중단 시 redelivery
- connection loss
- RabbitMQ message 재전달
- retry 없는 경우에도 재실행 가능
- run_id unique 처리
- 상태 전이 제어
- RUNNING to COMPLETE
- RUNNING to FAILED
- 이미 COMPLETE인 작업 재실행 방지
- DB lock
- optimistic check
- partial result 저장

### 설명 포인트

- 같은 run_id 작업이 두 번 실행될 수 있음
- 이미 완료된 작업은 다시 완료 처리하지 않도록 막아야 함
- 실패 시 partial result를 남기면 원인 분석에 도움
- 상태 전이를 명확히 정의해야 운영 혼란을 줄일 수 있음

### 강조 메시지

- 중복 실행은 버그가 아니라 분산 작업 시스템에서 고려해야 할 정상적인 위험이다.

## 7.10 Loki 로그 수집

### 구성 의도

- 운영 로그를 단순 파일이 아니라 검색 가능한 관측 데이터로 다루는 방식을 설명한다.

### 주요 키워드

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
- 로그 보존 정책
- 민감 정보 마스킹

### 검색 쿼리 후보

```logql
{service="da-dai"} |= "run_id=..."
```

```logql
{service="da-dai", component="worker"} |= "ERROR"
```

### 강조 메시지

- 로그는 남기는 것보다 찾을 수 있게 만드는 것이 중요하다.

## 7.11 Grafana Dashboard

### 구성 의도

- Agent 운영 상태를 대시보드로 보는 관점을 설명한다.
- 개발자 로그 확인을 넘어 운영자가 볼 수 있는 지표로 확장한다.

### 주요 키워드

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

### Dashboard 후보

- Agent Job Overview
- Node Execution
- LLM Error
- Celery/RabbitMQ
- Operation/Business View

### 지표 후보

- 요청 수
- 성공 수
- 실패 수
- 평균 실행 시간
- P95 실행 시간
- Node별 latency
- Node별 실패율
- queue backlog
- worker alive
- redelivery
- severity_score 분포

### 강조 메시지

- 운영자는 "성공했는가"뿐 아니라 "어디서 오래 걸리고 어디서 실패하는가"를 볼 수 있어야 한다.

## 7.12 Supervisor/tmux 운영 관리 후보

### 구성 의도

- 실제 운영 환경에서 프로세스를 어떻게 관리할지 후보를 정리한다.
- 배포와 재시작, 장애 확인 절차를 다룬다.

### 주요 키워드

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

### 강조 메시지

- 운영 시스템은 실행하는 방법보다 다시 시작하고 확인하는 방법이 더 중요해질 때가 많다.

## 7.13 운영 전환 체크리스트

### 구성 의도

- 3일차 내용을 실제 운영 준비 항목으로 정리한다.
- 4일차 신뢰성/평가 파트로 넘어가기 전 운영 기반을 닫는다.

### 체크리스트 키워드

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

### 강조 메시지

- 운영 전환은 배포가 아니라 추적, 복구, 검증, 개선이 가능한 상태를 만드는 것이다.

---

# 8. 발표자료 후보 슬라이드

1. 3일차 제목: Agent는 잘 만들어도 운영되지 않으면 서비스가 아니다
2. 2일차 복습: Agent Workflow
3. 데모와 운영 서비스의 차이
4. DaOps 운영 아키텍처 전체 그림
5. FastAPI 역할
6. Celery/RabbitMQ 역할
7. run_id 기반 추적
8. 중복 실행과 idempotency
9. Loki 로그 수집 구조
10. Grafana Dashboard 구성
11. 운영 체크리스트
12. 실제 운영 중 겪은 문제
13. 3일차 요약 및 4일차 예고

# 9. 3일차에서 사용할 반복 예시

## 예시 질문

> DB가 느립니다. 원인을 분석해 주세요.

## 운영 처리 관점

- FastAPI가 요청을 수신하고 run_id를 생성
- 요청 정보를 DB에 저장
- Celery task를 발행
- RabbitMQ에 작업 메시지가 적재
- Worker가 LangGraph Agent workflow를 실행
- 각 Node 로그에 run_id와 node_name을 포함
- 결과를 DB에 저장하고 상태를 COMPLETE로 변경
- 실패 시 FAILED 상태와 오류 로그를 남김
- Grafana에서 run_id로 전체 실행 흐름을 추적

## 3일차 활용 방식

- 비동기 처리 필요성 설명
- run_id 추적 설명
- 중복 실행/idempotency 설명
- Loki/Grafana 검색 예시 설명

# 10. 3일차 마무리 요약

- 데모와 운영 서비스는 다르다.
- LLM Agent는 장기 실행과 실패 가능성이 있으므로 API와 Worker를 분리해야 한다.
- FastAPI, Celery, RabbitMQ는 요청 접수, 비동기 실행, 작업 전달을 나누는 구조다.
- run_id는 로그, DB, Dashboard, 사용자 문의 대응을 연결하는 핵심 키다.
- Loki와 Grafana는 Agent 실행을 관측 가능한 운영 대상으로 만든다.

# 11. 4일차 예고

- 4일차에서는 Agent 결과를 어떻게 믿을 수 있게 만들지 다룬다.
- structured output, Quality Gate, LLM-as-a-judge, 평가 데이터셋을 설명한다.
- HITL, MCP, A2A를 DaOps의 향후 확장 방향으로 정리한다.
