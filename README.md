# AACS-Core

> **Autonomous Agent Coding System** — A multi-agent framework where 33 specialized AI agents collaborate to autonomously plan, build, test, and deliver production software.

![Build](https://img.shields.io/badge/build-passing-brightgreen)
![Tests](https://img.shields.io/badge/tests-2314%20passed-success)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![License](https://img.shields.io/badge/license-MIT-informational)
![WebUI](https://img.shields.io/badge/WebUI-16%20pages-blueviolet)
![Embedding](https://img.shields.io/badge/Embedding-8%20providers-orange)

---

## Overview

AACS-Core is a production-grade autonomous coding system built around a multi-agent architecture. Instead of relying on a single LLM to generate code, it decomposes the entire software development lifecycle into specialized roles — each handled by a dedicated AI agent with a focused role description, quality checklist, and permitted toolset.

The system is orchestrated by a central **Orchestrator** that never writes code itself. It receives project briefs, decomposes them into phased task queues, dispatches work to the most appropriate specialist agents, enforces phase-gate quality criteria, and handles escalations when agents encounter blockers.

Each specialist agent executes an **8-stage cognitive cycle**: Reception → Plan → Context Loading → Execution → Self-Review → Verification → Impact Assessment → Reporting. This cycle includes built-in loop prevention with 5 escalation grades, automatic self-correction of critical findings, and comprehensive impact analysis before results are submitted.

The system is underpinned by five core subsystems:

- **LLM Engine** — Context assembly pipeline with 6-stage prompt construction, token budgeting, circuit breaker pattern, streaming support, and per-project cost tracking
- **Embedding Engine** — 8-provider embedding system (OpenAI, HuggingFace, Cohere, Google, Ollama, Jina, Voyage, Mock) with LRU cache, circuit breaker, fallback chain, parallel batch processing, auto-chunking, and cost tracking
- **Virtual Filesystem Engine (VFE)** — In-memory code storage with segment-based parsing (Python, JS/TS), version history, atomic transactions, and hash-verified disk sync
- **Memory Engine** — Three-tier memory: per-agent working memory (dict), episodic memory (SQLite + FTS5 + ChromaDB vector search with custom embedding), and project memory (JSON) for conventions and decisions
- **Knowledge Graph** — NetworkX DiGraph with SQLite persistence supporting 5 query types: tech compatibility, pattern recommendation, tech pairing, security checks, and lesson retrieval
- **Message Bus** — Async priority queue with SQLite persistence delivering targeted and broadcast messages between agents
- **Text Chunker** — 4-strategy text chunking (fixed, sentence, paragraph, word) with configurable overlap for long document embedding

---

## Architecture

```
                              ┌─────────────────────────────────────────────────────────┐
                              │                    USER LAYER                          │
                              │   CLI (click)  │  WebUI (FastAPI+WS)  │  REST API     │
                              └──────┬─────────────┬──────────────────────┬────────────┘
                                     │             │                      │
                                     ▼             ▼                      ▼
                              ┌─────────────────────────────────────────────────────────┐
                              │              MESSAGE BUS (SQLite-backed)               │
                              │         Priority Queue │ Targeted │ Broadcast          │
                              └──────────┬──────────────────────┬────────────────────┘
                                         │                      │
                                         ▼                      ▼
                              ┌──────────────┐        ┌──────────────────┐
                              │  ORCHESTRATOR │◄───────│ HUMAN INTERFACE  │
                              │   (Dispatches │        │    AGENT         │
                              │   task queue, │        │                  │
                              │   phase gates)│        └──────────────────┘
                              └──────┬───────┘
                                     │
              ┌──────────────────────┼────────────────────────────────────┐
              │                      │                                    │
              ▼                      ▼                                    ▼
    ┌──────────────────┐  ┌──────────────────┐               ┌──────────────────┐
    │  PROJECT MANAGER │  │    ARCHITECT     │               │  BACKEND DEV     │
    ├──────────────────┤  ├──────────────────┤               ├──────────────────┤
    │  FRONTEND DEV    │  │  DATABASE DEV    │               │  AUTH DEV        │
    ├──────────────────┤  ├──────────────────┤               ├──────────────────┤
    │  CONFIG DEV      │  │  TEST WRITER     │               │  E2E TESTER      │
    ├──────────────────┤  ├──────────────────┤               ├──────────────────┤
    │  DEBUGGER        │  │  CODE REVIEWER   │               │  SECURITY CHECK  │
    ├──────────────────┤  ├──────────────────┤               ├──────────────────┤
    │  OPTIMIZER       │  │  AUDITOR         │               │  GITHUB MANAGER  │
    ├──────────────────┤  ├──────────────────┤               ├──────────────────┤
    │  DOCS WRITER     │  │  DELIVERY AGENT  │               ├──────────────────┤
    ├──────────────────┤  ├──────────────────┤               │  KNOWLEDGE AGENT │
    │  MEMORY KEEPER   │  │  INTEGRATION TST │               ├──────────────────┤
    └──────────────────┘  └──────────────────┘               │PERFORMANCE TST  │
                                                            └──────────────────┘
              │                      │                                    │
              └──────────────────────┼────────────────────────────────────┘
                                     │
                                     ▼
                    ┌─────────────────────────────────────────┐
                    │            SHARED INFRASTRUCTURE         │
                    │                                         │
                    │  ┌───────────┐  ┌────────────────────┐  │
                    │  │    VFE    │  │   LLM ENGINE       │  │
                    │  │ (In-mem   │  │ (Context Assembly,  │  │
                    │  │  files +  │  │  Token Budgeting,  │  │
                    │  │  segments)│  │  Cost Tracking)    │  │
                    │  └───────────┘  └────────────────────┘  │
                    │                                         │
                    │  ┌───────────┐  ┌────────────────────┐  │
                    │  │ TOOL      │  │   MEMORY ENGINE     │  │
                    │  │ REGISTRY  │  │ (Working + Episodic │  │
                    │  │(53 tools, │  │  + Project + Skills)│  │
                    │  │ perms,    │  │ (SQLite+ChromaDB)   │  │
                    │  │ timeouts) │  └────────────────────┘  │
                    │  └───────────┘  ┌────────────────────┐  │
                    │  ┌───────────┐  │  KNOWLEDGE GRAPH   │  │
                    │  │ STATE     │  │ (NetworkX DiGraph, │  │
                    │  │ STORE     │  │  5 query types,    │  │
                    │  │(JSON per- │  │  SQLite persist)   │  │
                    │  │ sist)     │  └────────────────────┘  │
                    │  └───────────┘  ┌────────────────────┐  │
                    │  ┌───────────┐  │  CHECKPOINT MGR    │  │
                    │  │  EVENT    │  │ (SHA256 verify,    │  │
                    │  │  SYSTEM   │  │  atomic swap,      │  │
                    │  │ (pub/sub) │  │  retention policy) │  │
                    │  └───────────┘  └────────────────────┘  │
                    │  ┌───────────┐  ┌────────────────────┐  │
                    │  │  SHELL    │  │  SECURITY          │  │
                    │  │  ENGINE   │  │ (Path validation,  │  │
                    │  │(sandboxed,│  │  shell quoting,    │  │
                    │  │ perms)    │  │  input sanitization)│  │
                    │  └───────────┘  └────────────────────┘  │
                    └─────────────────────────────────────────┘
```

---

## Features

- 🤖 **33 Specialized Agents** — Each with focused role descriptions, quality checklists, and permitted toolsets, ensuring clear separation of concerns
- 🔄 **8-Stage Cognitive Cycle** — Reception → Plan → Context → Execute → Self-Review → Verify → Impact → Report
- 📁 **Virtual Filesystem Engine** — In-memory code storage with segment-based parsing (Python, JS/TS), version history, atomic transactions, and hash-verified disk sync
- 🧠 **Three-Tier Memory** — Working memory (per-agent dict), Episodic memory (SQLite + FTS5 + ChromaDB with custom embeddings), Project memory (JSON)
- 🔍 **Knowledge Graph** — NetworkX DiGraph with 5 query types: tech compatibility, pattern recommendation, tech pairing, security check, lesson retrieval
- 📨 **Inter-Agent Communication** — Message Bus with SQLite persistence, targeted and broadcast routing, priority levels (critical/high/normal/low)
- 🚦 **Phase-Gated Lifecycle** — 9 phases: Intake → Architecture → Infrastructure → Development → Integration → QA → Audit → Delivery → Delivered
- 📋 **Task Queue** — Dependency resolution, priority routing, auto-retry with exponential backoff, concurrent dispatch
- 🔄 **Loop Prevention** — 5 escalation grades from context refresh (grade 0) to human required (grade 4)
- 💾 **Checkpoint/Restore** — SHA256 integrity verification, atomic temp-file writes, configurable retention policy, periodic scheduling
- 🐚 **Shell Sandboxing** — Permission-checked command execution, timeout enforcement, cross-platform command translation
- 🌐 **WebUI (16 pages)** — FastAPI server with WebSocket real-time events, token-based authentication, dashboard, agents view, task management, file browser, chat, embeddings config, skill management, checkpoints, logs, budget tracker, settings
- 🧩 **Embedding Model Config** — Frontend UI to add/manage embedding models from 8 providers (just like LLM model config)
- 📝 **Skill Management** — Frontend UI to add/edit/delete/validate skills, auto-configure skills into the system
- 💻 **CLI Interface** — Click-based commands for start/stop, project management, agent status, diagnostics
- ⚡ **Atomic Persistence** — All state files use temp-file + rename to prevent corruption (StateStore, MessageBus, VFE, Checkpoints)
- 🛡️ **Security** — Path traversal prevention, shell quoting, input sanitization, branch name validation, env var protection
- 📊 **Cost Tracking** — Per-project and daily budget limits with warnings, token estimation, cost-per-call calculation
- 🔧 **53 Registered Tools** — Build, test, lint, security scan, code analysis, git operations, monitoring, search, knowledge queries, semantic search
- ✂️ **Text Chunking** — 4 strategies (fixed, sentence, paragraph, word) with configurable size and overlap for long document embedding
- 📊 **Shared Vector Utils** — Cosine similarity, mean pooling, Euclidean distance, L2 normalization

---

## Quick Start

### Prerequisites

- **Python 3.10+**
- **OpenAI API key** (or compatible endpoint)
- [Optional] Git, Docker, Node.js for full tooling support

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/aacs-core.git
cd aacs-core

# Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt

# Install test dependencies (optional)
pip install pytest pytest-asyncio pytest-cov pytest-mock
```

### Configuration

1. Create a config file at `~/.aacs-core/config.yaml`:

```yaml
version: "1.0"

llm:
  base_url: "https://api.openai.com/v1"
  api_key: "vault:llm_api_key"        # Stored in OS keychain
  model_id: "gpt-4o"
  fast_model_id: "gpt-4o-mini"         # Optional: cheaper model for planning
  max_tokens: 4096
  temperature: 0.1

github:
  token: "vault:github_token"          # Stored in OS keychain
  username: "your-username"

resources:
  max_concurrent_agents: 3
  daily_budget_usd: 20.0
  project_budget_usd: 50.0
  checkpoint_interval_minutes: 30

quality:
  min_test_coverage: 80
```

2. Store your API key securely in the OS keychain:

```bash
# Use the CLI or keyring directly
python -c "import keyring; keyring.set_password('aacs-core', 'llm_api_key', 'sk-...')"
```

### Run Modes

```bash
# Full system (WebUI + all agents + dispatch loop)
python main.py

# CLI mode
python cli.py start
python cli.py status
python cli.py project-new -b "Build a REST API with FastAPI"

# WebUI only
python -m webui.server
```

Access the WebUI at **http://localhost:8080**. Set a password on first login.

### Run Tests

```bash
# Run full test suite (2314 tests)
pytest

# With coverage report
pytest --cov=. --cov-report=term-missing

# Run a specific test file
pytest tests/test_orchestrator.py -v
```

---

## Project Structure

```
aacs-core/
├── main.py                   # System bootstrap — initializes all engines and agents
├── cli.py                    # Click-based CLI interface
├── config.py                 # Configuration loading, validation, credential vault
├── requirements.txt          # Python dependencies
├── pytest.ini                # Test configuration
│
├── core/                     # Core engine subsystems
│   ├── types.py              # All enums and dataclasses (23 agent types, 11 task types, etc.)
│   ├── exceptions.py         # Custom exception hierarchy (LLM, VFE, Shell, Git, Task, Phase)
│   ├── state.py              # StateStore, EventSystem, MessageBus, ToolRegistry
│   ├── llm.py                # LLM Engine — context assembly, API calls, cost tracking
│   ├── vfe.py                # Virtual Filesystem Engine — segment parsing, versioning, disk sync
│   ├── memory.py             # Memory Engine — working/episodic/project memory, skills registry
│   ├── embedding.py          # Embedding Engine — 8 providers, cache, circuit breaker, chunking
│   ├── chunker.py            # Text Chunker — 4 strategies for long document embedding
│   ├── vector_utils.py       # Vector math — cosine similarity, mean pooling, normalization
│   ├── knowledge.py          # Knowledge Graph — NetworkX DiGraph, 5 query types, SQLite persist
│   ├── checkpoint.py         # Checkpoint Manager — create/verify/restore with SHA256
│   ├── shell.py              # Shell Engine — sandboxed execution, permission checks, output parsing
│   ├── security.py           # Security utilities — path validation, shell quoting, sanitization
│   └── github.py              # GitHub Bridge — API wrapper for repos, PRs, CI, releases
│
├── agents/                   # 23 specialized agents
│   ├── base.py               # BaseAgent — 8-stage cognitive cycle, loop prevention, message protocol
│   ├── orchestrator.py       # Orchestrator — task queue, dispatch loop, phase gates, escalation
│   ├── human_interface.py    # Human-facing agent — chat, brief processing
│   ├── project_manager.py    # Project planning, backlog management, milestone tracking
│   ├── architect.py          # System architecture, tech stack decisions, design docs
│   ├── backend_developer.py  # Backend code generation (Python, Node, Go)
│   ├── frontend_developer.py # Frontend code generation (React, Vue, HTML/CSS)
│   ├── database_developer.py # Database schema, migrations, queries
│   ├── auth_developer.py     # Authentication & authorization implementation
│   ├── config_developer.py   # DevOps, Docker, CI/CD, infrastructure config
│   ├── test_writer.py        # Unit test generation
│   ├── e2e_tester.py         # End-to-end test design
│   ├── integration_tester.py # Integration test planning
│   ├── performance_tester.py # Performance test planning
│   ├── debugger.py           # Bug diagnosis and fix implementation
│   ├── code_reviewer.py      # Code review with severity findings
│   ├── security_checker.py   # Security vulnerability scanning
│   ├── optimizer.py          # Code optimization and refactoring
│   ├── auditor.py            # Final quality audit
│   ├── github_manager.py     # GitHub operations (PRs, branches, CI)
│   ├── docs_writer.py        # Documentation generation
│   ├── delivery_agent.py     # Build, package, and deployment
│   ├── knowledge_agent.py    # Knowledge graph queries and updates (semantic search)
│   ├── memory_keeper.py      # Memory consolidation and management (semantic search)
│   ├── skill_manager.py      # Skill lifecycle management — CRUD, validation, auto-configure
│   ├── api_specialist.py     # API design and implementation specialist
│   ├── cloud_architect.py    # Cloud infrastructure design
│   ├── data_engineer.py      # Data pipeline and ETL specialist
│   ├── ml_engineer.py        # Machine learning model integration
│   ├── mobile_developer.py   # Mobile app development
│   ├── ui_ux_designer.py     # UI/UX design implementation
│   ├── release_manager.py    # Release management and deployment
│   ├── compliance_agent.py   # Standards compliance auditor
│   └── performance_engineer.py # Performance optimization specialist
│
├── tools/                    # Tool implementations (53 registered tools)
│   ├── filesystem.py         # VFE file operations (9 tools)
│   ├── code_analysis.py      # Symbol search, impact analysis, complexity (7 tools)
│   ├── build_test.py         # Build, lint, test, migration (13 tools)
│   ├── git_tools.py          # Git operations + GitHub API (10 tools)
│   ├── search.py             # Package search, docs search, CVE lookup, knowledge queries (7 tools)
│   └── monitoring.py         # System health, budget tracking, admin tools (7 tools)
│
├── webui/                    # Web interface
│   ├── server.py             # FastAPI app — REST API, WebSocket, auth, static files (40+ endpoints)
│   ├── routes.py             # Additional route definitions
│   └── frontend/             # Frontend assets
│       ├── index.html        # Main SPA entry point
│       ├── css/              # Stylesheets (main.css, components.css)
│       └── js/               # JavaScript pages (13 page modules)
│           ├── app.js        # Core framework, router, API client
│           └── pages/       # Page modules
│               ├── dashboard.js    # System dashboard
│               ├── agents.js       # Agent monitoring
│               ├── tasks.js        # Task queue management
│               ├── chat.js         # Chat interface
│               ├── files.js        # File browser
│               ├── settings.js     # LLM & system config
│               ├── embeddings.js  # Embedding model config
│               ├── skills.js       # Skill management (CRUD)
│               ├── checkpoints.js # Checkpoint management
│               ├── logs.js         # Real-time event logs
│               ├── budget.js       # Budget tracking
│               └── ...
│
├── skills/                   # Skill documents for agent knowledge
│   ├── general_coding.md     # General coding best practices
│   ├── python_fastapi.md     # FastAPI/Python patterns
│   ├── react_frontend.md     # React frontend patterns
│   ├── node_express.md       # Node.js/Express patterns
│   ├── postgresql.md         # PostgreSQL database patterns
│   ├── docker_deployment.md  # Docker & deployment patterns
│   └── authentication.md     # Auth patterns (JWT, OAuth, session)
│
├── data/                     # Static data
│   ├── knowledge_seed.json   # Initial knowledge graph seed data
│   └── project_template.json # Project template
│
├── tests/                    # Test suite (2314 tests)
│   ├── conftest.py           # 15 shared fixtures
│   ├── test_types.py         # 57 tests — type system validation
│   ├── test_exceptions.py    # 85 tests — exception hierarchy
│   ├── test_security.py      # 85 tests — security utilities
│   ├── test_state.py         # 91 tests — state management
│   ├── test_message_bus.py   # 29 tests — message bus
│   ├── test_base_agent.py    # 55 tests — agent cognitive cycle
│   ├── test_orchestrator.py  # 82 tests — orchestrator dispatch
│   ├── test_checkpoint.py    # 29 tests — checkpoint management
│   ├── test_memory.py        # 38 tests — memory engine
│   ├── test_knowledge.py     # 30 tests — knowledge graph
│   ├── test_config.py        # 27 tests — configuration
│   ├── test_webui.py         # 30 tests — web interface
│   ├── test_tools.py         # 59 tests — tool execution
│   ├── test_embedding.py     # 776 tests — embedding engine, providers, cache, batch
│   ├── test_llm.py           # LLM engine tests
│   ├── test_analytics.py     # Analytics and metrics tests
│   ├── test_api_gateway.py   # API gateway tests
│   ├── test_vector_utils.py  # 19 tests — cosine similarity, mean pool, normalization
│   ├── test_chunker.py       # 11 tests — text chunking strategies
│
└── docs/                     # Documentation
    ├── CONFIGURATION.md     # Configuration reference
    └── ARCHITECTURE.md       # Detailed architecture document
```

---

## Agent Catalog

| Agent | Role | Key Responsibilities |
|---|---|---|
| **Orchestrator** | Central coordinator | Task decomposition, dispatch, phase gates, escalation handling, progress tracking |
| **Human Interface** | User liaison | Chat processing, brief intake, status reporting, human input requests |
| **Project Manager** | Planning lead | Backlog management, milestone tracking, effort estimation, acceptance criteria |
| **Architect** | System designer | Tech stack selection, architecture diagrams, API design, system decomposition |
| **Backend Developer** | Backend coder | Python/Node/Go code generation, API implementation, business logic |
| **Frontend Developer** | Frontend coder | React/Vue/HTML/CSS code generation, component design, UI implementation |
| **Database Developer** | Data layer expert | Schema design, migrations, query optimization, ORM models |
| **Auth Developer** | Security coder | Authentication/authorization, JWT/OAuth, RBAC, session management |
| **Config Developer** | DevOps engineer | Docker, CI/CD pipelines, environment config, infrastructure-as-code |
| **Test Writer** | Unit test specialist | Test case generation, mocking, test structure, coverage targets |
| **E2E Tester** | Integration tester | End-to-end test scenarios, Playwright/Cypress test design |
| **Integration Tester** | System tester | Integration test planning, API contract testing, service testing |
| **Performance Tester** | Load tester | Performance benchmarks, k6/Gatling test plans, profiling |
| **Debugger** | Bug hunter | Error diagnosis, root cause analysis, fix implementation, regression check |
| **Code Reviewer** | Quality gate | Review findings, code quality assessment, best practice violations |
| **Security Checker** | Security auditor | Vulnerability scanning, OWASP analysis, dependency audit |
| **Optimizer** | Refactoring specialist | Code optimization, dead code removal, performance improvements |
| **Auditor** | Final reviewer | Pre-delivery audit, standards compliance, documentation review |
| **GitHub Manager** | Git operations | Branch management, PR creation, CI monitoring, release management |
| **Docs Writer** | Technical writer | README, API docs, guides, inline documentation |
| **Delivery Agent** | Deployment specialist | Build, package, deploy, release notes, handoff |
| **Knowledge Agent** | Knowledge curator | Graph queries, pattern matching, tech compatibility checks |
| **Memory Keeper** | Memory manager | Memory consolidation, compression, convention enforcement |

---

## Configuration

Configuration is managed through `~/.aacs-core/config.yaml`. Sensitive values (API keys, tokens) use the OS-native keyring via `vault:` prefixes.

See [docs/CONFIGURATION.md](docs/CONFIGURATION.md) for the full configuration reference including:

- **LLM Settings** — Base URL, model selection, token limits, temperature, cost tracking
- **GitHub Integration** — Token, username, default visibility, auto-repo creation
- **WebUI** — Port, bind address, authentication
- **Resources** — Concurrency limits, budgets, checkpoint intervals
- **Quality** — Test coverage targets, loop prevention thresholds
- **Storage** — Data directories, project directories, checkpoint paths

### Environment Variables

API keys can also be provided via environment variables as fallback:

```bash
export OPENAI_API_KEY="sk-..."
export GITHUB_TOKEN="ghp_..."
```

---

## Development

### Setup

```bash
# Clone and install
git clone https://github.com/your-org/aacs-core.git
cd aacs-core
pip install -r requirements.txt
pip install pytest pytest-asyncio pytest-cov pytest-mock

# Run tests
pytest
```

### Test Suite

The project includes 2314 tests across 14+ test files covering all core modules:

```
tests/test_types.py         57 tests  — Enum values, dataclass validation
tests/test_exceptions.py    85 tests  — Exception hierarchy, attributes
tests/test_security.py      85 tests  — Shell quoting, path validation, sanitization
tests/test_state.py         91 tests  — StateStore, EventSystem, MessageBus, ToolRegistry
tests/test_message_bus.py   29 tests  — Pub/sub, priority routing, persistence
tests/test_base_agent.py    55 tests  — Cognitive cycle stages, loop prevention
tests/test_orchestrator.py  82 tests  — Task queue, dispatch, phase gates
tests/test_checkpoint.py    29 tests  — Create/verify/restore, retention
tests/test_memory.py        38 tests  — Working/episodic/project memory
tests/test_knowledge.py     30 tests  — Graph queries, persistence
tests/test_config.py        27 tests  — YAML loading, vault resolution
tests/test_webui.py         30 tests  — API endpoints, auth, WebSocket
tests/test_tools.py         59 tests  — Tool registration, execution, permissions
tests/test_embedding.py     776 tests — Embedding engine, 8 providers, cache, batch, retry, fallback
tests/test_llm.py           LLM engine tests — API calls, cost tracking, streaming
tests/test_analytics.py     Analytics, metrics, workflow API
tests/test_api_gateway.py   API gateway — webhook, rate limiting, client
tests/test_vector_utils.py  19 tests  — Cosine similarity, mean pool, normalization
tests/test_chunker.py       11 tests  — Text chunking strategies (fixed/sentence/paragraph/word)
```

### Project Stats

- **226 Python files** (production code + tests)
- **178K+ lines of code**
- **33 specialized agents**
- **53 registered tools**
- **8 embedding providers** (OpenAI, HuggingFace, Cohere, Google, Ollama, Jina, Voyage, Mock)
- **12 built-in skills** (Python, FastAPI, React, Node, Go, Rust, Java, PostgreSQL, Docker, DevOps, Security, Authentication)
- **16 WebUI pages** (Dashboard, Agents, Tasks, Files, Chat, Settings, Embeddings, Skills, Checkpoints, Logs, Budget)
- **40+ REST API endpoints**
- **2314 passing tests**
- **Text chunking** (4 strategies) with auto-embedding integration

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
