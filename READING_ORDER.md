# Skyvern 코드베이스 읽기 순서 가이드

## 개요

이 문서는 Skyvern 코드베이스를 가장 효율적으로 이해하기 위한 **구체적인 파일 읽기 순서**를 제시합니다. 각 단계에서 무엇을 주목해야 하는지 명확히 안내합니다.

---

## 📚 학습 접근 방법

### 원칙
1. **Top-down 접근**: 전체 구조 → 세부 구현
2. **Data flow 추적**: 요청이 시스템을 어떻게 흐르는지 따라가기
3. **Use case 중심**: 실제 사용 시나리오 기반 학습
4. **점진적 깊이**: 첫 읽기에서는 흐름만, 두 번째에서 세부사항

---

## 🎯 Phase 1: 시스템 전체 구조 파악 (30분)

### 목표
전체 시스템이 어떻게 구성되어 있는지 큰 그림 이해

### 읽기 순서

#### 1.1 프로젝트 개요 (5분)
```
📄 README.md
📄 CLAUDE.md
```
**주목할 점:**
- Skyvern이 무엇을 하는지
- 핵심 기능 3가지
- 개발 명령어

#### 1.2 설정과 상수 (10분)
```
📄 skyvern/config.py (처음 100줄만)
📄 skyvern/constants.py
```
**주목할 점:**
- `Settings` 클래스 구조
- LLM 관련 설정 (`LLM_KEY`, `SECONDARY_LLM_KEY`)
- 브라우저 관련 설정 (`MAX_STEPS_PER_RUN`, `BROWSER_TYPE`)
- 멀티테넌시 설정

#### 1.3 데이터베이스 스키마 (15분)
```
📄 skyvern/forge/sdk/db/models.py (스크롤하며 훑기)
```
**주목할 점:**
- 어떤 테이블들이 있는지 (이름만)
- `organization_id`가 거의 모든 테이블에 있음 (멀티테넌시)
- 주요 관계: Workflow → WorkflowRun → WorkflowRunBlock
- Task → Step → Action

**체크리스트:**
- [ ] Workflow와 WorkflowRun의 차이 이해
- [ ] Task와 Step의 관계 이해
- [ ] organization_id의 역할 이해

---

## 🚀 Phase 2: 단순한 Task 실행 흐름 따라가기 (1시간)

### 목표
가장 단순한 시나리오(Task 하나 실행)의 전체 흐름 이해

### 시나리오
"사용자가 'google.com에 가서 Skyvern 검색' Task 생성 → 실행 → 결과 반환"

### 읽기 순서

#### 2.1 API 진입점 (10분)
```
📄 skyvern/forge/api_app.py
```
**주목할 점:**
- FastAPI 앱 생성 (`create_app()`)
- 라우터 등록 (`/api/v1`, `/api/v2`)
- 미들웨어 설정
- Lifespan 이벤트 (startup/shutdown)

**다음 파일로 이동:**
```
📄 skyvern/forge/sdk/routes/agent_protocol.py
   → `POST /v1/tasks` 엔드포인트 찾기
```

#### 2.2 Task 생성 API (15분)
```
📄 skyvern/forge/sdk/routes/agent_protocol.py
   함수: create_agent_task()
```
**주목할 점:**
- Request 모델: `TaskRequest`
- Validation 로직
- `AgentDB.create_task()` 호출
- Response 반환

**흐름:**
```
HTTP Request → Pydantic 검증 → DB 저장 → Task 객체 반환
```

#### 2.3 Task 실행 서비스 (20분)
```
📄 skyvern/services/task_v2_service.py
   클래스: TaskV2Service
   핵심 메서드: execute_task()
```
**주목할 점:**
- Task 상태 전이: CREATED → RUNNING → COMPLETED
- `_execute_step()` 호출 (반복)
- LLM 호출 부분
- 에러 처리

**핵심 로직:**
```python
while not task_completed and step_count < max_steps:
    1. 스크린샷 캡처
    2. LLM에게 현재 상태 + 목표 전달
    3. LLM이 다음 액션 결정
    4. 액션 실행
    5. 결과 확인
    6. 목표 달성 여부 검증
```

#### 2.4 브라우저 제어 (15분)
```
📄 skyvern/webeye/browser_manager.py
   클래스: BrowserManager
   메서드: get_or_create_page()
```
**주목할 점:**
- Playwright 초기화
- 브라우저 컨텍스트 관리
- 페이지 생성 및 재사용

```
📄 skyvern/webeye/scraper/scraper.py
   함수: scrape_website()
```
**주목할 점:**
- DOM 트리 분석
- 상호작용 가능한 요소 식별
- Element ID 할당

