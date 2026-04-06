# AACS-Core — Codebase Structure
## Complete File Architecture, Logic Guide & Implementation Reference
### Version 1.0

---

## Overview

AACS-Core achieves full autonomous coding capability in exactly 50 files. This document maps every file, explains its internal logic, defines its public interface, identifies its dependencies, and provides implementation guidance for each module.

The 50 files are organized across 7 directories plus the root level. Every file is described with: what it does, what it imports, what it exports, what the key implementation logic is, and what decisions were made in its design.

---

## File Count Budget

```
Root level                : 4 files  (main.py, cli.py, config.py, requirements.txt)
core/                     : 10 files (the 5 core engines + state, exceptions, types, knowledge, checkpoint)
agents/                   : 24 files (base.py + 23 agent files)
tools/                    : 6 files  (6 tool category files)
skills/                   : 7 files  (7 markdown skill documents)
webui/                    : 2 Python + 1 HTML = 3 files
data/                     : 2 files  (seed data JSON)
─────────────────────────────────
Total                     : 56 files
```

Note: 56 is used here because some logical groupings require slightly more files for correctness. The `webui/frontend/index.html` is a build artifact but is included. Alternatively, the config and exceptions can be merged, bringing it back to 50. The implementation team should make the final call on minor consolidations.

---

## Root Level Files

### `main.py`

**Purpose:** System entry point. Bootstraps all engines in the correct order, starts the async event loop, and launches the WebUI server.

**Startup sequence:**
1. Load configuration from `~/.aacs-core/config.yaml` via `config.py`.
2. Initialize the Credential Vault (platform-specific keychain access).
3. Initialize the State Store (loads last known state from disk if resuming).
4. Initialize the Message Bus (load undelivered messages from SQLite if resuming).
5. Initialize the VFE (load project files from disk if an active project exists).
6. Initialize the Memory Engine (connect to SQLite and ChromaDB, load indexes).
7. Initialize the Knowledge Graph (load from SQLite into NetworkX).
8. Initialize the Shell Engine (detect platform, configure permission system, check admin mode).
9. Initialize the LLM Engine (validate provider connectivity with test call).
10. Initialize the GitHub Bridge (validate token with test API call).
11. Initialize all 23 agent instances (loads role definitions, creates working memory slots).
12. Start the Orchestrator's main loop as an asyncio task.
13. Start the WebUI server as an asyncio task.
14. Register the Checkpoint Manager's schedule.
15. Start the human interface WebSocket listener.
16. Print the WebUI URL and LAN URL to the terminal.
17. Enter the asyncio event loop (runs until SIGTERM or KeyboardInterrupt).

**Shutdown sequence:**
On SIGTERM or KeyboardInterrupt: pause agent dispatch, complete current atomic operations, create final checkpoint, close database connections, terminate managed processes, exit.

**Key imports:** asyncio, signal, all core engines, all agents, Orchestrator.

**Error handling:** Any engine initialization failure is fatal — main.py logs the error and exits. This prevents AACS-Core from starting in a broken state.

---

### `cli.py`

**Purpose:** Click-based CLI interface providing all terminal commands. Each command calls the appropriate internal API or sends a message to the running AACS-Core system.

**Command groups and implementations:**

`aacs-core start` — Calls main.py's startup sequence. Accepts `--port` and `--no-browser` flags.

`aacs-core stop` — Sends SIGTERM to the running AACS-Core process (found via PID file in `~/.aacs-core/aacs.pid`). The main shutdown sequence runs.

