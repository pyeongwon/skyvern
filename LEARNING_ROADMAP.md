# Skyvern Codebase Learning Roadmap

## Introduction

This document provides a comprehensive learning roadmap for understanding the Skyvern codebase. Skyvern is a browser automation platform that uses LLMs and computer vision to interact with websites. This guide will take you from beginner to advanced understanding through a structured learning path.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture at a Glance](#architecture-at-a-glance)
3. [Learning Path](#learning-path)
4. [Core Components Deep Dive](#core-components-deep-dive)
5. [Key Concepts](#key-concepts)
6. [File Reading Order](#file-reading-order)
7. [Hands-On Exercises](#hands-on-exercises)
8. [Advanced Topics](#advanced-topics)

---

## Project Overview

### What is Skyvern?

Skyvern is an LLM-powered browser automation platform that can:
- Navigate websites using natural language goals
- Extract structured data from web pages
- Execute complex multi-step workflows
- Handle dynamic websites without fixed selectors
- Provide visual feedback through screenshot analysis

### Technology Stack

**Backend:**
- Python 3.11+ with async/await (asyncio)
- FastAPI for REST API
- SQLAlchemy AsyncORM for database
- PostgreSQL for data persistence
- Playwright for browser automation
- Pydantic for data validation

**Frontend:**
- React 18 + TypeScript
- Vite build system
- Tailwind CSS + shadcn/ui
- Zustand for state management
- React Router v6

**LLM Integration:**
- OpenAI (GPT-4, GPT-4 Vision)
- Anthropic Claude
- Azure OpenAI
- AWS Bedrock
- Google Vertex AI
- Ollama (local models)

---

## Architecture at a Glance

```
┌─────────────────────────────────────────────────────────────┐
│                     React Frontend (UI)                      │
│              (skyvern-frontend/src/routes/)                  │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP/WebSocket
┌──────────────────────────▼──────────────────────────────────┐
│                   FastAPI Server (Forge)                     │
│          (skyvern/forge/api_app.py, sdk/routes/)            │
│  - REST API endpoints (/v1, /api/v1, /api/v2)              │
│  - WebSocket connections for streaming                      │
│  - Authentication & authorization                           │
└──────────────┬────────────────────────────┬─────────────────┘
               │                            │
    ┌──────────▼─────────┐       ┌──────────▼─────────┐
    │   Agent System     │       │  Workflow Engine   │
    │ (forge/agent.py)   │       │  (forge/sdk/       │
    │ - LLM interactions │       │   workflow/)       │
    │ - Vision analysis  │       │ - Block execution  │
    │ - Action planning  │       │ - Parameter mgmt   │
    │ - Error recovery   │       │ - Context tracking │
    └────────┬───────────┘       └──────────┬────────┘
             │                              │
    ┌────────▼──────────────────────────────▼────────┐
    │      Browser Engine (WebEye)                   │
    │    (skyvern/webeye/)                          │
    │ - Playwright browser automation               │
    │ - DOM scraping & analysis                     │
    │ - Action execution (click, input, select)    │
    │ - Screenshot capture & vision processing    │
    │ - Browser session persistence                │
    └────────┬──────────────────────────────────────┘
             │
    ┌────────▼──────────────────────────────────────┐
    │      Database Layer (PostgreSQL)              │
    │    (forge/sdk/db/)                           │
    │ - Models: Tasks, Workflows, Runs, Steps     │
    │ - ORM: SQLAlchemy AsyncORM                   │
    │ - Migrations: Alembic                        │
    └───────────────────────────────────────────────┘
```

---

## Learning Path

### Phase 1: Foundation (Week 1)

**Goals:**
- Understand the project purpose and capabilities
- Set up local development environment
- Run your first task
- Understand basic configuration

**Tasks:**

1. **Read documentation** (1-2 hours)
   - [ ] `README.md` - Project overview
   - [ ] `CLAUDE.md` - Development commands and architecture
   - [ ] `.env.example` - Configuration options

2. **Environment setup** (2-3 hours)
   - [ ] Install Python 3.11+ and Node.js
   - [ ] Run `skyvern quickstart` for initial setup
   - [ ] Start services: `skyvern run all`
   - [ ] Access UI at http://localhost:8080

3. **First task execution** (1-2 hours)
   - [ ] Create a simple task via UI (e.g., "Navigate to google.com and search for Skyvern")
   - [ ] Watch the browser automation in action
   - [ ] Review the task results and logs
   - [ ] Examine screenshots and action history

4. **Core file review** (2-3 hours)
   - [ ] `skyvern/config.py` - Configuration settings
   - [ ] `skyvern/constants.py` - Global constants
   - [ ] `pyproject.toml` - Dependencies and project setup

**Checkpoint:** You should be able to run Skyvern locally and execute simple tasks.

---

### Phase 2: Core Architecture (Week 2)

**Goals:**
- Understand the main application structure
- Learn how components interact
- Follow a task execution flow
- Understand database models

**Tasks:**

1. **Application initialization** (3-4 hours)
   - [ ] `skyvern/forge/app.py` - Singleton instances (DATABASE, BROWSER_MANAGER, LLM_API_HANDLER)
   - [ ] `skyvern/forge/api_app.py` - FastAPI app setup, middleware, lifespan
   - [ ] Understand initialization order and dependencies

2. **Agent system** (4-5 hours)
   - [ ] `skyvern/forge/agent.py` - ForgeAgent class (main orchestration)
   - [ ] `skyvern/forge/prompts.py` - LLM prompt templates
   - [ ] `skyvern/forge/agent_functions.py` - Helper functions
   - [ ] Trace how a task is picked up and executed

3. **Browser engine basics** (3-4 hours)
   - [ ] `skyvern/webeye/browser_manager.py` - Browser instance management
   - [ ] `skyvern/webeye/browser_factory.py` - Browser creation and configuration
   - [ ] Understand how Playwright is integrated

4. **Database layer** (4-5 hours)
   - [ ] `skyvern/forge/sdk/db/models.py` - All database models
   - [ ] `skyvern/forge/sdk/db/client.py` - Database client setup
   - [ ] `alembic/versions/` - Browse recent migrations
   - [ ] Key models: Task, WorkflowRun, Step, Artifact

5. **Hands-on tracing** (2-3 hours)
   - [ ] Set breakpoints in `agent.py`
   - [ ] Run a task in debug mode
   - [ ] Follow execution from API → Agent → Browser → Database
   - [ ] Examine database entries created during execution

**Checkpoint:** You should understand how a request flows through the system.

---

### Phase 3: API and Services Layer (Week 3)

**Goals:**
- Understand REST API structure
- Learn business logic organization
- Study credential management
- Understand webhooks and async operations

**Tasks:**

1. **API routes** (4-5 hours)
   - [ ] `skyvern/forge/sdk/routes/routers.py` - Route registry
   - [ ] `skyvern/forge/sdk/routes/workflows.py` - Workflow CRUD
   - [ ] `skyvern/forge/sdk/routes/tasks.py` - Task endpoints
   - [ ] `skyvern/forge/sdk/routes/credentials.py` - Credential storage
   - [ ] `skyvern/forge/sdk/routes/browser_sessions.py` - Browser session management
   - [ ] Test endpoints using Postman or curl

2. **Service layer** (5-6 hours)
   - [ ] `skyvern/services/workflow_service.py` - Workflow orchestration
   - [ ] `skyvern/services/run_service.py` - Run execution
   - [ ] `skyvern/services/task_v2_service.py` - Task API v2
   - [ ] `skyvern/services/action_service.py` - Action logging
   - [ ] `skyvern/services/browser_session_service.py` - Session lifecycle

3. **Data schemas** (3-4 hours)
   - [ ] `skyvern/schemas/workflows.py` - Workflow models
   - [ ] `skyvern/schemas/runs.py` - Run models
   - [ ] `skyvern/schemas/steps.py` - Step models
   - [ ] `skyvern/schemas/artifacts.py` - Artifact models
   - [ ] Understand Pydantic validation

4. **Credential system** (2-3 hours)
   - [ ] `skyvern/forge/sdk/services/credential_service.py`
   - [ ] Integration with Bitwarden, 1Password, Azure Key Vault
   - [ ] Understand credential types and usage

5. **Async operations** (2-3 hours)
   - [ ] `skyvern/forge/async_operations.py` - Operation pooling
   - [ ] `skyvern/services/webhook_service.py` - Webhook dispatch
   - [ ] Understand async/await patterns

**Checkpoint:** You should be able to create custom API endpoints and services.

---

### Phase 4: Workflow System (Week 4)

**Goals:**
- Master workflow definitions
- Understand block system
- Learn parameter passing
- Build custom workflows

**Tasks:**

1. **Workflow fundamentals** (4-5 hours)
   - [ ] `skyvern/forge/sdk/workflow/models/workflow.py` - Workflow definition
   - [ ] `skyvern/forge/sdk/workflow/models/parameter.py` - Parameter system
   - [ ] `skyvern/forge/sdk/workflow/service.py` - Workflow service
   - [ ] Study example workflows in the repository

2. **Block system deep dive** (6-7 hours)
   - [ ] `skyvern/forge/sdk/workflow/models/block.py` - All block types
   - [ ] BaseBlock - Common functionality
   - [ ] ActionBlock - LLM-guided browser automation
   - [ ] NavigationBlock - URL navigation
   - [ ] ValidationBlock - Assertion checking
   - [ ] LoopBlock - Iteration control
   - [ ] FileParserBlock - Document parsing
   - [ ] ExtractBlock - Data extraction
   - [ ] ScriptBlock - Code execution

3. **Block execution** (3-4 hours)
   - [ ] `skyvern/services/block_service.py` - Block execution logic
   - [ ] `skyvern/services/run_service.py` - Run orchestration
   - [ ] Understand execution order and dependencies
   - [ ] Study parameter resolution (${blocks.blockA.output})

4. **Workflow builder UI** (2-3 hours)
   - [ ] `skyvern-frontend/src/routes/workflows/editor/` - Visual editor
   - [ ] Explore the drag-and-drop interface
   - [ ] Understand block configuration UI
   - [ ] Study parameter binding UI

5. **Hands-on workflow creation** (3-4 hours)
   - [ ] Create a multi-step workflow via UI
   - [ ] Use NavigationBlock → ActionBlock → ExtractBlock
   - [ ] Test parameter passing between blocks
   - [ ] Debug workflow execution using the debugger
   - [ ] Create a workflow with LoopBlock

**Checkpoint:** You should be able to design and execute complex workflows.

---

### Phase 5: Browser Automation (Week 5)

**Goals:**
- Understand DOM scraping
- Learn action execution
- Study vision processing
- Master browser session management

**Tasks:**

1. **DOM scraping** (4-5 hours)
   - [ ] `skyvern/webeye/scraper/scraper.py` - Main scraper
   - [ ] `skyvern/webeye/scraper/dom_util.py` - DOM utilities
   - [ ] Understand element detection and tree building
   - [ ] Study interactable element identification
   - [ ] Learn how accessibility tree is analyzed

2. **Action system** (5-6 hours)
   - [ ] `skyvern/webeye/actions/actions.py` - Action models
   - [ ] `skyvern/webeye/actions/handler.py` - Action execution
   - [ ] `skyvern/webeye/actions/responses.py` - Action results
   - [ ] Action types:
     - ClickAction - Element interaction
     - InputAction - Text input
     - SelectAction - Dropdown handling
     - UploadAction - File upload
     - ScrollAction - Page navigation
     - WaitAction - Timing control

3. **Vision processing** (3-4 hours)
   - [ ] Screenshot capture mechanism
   - [ ] Element bounding boxes and annotations
   - [ ] Vision LLM integration (GPT-4V, Claude)
   - [ ] Understanding visual context for action planning

4. **Browser session management** (3-4 hours)
   - [ ] `skyvern/webeye/persistent_sessions_manager.py`
   - [ ] Session persistence across workflow steps
   - [ ] Cookie and localStorage handling
   - [ ] Multi-tab and window management

5. **Playwright integration** (2-3 hours)
   - [ ] Study how Playwright API is used
   - [ ] Browser context configuration
   - [ ] Page event handling
   - [ ] Network interception (optional)

**Checkpoint:** You should understand how browser automation works at a low level.

---

### Phase 6: Frontend Development (Week 6)

**Goals:**
- Understand React application structure
- Learn component organization
- Study state management
- Master real-time streaming

**Tasks:**

1. **Frontend structure** (3-4 hours)
   - [ ] `skyvern-frontend/src/App.tsx` - Root component
   - [ ] `skyvern-frontend/src/router.tsx` - Routing setup
   - [ ] `skyvern-frontend/src/main.tsx` - Entry point
   - [ ] Browse `src/routes/` directory structure

2. **Key routes** (5-6 hours)
   - [ ] `src/routes/workflows/` - Workflow management
   - [ ] `src/routes/workflows/editor/` - Workflow builder
   - [ ] `src/routes/workflows/debugger/` - Debugger UI
   - [ ] `src/routes/tasks/` - Task management
   - [ ] `src/routes/runs/` - Run history

3. **API client** (2-3 hours)
   - [ ] `src/api/` - Generated client
   - [ ] API client initialization
   - [ ] Request/response handling
   - [ ] Error handling

4. **State management** (2-3 hours)
   - [ ] `src/store/` - Zustand stores
   - [ ] Study state patterns used
   - [ ] Understand global vs. local state

5. **Real-time streaming** (3-4 hours)
   - [ ] `src/components/BrowserStream.tsx` - Live browser view
   - [ ] WebSocket connection handling
   - [ ] Stream data rendering
   - [ ] Video/screenshot display

6. **UI components** (2-3 hours)
   - [ ] `src/components/ui/` - shadcn/ui primitives
   - [ ] Custom components in `src/components/`
   - [ ] Styling with Tailwind CSS

**Checkpoint:** You should be able to add new UI features and modify existing ones.

---

### Phase 7: Advanced Topics (Weeks 7-8)

**Goals:**
- Master multi-tenancy
- Understand LLM integration
- Learn error recovery strategies
- Study performance optimization

**Tasks:**

1. **Multi-tenancy** (3-4 hours)
   - [ ] Organization model in database
   - [ ] Row-level security (organization_id filtering)
   - [ ] Credential isolation
   - [ ] User authentication and authorization

2. **LLM integration** (5-6 hours)
   - [ ] `skyvern/forge/sdk/core/llm_api_handler.py` - LLM abstraction
   - [ ] Provider-specific implementations
   - [ ] Prompt engineering patterns
   - [ ] Vision model integration
   - [ ] Response parsing and validation
   - [ ] Token usage tracking

3. **Error handling and recovery** (4-5 hours)
   - [ ] `skyvern/errors/` - Error definitions
   - [ ] `skyvern/exceptions.py` - HTTP exceptions
   - [ ] Retry mechanisms with exponential backoff
   - [ ] Custom error code mapping
   - [ ] User-defined error handling in workflows

4. **Performance optimization** (3-4 hours)
   - [ ] `skyvern/forge/sdk/cache/` - LLM response caching
   - [ ] Database query optimization
   - [ ] Concurrent task execution
   - [ ] Browser resource management

5. **Artifact storage** (2-3 hours)
   - [ ] `skyvern/forge/sdk/artifact/` - Storage abstraction
   - [ ] S3 integration
   - [ ] Azure Blob Storage
   - [ ] Local filesystem storage

6. **Integration systems** (3-4 hours)
   - [ ] `integrations/n8n/` - n8n workflow platform
   - [ ] `integrations/make/` - Make.com
   - [ ] `integrations/langchain/` - LangChain connector
   - [ ] `integrations/llama_index/` - LlamaIndex connector

7. **CLI development** (2-3 hours)
   - [ ] `skyvern/cli/commands.py` - Command structure
   - [ ] `skyvern/cli/run_commands.py` - Run subcommands
   - [ ] `skyvern/cli/quickstart.py` - Setup wizard

8. **Python SDK** (3-4 hours)
   - [ ] `skyvern/library/skyvern.py` - Public SDK API
   - [ ] `skyvern/library/skyvern_browser.py` - Browser control
   - [ ] Embedded server mode
   - [ ] SDK usage examples

**Checkpoint:** You should have expert-level understanding of the entire system.

---

## Core Components Deep Dive

### 1. Forge Package (`skyvern/forge/`)

**Purpose:** Main application server and SDK

| Component | File | Responsibility |
|-----------|------|----------------|
| Singletons | `app.py` | Initialize DATABASE, BROWSER_MANAGER, LLM_API_HANDLER, etc. |
| FastAPI App | `api_app.py` | Create app, configure middleware, manage lifespan |
| Agent | `agent.py` | Main ForgeAgent class - orchestrate workflow execution |
| Prompts | `prompts.py` | LLM prompt templates and formatting |
| Async Ops | `async_operations.py` | Operation pooling for concurrent tasks |
| Routes | `sdk/routes/*.py` | API endpoint definitions |
| Database | `sdk/db/` | Models, client, migrations |
| Workflows | `sdk/workflow/` | Workflow models, blocks, services |

**Key APIs:**
- `POST /api/v1/tasks` - Create task
- `POST /api/v1/workflows/{id}/run` - Execute workflow
- `GET /api/v1/workflows/{id}/runs` - List runs
- `POST /api/v1/credentials` - Store credentials
- `GET /api/v1/browser-sessions` - List browser sessions

### 2. WebEye Package (`skyvern/webeye/`)

**Purpose:** Browser automation engine using Playwright

| Component | File | Responsibility |
|-----------|------|----------------|
| Browser Manager | `browser_manager.py` | Manage browser instances, contexts, pages |
| Browser Factory | `browser_factory.py` | Create and configure browsers |
| Session Manager | `persistent_sessions_manager.py` | Handle persistent sessions |
| Scraper | `scraper/scraper.py` | DOM scraping, element detection, tree building |
| Actions | `actions/actions.py` | Action models (Click, Input, Select, etc.) |
| Handler | `actions/handler.py` | Execute actions via Playwright |
| Responses | `actions/responses.py` | Action result data structures |

**Action Flow:**
1. Agent determines action from LLM analysis
2. Action object created (e.g., ClickAction)
3. Handler executes via Playwright
4. Response captured (success/failure, data extracted)
5. Result stored in database

### 3. Services Package (`skyvern/services/`)

**Purpose:** Business logic orchestration

| Service | File | Responsibility |
|---------|------|----------------|
| Workflow | `workflow_service.py` | Workflow CRUD, execution orchestration |
| Run | `run_service.py` | Workflow run execution, state management |
| Task | `task_v2_service.py` | Task API with iterative execution |
| Browser Session | `browser_session_service.py` | Session lifecycle management |
| Action | `action_service.py` | Action execution and logging |
| Block | `block_service.py` | Block execution logic |
| Webhook | `webhook_service.py` | Webhook dispatch and retry |
| OTP | `otp_service.py` | OTP/TOTP code generation |

### 4. Database Models (`skyvern/forge/sdk/db/models.py`)

**Core Tables:**

```python
# Multi-tenancy
Organization
  └── id, name, created_at

# Workflows
Workflow
  ├── workflow_id (UUID)
  ├── organization_id (FK)
  ├── title
  ├── description
  ├── workflow_definition (JSON)
  └── version

WorkflowRun
  ├── workflow_run_id (UUID)
  ├── workflow_id (FK)
  ├── organization_id (FK)
  ├── status (QUEUED, RUNNING, COMPLETED, FAILED, etc.)
  ├── run_blocks (relationship)
  └── output (JSON)

RunBlock
  ├── run_block_id (UUID)
  ├── workflow_run_id (FK)
  ├── block_type
  ├── status
  ├── output (JSON)
  └── error_code

# Tasks (v1/v2)
Task
  ├── task_id (UUID)
  ├── organization_id (FK)
  ├── url
  ├── goal
  ├── navigation_goal
  ├── extraction_goal
  ├── status
  └── steps (relationship)

Step
  ├── step_id (UUID)
  ├── task_id (FK)
  ├── order
  ├── action (JSON)
  ├── output (JSON)
  └── screenshots

# Credentials
Credential
  ├── credential_id (UUID)
  ├── organization_id (FK)
  ├── credential_type (password, secret_key, oauth, etc.)
  └── encrypted_data (JSON)

# Browser Sessions
BrowserSession
  ├── browser_session_id (UUID)
  ├── organization_id (FK)
  ├── status (ACTIVE, INACTIVE, CLOSED)
  └── session_data (JSON)
```

---

## Key Concepts

### 1. Vision-Based Automation

Unlike traditional automation that relies on fixed selectors (CSS, XPath), Skyvern:
- Takes screenshots of the page
- Sends screenshots to vision LLM (GPT-4V, Claude)
- LLM analyzes visual layout and identifies elements
- Plans actions based on natural language goals
- Adapts to UI changes automatically

### 2. Multi-Tenancy

Every major entity has `organization_id`:
- Row-level security at database level
- Credential isolation per organization
- Separate workflow/task namespaces
- Authentication via JWT with org context

### 3. Async/Await Architecture

Entire codebase uses asyncio:
```python
async def execute_workflow(workflow_id: str):
    async with database.session() as session:
        workflow = await session.get(Workflow, workflow_id)
        async with browser_manager.create_context() as context:
            await agent.execute(workflow, context)
```

### 4. Block System

Workflows are composed of blocks:
```json
{
  "blocks": [
    {
      "block_type": "navigation",
      "url": "https://example.com"
    },
    {
      "block_type": "action",
      "goal": "Fill out the form"
    },
    {
      "block_type": "extract",
      "schema": {"name": "string", "email": "string"}
    }
  ]
}
```

Blocks can reference outputs:
```json
{
  "url": "${blocks.navigation_1.output.final_url}"
}
```

### 5. Error Recovery

Multi-level retry strategy:
1. **Action-level:** Retry failed actions (max 5 retries)
2. **Step-level:** Retry entire step if action fails
3. **Task-level:** Custom error handling via user-defined blocks
4. **Workflow-level:** Error blocks for recovery flows

---

## File Reading Order

### Beginner Path

1. `README.md` - Project overview
2. `CLAUDE.md` - Development guide
3. `skyvern/config.py` - Configuration
4. `skyvern/constants.py` - Constants
5. `skyvern/forge/app.py` - Application initialization
6. `skyvern/forge/api_app.py` - FastAPI setup
7. `skyvern/forge/sdk/db/models.py` - Database models (skim)
8. `skyvern/schemas/workflows.py` - Workflow schemas

### Intermediate Path

9. `skyvern/forge/agent.py` - Agent orchestration
10. `skyvern/forge/prompts.py` - LLM prompts
11. `skyvern/services/workflow_service.py` - Workflow logic
12. `skyvern/services/run_service.py` - Run execution
13. `skyvern/forge/sdk/workflow/models/block.py` - Block system
14. `skyvern/webeye/browser_manager.py` - Browser management
15. `skyvern/webeye/scraper/scraper.py` - DOM scraping

### Advanced Path

16. `skyvern/webeye/actions/handler.py` - Action execution
17. `skyvern/forge/sdk/routes/*.py` - API routes (all)
18. `skyvern/services/*.py` - All services (deep dive)
19. `skyvern/forge/sdk/workflow/service.py` - Workflow service
20. `skyvern/library/skyvern.py` - Python SDK

---

## Hands-On Exercises

### Exercise 1: Simple Task

**Goal:** Create a task that navigates to a website and extracts data.

```python
# Using Python SDK
from skyvern import Skyvern

skyvern = Skyvern()
task = skyvern.task(
    url="https://news.ycombinator.com",
    goal="Extract the top 5 post titles",
    extraction_schema={
        "titles": "list[string]"
    }
)
result = task.execute()
print(result.extracted_data)
```

**Learning objectives:**
- Understand task API
- See LLM planning in action
- Review screenshots and action history

### Exercise 2: Multi-Step Workflow

**Goal:** Build a workflow with navigation, action, and extraction blocks.

1. Create workflow via UI
2. Add NavigationBlock (go to website)
3. Add ActionBlock (interact with page)
4. Add ExtractBlock (get structured data)
5. Run and debug

**Learning objectives:**
- Understand block composition
- Learn parameter passing
- Use workflow debugger

### Exercise 3: Loop Over Data

**Goal:** Create a workflow that loops over a list of URLs.

```json
{
  "parameters": [
    {"name": "urls", "type": "array"}
  ],
  "blocks": [
    {
      "block_type": "loop",
      "loop_over": "${workflow.parameters.urls}",
      "loop_block": {
        "block_type": "navigation",
        "url": "${loop.value}"
      }
    }
  ]
}
```

**Learning objectives:**
- Master LoopBlock
- Understand iteration patterns
- Handle dynamic data

### Exercise 4: Custom Block

**Goal:** Create a custom block type for specific business logic.

1. Define new block class in `forge/sdk/workflow/models/block.py`
2. Implement execution logic
3. Register block type
4. Test in workflow

**Learning objectives:**
- Extend block system
- Implement custom logic
- Integrate with existing blocks

### Exercise 5: Error Handling

**Goal:** Build a workflow with error recovery.

1. Create workflow with ValidationBlock
2. Add TerminateBlock on validation failure
3. Test with invalid data
4. Add retry logic

**Learning objectives:**
- Master error handling
- Implement validation
- Build robust workflows

---

## Advanced Topics

### 1. Custom LLM Provider

Implement custom LLM provider:

```python
# skyvern/forge/sdk/core/llm/custom_provider.py
from skyvern.forge.sdk.core.llm import BaseLLMAPIHandler

class CustomLLMHandler(BaseLLMAPIHandler):
    async def generate_completion(self, prompt: str) -> str:
        # Your implementation
        pass
```

### 2. Custom Action Type

Add new action type:

```python
# skyvern/webeye/actions/actions.py
class CustomAction(Action):
    action_type: ActionType = ActionType.CUSTOM

    # Your fields

# skyvern/webeye/actions/handler.py
async def handle_custom_action(action: CustomAction, page: Page):
    # Your implementation
```

### 3. Webhook Integration

Set up webhook for task completion:

```python
task = skyvern.task(
    url="https://example.com",
    goal="Extract data",
    webhook_callback_url="https://your-server.com/webhook"
)
```

Webhook payload:
```json
{
  "task_id": "...",
  "status": "completed",
  "extracted_data": {...}
}
```

### 4. Performance Tuning

Optimize for high throughput:

```python
# config.py
MAX_CONCURRENT_TASKS = 10
BROWSER_POOL_SIZE = 20
DATABASE_CONNECTION_POOL_SIZE = 30
```

---

## Summary

This roadmap provides a structured path from beginner to expert:

1. **Week 1:** Foundation - Setup, first task, basic config
2. **Week 2:** Core Architecture - Agent, browser, database
3. **Week 3:** API & Services - Routes, business logic, credentials
4. **Week 4:** Workflows - Blocks, parameters, execution
5. **Week 5:** Browser Automation - Scraping, actions, vision
6. **Week 6:** Frontend - React, routing, state, streaming
7. **Weeks 7-8:** Advanced - Multi-tenancy, LLM, optimization, integrations

**Next Steps:**
- Pick a phase based on your current knowledge
- Follow the checklist systematically
- Complete hands-on exercises
- Build your own custom workflows
- Contribute back to the project!

**Resources:**
- Documentation: https://docs.skyvern.com
- GitHub: https://github.com/skyvern-ai/skyvern
- Discord: Join for community support

Happy learning!