**체크리스트:**
- [ ] Task가 어떻게 Step으로 나뉘는지 이해
- [ ] LLM이 어디서 호출되는지 확인
- [ ] 브라우저 액션이 어떻게 실행되는지 이해

---

## 🔄 Phase 3: Workflow 시스템 이해하기 (1.5시간)

### 목표
복잡한 다단계 워크플로우가 어떻게 실행되는지 이해

### 시나리오
"NavigationBlock → ActionBlock → ExtractionBlock 순서의 워크플로우 실행"

### 읽기 순서

#### 3.1 Workflow 데이터 구조 (20분)
```
📄 skyvern/schemas/workflows.py
```
**주목할 점:**
- `BlockType` enum (모든 블록 타입)
- `BlockStatus` enum
- `BlockResult` 구조

```
📄 skyvern/forge/sdk/workflow/models/workflow.py
   클래스: Workflow, WorkflowDefinition
```
**주목할 점:**
- `blocks` 리스트
- `parameters` 정의
- 블록 간 의존성

#### 3.2 블록 시스템 (30분)
```
📄 skyvern/forge/sdk/workflow/models/block.py (처음 500줄)
```
**읽는 순서:**
1. `Block` 베이스 클래스 (line ~121)
   - 모든 블록의 공통 속성
   - `output_parameter`
   - `continue_on_failure`

2. 구체적인 블록 클래스들:
   ```python
   NavigationBlock     # URL로 이동
   ActionBlock         # LLM 기반 상호작용
   ExtractionBlock     # 데이터 추출
   ValidationBlock     # 결과 검증
   LoopBlock          # 반복 실행
   ```

**각 블록에서 주목할 점:**
- `execute()` 메서드 시그니처
- 필수 파라미터
- 출력 형식

#### 3.3 Workflow 실행 서비스 (40분)
```
📄 skyvern/services/workflow_service.py
   클래스: WorkflowService
```
**핵심 메서드 읽기 순서:**

1. `setup_workflow_run()` - 워크플로우 실행 초기화
2. `execute_workflow()` - 전체 오케스트레이션

```
📄 skyvern/services/run_service.py
   클래스: RunService
```
**핵심 메서드:**

1. `initialize_workflow_run()` - RunBlock 생성
2. `execute_workflow_run()` - 메인 실행 루프
3. `_execute_block()` - 개별 블록 실행

**실행 흐름:**
```
1. WorkflowRun 생성 (status=QUEUED)
2. 각 Block에 대해 WorkflowRunBlock 생성
3. 블록 순회:
   for block in blocks:
       - 파라미터 해결 (이전 블록 출력 참조)
       - block.execute() 호출
       - 결과 저장
       - 다음 블록으로 전달
4. 최종 output 집계
5. WorkflowRun 상태 업데이트 (COMPLETED/FAILED)
```

#### 3.4 파라미터 시스템 (30분)
```
📄 skyvern/forge/sdk/workflow/models/parameter.py
```
**주목할 점:**
- `WorkflowParameter` - 입력 파라미터
- `OutputParameter` - 블록 출력
- `ContextParameter` - 런타임 컨텍스트
- 파라미터 타입들

**파라미터 참조 문법:**
```python
# 워크플로우 파라미터
"${workflow.parameters.url}"

# 이전 블록 출력
"${blocks.navigation_block.output.final_url}"

# 컨텍스트
"${context.organization_id}"
```

```
📄 skyvern/forge/sdk/workflow/context_manager.py
   클래스: WorkflowRunContext
```
**주목할 점:**
- 실행 컨텍스트 관리
- 파라미터 해결 로직
- 블록 간 데이터 전달

**체크리스트:**
- [ ] 블록이 어떻게 순차 실행되는지 이해
- [ ] 파라미터가 블록 간 어떻게 전달되는지 이해
- [ ] WorkflowRun과 WorkflowRunBlock의 관계 이해

---

## 🔐 Phase 4: 자격증명 시스템 (45분)

### 목표
민감한 정보가 어떻게 안전하게 관리되는지 이해

### 읽기 순서

#### 4.1 Credential 모델 (10분)
```
📄 skyvern/forge/sdk/db/models.py
   - CredentialModel (line ~867)
   - CredentialParameterModel (line ~474)
   - BitwardenLoginCredentialParameterModel
   - TOTPCodeModel (line ~608)
```

#### 4.2 Credential 서비스 (20분)
```
📄 skyvern/forge/sdk/services/credential/credential_vault_service.py
   클래스: CredentialVaultService
```
**주목할 점:**
- `get_credential()` - 자격증명 조회
- `inject_credentials()` - 워크플로우에 주입

