# Skyvern Task 실행 엔진 가이드

## 목차
1. [개요](#개요)
2. [RunEngine vs RunType](#runengine-vs-runtype)
3. [엔진 상세 설명](#엔진-상세-설명)
4. [엔진 비교표](#엔진-비교표)
5. [사용 가이드](#사용-가이드)
6. [성능 및 비용](#성능-및-비용)

---

## 개요

Skyvern은 다양한 실행 엔진을 제공하여 **서로 다른 방식**으로 웹 자동화 작업을 수행할 수 있습니다.

### 엔진 종류

```python
class RunEngine(StrEnum):
    skyvern_v1 = "skyvern-1.0"      # Skyvern 오리지널 엔진
    skyvern_v2 = "skyvern-2.0"      # Skyvern 차세대 엔진
    openai_cua = "openai-cua"       # OpenAI Computer Use Agent
    anthropic_cua = "anthropic-cua" # Anthropic Computer Use Agent
    ui_tars = "ui-tars"             # UI-TARS 엔진
```

---

## RunEngine vs RunType

### RunEngine (실행 방식)
어떤 **알고리즘/접근 방식**으로 태스크를 실행할지 결정

### RunType (실행 타입)
어떤 **카테고리**의 실행인지 분류 (메트릭, 로깅 용도)

```python
class RunType(StrEnum):
    task_v1 = "task_v1"           # Task v1 API 사용
    task_v2 = "task_v2"           # Task v2 API 사용
    workflow_run = "workflow_run" # Workflow 실행
    openai_cua = "openai_cua"     # OpenAI CUA 방식
    anthropic_cua = "anthropic_cua" # Anthropic CUA 방식
    ui_tars = "ui_tars"           # UI-TARS 방식
```

**관계:**
- `RunEngine.skyvern_v1` → `RunType.task_v1`
- `RunEngine.skyvern_v2` → `RunType.task_v2`
- `RunEngine.openai_cua` → `RunType.openai_cua`
- `RunEngine.anthropic_cua` → `RunType.anthropic_cua`
- `RunEngine.ui_tars` → `RunType.ui_tars`

---

## 엔진 상세 설명

## 1. Skyvern v1 (skyvern-1.0) 🏆

### 특징
**Skyvern의 오리지널 엔진** - 검증된 안정성

### 작동 방식

```
1. 스크린샷 캡처
   ↓
2. DOM 스크래핑 (모든 상호작용 가능한 요소 추출)
   ↓
3. LLM에게 전달
   - 스크린샷 (Vision LLM)
   - DOM 트리 (텍스트)
   - 현재 목표
   ↓
4. LLM이 다음 액션 결정
   - 여러 액션을 한 번에 계획 가능
   - 액션: [Click, Input, Select, Upload, etc.]
   ↓
5. 모든 액션 실행
   ↓
6. 결과 검증 및 목표 달성 여부 확인
   ↓
7. 목표 미달성 시 반복 (최대 max_steps까지)
```

### 장점
- ✅ **배치 실행**: 한 스텝에서 여러 액션 동시 계획
- ✅ **빠른 속도**: 액션을 한꺼번에 실행
- ✅ **검증된 안정성**: 가장 오래 사용된 엔진
- ✅ **비용 효율적**: LLM 호출 횟수 적음

### 단점
- ❌ **복잡한 인터랙션 제한**: 단계별 피드백 부족
- ❌ **에러 복구 느림**: 여러 액션 중 하나가 실패하면 전체 재시도

### 사용 예시

```python
# REST API
curl -X POST https://api.skyvern.com/v1/tasks \
  -d '{
    "url": "https://example.com",
    "goal": "Fill out the contact form",
    "engine": "skyvern-1.0"
  }'

# Python SDK
task = skyvern.task(
    url="https://example.com",
    goal="Fill out the contact form",
    engine=RunEngine.skyvern_v1  # 기본값
)
result = task.execute()
```

### 적합한 경우
- 📋 **간단한 폼 작성**
- 🔍 **데이터 추출**
- ⚡ **빠른 실행이 필요한 경우**
- 💰 **비용 절감이 중요한 경우**

---

## 2. Skyvern v2 (skyvern-2.0) 🚀

### 특징
**차세대 엔진** - 반복적 계획 및 실행

### 작동 방식

```
1. 큰 목표를 여러 미니 목표로 분해
   ↓
2. 각 미니 목표마다:
   a. 현재 상태 분석 (스크린샷 + DOM)
   b. 다음 액션 계획
   c. 액션 실행
   d. 결과 확인
   e. 목표 달성 여부 판단
   ↓
3. 다음 미니 목표로 진행
   ↓
4. 모든 미니 목표 완료 시 종료
```

### 핵심 개념: Thought (사고 과정)

```python
class ThoughtType(StrEnum):
    plan = "plan"           # 계획 수립
    execute = "execute"     # 액션 실행
    extract = "extract"     # 데이터 추출
    validate = "validate"   # 검증
```

**Thought 기록:**
```python
ThoughtModel:
  - observer_thought_id
  - observer_cruise_id (task_v2_id)
  - user_input: 사용자 요청
  - observation: 현재 페이지 관찰
  - thought: LLM의 추론 과정
  - answer: 실행 결과
  - thought_type: plan/execute/extract/validate
```

### 장점
- ✅ **복잡한 작업 처리**: 다단계 인터랙션
- ✅ **반복적 개선**: 각 단계마다 피드백
- ✅ **명확한 추론**: Thought 기록으로 디버깅 용이
- ✅ **목표 지향적**: 미니 목표 기반 진행

### 단점
- ❌ **느린 속도**: 단계별 LLM 호출
- ❌ **높은 비용**: LLM 호출 횟수 증가
- ❌ **복잡한 구조**: 디버깅 시 많은 Thought 추적 필요

### 사용 예시

```python
# REST API
curl -X POST https://api.skyvern.com/v2/tasks \
  -d '{
    "url": "https://example.com",
    "prompt": "Navigate through the multi-step checkout process",
    "engine": "skyvern-2.0"
  }'

# Python SDK
from skyvern.schemas.runs import RunEngine

task = skyvern.task_v2(
    url="https://example.com",
    prompt="Navigate through the multi-step checkout process",
    engine=RunEngine.skyvern_v2
)
result = task.execute()

# Thought 확인
thoughts = skyvern.get_task_thoughts(task.task_id)
for thought in thoughts:
    print(f"{thought.thought_type}: {thought.thought}")
```

### 적합한 경우
- 🎯 **복잡한 다단계 프로세스**
- 🔄 **동적 상황 대응** (페이지가 계속 변하는 경우)
- 🧪 **실험적 작업** (새로운 사이트 탐색)
- 📊 **상세한 로깅 필요**

---

## 3. OpenAI CUA (openai-cua) 🤖

### 특징
**OpenAI의 Computer Use Agent** - GPT-4V 기반

### 작동 방식

```
Computer Use Agent (CUA) 패러다임:
1. 스크린샷만 사용 (DOM 불필요)
   ↓
2. Vision LLM이 화면을 보고 액션 결정
   - 한 번에 한 액션만
   ↓
3. 액션 실행
   ↓
4. 다시 스크린샷
   ↓
5. 반복 (step-by-step)
```

### OpenAI Tool Calls

```python
# OpenAI가 반환하는 Tool Call 형식
{
  "tool_calls": [
    {
      "type": "function",
      "function": {
        "name": "click",
        "arguments": {"x": 100, "y": 200}
      }
    }
  ]
}
```

### 장점
- ✅ **단순한 접근**: DOM 불필요
- ✅ **Step-by-step 실행**: 각 액션마다 피드백
- ✅ **OpenAI 최신 모델**: GPT-4V의 강력한 Vision 능력

### 단점
- ❌ **매우 느림**: 액션마다 LLM 호출
- ❌ **매우 비쌈**: OpenAI API 비용
- ❌ **완료 검증 없음**: CUA는 자체 완료 판단 없음

### 사용 예시

```python
# REST API
curl -X POST https://api.skyvern.com/v1/tasks \
  -d '{
    "url": "https://example.com",
    "goal": "Click the login button",
    "engine": "openai-cua"
  }'

# Python SDK
task = skyvern.task(
    url="https://example.com",
    goal="Click the login button",
    engine=RunEngine.openai_cua
)
result = task.execute()
```

### 적합한 경우
- 🖼️ **시각적으로 복잡한 UI**
- 🎨 **이미지 기반 인터페이스**
- 🔬 **OpenAI 모델 테스트**

---

## 4. Anthropic CUA (anthropic-cua) 🧠

### 특징
**Anthropic의 Computer Use Agent** - Claude 기반

### 작동 방식

OpenAI CUA와 유사하지만 **Anthropic Claude** 모델 사용

```
1. 스크린샷 캡처
   ↓
2. Claude Vision에게 전달
   ↓
3. Claude가 Tool Call로 액션 반환
   ↓
4. 액션 실행 (한 번에 하나)
   ↓
5. 반복
```

### Anthropic Tool Calls

```python
# Claude가 반환하는 Tool Call 형식
{
  "content": [
    {
      "type": "tool_use",
      "name": "computer",
      "input": {
        "action": "left_click",
        "coordinate": [100, 200]
      }
    }
  ]
}
```

### 장점
- ✅ **Claude의 추론 능력**: Anthropic의 강력한 모델
- ✅ **더 긴 컨텍스트**: Claude의 큰 컨텍스트 윈도우
- ✅ **Step-by-step 실행**: 세밀한 제어

### 단점
- ❌ **느린 속도**: 단계별 실행
- ❌ **높은 비용**: Claude API 비용
- ❌ **완료 검증 없음**: 자체 완료 판단 없음

### 사용 예시

```python
task = skyvern.task(
    url="https://example.com",
    goal="Navigate and extract data",
    engine=RunEngine.anthropic_cua
)
result = task.execute()
```

### 적합한 경우
- 🧠 **복잡한 추론 필요**
- 📝 **긴 컨텍스트 처리**
- 🔬 **Claude 모델 선호**

---

## 5. UI-TARS (ui-tars) 🎯

### 특징
**UI-TARS 연구 프로젝트** 기반 엔진

### 작동 방식

UI-TARS (UI Task Automation via Reinforcement Learning and Scaling) 논문 구현

```
1. 스크린샷 기반 액션 예측
   ↓
2. 강화학습 기반 정책
   ↓
3. 액션 실행
```

### 장점
- ✅ **연구 기반**: 최신 연구 결과 적용
- ✅ **특화된 접근**: UI 자동화에 최적화

### 단점
- ❌ **실험적**: 안정성 검증 필요
- ❌ **제한적 지원**: 특정 케이스에만 적합

### 사용 예시

```python
task = skyvern.task(
    url="https://example.com",
    goal="Automated UI testing",
    engine=RunEngine.ui_tars
)
result = task.execute()
```

---

## 엔진 비교표

| 특성 | Skyvern v1 | Skyvern v2 | OpenAI CUA | Anthropic CUA | UI-TARS |
|------|-----------|-----------|------------|---------------|---------|
| **속도** | ⚡⚡⚡ 빠름 | ⚡⚡ 보통 | ⚡ 느림 | ⚡ 느림 | ⚡⚡ 보통 |
| **비용** | 💰 저렴 | 💰💰 보통 | 💰💰💰 비쌈 | 💰💰💰 비쌈 | 💰💰 보통 |
| **복잡도 처리** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **안정성** | ✅ 높음 | ✅ 높음 | ⚠️ 보통 | ⚠️ 보통 | ⚠️ 실험적 |
| **배치 액션** | ✅ 가능 | ❌ 불가 | ❌ 불가 | ❌ 불가 | ❌ 불가 |
| **DOM 사용** | ✅ 사용 | ✅ 사용 | ❌ 미사용 | ❌ 미사용 | ❌ 미사용 |
| **완료 검증** | ✅ 자동 | ✅ 자동 | ❌ 없음 | ❌ 없음 | ✅ 자동 |
| **Thought 기록** | ❌ | ✅ | ❌ | ❌ | ❌ |
| **추천 용도** | 일반 작업 | 복잡한 작업 | 시각 중심 | 추론 중심 | 연구/실험 |

---

## 사용 가이드

### 엔진 선택 기준

#### Skyvern v1 선택 시 ✅
```python
- 간단한 폼 작성
- 데이터 추출
- 빠른 실행 필요
- 비용 절감 중요
- 검증된 안정성 필요

# 예시
task = skyvern.task(
    url="https://form.example.com",
    goal="Fill out contact form with name and email",
    engine=RunEngine.skyvern_v1
)
```

#### Skyvern v2 선택 시 ✅
```python
- 복잡한 다단계 프로세스
- 동적인 페이지 변화 대응
- 상세한 실행 로그 필요
- 명확한 추론 과정 추적

# 예시
task = skyvern.task_v2(
    url="https://checkout.example.com",
    prompt="Complete the entire checkout process including shipping and payment",
    engine=RunEngine.skyvern_v2
)
```

#### OpenAI CUA 선택 시 ✅
```python
- 이미지 기반 UI
- OpenAI 모델 선호
- DOM 접근 어려운 경우

# 예시
task = skyvern.task(
    url="https://canvas-app.example.com",
    goal="Draw a circle on the canvas",
    engine=RunEngine.openai_cua
)
```

#### Anthropic CUA 선택 시 ✅
```python
- 복잡한 추론 필요
- 긴 컨텍스트 처리
- Claude 모델 선호

# 예시
task = skyvern.task(
    url="https://complex-app.example.com",
    goal="Analyze and summarize the entire dashboard",
    engine=RunEngine.anthropic_cua
)
```

---

## API 사용법

### Task v1 API (Skyvern v1 전용)

```python
# POST /v1/tasks
{
  "url": "https://example.com",
  "goal": "Fill out the form",
  "navigation_goal": "Navigate to the contact page",
  "data_extraction_goal": "Extract contact information",
  "engine": "skyvern-1.0"  # 선택적 (기본값)
}
```

### Task v2 API (모든 엔진 지원)

```python
# POST /v2/tasks
{
  "url": "https://example.com",
  "prompt": "Complete the entire registration process",
  "engine": "skyvern-2.0",
  "max_steps": 50,
  "data_extraction_schema": {...}
}
```

### Workflow에서 엔진 지정

```python
# TaskBlock에서 엔진 지정
{
  "block_type": "task",
  "label": "registration_task",
  "url": "https://example.com/register",
  "goal": "Register a new account",
  "engine": "skyvern-2.0"  # 블록별로 다른 엔진 사용 가능
}
```

---

## 성능 및 비용

### 실행 시간 비교 (예시: 폼 작성)

```
Task: 5개 필드 폼 작성

Skyvern v1:
  - 스텝: 1-2개
  - LLM 호출: 2-3회
  - 실행 시간: ~10초
  - 비용: $0.02

Skyvern v2:
  - 스텝: 5-10개
  - LLM 호출: 10-15회
  - 실행 시간: ~30초
  - 비용: $0.08

OpenAI CUA:
  - 스텝: 10-15개
  - LLM 호출: 15-20회
  - 실행 시간: ~60초
  - 비용: $0.30

Anthropic CUA:
  - 스텝: 10-15개
  - LLM 호출: 15-20회
  - 실행 시간: ~60초
  - 비용: $0.25
```

### 비용 최적화 팁

1. **간단한 작업은 v1 사용**
```python
if task_complexity == "simple":
    engine = RunEngine.skyvern_v1
```

2. **복잡한 작업만 v2 사용**
```python
if task_complexity == "complex":
    engine = RunEngine.skyvern_v2
```

3. **CUA는 특수 케이스에만**
```python
if requires_vision_only:
    engine = RunEngine.anthropic_cua
```

4. **max_steps 제한**
```python
task = skyvern.task(
    ...,
    max_steps=10,  # 비용 폭증 방지
)
```

---

## 실전 예시

### 예시 1: E-commerce 상품 검색

```python
# Skyvern v1으로 충분
task = skyvern.task(
    url="https://amazon.com",
    goal="Search for 'wireless mouse' and extract top 5 products",
    extraction_schema={
        "products": [{
            "name": "string",
            "price": "string",
            "url": "string"
        }]
    },
    engine=RunEngine.skyvern_v1  # 빠르고 저렴
)
```

### 예시 2: 복잡한 주문 프로세스

```python
# Skyvern v2 권장
task = skyvern.task_v2(
    url="https://store.example.com",
    prompt="""
    1. Add product to cart
    2. Go to checkout
    3. Fill in shipping address
    4. Select shipping method
    5. Enter payment details
    6. Review and confirm order
    """,
    engine=RunEngine.skyvern_v2  # 다단계 처리
)
```

### 예시 3: Canvas 기반 애플리케이션

```python
# OpenAI CUA 필요
task = skyvern.task(
    url="https://canvas-editor.example.com",
    goal="Draw a red circle and add text 'Hello'",
    engine=RunEngine.openai_cua  # 시각적 이해 필요
)
```

---

## 디버깅 및 모니터링

### Skyvern v1 디버깅

```python
# Step 로그 확인
steps = skyvern.get_task_steps(task.task_id)
for step in steps:
    print(f"Step {step.order}:")
    print(f"  Status: {step.status}")
    print(f"  Actions: {len(step.output.action_results)}")
```

### Skyvern v2 디버깅

```python
# Thought 추적
thoughts = skyvern.get_task_thoughts(task.task_id)
for thought in thoughts:
    print(f"{thought.thought_type}:")
    print(f"  Observation: {thought.observation}")
    print(f"  Thought: {thought.thought}")
    print(f"  Answer: {thought.answer}")
```

### CUA 디버깅

```python
# Tool Call 확인
steps = skyvern.get_task_steps(task.task_id)
for step in steps:
    # CUA는 각 스텝마다 하나의 액션만
    assert len(step.output.action_results) == 1
    action = step.output.action_results[0]
    print(f"Action: {action.action_type}")
```

---

## FAQ

### Q: 어떤 엔진을 기본으로 사용해야 하나요?
**A:** 대부분의 경우 **Skyvern v1**으로 시작하세요. 작동하지 않거나 복잡한 경우에만 v2를 사용하세요.

### Q: CUA 엔진은 언제 사용하나요?
**A:** DOM 접근이 어렵거나 순수하게 시각적 판단이 필요한 경우에만 사용하세요. 비용과 속도를 고려하면 일반적으로 권장하지 않습니다.

### Q: Workflow에서 블록마다 다른 엔진을 사용할 수 있나요?
**A:** 네! TaskBlock마다 다른 엔진을 지정할 수 있습니다.

```python
workflow = {
    "blocks": [
        {
            "block_type": "task",
            "label": "simple_task",
            "engine": "skyvern-1.0"  # 간단한 작업
        },
        {
            "block_type": "task",
            "label": "complex_task",
            "engine": "skyvern-2.0"  # 복잡한 작업
        }
    ]
}
```

### Q: 엔진 간 전환이 가능한가요?
**A:** 새로운 task를 생성할 때만 엔진을 지정할 수 있습니다. 실행 중 변경은 불가능합니다.

### Q: v2가 v1보다 항상 좋나요?
**A:** 아니요. v2는 복잡한 작업에는 더 좋지만, 간단한 작업에서는 v1이 더 빠르고 저렴합니다.

---

## 요약

| 엔진 | 추천 용도 | 속도 | 비용 |
|------|----------|------|------|
| **Skyvern v1** | 일반적인 작업, 폼 작성, 데이터 추출 | ⚡⚡⚡ | 💰 |
| **Skyvern v2** | 복잡한 다단계 프로세스, 동적 페이지 | ⚡⚡ | 💰💰 |
| **OpenAI CUA** | 시각 중심 UI, 이미지 기반 앱 | ⚡ | 💰💰💰 |
| **Anthropic CUA** | 복잡한 추론, 긴 컨텍스트 | ⚡ | 💰💰💰 |
| **UI-TARS** | 연구/실험적 작업 | ⚡⚡ | 💰💰 |

**권장 사항:**
1. 기본적으로 **Skyvern v1** 사용
2. 복잡하면 **Skyvern v2** 시도
3. 특수한 경우에만 **CUA** 고려
4. 비용과 속도를 항상 모니터링

---

**Happy Automating! 🚀**
