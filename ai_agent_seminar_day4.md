# 4일차 세미나 구성안: 신뢰성, 평가, Harness Engineering, 확장 방향

## 1. 세미나 주제

**AI Agent를 믿고 운영하려면 무엇이 필요한가**

## 2. 4일차 목표

- LLM 답변을 그대로 신뢰하면 안 되는 이유를 이해한다.
- 근거 중심 응답 설계와 structured output의 필요성을 이해한다.
- Pydantic Schema와 Quality Gate가 Agent 신뢰성에 어떤 역할을 하는지 이해한다.
- LLM-as-a-judge와 평가 데이터셋 기반 개선 루프를 이해한다.
- Harness Engineering 관점에서 DaOps 구조를 재해석한다.
- HITL, MCP, A2A 기반 향후 확장 방향을 이해한다.

## 3. 대상자

- Agent 결과의 품질과 신뢰성을 고민하는 DBA/SRE/개발자
- 운영 환경에서 AI 답변을 어떻게 검증할지 알고 싶은 구성원
- LLM-as-a-judge, 평가 데이터셋, Prompt 개선 루프에 관심 있는 구성원
- DaOps를 향후 HITL, MCP, A2A 구조로 확장하는 방향을 이해하고 싶은 구성원

## 4. 사전 지식

- 1일차의 LLM 한계와 Agent 구성요소
- 2일차의 DaOps LangGraph workflow
- 3일차의 운영 아키텍처와 Observability 구조
- Prompt, Schema, 로그, 평가에 대한 기초 이해

## 5. 4일차 핵심 메시지

- LLM 답변은 그럴듯할 수 있지만, 운영 환경에서는 근거와 검증이 필요하다.
- Agent 신뢰성은 Prompt만으로 확보되지 않고 Schema, Quality Gate, Evaluation으로 보완해야 한다.
- 평가 데이터셋과 LLM-as-a-judge는 Agent 개선 루프의 기반이 된다.
- Harness Engineering은 Model 주변의 Tool, State, Verification, Observability, Permission, Failure Handling을 함께 설계하는 관점이다.
- HITL, MCP, A2A는 현재 DaOps의 과장된 설명이 아니라 향후 확장 방향으로 다루는 것이 적절하다.

## 6. 권장 시간 구성

| 시간 | 목차 | 핵심 질문 |
|---:|---|---|
| 0~5분 | 3일차 복습 | 운영 가능한 Agent 시스템에는 무엇이 필요했는가? |
| 5~13분 | 왜 신뢰성 확보가 필요한가 | LLM 답변을 그대로 믿으면 어떤 위험이 있는가? |
| 13~24분 | 근거 중심 응답과 structured output | 답변을 어떻게 구조화하고 검증 가능하게 만들 것인가? |
| 24~34분 | Quality Gate | 최종 답변을 어떤 기준으로 통과/수정/거절할 것인가? |
| 34~44분 | Evaluation과 LLM-as-a-judge | Agent 품질을 어떻게 측정하고 개선할 것인가? |
| 44~52분 | Harness Engineering | DaOps를 시스템 설계 관점으로 어떻게 재해석할 것인가? |
| 52~58분 | HITL/MCP/A2A 확장 방향 | 다음 단계 확장은 어떤 방향이 적절한가? |
| 58~60분 | 전체 요약 및 Q&A | 4일 전체 핵심 메시지는 무엇인가? |

---

# 7. 상세 목차 및 구성 키워드

## 7.1 3일차 복습: 운영 가능한 Agent 시스템

### 구성 의도

- 3일차의 운영 아키텍처를 신뢰성/평가 주제로 연결한다.
- 운영 가능성과 신뢰 가능성은 다르다는 점을 강조한다.

### 주요 키워드

- FastAPI
- Celery
- RabbitMQ
- Worker
- run_id
- 상태 저장
- Loki
- Grafana
- Dashboard
- timeout
- retry
- 중복 실행
- 장애 대응

### 강조 메시지

- 3일차가 "Agent를 어떻게 운영할 것인가"였다면, 4일차는 "Agent 결과를 어떻게 믿고 개선할 것인가"다.

## 7.2 왜 LLM 답변을 그대로 믿으면 안 되는가

### 구성 의도

- 운영 환경에서 LLM 답변이 갖는 위험을 구체적으로 보여준다.
- 신뢰성 확보 장치가 왜 필요한지 문제의식을 만든다.

### 주요 키워드

- Hallucination
- 그럴듯한 오답
- 근거 누락
- 최신 데이터 부재
- 도메인 해석 오류
- Metric 단위 오류
- 시간 범위 오류
- DB/Host 혼동
- 심각도 과대 판단
- 심각도 과소 판단
- 운영 조치 리스크
- DBA/SRE 검토 필요

### 운영 리스크 예시