```
📄 skyvern/forge/sdk/services/credential/bitwarden_credential_service.py
```
**주목할 점:**
- Bitwarden SDK 통합
- 자격증명 복호화

#### 4.3 TOTP 생성 (15분)
```
📄 skyvern/services/otp_service.py
```
**주목할 점:**
- `generate_totp_code()` - pyotp 사용
- 이메일에서 OTP 파싱
- 코드 만료 관리

**체크리스트:**
- [ ] Credential이 어떻게 저장되는지 이해
- [ ] TOTP가 어떻게 생성되는지 이해
- [ ] 워크플로우에서 자격증명 사용 방법 이해

---

## 🌐 Phase 5: 브라우저 자동화 깊이 파기 (1시간)

### 목표
Playwright와의 통합 및 DOM 조작 상세 이해

### 읽기 순서

#### 5.1 액션 시스템 (20분)
```
📄 skyvern/webeye/actions/actions.py
```
**액션 클래스들 확인:**
```python
ClickAction          # 클릭
InputAction          # 텍스트 입력
SelectAction         # 드롭다운 선택
UploadFileAction     # 파일 업로드
WaitAction           # 대기
CompleteAction       # 완료 신호
TerminateAction      # 종료
```

**각 액션의 구조:**
- `action_type`: ActionType enum
- `element_id`: 대상 요소 ID
- `reasoning`: LLM의 설명
- `confidence_float`: 확신도

#### 5.2 액션 실행 핸들러 (30분)
```
📄 skyvern/webeye/actions/handler.py
   클래스: ActionHandler
```
**핵심 메서드:**
```python
handle(action, page) → ActionResult
    ├─ _handle_click_action()
    ├─ _handle_input_action()
    ├─ _handle_select_action()
    └─ ...
```

**주목할 점:**
- Playwright API 사용법
- 요소 찾기 로직
- 에러 처리 및 재시도
- 스크린샷 캡처 타이밍

#### 5.3 DOM 스크래핑 (30분)
```
📄 skyvern/webeye/scraper/scraper.py
   함수: scrape_website()
```
**흐름:**
```
1. 페이지 HTML 가져오기
2. BeautifulSoup로 파싱
3. 상호작용 가능한 요소 식별
   - <button>, <input>, <select>, <a> 등
4. 요소별 메타데이터 추출
   - attributes, text, position
5. Element tree 구축
6. Element ID 매핑 생성
```

**주목할 점:**
- `SkyvernElement` 클래스
- Accessibility tree 활용
- 요소 중복 제거 (deduplication)

```
📄 skyvern/webeye/scraper/dom_util.py
```
**유틸리티 함수들:**
- `is_interactable()` - 상호작용 가능 여부
- `get_element_position()` - 요소 위치
- `is_visible()` - 가시성 확인

**체크리스트:**
- [ ] 액션이 어떻게 실행되는지 상세히 이해
- [ ] DOM에서 요소를 어떻게 찾는지 이해
- [ ] LLM이 받는 요소 정보 형식 이해

---

## 🎨 Phase 6: Frontend 구조 (1시간)

### 목표
React UI가 어떻게 백엔드와 통신하는지 이해

### 읽기 순서

#### 6.1 라우팅 구조 (10분)
```
📄 skyvern-frontend/src/router.tsx
📄 skyvern-frontend/src/App.tsx
```
**주목할 점:**
- 주요 라우트들
- 인증 가드
- Layout 구조

#### 6.2 API 클라이언트 (15분)
```
📄 skyvern-frontend/src/api/AxiosClient.ts
```
**주목할 점:**
- Axios 인스턴스 설정
- 인터셉터 (토큰 추가)
- 에러 핸들링

**Generated client:**
```
📄 skyvern-frontend/src/api/generated/ (훑어보기만)
```

#### 6.3 Workflow Editor (20분)
```
📄 skyvern-frontend/src/routes/workflows/editor/FlowRenderer.tsx
```
**주목할 점:**
- React Flow 사용
- 노드(블록) 렌더링
- 연결선(데이터 흐름) 표시

```
📄 skyvern-frontend/src/routes/workflows/editor/nodes/
```
**각 블록 타입의 UI 컴포넌트**

#### 6.4 Task 실행 모니터링 (15분)
```
📄 skyvern-frontend/src/routes/tasks/detail/TaskDetails.tsx
```
**주목할 점:**
- Task 상태 폴링
- Step 목록 표시
- Action 히스토리

```
📄 skyvern-frontend/src/components/BrowserStream.tsx
```
**주목할 점:**
- WebSocket 연결
- 실시간 스크린샷 스트리밍

