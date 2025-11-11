# Skyvern 프로젝트 DDD (Domain-Driven Design) 분석

## 목차
1. [개요](#개요)
2. [Bounded Contexts (경계 지어진 컨텍스트)](#bounded-contexts-경계-지어진-컨텍스트)
3. [Domain Models (도메인 모델)](#domain-models-도메인-모델)
4. [Aggregates (애그리게이트)](#aggregates-애그리게이트)
5. [Domain Services (도메인 서비스)](#domain-services-도메인-서비스)
6. [Repositories (리포지토리)](#repositories-리포지토리)
7. [Context Map (컨텍스트 맵)](#context-map-컨텍스트-맵)
8. [Ubiquitous Language (보편 언어)](#ubiquitous-language-보편-언어)

---

## 개요

Skyvern은 LLM과 컴퓨터 비전을 활용한 브라우저 자동화 플랫폼입니다. DDD 관점에서 분석하면 여러 명확한 도메인과 경계가 존재합니다.

**핵심 비즈니스 가치:**
- 자연어 기반 웹 자동화
- 동적 웹사이트 상호작용
- 복잡한 워크플로우 오케스트레이션
- 멀티테넌트 SaaS 플랫폼

---

## Bounded Contexts (경계 지어진 컨텍스트)

### 1. **Workflow Management Context** (워크플로우 관리 컨텍스트)
**책임:** 복잡한 다단계 워크플로우의 정의, 실행, 모니터링

**핵심 개념:**
- Workflow (워크플로우)
- Block (블록)
- Run (실행)
- Parameter (파라미터)

**데이터베이스 모델:**
- `WorkflowModel`
- `WorkflowRunModel`
- `WorkflowRunBlockModel`
- `WorkflowParameterModel`
- `OutputParameterModel`

**서비스:**
- `WorkflowService` (skyvern/services/workflow_service.py)
- `RunService` (skyvern/services/run_service.py)
- `BlockService` (skyvern/services/block_service.py)

**특징:**
- 블록 기반 워크플로우 조합
- 파라미터 바인딩 및 데이터 흐름
- 상태 머신 관리 (QUEUED → RUNNING → COMPLETED/FAILED)
- 버전 관리 (workflow_permanent_id + version)

---

### 2. **Task Execution Context** (태스크 실행 컨텍스트)
**책임:** 단일 웹 자동화 태스크의 실행과 관리

**핵심 개념:**
- Task (태스크)
- Step (단계)
- Action (액션)
- Thought (사고 과정)

**데이터베이스 모델:**
- `TaskModel` (v1)
- `TaskV2Model` (observer_cruises - v2)
- `StepModel`
- `ActionModel`
- `ThoughtModel` (observer_thoughts)

**서비스:**
- `TaskV1Service` (skyvern/services/task_v1_service.py)
- `TaskV2Service` (skyvern/services/task_v2_service.py)
- `ActionService` (skyvern/services/action_service.py)

**특징:**
- LLM 기반 의사결정
- 반복적 실행 (iterative execution)
- 스크린샷 기반 분석
- 목표 지향적 자동화

---

### 3. **Browser Automation Context** (브라우저 자동화 컨텍스트)
**책임:** 웹 브라우저 제어, DOM 조작, 세션 관리

**핵심 개념:**
- BrowserSession (브라우저 세션)
- BrowserProfile (브라우저 프로필)
- Page (페이지)
- Element (요소)

**데이터베이스 모델:**
- `PersistentBrowserSessionModel`
- `BrowserProfileModel`

**서비스:**
- `BrowserSessionService` (skyvern/services/browser_session_service.py)
- `BrowserManager` (skyvern/webeye/browser_manager.py)
- `BrowserFactory` (skyvern/webeye/browser_factory.py)

**특징:**
- Playwright 기반 브라우저 제어
- 영속적 세션 지원
- 프록시 및 헤더 커스터마이징
- 멀티 탭/윈도우 관리

---

### 4. **Credential Management Context** (자격증명 관리 컨텍스트)
**책임:** 민감한 자격증명의 안전한 저장 및 관리

**핵심 개념:**
- Credential (자격증명)
- Vault (저장소)
- TOTP (Time-based One-Time Password)

**데이터베이스 모델:**
- `CredentialModel`
- `CredentialParameterModel`
- `BitwardenLoginCredentialParameterModel`
- `BitwardenSensitiveInformationParameterModel`
- `BitwardenCreditCardDataParameterModel`
- `OnePasswordCredentialParameterModel`
- `AzureVaultCredentialParameterModel`
- `AWSSecretParameterModel`
- `TOTPCodeModel`

**서비스:**
- `CredentialVaultService` (skyvern/forge/sdk/services/credential/)
- `BitwardenCredentialService`
- `AzureCredentialVaultService`
- `OTPService` (skyvern/services/otp_service.py)

**특징:**
- 다중 vault 통합 (Bitwarden, 1Password, Azure Key Vault, AWS Secrets)
- 암호화된 저장
- TOTP 코드 자동 생성
- 조직별 격리

---

### 5. **Organization & Identity Context** (조직 및 신원 컨텍스트)
**책임:** 멀티테넌시, 인증, 권한 관리

**핵심 개념:**
- Organization (조직)
- AuthToken (인증 토큰)
- User (사용자)

**데이터베이스 모델:**
- `OrganizationModel`
- `OrganizationAuthTokenModel`
- `OrganizationBitwardenCollectionModel`

**특징:**
- Row-level 멀티테넌시
- JWT 기반 인증
- 조직별 설정 (max_steps_per_run, webhook_callback_url 등)
- 도메인 기반 조직 식별

---

### 6. **Artifact Storage Context** (아티팩트 저장 컨텍스트)
**책임:** 스크린샷, 파일, 실행 결과의 저장 및 관리

**핵심 개념:**
- Artifact (아티팩트)
- Storage (저장소)

**데이터베이스 모델:**
- `ArtifactModel`

**서비스:**
- Artifact storage abstraction (skyvern/forge/sdk/artifact/)
- S3/Azure Blob Storage integrations

**특징:**
- 다중 스토리지 백엔드 (S3, Azure, Local)
- 아티팩트 타입 구분 (screenshot, recording, LLM response, HTML 등)
- 실행 추적 가능성

---

### 7. **Script Generation Context** (스크립트 생성 컨텍스트)
**책임:** 실행된 워크플로우로부터 재사용 가능한 코드 생성

**핵심 개념:**
- Script (스크립트)
- ScriptRevision (스크립트 버전)
- ScriptFile (스크립트 파일)
- ScriptBlock (스크립트 블록)

**데이터베이스 모델:**
- `ScriptModel`
- `ScriptFileModel`
- `ScriptBlockModel`
- `WorkflowScriptModel`

**서비스:**
- `ScriptService` (skyvern/services/script_service.py)
- `WorkflowScriptService` (skyvern/services/workflow_script_service.py)
- `RunCodeService` (skyvern/services/run_code_service.py)

**특징:**
- AI 기반 코드 생성
- 워크플로우 → 코드 변환
- 버전 관리
- 캐시 키 기반 스크립트 재사용

---

### 8. **Content Organization Context** (콘텐츠 구성 컨텍스트)
**책임:** 워크플로우 및 리소스의 논리적 구성

**핵심 개념:**
- Folder (폴더)
- TaskRun (태스크 실행 이력)

**데이터베이스 모델:**
- `FolderModel`
- `TaskRunModel`

**특징:**
- 계층적 구성
- 조직별 폴더 관리
- 실행 이력 추적

---

### 9. **Debugging & Development Context** (디버깅 및 개발 컨텍스트)
**책임:** 워크플로우 개발 및 디버깅 지원

**핵심 개념:**
- DebugSession (디버그 세션)
- BlockRun (블록 실행)

**데이터베이스 모델:**
- `DebugSessionModel`
- `BlockRunModel`

**특징:**
- 인터랙티브 디버깅
- 블록별 실행 및 테스트
- VNC 스트리밍 지원

---

## Domain Models (도메인 모델)

### 1. Workflow Aggregate

#### Entities (엔티티)

**Workflow** (워크플로우)
```
- workflow_id: UUID (PK)
- workflow_permanent_id: UUID (비즈니스 키)
- version: int
- organization_id: UUID (FK)
- title: string
- description: string
- workflow_definition: JSON (블록 정의)
- status: WorkflowStatus
- folder_id: UUID (FK, optional)
```

**WorkflowRun** (워크플로우 실행)
```
- workflow_run_id: UUID (PK)
- workflow_id: UUID (FK)
- workflow_permanent_id: UUID
- organization_id: UUID (FK)
- status: RunStatus
- browser_session_id: UUID (FK, optional)
- proxy_location: ProxyLocation
- queued_at, started_at, finished_at: DateTime
```

**WorkflowRunBlock** (워크플로우 실행 블록)
```
- workflow_run_block_id: UUID (PK)
- workflow_run_id: UUID (FK)
- block_type: BlockType
- label: string
- status: BlockStatus
- output: JSON
- failure_reason: string (optional)
```

#### Value Objects (값 객체)

- `BlockType`: TASK, FOR_LOOP, CODE, VALIDATION, ACTION, NAVIGATION, etc.
- `WorkflowStatus`: published, draft, auto_generated, importing, import_failed
- `BlockStatus`: running, completed, failed, terminated, canceled, timed_out
- `ProxyLocation`: US, EU, ASIA, etc.

---

### 2. Task Aggregate

#### Entities

**Task** (v1)
```
- task_id: UUID (PK)
- organization_id: UUID (FK)
- url: string
- navigation_goal: string
- data_extraction_goal: string
- status: TaskStatus
- workflow_run_id: UUID (FK, optional)
- browser_session_id: UUID (FK, optional)
```

**TaskV2** (observer_cruise)
```
- observer_cruise_id: UUID (PK)
- organization_id: UUID (FK)
- workflow_run_id: UUID (FK, optional)
- prompt: text
- url: string
- status: TaskV2Status
- output: JSON
- max_steps: int
```

**Step** (단계)
```
- step_id: UUID (PK)
- task_id: UUID (FK)
- order: int
- status: string
- output: JSON
- is_last: bool
- retry_index: int
- input_token_count, output_token_count: int
- step_cost: Decimal
```

**Action** (액션)
```
- action_id: UUID (PK)
- task_id: UUID (FK)
- step_id: UUID (FK)
- action_type: ActionType
- status: string
- reasoning: string
- intention: string
- element_id: string
- action_json: JSON
- confidence_float: Decimal
```

**Thought** (사고 과정 - v2)
```
- observer_thought_id: UUID (PK)
- observer_cruise_id: UUID (FK)
- user_input: text
- observation: string
- thought: string
- answer: string
- observer_thought_type: ThoughtType (plan, execute, extract, etc.)
```

#### Value Objects

- `TaskStatus`: queued, running, completed, failed, canceled, terminated
- `ActionType`: CLICK, INPUT_TEXT, SELECT_OPTION, UPLOAD_FILE, etc.
- `ThoughtType`: plan, execute, extract, validate

---

### 3. Credential Aggregate

#### Entities

**Credential**
```
- credential_id: UUID (PK)
- organization_id: UUID (FK)
- name: string
- credential_type: CredentialType
- vault_type: VaultType (optional)
- username: string (optional)
- totp_type: TOTPType
- totp_identifier: string (optional)
```

**TOTPCode**
```
- totp_code_id: UUID (PK)
- totp_identifier: string
- organization_id: UUID (FK)
- code: string
- content: string
- expired_at: DateTime
- otp_type: string
```

#### Value Objects

- `CredentialType`: api_key, oauth_token, cookie, password, etc.
- `VaultType`: bitwarden, onepassword, azure_vault, aws_secret
- `TOTPType`: none, totp, hotp

---

### 4. BrowserSession Aggregate

#### Entities

**PersistentBrowserSession**
```
- persistent_browser_session_id: UUID (PK)
- organization_id: UUID (FK)
- runnable_type: string (workflow, task)
- runnable_id: UUID
- browser_id: string
- status: SessionStatus
- timeout_minutes: int
- proxy_location: ProxyLocation
```

**BrowserProfile**
```
- browser_profile_id: UUID (PK)
- organization_id: UUID (FK)
- name: string (unique per org)
- description: string
```

#### Value Objects

- `SessionStatus`: created, active, inactive, closed

---

### 5. Organization Aggregate

#### Root Entity

**Organization**
```
- organization_id: UUID (PK)
- organization_name: string
- domain: string (optional, indexed)
- webhook_callback_url: URL
- max_steps_per_run: int
- max_retries_per_step: int
- bw_organization_id: string (Bitwarden integration)
```

**OrganizationAuthToken**
```
- id: UUID (PK)
- organization_id: UUID (FK)
- token_type: TokenType
- token: string (indexed)
- encrypted_token: string
- valid: bool
```

---

### 6. Artifact Aggregate

#### Entity

**Artifact**
```
- artifact_id: UUID (PK)
- organization_id: UUID (FK)
- workflow_run_id: UUID (optional)
- task_id: UUID (optional)
- step_id: UUID (optional)
- artifact_type: ArtifactType
- uri: string (storage location)
```

#### Value Objects

- `ArtifactType`:
  - SCREENSHOT_LLM
  - SCREENSHOT_ACTION
  - RECORDING
  - VISIBLE_ELEMENTS_TREE
  - VISIBLE_ELEMENTS_ID_MAP
  - HTML_SCRAPE
  - LLM_RESPONSE
  - LLM_PROMPT

---

## Aggregates (애그리게이트)

### Aggregate 규칙 및 경계

DDD에서 Aggregate는 일관성 경계를 나타냅니다. Skyvern에서 식별된 주요 Aggregate:

#### 1. **Workflow Aggregate**
**Root:** Workflow
**포함 엔티티:**
- WorkflowParameter
- OutputParameter
- AWSSecretParameter
- BitwardenLoginCredentialParameter
- CredentialParameter
- etc.

**불변 조건:**
- Workflow의 모든 파라미터는 workflow_id로 연결
- 동일한 (organization_id, workflow_permanent_id, version)은 유니크
- 파라미터 키는 워크플로우 내에서 유니크

**라이프사이클:**
- CREATE → DRAFT → PUBLISHED
- 버전 업데이트 시 새로운 Workflow 엔티티 생성

---

#### 2. **WorkflowRun Aggregate**
**Root:** WorkflowRun
**포함 엔티티:**
- WorkflowRunBlock (여러 개)
- WorkflowRunParameter (여러 개)
- WorkflowRunOutputParameter (여러 개)
- Task (선택적 - TaskBlock이 있는 경우)

**불변 조건:**
- 모든 블록은 workflow_run_id로 연결
- 블록 실행 순서는 워크플로우 정의에 따름
- 파라미터 참조는 실행 시점에 해결되어야 함

**라이프사이클:**
- QUEUED → RUNNING → COMPLETED/FAILED/TERMINATED/CANCELED

---

#### 3. **Task Aggregate**
**Root:** Task (v1) 또는 TaskV2
**포함 엔티티:**
- Step (여러 개)
- Action (Step별로 여러 개)
- Thought (v2 only, 여러 개)
- Artifact (여러 개)

**불변 조건:**
- Step 순서는 order 필드로 관리
- 각 Step은 여러 Action을 가질 수 있음
- max_steps_per_run 제한 준수

**라이프사이클:**
- CREATED → QUEUED → RUNNING → COMPLETED/FAILED/TERMINATED

---

#### 4. **Credential Aggregate**
**Root:** Credential
**포함 엔티티:**
- TOTPCode (시간 제한적 연관)

**불변 조건:**
- Credential은 organization_id로 격리
- TOTP 코드는 만료 시간이 있음
- 삭제는 soft delete (deleted_at)

---

#### 5. **Organization Aggregate**
**Root:** Organization
**포함 엔티티:**
- OrganizationAuthToken (여러 개)
- OrganizationBitwardenCollection (여러 개)

**불변 조건:**
- 모든 리소스는 organization_id로 격리
- 도메인은 조직별로 유니크 (optional)

---

#### 6. **BrowserSession Aggregate**
**Root:** PersistentBrowserSession
**연관:**
- BrowserProfile (참조로만 연결)

**불변 조건:**
- 세션은 timeout_minutes 후 자동 종료
- 한 번에 하나의 runnable (workflow 또는 task)만 연결

---

#### 7. **Script Aggregate**
**Root:** Script (ScriptModel)
**포함 엔티티:**
- ScriptFile (여러 개)
- ScriptBlock (여러 개)

**불변 조건:**
- 동일한 (organization_id, script_id, version)은 유니크
- 파일 경로는 script_revision_id 내에서 유니크

---

## Domain Services (도메인 서비스)

Domain Service는 여러 Aggregate를 조율하거나 특정 Aggregate에 속하지 않는 비즈니스 로직을 캡슐화합니다.

### 1. **WorkflowOrchestrationService**
**위치:** `skyvern/services/workflow_service.py`, `skyvern/services/run_service.py`

**책임:**
- 워크플로우 실행 조율
- 블록 간 데이터 흐름 관리
- 병렬/순차 실행 제어
- 에러 처리 및 재시도

**주요 메서드:**
```python
async def execute_workflow(workflow_id, parameters) -> WorkflowRun
async def execute_block(block, context) -> BlockResult
async def resolve_parameters(block, context) -> dict
```

---

### 2. **TaskExecutionService**
**위치:** `skyvern/services/task_v2_service.py`

**책임:**
- LLM 기반 태스크 실행
- 반복적 계획 및 실행
- 스크린샷 분석
- 목표 달성 검증

**주요 메서드:**
```python
async def execute_task(task_id) -> TaskOutput
async def plan_next_step(task, current_state) -> Plan
async def execute_action(action, page) -> ActionResult
```

---

### 3. **BrowserAutomationService**
**위치:** `skyvern/webeye/browser_manager.py`, `skyvern/services/browser_session_service.py`

**책임:**
- 브라우저 인스턴스 생성 및 관리
- 세션 라이프사이클 관리
- 페이지 스크래핑 및 상호작용
- 스크린샷 캡처

**주요 메서드:**
```python
async def create_browser_session(organization_id, config) -> BrowserSession
async def get_page(session_id) -> Page
async def scrape_page(page) -> ScrapedData
```

---

### 4. **CredentialResolutionService**
**위치:** `skyvern/forge/sdk/services/credential/`

**책임:**
- 다양한 Vault로부터 자격증명 조회
- TOTP 코드 생성
- 자격증명 주입 (파라미터 해결)

**주요 메서드:**
```python
async def resolve_credential(credential_id) -> CredentialData
async def generate_totp_code(totp_identifier) -> str
async def inject_credentials(workflow_run_context) -> None
```

---

### 5. **ArtifactStorageService**
**위치:** `skyvern/forge/sdk/artifact/`

**책임:**
- 아티팩트 저장 및 조회
- 스토리지 추상화 (S3, Azure, Local)
- 아티팩트 메타데이터 관리

**주요 메서드:**
```python
async def store_artifact(artifact_type, data, metadata) -> Artifact
async def retrieve_artifact(artifact_id) -> bytes
```

---

### 6. **WebhookDispatchService**
**위치:** `skyvern/services/webhook_service.py`

**책임:**
- 워크플로우/태스크 완료 시 웹훅 발송
- 재시도 로직
- 실패 추적

**주요 메서드:**
```python
async def dispatch_webhook(url, payload) -> bool
async def handle_webhook_failure(run_id, reason) -> None
```

---

### 7. **ScriptGenerationService**
**위치:** `skyvern/services/script_service.py`

**책임:**
- 워크플로우 실행으로부터 코드 생성
- 스크립트 캐싱
- 코드 실행

**주요 메서드:**
```python
async def generate_script_from_workflow_run(workflow_run_id) -> Script
async def execute_script(script_id, parameters) -> ExecutionResult
```

---

### 8. **OTPGenerationService**
**위치:** `skyvern/services/otp_service.py`

**책임:**
- TOTP/HOTP 코드 생성
- 코드 만료 관리
- 다양한 소스로부터 OTP 파싱

**주요 메서드:**
```python
async def generate_totp_code(identifier, secret) -> str
async def parse_otp_from_email(email_content) -> str
```

---

## Repositories (리포지토리)

Repository는 Aggregate의 영속성을 관리합니다. Skyvern에서는 주로 `skyvern/forge/sdk/db/` 패키지에서 관리됩니다.

### Repository 패턴 구현

**위치:** `skyvern/forge/sdk/db/client.py`

```python
class AgentDB:
    async def get_task(task_id: str, organization_id: str) -> Task
    async def create_task(...) -> Task
    async def update_task(task_id: str, ...) -> Task

    async def get_workflow(workflow_id: str, organization_id: str) -> Workflow
    async def create_workflow(...) -> Workflow

    async def get_workflow_run(workflow_run_id: str) -> WorkflowRun
    async def create_workflow_run(...) -> WorkflowRun

    async def get_credential(credential_id: str, organization_id: str) -> Credential
    async def create_credential(...) -> Credential

    # ... 기타 등등
```

### 주요 Repository 인터페이스

#### 1. **WorkflowRepository**
```python
- get_workflow_by_id(workflow_id, organization_id) -> Workflow
- get_workflow_by_permanent_id(permanent_id, version, org_id) -> Workflow
- list_workflows(organization_id, filters) -> List[Workflow]
- create_workflow(workflow_data) -> Workflow
- update_workflow(workflow_id, updates) -> Workflow
- delete_workflow(workflow_id) -> bool (soft delete)
```

#### 2. **WorkflowRunRepository**
```python
- get_workflow_run(workflow_run_id) -> WorkflowRun
- create_workflow_run(run_data) -> WorkflowRun
- update_workflow_run_status(run_id, status) -> WorkflowRun
- get_run_blocks(workflow_run_id) -> List[WorkflowRunBlock]
- create_run_block(block_data) -> WorkflowRunBlock
- update_run_block(block_id, updates) -> WorkflowRunBlock
```

#### 3. **TaskRepository**
```python
- get_task(task_id, organization_id) -> Task
- create_task(task_data) -> Task
- update_task(task_id, updates) -> Task
- get_task_steps(task_id) -> List[Step]
- create_step(step_data) -> Step
- get_step_actions(step_id) -> List[Action]
- create_action(action_data) -> Action
```

#### 4. **CredentialRepository**
```python
- get_credential(credential_id, organization_id) -> Credential
- list_credentials(organization_id) -> List[Credential]
- create_credential(credential_data) -> Credential
- update_credential(credential_id, updates) -> Credential
- delete_credential(credential_id) -> bool
```

#### 5. **ArtifactRepository**
```python
- get_artifact(artifact_id) -> Artifact
- create_artifact(artifact_data) -> Artifact
- list_artifacts(filters) -> List[Artifact]
```

#### 6. **OrganizationRepository**
```python
- get_organization(organization_id) -> Organization
- get_organization_by_domain(domain) -> Organization
- create_organization(org_data) -> Organization
- update_organization(organization_id, updates) -> Organization
```

### Query Patterns

**조직별 격리:**
```sql
SELECT * FROM workflows
WHERE organization_id = ? AND workflow_id = ?
```

**복합 인덱스 활용:**
```sql
-- models.py의 Index 정의
Index("idx_tasks_org_created", "organization_id", "created_at")
Index("org_task_index", "organization_id", "task_id")
Index("workflow_oid_status_idx", "organization_id", "status")
```

**Soft Delete 패턴:**
```sql
SELECT * FROM workflows
WHERE organization_id = ?
  AND deleted_at IS NULL
```

---

## Context Map (컨텍스트 맵)

### Context 간 관계도

```
┌─────────────────────────────────────────────────────────────────┐
│                    Organization & Identity                       │
│                          (Core Domain)                           │
└───────────────────────┬─────────────────────────────────────────┘
                        │ (Authentication & Authorization)
                        │
        ┌───────────────┼───────────────────────┐
        │               │                       │
        ▼               ▼                       ▼
┌──────────────┐  ┌───────────────────┐  ┌──────────────────┐
│  Workflow    │  │   Task Execution  │  │   Credential     │
│  Management  │◄─┤   (Core Domain)   │─►│   Management     │
└──────┬───────┘  └─────────┬─────────┘  └──────────────────┘
       │                    │
       │ (Orchestrates)     │ (Uses)
       │                    │
       ▼                    ▼
┌────────────────────────────────────────┐
│       Browser Automation               │
│       (Supporting Domain)              │
└────────────────┬───────────────────────┘
                 │
                 │ (Produces)
                 ▼
┌────────────────────────────────────────┐
│       Artifact Storage                 │
│       (Generic Subdomain)              │
└────────────────────────────────────────┘

     ┌──────────────────────────┐
     │  Script Generation       │
     │  (Supporting Domain)     │
     └──────────────────────────┘
             ▲
             │ (Generates from)
             │
     ┌───────┴──────────────────┐
     │                           │
Workflow Run              Task Execution
```

### Context 간 통합 패턴

#### 1. **Shared Kernel**
- **Organization & Identity ↔ All Contexts**
  - `organization_id`는 모든 Aggregate의 공통 필드
  - 모든 컨텍스트가 Organization을 참조

#### 2. **Customer-Supplier**
- **Workflow Management (Upstream) → Task Execution (Downstream)**
  - Workflow가 Task를 생성하고 관리
  - Task는 Workflow의 요구사항을 따름

- **Task Execution (Upstream) → Browser Automation (Downstream)**
  - Task가 브라우저 액션을 요청
  - Browser Automation은 Task의 명령을 실행

#### 3. **Conformist**
- **All Contexts → Artifact Storage**
  - 모든 컨텍스트가 Artifact Storage의 인터페이스를 따름
  - 저장소 구현 변경에 유연하게 대응

#### 4. **Separate Ways**
- **Script Generation Context**
  - 독립적으로 작동
  - 다른 컨텍스트와 느슨하게 결합
  - Workflow Run 데이터를 읽기만 함

#### 5. **Published Language**
- **Webhook Events**
  - 표준화된 이벤트 페이로드
  - 외부 시스템과의 통합

---

## Ubiquitous Language (보편 언어)

Skyvern 팀과 도메인 전문가가 공통으로 사용하는 용어:

### Core Terms

| 용어 | 정의 | 예시 |
|------|------|------|
| **Workflow** | 재사용 가능한 자동화 프로세스 정의 | "로그인 후 데이터 추출 워크플로우" |
| **Block** | 워크플로우의 단일 실행 단위 | NavigationBlock, ActionBlock, ExtractionBlock |
| **Run** | 워크플로우의 특정 실행 인스턴스 | workflow_run_id로 식별 |
| **Task** | 단일 목표 지향 자동화 작업 | "회사 경력 페이지에서 채용 정보 추출" |
| **Step** | Task 실행 중 하나의 사고-행동 사이클 | LLM이 분석하고 액션을 결정하는 단위 |
| **Action** | 브라우저에서 실행되는 구체적인 조작 | CLICK, INPUT_TEXT, SELECT_OPTION |
| **Thought** | LLM의 추론 및 의사결정 과정 | "로그인 버튼을 찾아야 함" |
| **Element** | 웹 페이지의 상호작용 가능한 요소 | Button, Input, Select |
| **Extraction** | 웹 페이지에서 구조화된 데이터 추출 | JSON 스키마 기반 데이터 추출 |
| **Credential** | 인증에 필요한 자격증명 | Username/Password, API Key, OAuth Token |
| **Vault** | 자격증명의 안전한 저장소 | Bitwarden, 1Password, Azure Key Vault |
| **TOTP** | 시간 기반 일회용 비밀번호 | 2FA 인증 코드 |
| **Artifact** | 실행 중 생성된 결과물 | Screenshot, HTML, Recording |
| **Organization** | 멀티테넌트 격리 단위 | 회사, 팀 단위 |
| **Browser Session** | 영속적 브라우저 상태 | 로그인 상태 유지 |
| **Proxy Location** | 지리적 프록시 위치 | US, EU, ASIA |
| **Parameter** | 워크플로우 입력/출력 변수 | `${workflow.parameters.url}` |
| **Script** | AI가 생성한 재사용 가능 코드 | 워크플로우를 Python 코드로 변환 |

### Process Terms

| 용어 | 정의 |
|------|------|
| **Orchestration** | 여러 블록의 실행을 조율하는 프로세스 |
| **Iteration** | Task가 목표 달성까지 반복 실행하는 과정 |
| **Resolution** | 파라미터나 자격증명을 구체적 값으로 변환 |
| **Scraping** | 웹 페이지의 DOM을 분석하여 요소 추출 |
| **Vision Analysis** | LLM을 사용한 스크린샷 분석 |
| **Persistence** | 브라우저 세션 상태를 유지하는 것 |
| **Termination** | 조건에 따라 실행을 중단하는 것 |
| **Validation** | 실행 결과의 정확성 검증 |

### Technical Terms

| 용어 | 정의 |
|------|------|
| **Block Chain** | 순차적으로 연결된 블록들 |
| **Parameter Binding** | 블록 간 데이터 흐름 연결 |
| **Retry Logic** | 실패 시 재시도 메커니즘 |
| **Soft Delete** | deleted_at 타임스탬프를 사용한 논리 삭제 |
| **Row-level Security** | organization_id 기반 데이터 격리 |
| **Jinja Templating** | 파라미터 참조 문법 (${...}) |
| **Vision LLM** | 스크린샷 분석 가능한 LLM (GPT-4V, Claude) |

---

## 도메인 이벤트 (Domain Events)

### 주요 이벤트

#### Workflow Events
- `WorkflowCreated`
- `WorkflowPublished`
- `WorkflowRunQueued`
- `WorkflowRunStarted`
- `WorkflowRunCompleted`
- `WorkflowRunFailed`
- `BlockExecutionStarted`
- `BlockExecutionCompleted`
- `BlockExecutionFailed`

#### Task Events
- `TaskCreated`
- `TaskQueued`
- `TaskStarted`
- `TaskCompleted`
- `TaskFailed`
- `StepStarted`
- `StepCompleted`
- `ActionExecuted`

#### Credential Events
- `CredentialCreated`
- `CredentialUpdated`
- `CredentialDeleted`
- `TOTPCodeGenerated`

#### Browser Events
- `BrowserSessionCreated`
- `BrowserSessionClosed`
- `PageNavigated`
- `ScreenshotCaptured`

### 이벤트 처리

**Webhook 디스패치:**
```python
# WorkflowRunCompleted 이벤트 발생 시
if workflow.webhook_callback_url:
    await webhook_service.dispatch_webhook(
        url=workflow.webhook_callback_url,
        payload={
            "event": "workflow_run_completed",
            "workflow_run_id": run.workflow_run_id,
            "status": run.status,
            "output": run.output
        }
    )
```

**아티팩트 생성:**
```python
# ScreenshotCaptured 이벤트
await artifact_repository.create_artifact(
    artifact_type=ArtifactType.SCREENSHOT_LLM,
    workflow_run_id=run_id,
    task_id=task_id,
    step_id=step_id,
    uri=s3_uri
)
```

---

## 전략적 설계 권장사항

### 1. **Bounded Context 강화**
현재 코드베이스는 여러 컨텍스트가 혼재되어 있습니다. 다음과 같은 구조 개선을 고려할 수 있습니다:

```
skyvern/
├── domain/
│   ├── workflow/           # Workflow Management Context
│   │   ├── models/
│   │   ├── services/
│   │   ├── repositories/
│   │   └── events/
│   ├── task/              # Task Execution Context
│   │   ├── models/
│   │   ├── services/
│   │   └── repositories/
│   ├── credential/        # Credential Management Context
│   └── organization/      # Organization & Identity Context
├── infrastructure/
│   ├── browser/           # Browser Automation (Supporting)
│   ├── storage/           # Artifact Storage
│   └── llm/              # LLM Integration
└── application/
    ├── api/              # API Layer
    └── cli/              # CLI Interface
```

### 2. **Aggregate 경계 명확화**
- WorkflowRun과 Task의 관계를 더 명확히 할 필요
- Credential과 Organization의 라이프사이클 분리

### 3. **Domain Service 추출**
많은 비즈니스 로직이 Application Service에 있음. 다음을 Domain Service로 추출:
- Parameter Resolution Logic
- Block Execution Logic
- Credential Injection Logic

### 4. **Value Object 활용 증대**
현재 많은 필드가 primitive type (string, int). 다음을 Value Object로:
- `WorkflowStatus`, `TaskStatus` (이미 Enum으로 존재)
- `URL` (validation 로직 포함)
- `ProxyLocation` (지역별 설정 포함)
- `BlockLabel` (naming convention 검증)

### 5. **Repository 인터페이스 명시화**
현재 `AgentDB` 클래스가 모든 역할을 담당. 다음과 같이 분리:
- `IWorkflowRepository`
- `ITaskRepository`
- `ICredentialRepository`
- 각각의 구현체 (`SqlAlchemyWorkflowRepository` 등)

---

## 결론

Skyvern 프로젝트는 복잡한 도메인을 다루고 있으며, 다음과 같은 명확한 Bounded Context를 가지고 있습니다:

**Core Domains (핵심 도메인):**
1. Task Execution Context - LLM 기반 웹 자동화
2. Workflow Management Context - 복잡한 프로세스 오케스트레이션
3. Organization & Identity Context - 멀티테넌시

**Supporting Domains (지원 도메인):**
4. Browser Automation Context
5. Credential Management Context
6. Script Generation Context

**Generic Subdomains (일반 서브도메인):**
7. Artifact Storage Context
8. Content Organization Context

각 도메인은 명확한 Aggregate, Entity, Value Object, Service를 가지고 있으며, Repository 패턴을 통해 영속성을 관리합니다.

프로젝트의 전략적 가치는 **LLM 기반 자동화**와 **블록 기반 워크플로우 조합**에 있으며, 이는 Core Domain인 Task Execution과 Workflow Management에서 구현됩니다.