- 근거 없는 원인 단정
- CPU 문제와 DB Wait 문제 혼동
- 시간 범위를 잘못 해석한 진단
- 위험한 SQL Kill 또는 파라미터 변경 추천
- 실제로는 데이터가 부족한데 확정적으로 표현

### 강조 메시지

- 운영 환경에서 LLM 답변은 "그럴듯함"보다 "근거와 검증 가능성"이 중요하다.

## 7.3 근거 중심 응답 설계

### 구성 의도

- 답변 품질을 높이기 위한 응답 설계 원칙을 설명한다.
- evidence-first prompting과 불확실성 표현을 강조한다.

### 주요 키워드

- 결론보다 근거 우선
- evidence-first prompting
- 수치 기반 판단
- 시계열 변화
- threshold
- abnormal pattern
- 원인 후보와 확정 원인 분리
- recommended_actions
- confidence
- uncertainty
- "확인 필요" 표현
- 과도한 단정 방지

### 응답 구성 후보

- 요약 결론
- 핵심 근거
- 원인 후보
- 확인이 필요한 항목
- 추천 조치
- 위험도/심각도
- 신뢰도 또는 불확실성

### 강조 메시지

- 좋은 Agent 답변은 정답처럼 말하는 답변이 아니라 근거와 불확실성을 함께 제시하는 답변이다.

## 7.4 structured output

### 구성 의도

- LLM 출력이 자유 텍스트일 때 생기는 문제를 설명한다.
- Schema 기반 출력이 후속 검증과 평가에 왜 유리한지 보여준다.

### 주요 키워드

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

### 구조화 대상 예시

- request_type
- target_nodes
- severity_score
- suspected_root_causes
- evidence
- recommended_actions
- confidence
- quality_gate_result

### 강조 메시지

- structured output은 답변을 예쁘게 만들기 위한 것이 아니라 후속 로직이 믿고 사용할 수 있게 만드는 장치다.

## 7.5 Quality Gate

### 구성 의도

- 최종 답변을 사용자에게 내보내기 전에 검증하는 단계를 설명한다.
- 운영형 Agent에서 Quality Gate가 안전장치 역할을 한다는 점을 보여준다.

### 주요 키워드

- 최종 답변 검증
- 질문에 답했는가
- 핵심 근거가 포함됐는가
- Agent 결과를 누락하지 않았는가
- 심각도 판단이 일관적인가
- 과도한 추측이 있는가
- 운영 액션이 위험하지 않은가
- 재생성 여부 판단
- 품질 점수
- Reject
- Revise
- Accept

### Quality Gate 기준 후보

- Answer Relevance: 사용자 질문에 직접 답하는가
- Evidence Coverage: 핵심 근거가 충분한가
- Consistency: Agent 결과와 최종 답변이 모순되지 않는가
- Safety: 위험한 조치를 단정적으로 권하지 않는가
- Completeness: 중요한 분석 결과를 누락하지 않았는가
- Uncertainty: 데이터 부족 시 확인 필요를 표현하는가

### 강조 메시지

- Quality Gate는 LLM 답변을 한 번 더 LLM으로 꾸미는 단계가 아니라 운영 품질을 통제하는 단계다.

## 7.6 Evaluation 체계

### 구성 의도

- Agent 품질을 감으로 판단하지 않고 데이터셋과 기준으로 평가하는 방법을 설명한다.
- Prompt 개선과 모델 변경을 검증 가능한 루프로 만든다.

### 주요 키워드

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

### 평가 데이터셋 후보

- DB 성능 저하 질문
- CPU 병목 사례
- Lock 대기 사례
- Temp 사용량 급증 사례
- Undo 이슈 사례
- 정상 상태 질문
- 데이터 부족 사례
- 복합 원인 사례

### 강조 메시지

- Agent 품질은 "이번 답변이 좋아 보인다"가 아니라 "고정된 사례에서 이전보다 좋아졌는가"로 봐야 한다.

## 7.7 LLM-as-a-judge

### 구성 의도

- LLM-as-a-judge를 평가 자동화의 한 방법으로 소개하되 한계도 함께 설명한다.
- 사람이 검토한 golden set의 필요성을 강조한다.

### 주요 키워드

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

### Rubric 후보

- 질문 응답성: 0~2점
- 근거 포함: 0~2점
- 도메인 정확성: 0~2점
- 액션 가능성: 0~2점
- 안전성: 0~2점
- 불확실성 표현: 0~2점

### 주의점

- judge도 LLM이므로 오류 가능
- 평가 기준이 모호하면 점수가 흔들림
- 사람이 검토한 기준 답변이 필요
- rule-based check와 함께 사용하는 것이 좋음

### 강조 메시지

- LLM-as-a-judge는 완전한 심판이 아니라 반복 평가를 돕는 보조 평가자다.