**체크리스트:**
- [ ] UI에서 워크플로우가 어떻게 생성되는지 이해
- [ ] 실행 모니터링이 어떻게 동작하는지 이해
- [ ] 실시간 스트리밍 메커니즘 이해

---

## 🧩 Phase 7: 고급 기능들 (2시간)

### 7.1 Script 생성 시스템 (30분)
```
📄 skyvern/services/script_service.py
```
**주목할 점:**
- 워크플로우 → 코드 변환
- AI 기반 코드 생성
- 스크립트 캐싱

```
📄 skyvern/services/run_code_service.py
```
**주목할 점:**
- 샌드박스 실행 환경
- 보안 검사
- 코드 실행 결과 반환

#### 7.2 LLM 통합 (30분)
```
📄 skyvern/forge/sdk/core/llm/api_handler_factory.py
```
**주목할 점:**
- 다양한 LLM 프로바이더 지원
- 추상화 레이어

```
📄 skyvern/forge/prompts.py (주요 프롬프트만)
```
**주목할 점:**
- `generate_action_prompt()` - 액션 결정
- `generate_extraction_prompt()` - 데이터 추출
- 프롬프트 구조

#### 7.3 아티팩트 저장 (20분)
```
📄 skyvern/forge/sdk/artifact/storage/
   - base.py (인터페이스)
   - s3.py (S3 구현)
   - local.py (로컬 구현)
```

#### 7.4 Webhook 시스템 (20분)
```
📄 skyvern/services/webhook_service.py
```
**주목할 점:**
- 완료 이벤트 발송
- 재시도 메커니즘
- 실패 처리

#### 7.5 멀티테넌시 (20분)
```
코드 전반에서 organization_id 사용 패턴 확인
```
**주목할 점:**
- 미들웨어에서 조직 식별
- 모든 DB 쿼리에 organization_id 필터
- Row-level 격리

---

## 🔍 Phase 8: 실전 디버깅 (실습)

### 목표
실제로 코드를 실행하며 흐름 추적

### 실습 단계

#### 8.1 로컬 환경 구동
```bash
skyvern quickstart
skyvern run all
```

#### 8.2 간단한 Task 실행
```python
# Python SDK 사용
from skyvern import Skyvern

skyvern = Skyvern()
task = skyvern.task(
    url="https://www.google.com",
    goal="Search for 'Skyvern' and click the first result"
)
result = task.execute()
```

#### 8.3 브레이크포인트 설정
디버거에서 다음 지점들에 브레이크포인트:
1. `skyvern/services/task_v2_service.py:execute_task()`
2. `skyvern/webeye/scraper/scraper.py:scrape_website()`
3. `skyvern/webeye/actions/handler.py:handle()`

#### 8.4 로그 확인
```bash
# Task 실행 로그
tail -f logs/skyvern.log

# 특정 task_id 필터링
grep "task_id=abc123" logs/skyvern.log
```

#### 8.5 데이터베이스 확인
```bash
# PostgreSQL 접속
psql $DATABASE_STRING

# Task 조회
SELECT * FROM tasks WHERE task_id = 'abc123';

# Steps 조회
SELECT * FROM steps WHERE task_id = 'abc123' ORDER BY "order";

# Actions 조회
SELECT * FROM actions WHERE task_id = 'abc123' ORDER BY action_order;
```

---

## 📊 학습 체크리스트

### Phase 1: 시스템 구조 ✅
- [ ] 전체 아키텍처 이해
- [ ] 데이터베이스 스키마 파악
- [ ] 주요 설정 확인

### Phase 2: Task 실행 ✅
- [ ] API 엔드포인트 → 서비스 → DB 흐름
- [ ] Task → Step → Action 관계
- [ ] LLM 호출 지점 확인
- [ ] 브라우저 제어 메커니즘

### Phase 3: Workflow 시스템 ✅
- [ ] 블록 시스템 이해
- [ ] 파라미터 전달 메커니즘
- [ ] 워크플로우 실행 오케스트레이션
- [ ] 에러 처리 및 재시도

### Phase 4: 자격증명 ✅
- [ ] Credential 저장 및 조회
- [ ] TOTP 생성
- [ ] Vault 통합

### Phase 5: 브라우저 자동화 ✅
- [ ] 액션 시스템 상세
- [ ] DOM 스크래핑 로직
- [ ] Playwright 통합

### Phase 6: Frontend ✅
- [ ] 라우팅 구조
- [ ] API 통신
- [ ] Workflow Editor
- [ ] 실시간 모니터링