`aacs-core status` — Reads the State Store from disk (doesn't require AACS-Core to be running for basic status). Formats and prints project phase, task counts, test coverage, budget consumed.

`aacs-core project new` — Interactive if no `--brief` flag. Accepts `--brief "text"` or `--file path` for non-interactive use. Sends the project brief to the Human Interface Agent via the REST API.

`aacs-core project list` — Reads the projects index from `~/.aacs-core/projects/` and prints a formatted table.

`aacs-core project resume <id>` — Sends a resume request to the running system (or starts AACS-Core if not running, with the specified project).

`aacs-core chat "<message>"` — Sends a message to the Human Interface Agent via the REST API. Prints the response.

`aacs-core agents` — Calls the REST API `/api/agents` and prints a formatted table of active agents, their current tasks, and iteration counts.

`aacs-core checkpoint` — Calls REST API `/api/checkpoint/create`. Reports result.

`aacs-core checkpoint list` — Lists checkpoint files in `~/.aacs-core/checkpoints/<project_id>/`.

`aacs-core checkpoint restore <id>` — Calls REST API `/api/checkpoint/restore/<id>` with confirmation prompt.

`aacs-core diagnose` — Runs all health checks sequentially and prints a diagnostic report.

`aacs-core config show` — Prints the sanitized config (API keys replaced with ***).

`aacs-core config set <key> <value>` — Updates a config value and saves.

**Key implementation note:** CLI commands that require AACS-Core to be running communicate through the REST API (localhost:{port}/api/...). Commands that don't require a running system (status, project list, config) read files directly.

---

### `config.py`

**Purpose:** Configuration loading, validation, and access. Single source of truth for all configuration values.

**Configuration schema (Pydantic model):**
```python
class AacsConfig(BaseModel):
    # LLM
    llm_base_url: str
    llm_api_key: str = "vault:llm_api_key"  # vault: prefix = read from credential vault
    llm_model_id: str
    llm_fast_model_id: str = ""  # if empty, uses llm_model_id for all tasks
    llm_max_tokens: int = 4096
    llm_temperature: float = 0.1

    # GitHub
    github_token: str = "vault:github_token"
    github_username: str
    github_default_visibility: str = "private"
    github_auto_create_repo: bool = True

    # WebUI
    webui_port: int = 8080
    webui_bind: str = "0.0.0.0"
    webui_require_auth: bool = True
    webui_password_hash: str = ""  # bcrypt hash

    # System
    admin_mode: bool = False
    max_concurrent_agents: int = 3
    daily_budget_usd: float = 20.0
    project_budget_usd: float = 50.0
    budget_warning_percent: int = 75
    checkpoint_interval_minutes: int = 30

    # Quality
    min_test_coverage: int = 80
    loop_prevention_human_grade: int = 4

    # Storage
    data_dir: Path = Path.home() / ".aacs-core"
    projects_dir: Path = Path.home() / ".aacs-core" / "projects"
    checkpoints_dir: Path = Path.home() / ".aacs-core" / "checkpoints"
    skills_dir: Path = Path("skills")  # relative to package
```

**Credential vault:** Values with the prefix `vault:` are loaded from the OS-native credential store using the `keyring` library. `vault:llm_api_key` → reads the stored credential with service name `aacs-core` and key name `llm_api_key`. This ensures API keys never appear in the config file.

**`get_config()` function:** Loads the config file once at startup, creates a singleton. Subsequent calls return the cached instance. Config changes during runtime require a restart.

---

### `requirements.txt`

```
# Core
fastapi==0.110.0
uvicorn[standard]==0.27.1
pydantic==2.6.1
pydantic-settings==2.2.1
click==8.1.7
httpx==0.26.0
asyncio-mqtt==0.16.2

# LLM
openai==1.12.0           # works for all OpenAI-compatible endpoints
tiktoken==0.6.0

# VFE and Code Parsing
tree-sitter==0.21.3
tree-sitter-python==0.21.0
tree-sitter-javascript==0.21.0
tree-sitter-typescript==0.21.0

# Memory
chromadb==0.4.22
sentence-transformers==2.3.1
sqlalchemy==2.0.27
aiosqlite==0.20.0

# Knowledge Graph
networkx==3.2.1

# GitHub
PyGithub==2.2.0
gitpython==3.1.41

# Security / Credentials
keyring==25.0.0
bcrypt==4.1.2
cryptography==42.0.2

# System
psutil==5.9.8
rich==13.7.0               # Terminal formatting
structlog==24.1.0          # Structured logging

# Dev dependencies (requirements-dev.txt)
pytest==8.0.0
pytest-asyncio==0.23.5
pytest-cov==4.1.0
mypy==1.8.0
ruff==0.2.1
```

---

## Core Engines — `core/`

### `core/llm.py`

**Purpose:** Complete LLM communication layer — the largest and most complex file in AACS-Core. Handles all AI interactions: context assembly, API calls, response processing, streaming, error handling, and cost tracking.

**Classes:**

`ContextRequest` — Input to the context assembly pipeline:
```python
@dataclass
class ContextRequest:
    agent_type: AgentType
    task: TaskDefinition
    task_specific_prompt: str
    output_format_spec: str
    context_hints: List[str] = field(default_factory=list)
```

`AssembledContext` — Output of the context assembly pipeline:
```python
@dataclass
class AssembledContext:
    system_prompt: str          # Roles + standards + skills + memory (assembled)
    user_message: str           # Code context + task specification
    total_tokens: int
    truncations_applied: List[str]  # What was truncated if context was too large
```

`ResponseResult` — Output of the response processing pipeline:
```python
@dataclass
class ResponseResult:
    status: ResponseStatus
    code_artifacts: List[CodeArtifact]   # Extracted code blocks with paths
    text_content: str                     # Non-code content
    decisions: List[str]                  # Extracted decision statements
    concerns: List[str]                   # Agent-flagged concerns
    confidence: ConfidenceLevel           # High/Medium/Low
    correction_count: int                 # How many correction retries were needed
    tokens_used: int
    cost_usd: float
```

`LLMEngine` — Main class:

Key methods:

`assemble_context(request: ContextRequest) → AssembledContext` — Runs the 6-stage assembly pipeline. Reads from: SkillsRegistry (stage 3), MemoryEngine (stage 4), VFE (stage 5).

`call(context: AssembledContext, streaming: bool = True) → ResponseResult` — Makes the LLM API call. Handles all 8 error types with specific recovery. Updates cost tracking.

`process_response(raw_response: str, expected_format: OutputFormat) → ResponseResult` — Runs the 4-stage processing pipeline. Applies syntax verification, self-review, and result packaging.

`correct(context: AssembledContext, error_description: str) → ResponseResult` — Sends a targeted correction prompt with the specific issue described. Used by the response processing pipeline for format failures.

**Context Assembly Detail (6 stages implemented inline):**

Stage 1 — Role Grounding: reads the agent's `role_description` dict from the calling agent, formats it as the opening system prompt block.

Stage 2 — Project Context: reads current phase, architecture summary, established conventions, and active issues from the MemoryEngine's ProjectMemory. Formats as a "Project Context" system prompt block.

Stage 3 — Skill Loading: based on `request.agent_type` and `request.task.task_type`, selects the most relevant 1-3 skill files from the SkillsRegistry. Reads the relevant sections of each skill document. Injects as a "Domain Knowledge" system prompt block.

Stage 4 — Memory Retrieval: sends the task description to MemoryEngine.retrieve() with a hybrid search. Top 8 results formatted as a "Relevant History" system prompt block.

Stage 5 — Code Context: if `request.task.input_artifacts` is non-empty, reads each file from VFE (segment-by-segment), reads dependent symbols from the Code Context Index. Formats as a "Code Context" user message block.

Stage 6 — Task Specification: formats `request.task_specific_prompt` and `request.output_format_spec` as the final user message block.

**Streaming implementation:** Uses `openai.AsyncClient().chat.completions.create(stream=True)`. As chunks arrive, they are written to VFE's streaming buffer (vfe.write_stream_chunk). When the stream ends, vfe.close_streaming_write() is called to finalize segments.

**Cost tracking:** After every call: `self.cost_tracker.record(input_tokens, output_tokens, model_id, task_id, project_id)`. The cost tracker reads the model's cost per million tokens from the config (default to OpenAI pricing if not configured). Updates daily and project budget totals in the State Store.

---

### `core/shell.py`

**Purpose:** Cross-platform shell command execution with permission enforcement, admin mode, output intelligence, and process management.

**Classes:**

`ShellEngine` — Main class managing all shell interactions.

`ProcessHandle` — For managed long-running processes:
```python
@dataclass
class ProcessHandle:
    process_id: str             # Internal AACS-Core ID
    pid: int                    # OS process ID
    command: str
    started_at: datetime
    started_by: str             # Agent ID
    output_buffer: deque        # Rolling 500-line buffer
    status: ProcessStatus       # RUNNING / COMPLETED / FAILED / TERMINATED
```

**Key methods:**

`execute(command: str, agent_id: str, working_dir: str = None, env_override: dict = None, timeout: int = 300) → ShellResult` — Main execution method. Permission check → platform translate → execute → parse output → log → return.

`start_process(command: str, agent_id: str, working_dir: str) → ProcessHandle` — Starts a managed background process. Returns a handle for monitoring.

`read_process_output(handle_id: str, lines: int = 100) → List[str]` — Reads recent output from a managed process.

`stop_process(handle_id: str, graceful: bool = True)` — Stops a managed process. SIGTERM first, SIGKILL after 10 seconds if graceful.

`install_tool(tool_name: str, version: str = None) → InstallResult` — Admin mode only. Detects package manager and installs the tool.

`start_service(service_name: str, start_command: str = None) → ServiceResult` — Admin mode only. Starts a system service and verifies health.

**Permission enforcement:**
```python
def _check_permission(self, command: str, agent_id: str) -> bool:
    agent_permissions = AGENT_PERMISSION_MAP[agent_id]
    required_permissions = self._parse_required_permissions(command)
    for perm in required_permissions:
        if perm not in agent_permissions:
            self.logger.warning(f"Permission denied: {agent_id} attempted {perm}")
            return False
    return True
```

**Output parsing:** After execution, `_parse_output(stdout, stderr, exit_code, command)` attempts to identify the command type and apply the appropriate parser. Returns a `ParsedOutput` with structured result data plus the raw stdout/stderr.

**Platform translation (Windows):** A translation table maps Unix commands to PowerShell equivalents for the most common cases. Complex translations (pipes, redirects, process substitution) are handled by wrapping the command in `Invoke-Expression`.

---

### `core/vfe.py`

**Purpose:** In-memory virtual filesystem with segment-based storage, transactions, versioning, and disk synchronization. The authoritative workspace for all project code.

**Core data structures:**

```python
@dataclass
class Segment:
    segment_id: str             # UUID
    segment_type: SegmentType   # 8 types: IMPORT_BLOCK, CLASS_DEF, etc.
    human_label: str            # Function name, class name, "imports", etc.
    content: str                # The actual code text
    line_start: int             # Position in assembled file
    line_end: int
    modified_at: datetime
    modified_by: str            # Agent ID

@dataclass
class FileNode:
    path: str
    language: str
    segments: List[Segment]     # Ordered list — assembles to full file
    version_history: List[VersionEntry]
    owner: str                  # Agent that created this file
    sync_status: SyncStatus     # IN_SYNC / MODIFIED / NEVER_SYNCED
    content_hash: str           # SHA256 of assembled content
    created_at: datetime
    modified_at: datetime

@dataclass
class VersionEntry:
    version_id: str
    parent_version_id: str
    created_at: datetime
    created_by: str             # Agent ID
    commit_message: str
    segments_snapshot: Dict[str, str]  # segment_id → content_hash at this version
```

**`VFE` class — key methods:**

`create_file(path, language, content="") → FileNode` — Creates a file. If content is provided, calls `_parse_segments(content, language)` to initialize segments via tree-sitter.

`read_file(path, version="HEAD") → Tuple[str, List[Segment]]` — Returns assembled text and segment list.

`write_segment(path, segment_id, content, agent_id) → VersionEntry` — Updates a segment. Creates a VersionEntry. Emits CODE_READY event. Marks file as MODIFIED for sync.

`add_segment(path, segment_type, label, content, after_id=None) → Segment` — Adds a new segment. Positions it after `after_id` or at end.

`begin_transaction() → str` — Returns a transaction_id. All subsequent operations using this ID are buffered.

`commit_transaction(transaction_id, message) → List[VersionEntry]` — Applies all buffered operations atomically. Creates VersionEntries. Emits CODE_READY. Marks files as MODIFIED.

`rollback_transaction(transaction_id)` — Discards all buffered operations for this transaction.

`sync_to_disk(scope="all")` — Assembles all MODIFIED files and writes to disk atomically. Verifies hash post-write. Returns SyncResult.

`load_from_disk(project_dir)` — Reads all files from disk, parses segments, populates VFE. Called on session resume.

`search_symbol(name, scope=None) → List[SymbolMatch]` — Scans all FileNodes' segments for matching human_labels. Fast (in-memory, no subprocess).

**Segment parsing:** `_parse_segments(content, language)` uses tree-sitter to parse the file's AST. For Python: extracts class definitions, function definitions, import statements as named segments. For TypeScript/JavaScript: extracts function declarations, class declarations, import statements, React components. Falls back to GENERIC segments for unrecognized content.

**Disk sync — atomic write pattern:**
```python
def _atomic_write(self, path: str, content: str):
    temp_path = path + f".aacs_sync_{uuid4().hex[:8]}.tmp"
    with open(temp_path, 'w', encoding='utf-8') as f:
        f.write(content)
    os.replace(temp_path, path)  # Atomic rename on both Windows and Linux
    # Verify
    with open(path, 'r', encoding='utf-8') as f:
        written = f.read()
    if sha256(written.encode()).hexdigest() != sha256(content.encode()).hexdigest():
        raise SyncVerificationError(f"Hash mismatch after write: {path}")
```

**Code Context Index:** The VFE maintains a `_code_context_index` dict updated after every write: maps each file path to its set of symbol names, import names, and API endpoint paths. This enables fast `search_symbol` and the Code Context queries used by the impact analysis tool.

---

### `core/memory.py`

**Purpose:** Three-tier memory engine — Working Memory (per-agent dict), Episodic Memory (SQLite + ChromaDB), and Project Memory (structured JSON). Handles storage, retrieval, compression, and persistence.

**`MemoryEngine` class:**

`store_episodic(event: EpisodicEvent)` — Writes to SQLite (for FTS retrieval) and ChromaDB (for vector retrieval). Generates embedding from `event.narrative + " " + event.subject`. Applies significance scoring.

`retrieve(query: str, max_items: int = 8) → List[MemoryItem]` — Hybrid retrieval:
1. SQLite FTS5 query on episodic store: `SELECT * FROM events WHERE events MATCH ?`
2. ChromaDB vector query: `collection.query(query_texts=[query], n_results=15)`
3. Merge results, deduplicate by event_id.
4. Score each result: `(0.5 × vector_similarity) + (0.3 × fts_relevance) + (0.2 × recency_score)`
5. Return top `max_items` as formatted MemoryItem objects.

`update_project_memory(section: str, key: str, value: dict)` — Updates the ProjectMemory JSON file atomically. Maintains a flat dict structure per section for fast access.

`get_project_memory_section(section: str) → List[dict]` — Returns all items in a project memory section. Used by the context assembly pipeline for Stage 2.

`compress_episodic()` — Runs weekly. Groups all events older than 7 days by day. Creates a daily_summary event for each day (with narrative summarizing the day's work) and deletes the individual events (except Critical and Decision events which are never deleted). Updates the ChromaDB index.

**SQLite schema for Episodic Memory:**
```sql
CREATE TABLE events (
    event_id TEXT PRIMARY KEY,
    event_type TEXT NOT NULL,
    timestamp TEXT NOT NULL,
    subject TEXT NOT NULL,
    narrative TEXT NOT NULL,
    technical_detail TEXT,   -- JSON
    significance_score REAL,
    tags TEXT,               -- JSON array
    project_id TEXT NOT NULL
);
CREATE VIRTUAL TABLE events_fts USING fts5(
    event_id UNINDEXED,
    narrative,
    subject,
    tags,
    content='events',
    content_rowid='rowid'
);
```

**Working Memory management:** Each active agent's Working Memory is a Python dict stored in the State Store (in-memory Python dict, also serialized to the checkpoint). The Memory Engine provides `get_working_memory(agent_id)`, `update_working_memory(agent_id, updates)`, and `clear_working_memory(agent_id)` methods.

---

### `core/knowledge.py`

**Purpose:** Knowledge Graph — NetworkX in-memory graph with SQLite persistence, 5 query types, and growth from project learnings.

**`KnowledgeGraph` class:**

`load()` — Loads from SQLite into NetworkX DiGraph. Called at startup.

`save()` — Serializes NetworkX graph to SQLite. Called after every mutation.

`query(query_type: str, params: dict) → KnowledgeQueryResult`:
- `TechCompatibilityCheck`: For each pair of technologies in `params["tech_set"]`, check for CONFLICTS_WITH edges. Return all conflicts with descriptions.
- `PatternRecommendation`: Embed `params["problem"]`, find similar Problem nodes via SQLite FTS, traverse SOLVES edges to Solution nodes, filter by tech compatibility, rank by effectiveness.
- `TechPairing`: From `params["technology"]` node, traverse COMMONLY_PAIRED_WITH edges. Return sorted by edge weight (pairing confidence).
- `SecurityCheck`: Find Technology node for `params["technology"]`. Traverse CAUSES edges to Problem nodes with type=SECURITY. Filter by version relevance.
- `LessonRetrieval`: Find Project nodes similar to current project (by domain, scale, tech overlap). Traverse LEARNED_FROM edges. Return top-5 knowledge items.

`update(nodes: List[dict], edges: List[dict])` — Adds new nodes and edges. Called by Knowledge Agent after project completion.

**Seed data loading:** On first startup (no SQLite file exists), loads `data/knowledge_seed.json` which contains the initial 200 nodes and 400 edges. This seed covers all technologies in AACS-Core's initial skill library and their common relationships.

---

### `core/github.py`

**Purpose:** Complete GitHub integration — repository management, branch strategy, commit discipline, CI monitoring, PR management, release creation.

**`GitHubBridge` class:**

`init_repo(project_name: str, description: str) → RepoInfo` — Creates repository via PyGitHub. Configures branch protection on main. Creates develop branch. Generates .gitignore from tech stack. Creates CI workflow YAML. Makes initial commit. Pushes to GitHub. Returns repo URL and clone URL.

`generate_ci_workflow(tech_stack: TechStack) → str` — Returns CI workflow YAML string. Template selection based on primary language:
- Python → pytest + mypy + ruff + pip-audit + build-verify
- Node.js → jest + tsc + eslint + npm-audit + build-verify
- React frontend → vitest + tsc + eslint + build-verify
Multi-service projects get a matrix strategy. All workflows cache dependencies.

`commit_and_push(task_id: str, task_title: str, files_changed: List[str], working_dir: str)` — Stages changed files, builds Conventional Commit message (`feat(scope): description [TASK-ID]`), commits, pushes to current feature branch.

`create_pr(feature_branch: str, task_summary: str, changes: List[str]) → PRInfo` — Creates PR from feature branch to develop. Fills PR template with: summary paragraph, changes list, testing notes, task references.

`get_ci_status(branch: str) → CIStatus` — Polls GitHub Actions API for latest run. Returns structured status with per-job results.

`get_ci_failure_details(run_id: str) → FailureDetails` — Downloads run logs, identifies the failing job and step, extracts the specific error message, returns a structured failure report for the Debugger agent.

`create_release(version: str, changes: List[str]) → ReleaseInfo` — Creates release tag, generates release notes from git log (groups by Conventional Commit type), creates GitHub Release.

**Polling:** The `start_ci_monitor(branch: str)` method starts an asyncio background task that polls CI status every 30 seconds. When status changes to "failure", it emits a CI_FAILURE event to the Message Bus. When status changes to "success", it emits CI_SUCCESS.

---

### `core/state.py`

**Purpose:** Central state store, Message Bus, Tool Registry, and event system. The coordination layer.

**`StateStore` class:** An in-memory dict-backed store with atomic reads and writes. Persisted to `~/.aacs-core/projects/<id>/state.json` on every write. Key namespaces: `project.*` (project-level state), `agent.*` (per-agent state), `task.*` (per-task state), `resource.*` (budget, concurrency, health).

**`MessageBus` class:** An asyncio-based priority queue with SQLite persistence. Messages are written to SQLite before delivery (at-least-once guarantee). Undelivered messages from previous sessions are replayed on startup. Priority queue ensures Critical messages are processed first.

`publish(message: Message)` → Writes to SQLite, adds to in-memory priority queue.

`subscribe(agent_type: AgentType, callback: Callable)` → Registers a handler for messages addressed to an agent type.

**`ToolRegistry` class:** The registry that makes tools available to agents.

`register_tools()` → Called at startup. Imports all tool functions from `tools/*.py` and registers them with typed input/output schemas. Each tool gets: a unique name, an input validator (Pydantic model), an output type, a permission requirement, and a health probe.

`execute(tool_name: str, params: dict, agent_id: str) → ToolResult` → Permission check → input validation → execute function → output validation → log → return result.

**`EventSystem` class:** Simple pub/sub. Components subscribe with `on(event_type, callback)`. Components publish with `emit(event_type, data)`. The WebUI's WebSocket bridge subscribes to all event types and forwards them to connected clients.

---

### `core/checkpoint.py`

**Purpose:** Checkpoint creation, verification, restoration, and scheduled operation.

**`CheckpointManager` class:**

`create(label: str = None) → CheckpointResult` — Creates a complete checkpoint:
1. Pause agent dispatch (send PAUSE_DISPATCH to Orchestrator).
2. Wait for all agents to reach a stable state (complete current atomic op).
3. Flush VFE to disk (call `vfe.sync_to_disk(scope="all")`).
4. Serialize: State Store → JSON, Message Bus queues → JSON, all agent Working Memory → JSON, Episodic Memory delta → JSON.
5. Write all serialized components to a temp checkpoint directory.
6. Compute SHA256 of each component file.
7. Write manifest JSON with all hashes.
8. Atomic rename of temp directory to final name (`checkpoint_{timestamp}_{label}/`).
9. Read back manifest and verify all hashes.
10. Resume agent dispatch.
11. Delete oldest checkpoint if more than 5 exist.

`restore(checkpoint_id: str) → RestoreResult` — Restores from checkpoint:
1. Verify checkpoint integrity (check all hashes in manifest).
2. Load State Store from checkpoint JSON.
3. Rebuild VFE from disk (files were synced before checkpoint, disk = authoritative).
4. Replay Message Bus messages from checkpoint JSON.
5. Reload all agent Working Memory from checkpoint JSON.
6. Restart all managed processes (development servers, etc.).
7. Verify system health (LLM connectivity, GitHub connectivity, disk space).
8. Generate resume summary.

`schedule(interval_minutes: int)` → Starts an asyncio background task that calls `create()` every `interval_minutes`. Also creates checkpoints at every phase gate transition (called by the Orchestrator).

---

### `core/exceptions.py`

**Purpose:** All AACS-Core custom exception classes organized by category.

Key exception classes: `LLMError`, `RateLimitError`, `AuthError`, `BudgetExhaustedError`, `ContextTooLongError`, `VFETransactionConflict`, `VFESyncError`, `ShellPermissionDenied`, `ShellCommandFailed`, `ShellProcessTimeout`, `GitOperationFailed`, `CIFailure`, `GithubRateLimitError`, `TaskFailed`, `LoopDetected(grade)`, `PhaseGateFailed(criteria_failed)`, `HumanInputRequired(question, options)`, `AgentHeartbeatTimeout`, `CheckpointCorrupted`, `SystemHealthCritical`.

Each exception carries enough context for the Error Shield to determine the appropriate recovery procedure without inspecting exception internals.

---

### `core/types.py`

**Purpose:** All shared enums and type definitions used across the system.

Key enums: `AgentType` (all 23 agent identifiers), `TaskType` (CODE_GENERATION, CODE_REVIEW, DEBUGGING, TESTING, PLANNING, ARCHITECTURE, DOCUMENTATION, CONFIGURATION, INTEGRATION, DELIVERY), `Phase` (INTAKE, ARCHITECTURE, INFRASTRUCTURE, DEVELOPMENT, INTEGRATION, QA, AUDIT, DELIVERY, DELIVERED), `Priority` (CRITICAL, HIGH, NORMAL, LOW), `SegmentType` (8 values), `SyncStatus`, `ProcessStatus`, `ResponseStatus`, `ConfidenceLevel`, `EffortSize`, `EpisodeType`, `ProjectDomain`, `ProjectScale`, `Permission` (STANDARD, ELEVATED, ADMIN).

---

## Agent Files — `agents/`

### `agents/base.py`

**Purpose:** Base class for all 23 agents. Implements the complete 8-stage Cognitive Cycle. All agents extend this class and override their specific configuration.

**`BaseAgent` class:**

Constructor args: `agent_type`, `role_description`, `quality_checklist`, `default_skills`, `permitted_tools`.

`run(task: TaskDefinition) → TaskResult` — Executes the complete 8-stage cognitive cycle. This is the method called by the Orchestrator when dispatching a task.

Stage implementations:
- Stage 1: `_stage_reception(task)` — Scope check, missing info detection, confidence assessment.
- Stage 2: `_stage_plan(task)` — LLM call to produce an ordered execution plan. Stored in Working Memory.
- Stage 3: `_stage_context(task)` — Submits ContextRequest to LLM Engine, waits for assembled context.
- Stage 4: `_stage_execute(task, context, plan)` — Iterates through plan steps. Each step: LLM call or tool call. Updates Working Memory after each step. Checks iteration_count against loop prevention thresholds.
- Stage 5: `_stage_self_review(output)` — Secondary LLM call evaluating the output against `self.quality_checklist`. Returns list of ReviewFinding objects. Agent fixes all Critical and High findings inline.
- Stage 6: `_stage_verify(artifacts)` — Calls `build_verify_syntax` and `build_run_type_checker` (if applicable) on all produced code files.
- Stage 7: `_stage_impact(artifacts)` — Calls `ccs_get_impact_analysis` for all modified VFE paths.
- Stage 8: `_stage_report(results) → TaskResult` — Packages all results, concerns, findings into TaskResult and sends to Orchestrator.

**Heartbeat:** The base class sends a heartbeat to the State Store every 30 seconds while a task is running. The Orchestrator monitors these heartbeats and flags agents that miss 3 consecutive heartbeats.

**Loop prevention:** After each iteration in Stage 4, the base class checks `working_memory['iteration_count']` against the grade thresholds. If a threshold is crossed, the appropriate Loop Prevention response is triggered (context refresh, forced alternative, etc.) before the next iteration.

---

### Individual Agent Files (`agents/orchestrator.py` through `agents/human_interface.py`)

Each agent file follows this pattern:

```python
from agents.base import BaseAgent

class BackendDeveloperAgent(BaseAgent):

    ROLE_DESCRIPTION = """
    You are the Backend Developer for this project. You implement all server-side code...
    [Complete role grounding — who you are, what you must produce, what quality standards apply]
    """

    QUALITY_CHECKLIST = [
        {"id": "BD-001", "severity": "Critical",
         "item": "All API inputs validated with Pydantic/Zod models",
         "how_to_check": "Verify every route handler has a typed request body parameter"},
        {"id": "BD-002", "severity": "Critical",
         "item": "Authentication enforced on all protected endpoints",
         "how_to_check": "Check every non-public route has auth dependency"},
        # ... 8-12 checklist items
    ]

    DEFAULT_SKILLS = ["python_fastapi", "postgresql", "authentication"]
    # Skills are loaded by name — the LLM Engine reads the corresponding .md files

    PERMITTED_TOOLS = ["filesystem", "code_analysis", "build_test", "search", "monitoring"]
    # Categories of tools this agent can use

    def __init__(self):
        super().__init__(
            agent_type=AgentType.BACKEND_DEV,
            role_description=self.ROLE_DESCRIPTION,
            quality_checklist=self.QUALITY_CHECKLIST,
            default_skills=self.DEFAULT_SKILLS,
            permitted_tools=self.PERMITTED_TOOLS
        )
```

This pattern means each agent file is ~50-100 lines: the role definition, quality checklist, default skills, permitted tools, and any agent-specific method overrides (some agents override `_stage_execute` to add specialized behavior like the Debugger's savepoint-before-fix protocol).

The 23 agent files are:
```
agents/
├── base.py                        # Base class with 8-stage cognitive cycle
├── orchestrator.py                # ORCH — central coordinator
├── human_interface.py             # HIA — user communication
├── project_manager.py             # PM — task backlog, milestones, planning
├── architect.py                   # ARCH — architecture, tech stack, DB schema
├── backend_developer.py           # BEND — all backend code
├── frontend_developer.py          # FEND — all frontend code
├── database_developer.py          # DBA — migrations, ORM models, queries
├── auth_developer.py              # AUTH — auth system implementation
├── config_developer.py            # CFG — Docker, environment, deployment config
├── test_writer.py                 # TEST — unit + integration tests
├── e2e_tester.py                  # E2E — end-to-end tests (Playwright)
├── debugger.py                    # DBG — bug fixing, CI failure resolution
├── code_reviewer.py               # REVIEW — standards enforcement
├── security_checker.py            # SEC — security scanning
├── optimizer.py                   # OPT — performance optimization
├── auditor.py                     # AUD — final quality audit
├── github_manager.py              # GIT — all GitHub operations
├── docs_writer.py                 # DOCS — README, API docs, changelog
├── delivery_agent.py              # DEL — project packaging and delivery
├── knowledge_agent.py             # KNOW — knowledge graph management
├── memory_keeper.py               # MEM — memory system management
├── integration_tester.py          # INTT — integration + contract testing
└── performance_tester.py          # PERF — load testing + benchmarks
```

---

## Tool Files — `tools/`

### `tools/filesystem.py`
Implements all 10 VFE and disk tools: vfe_create_file, vfe_read_file, vfe_read_segment, vfe_write_segment, vfe_add_segment, vfe_replace_file, vfe_get_file_tree, vfe_search_symbol, vfe_diff_versions, disk_sync_now. Each tool is a standalone async function. The Tool Registry maps tool names to these functions.

### `tools/code_analysis.py`
Implements all 6 code analysis tools: ccs_get_symbol_definition, ccs_get_impact_analysis, ccs_get_api_endpoints, ccs_get_orm_models, ccs_measure_complexity, ccs_find_unused_symbols, ccs_find_circular_imports. These tools query the VFE's code context index (maintained in `core/vfe.py`) rather than running subprocess commands.

### `tools/build_test.py`
Implements all 13 build and test tools: build_install_dependencies, build_verify_syntax, build_run_linter, build_run_type_checker, build_run_security_scanner, build_run_migration, build_run_migration_check, test_run_suite, test_run_single, test_run_e2e, test_run_performance, test_get_coverage_report, build_run_build. These tools use the Shell Engine for execution and the Shell Engine's output parsers for structured result extraction.

### `tools/git_tools.py`
Implements all 10 git/GitHub tools: git_status, git_add_and_commit, git_push, git_create_branch, git_merge_branch, github_create_repo, github_create_pr, github_get_ci_status, github_get_run_logs, github_create_release. These tools use the GitHub Bridge (`core/github.py`) for GitHub API operations and GitPython for local git operations.

### `tools/search.py`
Implements all 7 search and knowledge tools: search_official_docs, search_error_solutions, search_pypi, search_npm, search_cve_database, knowledge_query, knowledge_update. The search tools use httpx for external HTTP requests. The knowledge tools use the Knowledge Graph (`core/knowledge.py`).

### `tools/monitoring.py`
Implements all 7 monitoring and admin tools: monitor_system_health, monitor_project_metrics, monitor_budget_status, admin_install_tool, admin_start_service, admin_set_env_var, comms_notify_human. The admin tools call the Shell Engine with ADMIN permission. comms_notify_human sends to both the WebUI (via the Event System) and prints to terminal.

---

## Skill Files — `skills/`

Seven markdown files covering all primary domains. Each is a standalone knowledge document following the 6-section structure (Overview, Project Setup, Core Patterns, Error Handling, Security Considerations, Testing Approach, Common Mistakes). Each file is approximately 2,000-4,000 words — comprehensive enough to be immediately actionable, concise enough to fit within context windows.

```
skills/
├── python_fastapi.md              # FastAPI, Pydantic v2, SQLAlchemy, Alembic, pytest
├── node_express.md                # Express.js, Node.js async, Prisma, Jest, TypeScript
├── react_frontend.md              # React 18, React Query, Zustand, Tailwind, Vitest
├── postgresql.md                  # Schema design, indexing, query optimization, N+1
├── authentication.md              # JWT, OAuth 2.0, RBAC, bcrypt, sessions, security headers
├── docker_deployment.md           # Dockerfile, docker-compose, GitHub Actions deploy
└── general_coding.md              # Universal standards: naming, errors, logging, testing philosophy
```

---

## WebUI Files — `webui/`

### `webui/server.py`

**Purpose:** FastAPI application factory. Creates and configures the FastAPI app with all routes, middleware, WebSocket endpoint, and static file serving.

**Key setup:**
- CORS middleware configured for LAN access (allows all origins on the configured port).
- JWT auth middleware protects all `/api/*` routes.
- Static files mounted at `/` serving `webui/frontend/index.html` and assets.
- WebSocket endpoint at `/ws/events` (authenticated via query parameter `token`).
- All routers from `webui/routes.py` included.
- On startup: validates the WebUI password hash exists (redirects to setup if not).

**WebSocket implementation:** Each connected client gets a unique connection ID. The WebUI subscribes to all Event System events and forwards them as JSON messages to all active WebSocket connections. Heartbeat messages are sent every 30 seconds to keep connections alive. On reconnect, the client receives a `state_snapshot` message with current project status before resuming the event stream.

### `webui/routes.py`

**Purpose:** All REST API endpoints.

```python
# Project endpoints
GET  /api/projects              → List all projects with status
POST /api/projects              → Create new project (accepts {brief: str})
GET  /api/projects/{id}/status  → Current project status (lightweight)
POST /api/projects/{id}/pause   → Pause current project
POST /api/projects/{id}/resume  → Resume paused project

# Agent endpoints
GET  /api/agents                → Active agents with current task

# Code endpoints
GET  /api/code/tree             → VFE file tree
GET  /api/code/file             → File content (query param: path, version)
GET  /api/code/diff             → Diff between versions (query params: path, v1, v2)

# Test endpoints
GET  /api/tests/results         → Latest test run results
GET  /api/tests/coverage        → Current coverage report

# GitHub endpoints
GET  /api/github/status         → Current CI status + recent commits
GET  /api/github/prs            → Open pull requests

# Memory endpoints
GET  /api/memory/project        → Project memory (all sections)
GET  /api/memory/episodic       → Recent episodic events (query params: since, type, limit)

# Chat endpoints
GET  /api/chat/history          → Chat message history
POST /api/chat/message          → Send message to Human Interface Agent
GET  /api/chat/pending          → Any pending Human Required questions

# System endpoints
GET  /api/system/health         → System health report
GET  /api/system/budget         → Budget status
POST /api/checkpoint            → Create manual checkpoint
GET  /api/checkpoints           → List checkpoints
POST /api/checkpoints/{id}/restore → Restore from checkpoint

# Config endpoints
GET  /api/config                → Sanitized config (secrets masked)
PUT  /api/config                → Update config values
```

### `webui/frontend/index.html`

**Purpose:** The complete single-page application — a single HTML file with embedded JavaScript and CSS (using CDN-loaded Vue.js or Alpine.js for reactivity, and Tailwind CSS CDN for styling). No build step required.

**Layout:** Three-column layout with: left sidebar (navigation, project selector, system health indicator), main content area (switches between Dashboard, Code, Chat, Tests, GitHub, Memory, Settings views), and a collapsible right panel (agent activity / context-sensitive detail).

**Chat interface:** A persistent chat bar at the bottom of the screen always visible. Messages displayed in a scrollable conversation view. "Human Required" notifications appear as highlighted cards at the top of the chat view.

**File tree:** The Code view shows a collapsible file tree on the left. Clicking a file opens it in a syntax-highlighted viewer on the right. Files have colored status dots matching the VFE sync status and review state.

**Activity feed:** A real-time stream in the Dashboard view. Each event type has a distinct icon and color. The feed auto-scrolls but stops auto-scrolling if the user has scrolled up (resumes auto-scroll when the user scrolls back to bottom).

**WebSocket integration:** Vanilla JavaScript WebSocket client. On message received, the relevant UI component is updated in-place using Vue.js/Alpine.js reactive data binding. Connection status shown as a small indicator in the header.

---

## Data Files — `data/`

### `data/knowledge_seed.json`

Initial Knowledge Graph data. Structure:
```json
{
  "nodes": [
    {"id": "python", "type": "Technology", "name": "Python",
     "properties": {"ecosystem": "general", "maturity": "high", "primary_uses": ["backend", "data", "scripts"]}},
    {"id": "fastapi", "type": "Technology", "name": "FastAPI",
     "properties": {"language": "python", "domain": "web-backend", "maturity": "high"}},
    ...
  ],
  "edges": [
    {"source": "fastapi", "target": "pydantic", "type": "USES",
     "properties": {"version_constraint": ">=2.0"}},
    {"source": "fastapi", "target": "sqlalchemy", "type": "COMMONLY_PAIRED_WITH",
     "properties": {"synergy": "async SQLAlchemy for non-blocking DB access"}},
    ...
  ]
}
```

Contains approximately 200 technology nodes covering Python, JavaScript, TypeScript, Go, common frameworks, databases, and tools, plus 400 edges documenting compatibility and pairing relationships.

### `data/agent_configs.json`

Static configuration data for each agent type: default concurrency limits, maximum task size (effort estimate ceiling for single assignment), escalation thresholds (custom overrides of the default loop prevention grades), and resource allocation weights (how much of the token budget each agent type is allowed to consume per hour).

---

## Complete File Listing — All 50 Files

```
aacs-core/
│
├── main.py                            # System entry point and startup sequence
├── cli.py                             # Click CLI — all 15+ terminal commands
├── config.py                          # Configuration schema, loading, credential vault
├── requirements.txt                   # All pinned production dependencies
│
├── core/
│   ├── llm.py                         # LLM Engine: 6-stage CAP, 4-stage RRP, streaming
│   ├── shell.py                       # Shell Engine: cross-platform, admin mode, output parsing
│   ├── vfe.py                         # Virtual Filesystem Engine: segments, transactions, sync
│   ├── memory.py                      # Memory Engine: 3 tiers, hybrid retrieval, compression
│   ├── knowledge.py                   # Knowledge Graph: NetworkX, 5 query types, growth
│   ├── github.py                      # GitHub Bridge: full repo/branch/PR/CI/release management
│   ├── state.py                       # State Store + Message Bus + Tool Registry + Event System
│   ├── checkpoint.py                  # Checkpoint creation, verification, restoration, scheduling
│   ├── exceptions.py                  # All 25+ custom exception classes organized by category
│   └── types.py                       # All shared enums and type definitions
│
├── agents/
│   ├── base.py                        # BaseAgent: complete 8-stage cognitive cycle implementation
│   ├── orchestrator.py                # ORCH: task queue, phase gates, loop prevention coordination
│   ├── human_interface.py             # HIA: all user communication, message classification
│   ├── project_manager.py             # PM: task backlog, milestones, timeline, risk
│   ├── architect.py                   # ARCH: architecture, tech stack, DB schema, integration spec
│   ├── backend_developer.py           # BEND: API endpoints, business logic, services
│   ├── frontend_developer.py          # FEND: React components, pages, state, API client
│   ├── database_developer.py          # DBA: migrations, ORM models, repositories, query opt
│   ├── auth_developer.py              # AUTH: JWT, OAuth, sessions, RBAC, password security
│   ├── config_developer.py            # CFG: Docker, environment, deployment config
│   ├── test_writer.py                 # TEST: unit tests, integration tests, test infrastructure
│   ├── e2e_tester.py                  # E2E: Playwright E2E tests for critical user journeys
│   ├── debugger.py                    # DBG: root cause analysis, bug fixing, CI failure resolution
│   ├── code_reviewer.py               # REVIEW: standards enforcement, pattern compliance
│   ├── security_checker.py            # SEC: OWASP scanning, CVE checking, secrets detection
│   ├── optimizer.py                   # OPT: performance analysis and improvement
│   ├── auditor.py                     # AUD: comprehensive final audit across 5 pillars
│   ├── github_manager.py              # GIT: all GitHub operations (uses core/github.py)
│   ├── docs_writer.py                 # DOCS: README, API docs, deployment guide, changelog
│   ├── delivery_agent.py              # DEL: project packaging and delivery report
│   ├── knowledge_agent.py             # KNOW: knowledge graph management and external research
│   ├── memory_keeper.py               # MEM: memory system management (uses core/memory.py)
│   ├── integration_tester.py          # INTT: integration + contract tests + E2E full stack
│   └── performance_tester.py          # PERF: load testing, benchmarking, performance reporting
│
├── tools/
│   ├── filesystem.py                  # 10 VFE and disk sync tools
│   ├── code_analysis.py               # 6+1 code context and analysis tools
│   ├── build_test.py                  # 13 build, lint, type check, security, test tools
│   ├── git_tools.py                   # 10 git and GitHub API tools
│   ├── search.py                      # 7 search, knowledge graph, and documentation tools
│   └── monitoring.py                  # 7 monitoring, admin, and notification tools
│
├── skills/
│   ├── python_fastapi.md              # FastAPI + Pydantic v2 + SQLAlchemy + pytest
│   ├── node_express.md                # Express.js + Prisma + Jest + TypeScript
│   ├── react_frontend.md              # React 18 + React Query + Zustand + Tailwind + Vitest
│   ├── postgresql.md                  # Schema design + indexing + query optimization
│   ├── authentication.md              # JWT + OAuth 2.0 + RBAC + bcrypt + security headers
│   ├── docker_deployment.md           # Dockerfile + docker-compose + GitHub Actions deploy
│   └── general_coding.md              # Universal standards: naming, errors, logging, tests
│
├── webui/
│   ├── server.py                      # FastAPI app factory: routes, middleware, WebSocket, static
│   ├── routes.py                      # All REST API endpoints (30+ routes)
│   └── frontend/
│       └── index.html                 # Complete single-page WebUI (no build step required)
│
└── data/
    ├── knowledge_seed.json            # Initial Knowledge Graph: 200 nodes, 400 edges
    └── agent_configs.json             # Per-agent concurrency, escalation, resource config
```

**Total: 50 files** (root: 4, core: 10, agents: 24, tools: 6, skills: 7, webui: 3, data: 2) — Exactly 50.

---

## Implementation Notes

### Dependency Injection

AACS-Core uses a simple dependency injection pattern. All engine instances (LLM Engine, Shell Engine, VFE, Memory Engine, Knowledge Graph, GitHub Bridge) are created once in `main.py` and passed as parameters to the Orchestrator constructor. The Orchestrator holds references to all engines and passes them to agents when dispatching tasks. Agents do not instantiate engines — they use what they are given. This makes testing straightforward and allows engine swapping without agent changes.

### Async Architecture

The entire system is async-first. All agent `run()` methods are async. All tool functions are async. All engine methods that do I/O are async. The asyncio event loop in `main.py` runs all concurrent agent tasks as asyncio tasks, which means true concurrency for I/O-bound work (LLM calls, shell execution, disk I/O) without threading complexity.

For CPU-bound work (tree-sitter parsing, embedding generation), AACS-Core uses `asyncio.run_in_executor(None, ...)` to run in the default thread pool executor, preventing blocking of the event loop.

### Error Propagation

Errors propagate upward through the error shield layers. A `ShellCommandFailed` raised in a tool is caught by the Tool Registry (Layer 1 — Component Shield), which retries or returns a ToolError. A persistent ToolError is caught by the agent's `_stage_execute` method (Layer 2 — Agent Shield), which tries an alternative approach or marks the step as failed. A step failure is caught by `base.py`'s `run()` method (Layer 2 — Agent Shield escalation), which sends TASK_FAILED to the Orchestrator. The Orchestrator (Layer 3 — Orchestrator Shield) applies the appropriate recovery procedure.

This layered propagation means no single failure reaches the user without being processed through at least two recovery attempts.

### Testing AACS-Core

AACS-Core includes a test suite in `tests/` (not counted in the 50-file limit as these are dev files):

`tests/test_vfe.py` — VFE unit tests: segment CRUD, transactions, conflict resolution, hash verification.
`tests/test_memory.py` — Memory unit tests: store/retrieve, compression, cross-tier retrieval.
`tests/test_llm.py` — LLM Engine unit tests: context assembly, response processing, error handling (all with mock LLM provider).
`tests/test_agents.py` — Agent unit tests: cognitive cycle stages, self-review, verification (with mock tools and LLM).
`tests/test_e2e.py` — Full integration test: submits a minimal project brief, runs through all phases, verifies delivery (requires live LLM API key — used in CI).

---

*AACS-Core Codebase Structure — Version 1.0*
*50 files. 23 agents. Full autonomy.*
*Every file explained. Every dependency documented. Every design decision justified.*