## 7.8 Prompt 개선 루프

### 구성 의도

- 평가 결과를 실제 개선으로 연결하는 과정을 설명한다.
- Prompt 수정, Schema 조정, Tool 개선, 재평가의 루프를 보여준다.

### 주요 키워드

- 실패 사례 수집
- 원인 분류
- Prompt 수정
- Schema 단순화
- Tool 결과 개선
- 평가 재실행
- 결과 비교
- 배포 여부 결정
- metadata DB 저장
- 코드 배포 없이 튜닝 가능한 구조 후보

### 실패 원인 후보

- 데이터 부족
- Prompt 불명확
- Schema 과도
- LLM 추론 실패
- Tool 결과 오류
- Join 누락
- 시간 범위 오류
- 도메인 용어 혼동

### 강조 메시지

- Agent 개선은 한 번의 Prompt 수정이 아니라 실패 사례를 모으고 평가를 반복하는 운영 루프다.

## 7.9 Harness Engineering으로 보는 DaOps

### 구성 의도

- 4일 전체 내용을 하나의 시스템 설계 프레임으로 묶는다.
- DaOps가 Model만이 아니라 주변 실행 환경으로 품질을 만드는 구조임을 설명한다.

### Model

- DS LLM API
- gpt-oss-120b
- 요약
- 분류
- 진단 보조

### Prompt

- system prompt
- user prompt
- evidence-first prompt
- classifier prompt
- judge prompt

### Tool

- Collector
- SQL query
- Chart generator
- Log query
- 향후 Runbook 검색

### State

- MainState
- target_nodes
- visited_nodes
- reason_data
- quality_gate_result

### Orchestration

- LangGraph
- conditional routing
- fan-out/fan-in
- Join
- Summary
- Quality Gate

### Verification

- Pydantic
- structured output
- Quality Gate
- rule-based check

### Observability

- Loki
- Grafana
- run_id
- node_name
- latency
- error trace

### Permission

- 향후 HITL
- Action 승인
- 권한 관리
- 감사 로그

### Failure Handling

- timeout
- retry
- fallback
- partial result
- redelivery 대응

### Evaluation

- LLM-as-a-judge
- regression test
- golden set
- prompt/model version

### 강조 메시지

- Harness Engineering은 모델 주변의 실행 환경을 설계하는 일이다.
- DaOps의 신뢰성은 Model 하나가 아니라 Tool, State, Verification, Observability, Evaluation이 함께 만든다.

## 7.10 HITL 확장 방향

### 구성 의도

- 운영 조치 자동화에서 사람 승인 흐름이 왜 필요한지 설명한다.
- DaOps의 향후 Action Agent 확장과 연결한다.

### 주요 키워드

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

### 강조 메시지

- 운영 환경에서는 모든 것을 자동 실행하는 것보다, 위험한 조치 앞에 사람 승인 단계를 두는 것이 현실적이다.

## 7.11 MCP 확장 방향

### 구성 의도

- 현재 Tool 구조와 MCP 기반 Tool 표준화 가능성을 비교한다.
- 1일차 개념을 4일차 확장 설계로 다시 연결한다.

### 현재 구조

- Python 함수/Collector가 Tool 역할
- Agent 내부에서 필요한 조회 기능 호출
- 서비스 내부 구현과 Tool 호출이 밀접하게 연결

### MCP 적용 후보

- Oracle Metric MCP Server
- Loki Search MCP Server
- Grafana Query MCP Server
- Runbook MCP Server
- File System MCP Server
- Ticket System MCP Server

### 장점

- Tool 표준화
- 시스템 간 연결성
- 재사용성
- Agent와 Tool 분리
- 다른 Agent/앱에서 같은 Tool 재사용 가능

### 주의점

- 권한
- 보안
- 네트워크
- 감사 로그
- Tool 호출 제한
- 민감 정보 마스킹

### 강조 메시지

- MCP는 현재 DaOps의 필수 요소라기보다 Tool 계층을 표준화할 수 있는 향후 확장 방향이다.

## 7.12 A2A 확장 방향

### 구성 의도

- 현재 LangGraph 내부 Sub Agent 구조와 독립 Agent 서비스 구조를 구분한다.
- A2A를 과장하지 않고 미래 확장 후보로 제시한다.

### 현재 구조

- 하나의 LangGraph 내부에서 Sub Agent 호출
- 중앙 workflow 안에서 Agent 책임 분리
- 상태와 병합을 한 프로세스/서비스 관점에서 관리

### A2A 적용 후보

- DB Agent 독립 서비스
- OS Agent 독립 서비스
- Log Agent 독립 서비스
- Report Agent 독립 서비스
- Alarm Agent 독립 서비스
- Ticket Agent

### 장점

- Agent 독립 배포
- 팀 간 Agent 연동
- 이기종 프레임워크 연동
- 중앙 Orchestrator와 전문 Agent 분리
- 조직 단위 확장 가능