### Phase 7: 고급 기능 ✅
- [ ] Script 생성
- [ ] LLM 통합
- [ ] 아티팩트 저장
- [ ] Webhook 시스템
- [ ] 멀티테넌시

### Phase 8: 실습 ✅
- [ ] 로컬 환경 구동
- [ ] Task 실행 및 디버깅
- [ ] 로그 분석
- [ ] DB 데이터 확인

---

## 💡 학습 팁

### 1. **한 번에 다 이해하려 하지 마세요**
- 첫 번째 읽기: 흐름만 파악
- 두 번째 읽기: 상세 로직 이해
- 세 번째 읽기: 예외 케이스와 최적화

### 2. **실행하면서 배우세요**
- 코드를 읽기만 하지 말고 실제로 실행
- 브레이크포인트로 단계별 확인
- 로그로 데이터 흐름 추적

### 3. **문서화하세요**
- 이해한 내용을 자신의 언어로 정리
- 흐름도 그리기
- 궁금한 점 메모

### 4. **테스트 코드 참고**
```
📄 tests/unit_tests/
```
- 각 컴포넌트의 사용법을 보여주는 예제

### 5. **커뮤니티 활용**
- GitHub Issues 읽기
- Discord에서 질문
- Pull Request 히스토리 확인

---

## 🎯 Use Case별 읽기 가이드

### "Task API 사용법을 알고 싶어요"
**짧은 경로:**
1. `skyvern/forge/sdk/routes/agent_protocol.py` - API
2. `skyvern/services/task_v2_service.py` - 실행 로직
3. `skyvern/schemas/task_v2.py` - 데이터 모델

### "Workflow를 만들고 싶어요"
**짧은 경로:**
1. `skyvern/schemas/workflows.py` - 블록 타입들
2. `skyvern/forge/sdk/workflow/models/block.py` - 블록 구현
3. `skyvern-frontend/src/routes/workflows/editor/` - UI
4. 예제: `examples/` 폴더

### "자격증명을 워크플로우에서 사용하고 싶어요"
**짧은 경로:**
1. `skyvern/forge/sdk/db/models.py` - Credential 모델
2. `skyvern/forge/sdk/services/credential/` - 서비스들
3. 워크플로우 파라미터에서 credential 타입 사용 예제

### "커스텀 블록을 만들고 싶어요"
**짧은 경로:**
1. `skyvern/forge/sdk/workflow/models/block.py` - 기존 블록 참고
2. `Block` 클래스 상속
3. `execute()` 메서드 구현
4. `BlockType` enum에 등록

### "LLM 프롬프트를 수정하고 싶어요"
**짧은 경로:**
1. `skyvern/forge/prompts.py` - 모든 프롬프트
2. 원하는 프롬프트 함수 찾기
3. 수정 후 테스트

---

## 📈 다음 단계

### 코드베이스 마스터했다면:

1. **기여하기**
   - GitHub Issues에서 "good first issue" 찾기
   - 버그 수정 PR 제출
   - 문서 개선

2. **확장하기**
   - 커스텀 블록 개발
   - 새로운 LLM 통합
   - 새로운 Vault 프로바이더 추가

3. **최적화하기**
   - 성능 병목 지점 찾기
   - 캐싱 개선
   - 병렬 처리 추가

4. **공유하기**
   - 블로그 포스트 작성
   - 튜토리얼 비디오 제작
   - 커뮤니티에서 다른 사람 돕기

---

## 🔗 참고 자료

- **Learning Roadmap**: `LEARNING_ROADMAP.md` - 8주 학습 계획
- **DDD Analysis**: `DDD_ANALYSIS.md` - 도메인 구조 분석
- **Official Docs**: https://docs.skyvern.com
- **GitHub**: https://github.com/skyvern-ai/skyvern
- **Discord**: 커뮤니티 지원

---

## ❓ 자주 묻는 질문

### Q: 모든 파일을 다 읽어야 하나요?
A: 아니요. 위 순서대로 핵심 파일들만 읽어도 충분합니다. 나머지는 필요할 때 참조하세요.

### Q: 얼마나 시간이 걸리나요?
A: Phase 1-5까지는 약 5-7시간, 전체는 10-15시간 정도 소요됩니다.

### Q: Python/TypeScript에 익숙하지 않은데 괜찮나요?
A: 기본 문법만 알면 됩니다. 코드를 읽으면서 배울 수 있습니다.

### Q: 막히면 어떻게 하나요?
A:
1. 로그 확인
2. 디버거로 단계별 실행
3. 테스트 코드 참고
4. Discord/GitHub Issues에 질문

---

**Happy Learning! 🚀**

이 가이드를 따라가면서 Skyvern 코드베이스를 마스터하세요!
