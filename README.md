# 🧠 AI-Driven DevOps Automation System Architecture

## 📦 전체 시스템 구성도

```plaintext
[K8s Cluster]
  ↓
[Monitoring (Prometheus / Loki / Grafana)]
  ↓
[Backend (FastAPI)]
  ├─ MongoDB 저장
  └─ 1차 이상탐지
  ↓
[Code Llama Model Server]
  ├─ Vector DB (RAG) 결합
  ├─ 로그/코드 원인분석
  └─ kubectl 제안 명령 생성
  ↓
[GPT API (검증)]
  ├─ 제안 명령 정책·안전성 검증
  └─ 실행 전/후 영향 분석
  ↓
[Frontend (React Dashboard)]
  ├─ 결과 요약/시각화
  └─ 승인 또는 거부
  ↓
[MCP Server]
  ├─ kubectl / helm / terraform 실제 실행
  └─ 결과 회신 → DB 기록 → UI 갱신
```
  ## 1️⃣ K8s ↔ Monitoring (점검·보완)

Prometheus와 Loki에서 수집된 메트릭 및 로그를 **Alertmanager 웹훅(Webhook)** 을 통해 백엔드로 전달합니다.  
이 과정에서 **이벤트 유실이나 지연**이 발생하지 않도록, **Kafka**, **NATS**, **Redis Streams** 같은 **이벤트 버스(Event Bus)** 를 중간 계층에 추가하는 것이 좋습니다.

이벤트 버스를 활용하면 다음과 같은 장점이 있습니다:
- **이상 감지 → 이벤트 발행 → 소비(Consume)** 흐름을 표준화할 수 있음  
- **재시도(Retry)**, **이벤트 보존(Persistence)**, **순서 보장(Order Guarantee)** 을 활성화할 수 있음  

따라서, Prometheus 알림 흐름은 다음과 같이 구성할 수 있습니다:

```plaintext
Prometheus → Alertmanager → Backend Webhook
         ↘︎ (Event Publish) → Kafka/NATS Topic → Consumer (Code Llama)
```

## 2️⃣ Backend ↔ Code Llama (점검·보완)

백엔드는 **이벤트 정규화(normalization)** 와 **1차 이상탐지(rule/statistics 기반)** 에 집중하고,  
Code Llama 서버는 **RAG(Vector DB 기반 검색)** 을 통해 다음과 같은 정보를 함께 받아야 합니다:

- 직전 **N분간의 메트릭 데이터**
- **유사 과거 인시던트** 로그 및 조치 내역
- 관련 **코드 스니펫 / YAML 설정 파일**

이렇게 다층 컨텍스트를 전달하면 Llama가 단일 로그만 분석할 때보다 훨씬 정확하게  
**원인 가설(cause hypothesis)** 과 **조치 제안(action plan)** 을 도출할 수 있습니다.

즉, 단순한 로그 입력 대신 **시간 + 유사도 기반 컨텍스트 검색 결과**를  
항상 포함하는 구조로 설계하는 것이 이상적입니다.

## 3️⃣ Code Llama ↔ GPT 검증 (점검·보완)

Code Llama가 생성한 명령은 **정책 기반 검증(Policy-based Validation)** 단계를 거쳐야 합니다.  
이때 GPT API는 단순히 명령의 문법이나 형태만 확인하는 것이 아니라,  
**시스템 정책 JSON**을 기반으로 명령의 안전성과 적합성을 평가합니다.

예를 들어, 정책 파일은 다음과 같이 정의할 수 있습니다:

```json
{
  "allowed_commands": ["kubectl scale", "helm upgrade"],
  "forbidden_keywords": ["delete", "rm", "drop"],
  "restricted_namespaces": ["kube-system", "longhorn-system"]
}
```
검증 과정에서 GPT는 위 정책을 참고하여 다음을 수행합니다:

- ✅ **명령어가 허용된 리스트 내에 존재하는가**  
- 🚫 **금지된 키워드가 포함되어 있는가**  
- 🔒 **보호된 네임스페이스를 수정하려는 시도가 있는가**

만약 위험한 명령으로 판단된다면, 단순히 폐기하지 말고  
**“리뷰 필요(Needs Review)”** 상태로 저장하여 운영자가 후속 검토할 수 있도록 합니다.  

또한 GPT 프롬프트에는 반드시 다음 항목을 포함해야 합니다:

1. 🧩 **정책 파일(JSON)**  
2. 📊 **현재 클러스터 상태 요약**  
3. 🔁 **롤백 경로 및 영향 범위**

이러한 구성을 통해 GPT 검증 단계는 단순한 문자열 필터링이 아니라  
**정책 준수 검증(Policy Compliance Validation)** 을 수행하는  
안전하고 신뢰성 있는 AI 판단 레이어로 발전할 수 있습니다.

## 4️⃣ GPT 검증 ↔ Frontend (점검·보완)

프론트엔드는 GPT가 검증한 결과를 운영자에게 **직관적이고 투명하게** 보여주는 역할을 수행합니다.  
이 단계에서는 **AI의 판단 근거(reasoning)**, **리스크 수준(Risk Score)**, **예상 영향 범위(Impact Scope)** 를 함께 시각화하여  
운영자가 한눈에 이해하고 승인 여부를 판단할 수 있어야 합니다.