### 주의점

- 통신 비용
- 장애 전파
- 인증/인가
- 메시지 표준
- 상태 일관성
- 관측성 복잡도

### 강조 메시지

- A2A는 Agent를 서비스 단위로 분리해야 할 필요가 생겼을 때 검토할 확장 방향이다.

## 7.13 최종 발전 방향

### 구성 의도

- 현재 DaOps의 위치와 다음 단계 확장 방향을 정리한다.
- 세미나 전체를 실무 로드맵으로 마무리한다.

### 현재 DaOps

- 분석 중심 Agent
- 근거 기반 진단
- 리포트 생성
- 로그 관측
- 비동기 실행
- Quality Gate

### 다음 단계

- HITL 기반 조치 추천
- Runbook RAG
- 장애 사례 검색
- MCP Tool Server
- Agent Evaluation Dashboard
- Action Agent
- 자동 티켓 생성
- 알림 정책 연동

### 최종 목표

- 반복 점검 자동화
- 장애 초기 분석 표준화
- 운영자 의사결정 보조
- 신뢰 가능한 AI 운영 시스템

### 강조 메시지

- DaOps의 목표는 운영자를 대체하는 것이 아니라, 반복 점검과 초기 분석을 표준화하고 의사결정을 돕는 것이다.

---

# 8. 발표자료 후보 슬라이드

1. 4일차 제목: AI Agent를 믿고 운영하려면 무엇이 필요한가
2. 3일차 복습: 운영 가능한 Agent 시스템
3. 왜 LLM 답변을 그대로 믿으면 안 되는가
4. 근거 중심 응답 설계
5. Structured Output과 Pydantic
6. Quality Gate
7. Evaluation 체계
8. LLM-as-a-judge
9. Prompt 개선 루프
10. Harness Engineering으로 보는 DaOps
11. HITL 확장 방향
12. MCP 확장 방향
13. A2A 확장 방향
14. 최종 발전 방향
15. 4일 전체 요약 및 Q&A

# 9. 4일차에서 사용할 반복 예시

## 예시 질문

> DB가 느립니다. 원인을 분석해 주세요.

## 신뢰성 관점 처리

- 답변이 실제 질문에 답했는지 확인
- Global Health와 Sub Agent 근거가 포함됐는지 확인
- CPU, Lock, Temp 등 원인 후보가 근거와 연결됐는지 확인
- 확정 원인과 의심 원인을 구분했는지 확인
- 위험한 운영 조치를 단정적으로 권하지 않았는지 확인
- 데이터 부족 시 확인 필요 항목을 명시했는지 확인
- 평가 데이터셋에서 동일 사례를 반복 검증

## 4일차 활용 방식

- Quality Gate 기준 설명
- LLM-as-a-judge rubric 설명
- Prompt 개선 루프 설명
- HITL 승인 흐름 예시 설명

# 10. 4일차 마무리 요약

- LLM 답변은 운영 환경에서 그대로 신뢰하기 어렵다.
- 근거 중심 응답, structured output, Quality Gate가 Agent 신뢰성을 높인다.
- Evaluation과 LLM-as-a-judge는 Agent 개선을 반복 가능한 루프로 만든다.
- Harness Engineering은 Model 주변의 Tool, State, Verification, Observability, Permission, Failure Handling을 함께 설계하는 관점이다.
- HITL, MCP, A2A는 DaOps의 다음 확장 방향으로 다룰 수 있다.

# 11. 4일 전체 세미나 최종 요약

- 1일차: AI, LLM, Agent의 개념 관계를 정리했다.
- 2일차: DaOps의 LangGraph 기반 Multi-Agent workflow를 살펴봤다.
- 3일차: Agent workflow를 운영 서비스로 만들기 위한 FastAPI, Celery, RabbitMQ, Loki, Grafana 구조를 다뤘다.
- 4일차: Agent 결과를 신뢰하고 개선하기 위한 Quality Gate, Evaluation, Harness Engineering, 확장 방향을 정리했다.

# 12. Q&A 예상 질문 후보

- DaOps는 기존 챗봇과 무엇이 다른가?
- LangChain만으로는 부족하고 LangGraph가 필요한 이유는 무엇인가?
- 모든 Sub Agent를 항상 호출하면 안 되는가?
- LLM 답변의 환각을 어떻게 줄일 수 있는가?
- Quality Gate도 LLM이면 완전히 믿을 수 있는가?
- RAG를 DaOps에 적용하면 어떤 부분에 가장 적합한가?
- MCP는 지금 꼭 필요한가, 향후 확장인가?
- A2A는 현재 구조에서 언제 필요해지는가?
- 사람이 승인해야 할 작업과 자동화 가능한 작업의 경계는 무엇인가?