---

### ✅ 핵심 구현 포인트
- 결과, 근거, 리스크 스코어를 함께 표시하여 **Human-in-the-Loop 승인 체계**를 강화  
- 승인/거부 내역을 **MongoDB Audit Log**에 저장  
  - `승인자`, `시간`, `사유`, `명령 해시(command hash)` 등의 필드를 포함  
- 명령의 **위험 등급(Risk Level)** 을 자동 분류하여  
  - 🔹 **Low Risk** → 자동 실행  
  - 🔸 **Medium Risk** → 운영자 알림 및 승인 필요  
  - 🔴 **High Risk** → 수동 승인 전까지 보류  

---

### 💡 예시 워크플로우
```plaintext
[GPT 검증 결과 수신]
        ↓
[Frontend Dashboard 표시]
        ↓
운영자 확인 → 승인/거부 클릭
        ↓
결과 MongoDB Audit Log에 기록
        ↓
MCP로 승인된 명령 전송
```
이 구조를 통해 AI 자동화의 신속성과 인간 검증의 안정성을 모두 확보할 수 있습니다.

## 5️⃣ Frontend ↔ MCP 실행 (점검·보완)

MCP는 실제 **명령 실행 계층(Execution Layer)** 으로, 프론트엔드에서 승인된 조치를 안전하게 수행하는 역할을 맡습니다.  
이 구간의 설계 목표는 **명령의 안전성 확보**, **롤백 가능성 보장**, **결과의 추적성 유지**입니다.

---

### ✅ 핵심 구현 포인트
1. **Dry-Run(시뮬레이션) 실행 필수화**  
   - 실제 명령 실행 전 반드시 `--dry-run` 모드 또는 시뮬레이션을 수행하여 명령의 유효성을 검증합니다.  
   - 예:  
     ```bash
     kubectl apply -f deployment.yaml --dry-run=client
     ```

2. **실행 전 스냅샷 저장 (Rollback 대비)**  
   - 변경 전 리소스 매니페스트 또는 Helm values 파일을 백업하여 실패나 비정상 상황 발생 시 즉시 복원 가능하도록 합니다.  
   - GitOps 환경에서는 `git revert` 또는 ArgoCD Rollback 기능을 활용할 수도 있습니다.

3. **실행 결과의 완전 기록 (Full Audit Trail)**  
   - 실행 명령, 결과 로그(stdout/stderr), 상태 코드, 실행 시간 등을 모두 DB에 저장합니다.  
   - 이러한 데이터는 차후 리포트 생성과 AI 학습 피드백 단계에서 재활용할 수 있습니다.

---

### 💡 실행 흐름 예시
```plaintext
승인된 명령 수신
   ↓
Dry-Run 수행 → 결과 검증
   ↓
실제 명령 실행 (kubectl / helm / terraform)
   ↓
실행 결과 로그 수집
   ↓
백엔드 DB 저장 → UI 갱신
```
이 과정을 통해 AI가 제안한 조치라도 항상 검증된 실행 경로를 따르게 되어,
시스템 안전성과 신뢰성을 동시에 확보할 수 있습니다.

## 6️⃣ 피드백 / 학습 루프 (점검·보완)

이 단계는 시스템의 **지속적인 자기개선(Self-Learning)** 을 담당합니다.  
AI가 수행한 조치의 결과를 단순히 기록하는 데서 끝나는 것이 아니라,  
이를 **재학습 데이터(RAG Knowledge)** 로 축적하여 다음 의사결정의 정확도를 높입니다.

---

### ✅ 핵심 구현 포인트
1. **사건-조치-결과(Incident → Action → Result) 데이터 구조화**  
   - 모든 인시던트에 대해 “무엇이 발생했고, 어떤 조치가 취해졌으며, 그 결과가 어땠는가”를 하나의 문서로 정규화합니다.  
   - 예:  
     ```json
     {
       "incident_id": "INC_20251021_003",
       "root_cause": "Memory Leak in Backend",
       "action": "kubectl rollout restart deployment backend",
       "result": "success",
       "timestamp": "2025-10-21T19:45:00Z"
     }
     ```

2. **Vector DB에 패턴 문서 저장 (RAG 기반 학습)**  
   - 사건의 로그, 코드 조각, 조치 내용, 결과 요약을 임베딩(embedding) 형태로 Vector DB에 저장합니다.  
   - 이후 유사 사건이 발생하면 Llama가 해당 벡터를 참조해 더 빠르고 정확한 원인 추론을 수행합니다.

3. **성능 지표 기반 튜닝 자동화**  
   - 정기적으로 “조치 성공률”, “평균 해결시간(MTTR)”, “모델 정확도” 등을 계산하여  
     이상탐지 임계값, 프롬프트 문장, 정책 설정 등을 자동 조정합니다.

---

### 💡 예시 흐름
```plaintext
Incident 발생
   ↓
Action 실행
   ↓
Result 수집 → MongoDB 저장
   ↓
Vector DB에 사건-조치-결과 패턴 추가
   ↓
다음 이상탐지 시 RAG 검색 → 유사 사건 기반 판단 강화
이 과정을 통해 시스템은 반복적인 패턴을 스스로 학습하며,
시간이 지날수록 더 정확하고 안정적인 자율운영(Autonomous Operation) 으로 진화하게 됩니다.

## 7️⃣ 전체 구조 요약

이 시스템은 **관측 → 인식 → 계획 → 검증 → 승인 → 실행 → 피드백**의 완전한 폐루프(Closed Loop)를 갖춘  
AI 기반 자율운영(Autonomous DevOps) 구조입니다.  
각 모듈은 역할이 명확히 분리되어 있으며, 인간의 개입(Human-in-the-Loop)을 통해 안정성과 투명성을 확보합니다.

---

### ✅ 핵심 구조 요약

| 구간 | 역할 | 주요 기능 | 보완 포인트 |
|------|------|------------|--------------|
| **Monitoring → Backend** | 이상 탐지 | 메트릭 수집 / 이벤트 트리거 | Kafka·NATS 등 이벤트 버스 추가 |
| **Backend → LLM** | 원인 분석 | 로그 정규화 / RAG 검색 | 컨텍스트 강화 (유사 사건 검색 포함) |
| **LLM → GPT 검증** | 정책 검증 | 안전성·정책 기준 검토 | 정책 JSON 및 리뷰 상태 저장 |
| **GPT → Frontend 승인** | 결과 검토 | 리스크 표시 / 승인·거부 | Risk Level 자동화 정책 도입 |
| **Frontend → MCP 실행** | 명령 실행 | kubectl·helm 실행 / 결과 회신 | Dry-run + Rollback 의무화 |
| **Feedback Loop** | 자기학습 | 사건-조치-결과 기록 / RAG 재학습 | 피드백 루프 자동화 |

---

### 💡 아키텍처 설계 요약

- **Event Bus 기반 비동기 구조**로 신뢰성 확보  
- **RAG + Policy 기반 판단 구조**로 정확성과 안전성 강화  
- **Human-in-the-Loop** 승인 체계로 실시간 운영 제어  
- **Rollback 및 로그 이력 관리**로 재현성과 복구성 보장  
- **Self-Learning 루프**로 지속적인 AI 운영 지능 고도화

---

이로써 시스템은 시간이 지날수록 더 정교하게 학습하며,  
“AI가 운영을 돕는 수준”을 넘어 **AI가 스스로 판단하고 조치하는 DevOps Agent**로 발전할 수 있습니다.

## 8️⃣ 실행 체크리스트

이 섹션은 실제 시스템 구현 및 배포 단계에서 반드시 확인해야 할 핵심 점검 항목들을 정리한 것입니다.  
각 단계별 실행 조건과 필수 설정을 따라가면, 프로덕션 환경에서도 안전하게 작동하는 자율운영 구조를 구성할 수 있습니다.

---

### ✅ 핵심 점검 항목

1. **이벤트 전달 경로 설정**  
   - `Prometheus → Alertmanager → Webhook → Event Bus(Kafka/NATS)`  
   - 이벤트 재시도, 순서 보장, 장애 시 큐 보존 기능 확인  

2. **RAG 컨텍스트 최소 보장**  
   - Code Llama 호출 시 다음 컨텍스트를 반드시 포함:  
     - 현재 로그  
     - 직전 N분간의 메트릭  
     - 유사 사건 로그  
     - 관련 코드 / YAML 설정  

3. **검증 프롬프트 구성**  
   - GPT 검증 요청 시 반드시 아래 정보를 포함:  
     - 정책 JSON  
     - 현재 클러스터 상태  
     - 롤백 경로 및 영향 범위  

4. **프론트엔드 승인 화면 구성**  
   - 리스크 등급, 근거 설명, 영향 범위를 시각화  
   - 승인/거부 내역은 MongoDB 감사 로그에 자동 기록  

5. **MCP 실행 보호 절차**  
   - 모든 명령 실행 전 `--dry-run` 필수  
   - 실행 전후 스냅샷(리소스/매니페스트) 자동 백업  
   - 실행 결과와 표준출력 로그는 DB에 저장  

6. **피드백 루프 동작 검증**  
   - 인시던트–조치–결과 패턴이 Vector DB에 정상 기록되는지 확인  
   - RAG 검색이 유사 사건을 참조하는지 주기적으로 테스트  

---

### 💡 운영 시점 팁
- 새 모델 버전 도입 시 기존 Vector DB를 백업 후 재임베딩(Re-embedding) 수행  
- Kafka 토픽 파티션 수와 리텐션 기간을 미리 조정하여 이벤트 폭주 대비  
- 감사 로그를 기반으로 리스크 분류 정책을 지속적으로 개선  
- Helm, Terraform 실행 로그는 별도 스토리지(MinIO/S3)에 백업 유지  

---

이 체크리스트를 따라가면 AI 기반 운영 자동화 시스템이  
**예측 가능하고 복원력 있는(Resilient)** 형태로 안정적으로 동작합니다.
