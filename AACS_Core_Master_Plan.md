# AACS-Core — Autonomous Coding System Core
## Master Architecture, Planning & Engineering Specification
### Version 1.0 — The Small Avatar of AACS

---

> AACS-Core is the compact, production-grade avatar of the full AACS system. It delivers the same autonomous software development capability — deep planning, full-stack coding, testing, debugging, auditing, GitHub management, and delivery — within a 50-file, 23-agent architecture that a single developer can deploy, understand, and extend. AACS-Core is not a reduced version. It is a concentrated version. Every feature present is complete and production-quality. What was distributed across 2,000 files in full AACS is compressed into intelligent, multi-role agents and consolidated modules without sacrificing correctness, reliability, or depth.

---

# PART ONE: FOUNDATIONS

---

## Chapter 1: Why AACS-Core Exists

### 1.1 The Gap

The full AACS system is architecturally complete but operationally massive. At ~2,000 files and 126 agent types, it demands significant infrastructure, longer build times, and deep familiarity before a developer can begin extending it. For a developer who wants to start building autonomous coding capability today — and scale to the full system later — AACS-Core is the bridge.

AACS-Core operates under a single guiding constraint: **maximum capability within 50 files and 23 agents.** Every design decision serves this constraint. Agents are multi-role rather than hyper-specialized. Modules are consolidated rather than split. Skills are single documents rather than multi-part packages. But the intelligence, the workflows, the quality standards, and the autonomous delivery contract remain intact.

### 1.2 What AACS-Core Delivers

A user hands AACS-Core a project brief — in natural language, as detailed or as rough as they choose — and AACS-Core produces:

A complete, working software product committed to a GitHub repository with clean history. A professional README and API documentation. A test suite with meaningful coverage. A passing CI/CD pipeline. A delivery report explaining what was built. The ability to continue the project after delivery through the same chat interface.

AACS-Core delivers this while running on a single machine, using the user's OpenAI-compatible LLM API key and GitHub PAT, accessible from any device on the LAN through a browser.

### 1.3 Design Philosophy

**Consolidation without compromise.** A multi-role agent that handles backend API development, auth, and data access is not worse than three separate agents — it is more efficient because it carries all the context needed for those concerns simultaneously, without inter-agent communication overhead.

**Shallow hierarchy, deep intelligence.** AACS-Core has one coordination layer (the Orchestrator) and one execution layer (all other agents). This flat structure means less message routing, less coordination overhead, and faster task execution. Depth lives in agent cognition, not organizational structure.

**Real systems, not mocks.** The Virtual Filesystem is real. The Memory System is real. The Knowledge Graph is real. The GitHub integration is real. Nothing is a placeholder. AACS-Core is a production system at a compact scale.

**Expandable foundation.** Every design choice is made with the full AACS upgrade path in mind. When the user is ready to scale, AACS-Core's modules expand into full AACS modules without a rewrite. The agent role definitions in AACS-Core are subsets of the full AACS definitions.

---

## Chapter 2: System Overview

### 2.1 What AACS-Core Is

AACS-Core is a Python-based autonomous software development system with the following operational profile:

It runs on Windows (PowerShell 7 in Administrator mode) and Linux (bash with sudo). It accepts a natural language project brief through a chat-style WebUI or terminal. It executes the complete software development lifecycle autonomously: requirements analysis, architecture, coding, testing, debugging, optimization, auditing, and delivery. It manages a GitHub repository throughout — creating it, managing branches, creating PRs, configuring CI/CD, monitoring CI runs, and debugging CI failures. It maintains persistent project memory across sessions. It checkpoints its state every 30 minutes and resumes from checkpoints after interruption. It allows unlimited post-delivery follow-up through the same chat interface.

### 2.2 User Configuration

AACS-Core requires exactly three pieces of user configuration:

**LLM Provider:** The user provides a Base URL (e.g., `https://api.openai.com/v1`), an API Key, and a Model ID. Any OpenAI-compatible endpoint works — OpenAI, Anthropic via proxy, local Ollama, Together.ai, Groq, or any custom endpoint that speaks the OpenAI chat completions protocol.

**GitHub:** The user provides a Personal Access Token with repo and workflow scopes, and their GitHub username or organization name.

**System Mode:** The user chooses whether to enable Administrator mode (full system control) or restricted mode (project directory only). For a dedicated development machine, Administrator mode is recommended and enables full tool installation, service management, and environment configuration.

These three configurations are entered once through the first-run wizard and stored securely in the system's credential store.

### 2.3 Operational Modes

**Interactive Mode:** The user runs `aacs-core start` and the WebUI launches. The user interacts through the browser interface — submitting project briefs, answering clarification questions, monitoring activity, browsing code, and sending chat messages. The system runs in the background, working continuously.

**CLI Mode:** The user can interact entirely through the terminal with commands like `aacs-core project new --brief "..."`, `aacs-core status`, `aacs-core chat "..."`, and `aacs-core project list`.

**Unattended Mode:** The user starts a project and steps away. AACS-Core works overnight or over multiple days without human interaction, checkpointing regularly, sending a morning summary when the user next opens the WebUI, and completing the project whenever it finishes.

---

## Chapter 3: Architecture

### 3.1 The Five Layers

AACS-Core has five architectural layers, each serving the one above it.

**Layer 1: Substrate.** The host OS (Windows/Linux), the filesystem, the network, and the terminal runtime. AACS-Core interacts with this layer through the Shell Engine in Admin Mode.

**Layer 2: Core Engines.** The five core engines that power everything: LLM Engine (AI communication), Shell Engine (OS command execution), Virtual Filesystem Engine (in-memory codebase), Memory Engine (3-tier persistent memory), and the GitHub Bridge (version control). These engines are always running.

**Layer 3: Intelligence Layer.** The Knowledge Graph (technology knowledge), the Skills Registry (7 markdown-based skill documents), the Error Shield (error handling and loop prevention), the Checkpoint System (state persistence), and the Cognitive Cycle (how agents think and work).

**Layer 4: Agent Layer.** The 23 agents, each running the Cognitive Cycle, using the Core Engines and Intelligence Layer to execute their assigned tasks. Agents communicate through the Message Bus.

**Layer 5: Interface Layer.** The WebUI (browser-based dashboard + real-time chat + file tree), the REST API (for external tool integration), and the Terminal Interface (CLI commands).

### 3.2 Data Flow

When a user submits a project brief, data flows as follows:

Brief enters at Layer 5 (WebUI chat or CLI). The Human Interface Agent (Layer 4) processes it and sends a message to the Orchestrator (Layer 4). The Orchestrator activates the Project Manager agent, which uses the LLM Engine (Layer 2) to produce a structured project specification. The Orchestrator then activates the Architect agent, which designs the system architecture. The Orchestrator dispatches coding tasks to development agents (Backend Dev, Frontend Dev, etc.), which write code to the VFE (Layer 2), which syncs to disk on schedule. As code is written, CODE_READY events trigger the Code Reviewer and Test Writer agents in parallel. The GitHub Bridge (Layer 2) commits code, pushes branches, creates PRs, and monitors CI. If CI fails, the Debugger agent is activated. When all quality gates pass, the Auditor and Delivery agents run. The final delivery is presented to the user through the WebUI (Layer 5).

### 3.3 Concurrency Model

AACS-Core uses Python's asyncio as its runtime. All agent execution is asynchronous. Multiple agents can be active simultaneously — the Orchestrator coordinates which agents are running at any time and enforces dependency ordering (the Backend Dev must finish the data models before the Test Writer writes data access tests for them).

The Orchestrator maintains a task queue. Agents that finish their tasks report back. The Orchestrator dispatches the next ready task to the next available agent. Tasks that can run in parallel (e.g., writing frontend components while backend tests are being written) are dispatched simultaneously to separate agent instances.

The maximum concurrent agent count is configurable (default 3 — balancing LLM rate limits with throughput). On high-rate-limit API plans, this can be increased to 5 or 8 for faster project completion.

---

# PART TWO: CORE ENGINES

---

## Chapter 4: LLM Engine

### 4.1 Responsibilities

The LLM Engine is the AI communication layer. It handles all interactions with the configured LLM provider: assembling contexts, making API calls, processing responses, handling errors, and tracking token usage.

### 4.2 Context Assembly

Every agent call goes through a 6-stage context assembly process. Stage 1 loads the agent's role definition (who this agent is and what it must produce). Stage 2 loads the project context (current project spec, architecture decisions, and established conventions). Stage 3 loads the relevant skill knowledge for the current task type (fetched from the Skills Registry). Stage 4 retrieves relevant memory items from the Memory Engine (recent decisions, past similar tasks, known issues). Stage 5 loads the task-specific code context from the VFE (the specific files and symbols relevant to this task). Stage 6 assembles the final prompt with the task specification and output format instructions.

Token counting is applied to the assembled context. If it exceeds the model's context window (with a 20% safety margin), priority-based truncation is applied: memory items are trimmed first, then skill examples, then code context, keeping the role definition and task specification intact at all times.

### 4.3 Response Processing

LLM responses go through a 4-stage processing pipeline. Stage 1 validates the structural format (if JSON was requested, validate JSON; if code was requested, verify code blocks are present). Stage 2 runs syntax verification on all generated code (ast.parse for Python, acorn for JavaScript, tsc --noEmit for TypeScript). Stage 3 applies the agent's self-review checklist using a secondary fast LLM call. Stage 4 packages the response into a typed ResponseResult with extracted code artifacts, decisions, concerns, and usage statistics.

If Stage 1 or Stage 2 fails, the LLM is re-called with a targeted correction prompt. Maximum 2 correction retries before escalating to the Error Shield.

### 4.4 Streaming

For code files over 150 lines, the LLM Engine uses streaming mode. Streamed chunks are written incrementally to a VFE streaming buffer. When a complete segment (function, class, route handler) is detected in the buffer, it is committed to the VFE as a named segment. If streaming is interrupted, the VFE rolls back to the last committed segment and the LLM is asked to continue from that point.

### 4.5 Cost Tracking

Every LLM call records: tokens used (input and output), cost calculated (tokens × configured cost per token), agent ID, task ID, project ID, and timestamp. A running project budget total is maintained. When the project budget reaches 75%, a warning is shown in the WebUI. When it reaches 95%, the system shifts to conservative mode (uses cheaper/faster model for non-critical tasks like formatting and documentation). When the configured limit is reached, the system pauses and notifies the user.

### 4.6 Error Handling

Eight LLM error types are handled with specific strategies: Rate Limit (exponential backoff with parallel queuing), Server Error (retry with 10-second delay, max 3 times), Context Too Long (aggressive context trimming then retry), Invalid Format (targeted correction prompt), Auth Failure (immediate pause, human notification), Timeout (fail fast, retry with smaller context), Content Policy (rephrase task, log occurrence), and Unexpected Response (escalate to higher tier model).

---

## Chapter 5: Shell Engine

### 5.1 Responsibilities

The Shell Engine executes system commands on the host OS on behalf of agents. In Administrator mode, it has the authority of a developer with full system access. Every command is permission-checked, executed, and logged before results are returned to the requesting agent.

### 5.2 Administrator Mode

When Admin Mode is enabled (the user has launched AACS-Core with elevated privileges and enabled admin_mode in config), the Shell Engine gains the following capabilities beyond basic file operations:

Installing development tools globally (Node.js, Python packages, databases, Docker), managing system services (start/stop PostgreSQL, Redis, etc.), modifying system environment variables permanently, managing the hosts file (for local domain mapping), creating firewall rules for development ports, and managing background system processes. Every administrative operation is logged to a dedicated admin.log file with timestamp, command, justification, and reversibility assessment.

### 5.3 Permission System

Agents have defined permission levels: STANDARD (filesystem within project directory, package install in project, git operations), ELEVATED (system-wide package install, service management), and ADMIN (all operations including hosts, firewall, environment). The Shell Engine checks the requesting agent's permission level before executing any command. Unauthorized commands are blocked and logged.

### 5.4 Cross-Platform

On Windows, the Shell Engine uses PowerShell 7 (pwsh.exe) with a persistent process pool. Commands are built as PowerShell commands when a direct PowerShell equivalent exists, or wrapped in Invoke-Expression for shell scripts. Cross-platform command translation handles common Unix commands transparently.

On Linux, the Shell Engine uses bash with sudo for elevated commands. It detects the available package manager (apt, dnf, pacman, brew) and uses the correct one for tool installations.

### 5.5 Output Intelligence

Shell output is parsed before being returned to agents. Key parsers: pytest output → structured TestRunResult with pass/fail counts and failure details. ESLint/ruff output → LinterReport with file, line, message. mypy/tsc output → TypeCheckReport with typed errors. Git output → structured Git objects. npm/pip install output → InstallResult with success/failure and any vulnerability warnings. For unrecognized commands, exit code and stderr patterns are used for success/failure determination.

### 5.6 Long-Running Processes

Development servers, database instances, and test watchers run as managed background processes with unique IDs. Agents can start, monitor, send signals to, read output from, and stop these processes. The Autonomous Service sub-system (built into the Orchestrator) monitors all running processes and restarts any that die unexpectedly.

---

## Chapter 6: Virtual Filesystem Engine

### 6.1 Why VFE

AI agents writing code through shell redirection is fragile — large files get corrupted, writes get interrupted, context windows can't hold entire files. The VFE is an in-memory filesystem that agents interact with through a clean API. Code lives in VFE during development; the disk receives it through a managed, atomic sync process.

### 6.2 Data Model

The VFE stores files as nodes in an in-memory tree. Each FileNode contains: path, language, content stored as an ordered list of named Segments, version history, ownership (which agent last modified it), sync status, and a SHA256 content hash.

Segments are the unit of granularity. A segment is a named, typed block of code within a file: a Python function, a class definition, an import block, a route handler, an ORM model, a test function, or a generic block. Agents read and write at the segment level when possible — adding a new method to a class is a segment add operation, not a full file rewrite. This prevents the common failure mode where an agent regenerates an entire file and accidentally drops unrelated content.

Eight segment types cover all cases: IMPORT_BLOCK, CLASS_DEF, METHOD_DEF, FUNCTION_DEF, ROUTE_HANDLER, ORM_MODEL, TEST_FUNCTION, and GENERIC. Language-specific parsing using tree-sitter identifies segments when a file is first loaded into VFE.

### 6.3 Transactions

All VFE write operations happen within transactions. An agent opens a transaction, makes changes (create file, write segment, add segment, delete segment), and commits. If anything fails during the transaction, it rolls back to the pre-transaction state. Committed transactions create version entries in the file's version history. The version history enables semantic diffs and rollbacks.

### 6.4 Sync to Disk

VFE-to-disk synchronization uses WriteBehind mode by default: changes are batched and flushed to disk every 30 seconds or when a batch of 10+ changes accumulates. Each sync writes files atomically (temp file + rename). After sync, hash verification confirms the disk content matches VFE. Pre-commit sync uses WriteThrough (every change immediately confirmed on disk) to ensure GitHub sees the final state.

Post-sync verification runs automatically: Python files are syntax-checked with ast.parse, JavaScript/TypeScript files are checked with node --check or tsc --noEmit, and import chains are verified. Any post-sync failure triggers immediate routing to the Debugger agent.

### 6.5 VFE API

Agents access the VFE through 15 typed operations: create_file, read_file, read_segment, write_segment, add_segment, delete_segment, replace_file, delete_file, get_file_tree, search_symbol, search_content, begin_transaction, commit_transaction, rollback_transaction, and diff_versions. These are wrapped as callable tools available to all agents.

### 6.6 Project Loading

When AACS-Core resumes a paused project or starts working on an existing project, the load_from_disk operation reads all project files from disk, parses them into segments using tree-sitter, and populates the VFE. After loading, the Code Context Index is rebuilt from the VFE state, giving all agents an accurate picture of the existing codebase.

---

## Chapter 7: Memory Engine

### 7.1 Three-Tier Architecture

AACS-Core implements a three-tier memory system that gives agents informed, context-aware intelligence throughout the project lifecycle.

**Tier 1: Working Memory.** Each active agent has a working memory — a Python dictionary stored in the State Store containing: the agent's current task definition, its execution plan (ordered steps), completed steps with their outputs, notes made during execution, concerns flagged, and iteration count (for loop detection). Working memory is private to the agent and is serialized to disk as part of checkpoints.

**Tier 2: Episodic Memory.** The project's event timeline. Every significant action is recorded: task completed, bug found, bug fixed, decision made, test failed, architecture changed, human input received. Stored in SQLite with FTS5 full-text search and a ChromaDB vector index for semantic retrieval. Episodic memory answers the question "what happened in this project related to X?" before an agent starts a task.

**Tier 3: Project Memory.** Structured, queryable knowledge about the current project. Organized into sections: Technical Decisions (all architectural and technology choices with rationale), Established Conventions (coding patterns, naming rules, file organization established during the project), Active Issues (known bugs or limitations with their status), Integration Notes (how components connect), and Technology Notes (project-specific knowledge about chosen technologies). Project Memory is updated by agents as they work and is the primary context source for the Context Assembly Pipeline.

### 7.2 Retrieval

When an agent needs memory, it sends a retrieval request with a query string. The Memory Engine performs hybrid retrieval: keyword search via SQLite FTS5 on the episodic store, and vector similarity search via ChromaDB using a locally-running sentence-transformers embedding model (all-MiniLM-L6-v2, ~80MB, runs on CPU). Results from both searches are merged, deduplicated, and ranked by a composite score: 50% semantic relevance + 30% keyword relevance + 20% recency. The top 8 results are returned as formatted memory items for injection into the agent's context.

### 7.3 Persistence

All memory tiers are persisted to the project's data directory. The episodic SQLite database is backed up as part of checkpoints. Project memory is stored as a structured JSON file, updated atomically on each write. The ChromaDB vector index is persisted to a subdirectory and reloaded on session resume.

### 7.4 Cross-Session Continuity

When a user returns to a project after a break (or the next morning after overnight operation), the Memory Engine provides the complete project history. The agent that picks up where work left off has access to every decision made, every bug found and fixed, every convention established, and every concern raised. This is what makes post-delivery continuation feel natural — the system remembers the entire project.

---

## Chapter 8: GitHub Bridge

### 8.1 Scope

The GitHub Bridge is not just a git wrapper. It is a complete GitHub project management system: repository lifecycle, branch strategy, commit discipline, pull request management, CI/CD workflow creation and monitoring, release management, and failure debugging. The GitHub Manager agent (dedicated to GitHub operations) uses the Bridge for all operations.

### 8.2 Repository Management

On project start: The Bridge creates a new private repository named from the project (slugified), sets up branch protection on main (require PR, require status checks), creates the develop branch, generates a .gitignore tailored to the project's tech stack, creates a basic README skeleton, and sets up the initial GitHub Actions CI workflow. The first commit is "chore: initialize repository with AACS-Core project scaffold".

During development: Every feature task gets a dedicated branch (`feature/TASK-XXX-short-description`). When a task is reviewed and verified, its branch is committed and a PR is created from feature → develop. The Bridge monitors all PR statuses.

At milestones: develop is merged into main with a release tag. GitHub Releases are created with auto-generated release notes.

### 8.3 Commit Strategy

AACS-Core follows Conventional Commits. Every commit message is built by the GitHub Manager agent: `{type}({scope}): {description} [TASK-XXX]`. Commits are atomic — one logical change per commit. Commit bodies explain WHY the change was made (the git diff shows the what). This produces a professional, navigable project history.

### 8.4 CI/CD Workflow Generation

The GitHub Bridge generates GitHub Actions workflow YAML files tailored to the project's tech stack. For Python FastAPI projects: lint (ruff), type check (mypy), unit tests (pytest with coverage), security scan (pip-audit + bandit), build verify. For Node.js projects: lint (ESLint), type check (tsc), unit tests (Jest), security scan (npm audit), build verify. For React frontends: lint, type check, component tests (Vitest), build verify.

All generated workflows cache dependencies, use matrix builds where appropriate, and upload test results as artifacts.

### 8.5 CI Monitoring and Failure Debugging

After every push, the Bridge polls the GitHub Actions API every 30 seconds for run status. The WebUI GitHub panel shows live CI status. When a CI run fails, the Bridge downloads the full run logs, parses them using the Shell Engine's output parsers, and routes the failure as a CI_FAILURE task to the Debugger agent. The Debugger analyzes the failure, generates a fix, and the cycle repeats until CI passes. The system never ignores a CI failure — a red CI is treated as a blocking issue.

### 8.6 Issue Management

Significant bugs found during development are tracked as GitHub Issues with AACS labels. When the Auditor finds a finding that is deferred (not blocking delivery but worth noting), it creates a labeled GitHub Issue with full context. This keeps the GitHub repository as the authoritative project management source.

---

# PART THREE: INTELLIGENCE LAYER

---

## Chapter 9: Knowledge Graph

### 9.1 Purpose

The Knowledge Graph gives AACS-Core structured, queryable knowledge about technologies, patterns, and best practices. Unlike flat documentation, the graph represents relationships — which technologies are commonly paired, which patterns solve which problems, which technology versions are incompatible. This enables agents to make informed decisions about technology selection, architectural choices, and implementation approaches without solely relying on the LLM's training data.

### 9.2 Implementation

The Knowledge Graph uses NetworkX as the in-memory graph library with a SQLite persistence layer. The graph loads from a seed file at startup (data/knowledge_seed.json, containing ~200 nodes and ~400 edges covering all common web development technologies) and grows as AACS-Core completes projects.

Node types: Technology (language, framework, library, database, tool), Pattern (design pattern, architectural pattern, integration pattern), Problem (common failure mode, known limitation, security concern), and Solution (proven approach to a problem type).

Edge types: COMMONLY_PAIRED_WITH (technologies that work well together), CONFLICTS_WITH (technologies with known incompatibilities at specific versions), IMPLEMENTS (technology or pattern implements a concept), SOLVES (solution solves a problem), LEARNED_FROM (knowledge derived from a past project), and SUPERSEDED_BY (older technology superseded by newer one).

### 9.3 Queries

Five query types are available to agents through the knowledge_query tool:

TechCompatibilityCheck: Given a set of selected technologies, return any known conflicts. Used by the Architect agent during technology selection.

PatternRecommendation: Given a problem description, return patterns that solve it. Used when an agent needs to implement something and wants to know the established approach.

TechPairing: Given a core technology, return commonly paired technologies. Used to suggest complementary libraries and tools.

SecurityCheck: Given a technology and version, return known CVEs or security concerns. Used by the Security Checker agent.

LessonRetrieval: Return lessons learned from past projects similar to the current one. Used by the Project Manager and Architect agents at the start of a new project.

### 9.4 Growth

After every project completion, the Knowledge Graph is updated with: new technology nodes for any new technologies used, new COMMONLY_PAIRED_WITH edges for technology combinations that proved effective, new LEARNED_FROM edges connecting knowledge items to the project, and any new patterns or solutions discovered during the project. This continuous growth means AACS-Core becomes more informed with each project it completes.

---

## Chapter 10: Skills Registry

### 10.1 Philosophy

Skills are structured knowledge documents that give agents domain-specific expertise beyond their LLM training data. In AACS-Core, each skill is a single markdown file (not the multi-part packages of full AACS — that complexity is not needed at this scale). A skill document is loaded into the agent's context during the Context Assembly Pipeline's skill-loading stage.

### 10.2 Skill Document Structure

Each skill document contains six sections: Overview (what the technology/domain is, when to use it, key characteristics), Project Setup (how to initialize a project with this technology, key dependencies, configuration), Core Patterns (the most important implementation patterns with code examples), Error Handling (technology-specific error handling approaches), Security Considerations (common security pitfalls and their solutions), Testing Approach (how to test code written with this technology, framework-specific test patterns), and Common Mistakes (anti-patterns specific to this technology that agents must avoid).

Skills are written at production engineer level — not tutorials, but the distilled knowledge of someone who has shipped multiple production systems with the technology.

### 10.3 Initial Skill Library (7 Skills)

`python_fastapi.md` — FastAPI, Pydantic v2, SQLAlchemy 2, Alembic, async Python patterns, dependency injection, error handling, pytest testing, OpenAPI generation.

`node_express.md` — Express.js, Node.js async patterns, middleware architecture, Prisma ORM, Jest testing, TypeScript with Express, environment management.

`react_frontend.md` — React 18, TypeScript, React Query (TanStack), Zustand state management, Tailwind CSS, React Testing Library, Vite, component patterns, accessibility basics.

`postgresql.md` — PostgreSQL schema design, indexing strategy, query optimization, N+1 prevention, migration strategy with Alembic/Prisma, connection pooling, JSONB usage.

`authentication.md` — JWT implementation, refresh token rotation, session management, OAuth 2.0 flows, RBAC implementation, password hashing (bcrypt/Argon2), security headers, CSRF protection.

`docker_deployment.md` — Dockerfile multi-stage builds, docker-compose for development and production, environment management, health checks, GitHub Actions deployment workflows, common deployment targets (Railway, Fly.io, DigitalOcean App Platform).

`general_coding.md` — Universal coding standards: naming conventions, error handling patterns, logging standards, code documentation, API design principles, REST conventions, testing philosophy (TDD principles, test naming, mock strategies), Git commit discipline.

### 10.4 Skill Loading

When the Context Assembly Pipeline loads skills for an agent task, it uses the task type and project tech stack to select the most relevant skill(s). A backend API task on a FastAPI project loads `python_fastapi.md` and `postgresql.md`. An authentication task loads `authentication.md`. A deployment task loads `docker_deployment.md`. The `general_coding.md` skill is always loaded as a universal quality baseline for all coding tasks. Maximum 3 skills are loaded simultaneously to avoid context overload.

---

## Chapter 11: Error Shield and Loop Prevention

### 11.1 The Error Shield

AACS-Core wraps all operations in a three-layer error protection system:

**Layer 1: Operation Shield.** Every LLM call, tool call, and file operation is wrapped in a try-catch with typed exception handling. Transient errors (network timeouts, rate limits) trigger automatic retry with backoff. Permanent errors (auth failure, invalid input) escalate to Layer 2.

**Layer 2: Agent Shield.** If an agent's current step fails despite Layer 1 recovery, the Agent Shield applies: try an alternative approach, refresh the agent's context, or escalate to Layer 3.

**Layer 3: Orchestrator Shield.** If an agent cannot complete a task after Layer 2 recovery, the Orchestrator receives a TASK_FAILED message with full context. The Orchestrator applies division-level recovery: reassign to a different agent instance, decompose the task into smaller tasks, or escalate to human.

Circuit breakers protect all external endpoints (the LLM API, GitHub API). After 3 consecutive failures, the circuit opens for 60 seconds, then tests with a single call before fully reopening. This prevents cascade failures during provider outages.

### 11.2 Loop Prevention System

The debug loop — where fixing one bug creates another, which gets "fixed" by reverting toward the original bug, in an infinite cycle — is the most expensive failure mode in autonomous coding. AACS-Core's Loop Prevention System (LPS) prevents this with 5 escalating grades.

**Grade 0 — Context Refresh.** Triggered at iteration 3 on the same task step. The agent's accumulated context is cleared and rebuilt fresh. Most loops are caused by context accumulation (contradictory information building up in the context window). A fresh context usually resolves them. No user notification.

**Grade 1 — Forced Alternative.** Triggered at iteration 5. The agent is explicitly instructed: "Your current approach has not succeeded in N attempts. You must take a completely different approach. Explain in 2 sentences why the current approach failed, then describe your new approach before implementing it." The agent cannot continue the old approach. No user notification.

**Grade 2 — Specialist Escalation.** Triggered at iteration 8. A second agent instance of a different type is spawned to attack the same problem from a different angle. For example, if the Backend Dev is stuck on a database query, the Debugger agent is spawned to analyze the problem independently. The Debugger's analysis is provided to the Backend Dev as authoritative guidance. Brief WebUI notification.

**Grade 3 — Task Decomposition.** Triggered at iteration 12. The task is returned to the Orchestrator with a full failure history. The Orchestrator routes it to the Project Manager agent, which decomposes it into 2-3 smaller, more atomic tasks. Each sub-task is dispatched fresh. WebUI notification with status update.

**Grade 4 — Human Required.** Triggered at iteration 18 or if the cost spent on a single task exceeds 3% of the project budget. The task is paused. A Human Required notification appears prominently in the WebUI with: what was attempted, every approach tried and why it failed, the current state of the code, and three specific options for the user to choose from. The system continues all other unblocked work while waiting. No timeout — AACS-Core waits as long as needed for human input.

### 11.3 Debug-Specific Protections

Before any bug fix is committed to VFE, a savepoint is created. The fix is applied within a VFE transaction. The relevant test suite is run (only the tests for the modified module). If any test that previously passed now fails (regression), the VFE transaction is rolled back immediately. AACS-Core never commits a fix that introduces a regression. This is non-negotiable — the system would rather escalate to Grade 3 decomposition than commit regressing code.

---

## Chapter 12: Checkpoint System

### 12.1 Why Checkpoints Matter

AACS-Core projects often run for hours or overnight. Power loss, unexpected system shutdown, or simple user interruption must never result in lost work. The checkpoint system ensures that every project can always be resumed from the point it was interrupted.

### 12.2 Checkpoint Content

A checkpoint captures the complete system state: the Global State Store (current phase, task statuses, agent states), the VFE state (all files are flushed to disk before checkpointing, so the checkpoint stores metadata only — file paths, version IDs, modification timestamps, and content hashes), all active agent Working Memory objects serialized to JSON, the Message Bus state (all undelivered messages), the Episodic Memory delta (new events since the last checkpoint), and checkpoint metadata (timestamp, project phase, SHA256 hash of all components).

Checkpoints are stored in the project's checkpoint directory. The last 5 checkpoints are retained, older ones are deleted. Each checkpoint is verified by reading it back and checking all SHA256 hashes before the old checkpoint is deleted. A corrupted checkpoint is discarded without deleting its predecessor.

### 12.3 Checkpoint Schedule

Checkpoints are created every 30 minutes during normal operation. Forced checkpoints are created at every phase gate transition, before any major GitHub operation (main branch merge, release creation), and before any administrative operation that modifies system configuration. The user can also request a manual checkpoint at any time via the WebUI or CLI.

Creating a checkpoint pauses agent execution for typically 15-45 seconds (the time needed to flush VFE to disk and serialize all state). Agents are notified of the pause via the Message Bus, complete their current atomic operation, and wait. After checkpoint completion, execution resumes from exactly where it paused.

### 12.4 Resume Protocol

On startup, AACS-Core checks if there is an active project with a valid checkpoint. If yes, it initiates resume: loads the Global State Store, rebuilds VFE from disk (reading all project files and parsing segments), redelivers all pending messages from the Message Bus state, restarts all managed background processes (dev servers, databases), reloads all active agent Working Memory, and verifies system health (LLM connectivity, GitHub connectivity, disk space). A resume summary is generated and shown in the WebUI and terminal: what was in progress, what will be re-run, what was completed before the interruption.

---

# PART FOUR: AGENT SYSTEM

---

## Chapter 13: Agent Cognitive Architecture

### 13.1 The Cognitive Cycle

Every agent in AACS-Core follows the same 8-stage cognitive cycle for every task it executes. This uniformity ensures consistent behavior, predictable quality, and systematic error detection regardless of which agent is running.

**Stage 1: Task Reception.** The agent reads the incoming TaskDefinition. It checks: is this task within my role's scope? Do I have all the information I need? Are there relevant past failures I should know about (checks Working Memory for previous attempts on similar tasks in this project)? Assesses confidence level.

**Stage 2: Plan Construction.** The agent constructs a step-by-step execution plan before doing anything. The plan is stored in Working Memory and updated as execution proceeds. For complex tasks, the plan is validated by the Orchestrator before execution begins.

**Stage 3: Context Loading.** The agent submits a ContextRequest. The Context Assembly Pipeline fetches role grounding, project context, skills, memory, and code context. The assembled context is returned ready for LLM use.

**Stage 4: Execution.** The agent executes its plan step by step. Each step uses LLM calls for code generation or reasoning, and tool calls for file operations, shell commands, or information retrieval. After each step, Working Memory is updated.

**Stage 5: Self-Review.** Before reporting completion, the agent evaluates its own output against its quality checklist. All Critical and High findings must be fixed before proceeding. Medium and Low findings are attached as notes.

**Stage 6: Output Verification.** Mechanical verification of outputs: syntax checks on all code, import verification, type checks where applicable. Verification failures trigger targeted correction (not a full restart).

**Stage 7: Impact Assessment.** The agent requests an impact analysis from the VFE for all modified files: what else might be affected by these changes? What tests are at risk? Are any API contracts changed? This information is attached to the task result.

**Stage 8: Result Reporting.** The agent sends TASK_RESULT to the Orchestrator with: all produced artifacts, verification results, impact assessment, self-review findings, concerns, execution time, and LLM call count.

### 13.2 Agent Working Memory

Each agent's Working Memory during a task contains:
- task_id: the current task
- execution_plan: ordered list of steps with status
- notes: free-form agent notes
- concerns: flagged issues for human or downstream review
- iteration_count: for loop prevention
- context_loaded_at: timestamp for staleness checking
- retry_history: previous attempts and why they failed

Working Memory is updated after every step and is available for inspection in the WebUI's agent detail panel.

---

## Chapter 14: Complete Agent Roster

### 14.1 Overview

AACS-Core has 23 agents. Each is described below with: its responsibilities, the skills it loads, the tools it uses, what triggers its activation, and what it produces.

---

### Agent 1: Orchestrator (ORCH)

**Role:** The central brain of AACS-Core. The Orchestrator never writes code or makes technical decisions. Its job is to keep the project moving: dispatch tasks, track progress, manage the task queue, enforce phase gates, handle escalations, and coordinate all other agents.

**Responsibilities:** Maintains the project task queue with priority ordering. Dispatches tasks to agents based on dependency readiness, agent availability, and priority. Receives TASK_RESULT from all agents and determines next steps. Applies the Loop Prevention System at Grades 3 and 4. Manages phase gate assessment (coordinates multi-agent parallel gate checks). Handles the Emergency Coordination Protocol when multiple tasks fail simultaneously. Generates status events for the WebUI. Manages the overnight operation schedule.

**Activation:** Always active when AACS-Core is running.

**Skills loaded:** general_coding.md (for quality standard awareness).

**Key tools:** message_bus (all operations), monitor_project_metrics, monitor_system_health, checkpoint_create.

**Does NOT use:** VFE write tools, git tools, LLM for code generation. The Orchestrator uses LLM calls only for reasoning tasks (phase gate assessment, escalation analysis, task decomposition).

---

### Agent 2: Human Interface Agent (HIA)

**Role:** The user's primary point of contact within AACS-Core. Manages all communication between the human user and the system.

**Responsibilities:** Processes incoming chat messages from the WebUI. Classifies messages: Is this a project brief (new project)? A clarification response? A scope change request? A question about the project? A direct command? Routes classified messages to the appropriate agent or action. Generates progress reports, morning summaries, phase completion notifications, and milestone announcements. Manages the clarification queue: when AACS-Core needs human input, the HIA formulates the minimal, most actionable question and presents it with structured options where possible. For post-delivery follow-up messages, the HIA classifies them (new feature, bug report, question, scope change) and routes them.

**Activation:** Always active (receives all user-originating messages).

**Skills loaded:** general_coding.md.

**Key tools:** comms_notify_human, memory_retrieve (for answering questions about the project), vfe_get_file_tree (for file-related questions).

---

### Agent 3: Project Manager (PM)

**Role:** Converts a validated project brief into a structured, executable project plan.

**Responsibilities:** Receives the structured ProjectSpecificationDocument from the Orchestrator. Creates a complete task backlog: every feature decomposes into atomic tasks, each with a description, acceptance criteria, estimated effort, division, and dependencies. Creates project milestones with clear completion criteria. Estimates project timeline (probabilistic: best/expected/worst case). Identifies risks and mitigation strategies. Maintains the project backlog throughout the project — adding tasks when new requirements emerge, adjusting priorities when blockers arise, re-estimating when actual progress diverges from plan.

**Activation:** Activated at project start (planning phase) and whenever the project plan needs revision (scope change, major blocker, phase gate failure).

**Skills loaded:** general_coding.md.

**Key tools:** memory_store_decision (records all planning decisions), vfe_create_file (creates the project spec document in VFE), memory_retrieve.

**Produces:** TaskBacklog (complete list of TaskDefinition objects), ProjectMilestones, RiskRegister.

---

### Agent 4: Architect (ARCH)

**Role:** Designs the complete technical architecture for the project.

**Responsibilities:** Reads the ProjectSpecificationDocument and ProjectMilestones. Designs the software architecture: component structure, layer definitions, responsibility boundaries, deployment topology. Selects the technology stack with documented rationale for every choice. Designs the database schema: all tables/collections, fields, relationships, indexes, constraints. Creates the Integration Specification: all API endpoints with request/response schemas, all WebSocket message types, all authentication flows. Defines project coding standards: naming conventions, error handling approach, logging format, API response structure. Queries the Knowledge Graph for technology compatibility and pairing recommendations before finalizing selections.

**Activation:** Activated at project start (architecture phase) and when architectural questions arise during development.

**Skills loaded:** python_fastapi.md or node_express.md (based on tech stack), postgresql.md, authentication.md.

**Key tools:** knowledge_query (TechCompatibilityCheck, PatternRecommendation, TechPairing), vfe_create_file, memory_store_decision, vfe_write_segment.

**Produces:** ArchitectureDocument, TechnologyStackDocument, DatabaseSchemaDocument, IntegrationSpecification, ProjectStandards.

---

### Agent 5: Backend Developer (BEND)

**Role:** Implements all server-side code. The most active coding agent in the system.

**Responsibilities:** Implements HTTP API endpoints based on the Integration Specification. Implements business logic and service classes. Implements ORM models and database interaction code. Implements authentication and authorization (works closely with the auth system designed by the Architect). Implements background job definitions. Implements external service integrations. Implements configuration management. For AACS-Core's scope, the Backend Developer handles all of these — it is a multi-role agent that carries the full backend context.

**Activation:** Primary development phase tasks assigned by the Orchestrator.

**Skills loaded:** python_fastapi.md (or node_express.md), postgresql.md, authentication.md.

**Key tools:** All VFE write tools, build_install_dependencies, build_run_linter, build_run_type_checker, build_verify_syntax, memory_retrieve, memory_store_convention, ccs_get_symbol_definition, ccs_get_impact_analysis.

**Self-Review Checklist Key Items:** Pydantic/Zod validation on all inputs. Appropriate HTTP status codes. Auth enforced on all protected endpoints. Error handling produces structured responses. No raw SQL string concatenation. No hardcoded secrets. Async functions where I/O is involved.

---

### Agent 6: Frontend Developer (FEND)

**Role:** Implements all client-side code: UI components, pages, routing, state management, and API integration.

**Responsibilities:** Builds reusable UI components (forms, tables, navigation, modals, cards). Assembles pages from components with complete routing. Implements state management (Zustand or equivalent). Implements API client layer (typed hooks using React Query). Implements authentication UI (login, registration, password reset flows). Implements responsive layouts and basic styling (Tailwind or equivalent). Ensures all forms have proper validation (client-side). Integrates with backend APIs using the Integration Specification as the contract.

**Activation:** Frontend development tasks assigned by Orchestrator (often parallel with backend development).

**Skills loaded:** react_frontend.md, general_coding.md.

**Key tools:** All VFE write tools, build_install_dependencies, build_run_linter, build_run_type_checker, build_verify_syntax, ccs_get_api_endpoints (reads the backend contract), memory_retrieve.

**Self-Review Checklist Key Items:** All props are typed. Loading and error states handled for all async operations. Forms have validation. No direct DOM manipulation. State management is consistent. No console.log left in production code.

---

### Agent 7: Database Developer (DBA)

**Role:** Implements the complete database layer: migrations, ORM models, repositories, and query optimization.

**Responsibilities:** Creates database migration files from the DatabaseSchemaDocument. Writes ORM model classes with all field definitions, relationships, validations, and indexes. Implements repository pattern classes (or equivalent data access layer). Writes optimized queries for complex data retrieval operations. Identifies and prevents N+1 query patterns. Configures connection pooling. Creates seed data scripts and fixture files for testing.

**Activation:** Activated early in development (database setup before API development depends on it).

**Skills loaded:** postgresql.md, python_fastapi.md or node_express.md.

**Key tools:** VFE write tools, build_run_migration (applies migrations to test database), build_run_migration_check (validates migrations), ccs_get_orm_models, memory_store_convention.

---

### Agent 8: Auth Developer (AUTH)

**Role:** Implements the complete authentication and authorization system.

**Responsibilities:** Implements JWT token generation, validation, and refresh. Implements user registration, login, and logout flows. Implements refresh token rotation. Implements role-based access control: permission definitions, role assignments, authorization decorators/middleware. Implements password hashing (bcrypt or Argon2). Implements OAuth 2.0 provider integration if required. Implements security headers and CSRF protection. Writes comprehensive tests for all auth flows including negative cases (invalid tokens, expired tokens, insufficient permissions).

**Activation:** Activated in development phase after core models are established.

**Skills loaded:** authentication.md, python_fastapi.md or node_express.md.

**Key tools:** VFE write tools, build_run_linter, build_run_type_checker, ccs_get_symbol_definition, memory_retrieve.

---

### Agent 9: Test Writer (TEST)

**Role:** Writes the complete automated test suite for the project.

**Responsibilities:** Writes unit tests for all business logic functions and classes. Writes integration tests for all API endpoints (using a real test database, not mocks). Writes contract tests verifying the frontend-backend API contract. Sets up test infrastructure: test database creation, fixture definitions, test configuration. Configures coverage reporting. Selects appropriate mocking strategies for external dependencies. Writes property-based tests for complex input-output functions where applicable. Ensures every test has a descriptive name following the pattern test_{what}_{given}_{expected_outcome}.

**Activation:** Parallel with development tasks — TEST is activated as soon as code is written (CODE_READY event), writing tests for the just-completed code.

**Skills loaded:** python_fastapi.md or node_express.md or react_frontend.md (based on what is being tested), general_coding.md.

**Key tools:** VFE write tools, test_run_suite, test_get_coverage_report, ccs_get_symbol_definition, ccs_get_test_coverage_map, build_verify_syntax.

---

### Agent 10: E2E Tester (E2E)

**Role:** Writes and executes end-to-end tests for all critical user journeys.

**Responsibilities:** Identifies all critical user journeys from the ProjectSpecificationDocument. Implements Playwright (default) or Cypress E2E tests for each journey. Implements the Page Object Model for maintainable E2E tests. Uses semantic selectors (role, accessible name) rather than fragile CSS selectors. Implements retry logic and explicit wait conditions (never fixed sleep calls). Manages test data setup and teardown. Runs E2E tests against the complete running application stack.

**Activation:** Activated after integration is complete (Phase 4 gate passed).

**Skills loaded:** react_frontend.md (for understanding UI structure), general_coding.md.

**Key tools:** VFE write tools, test_run_e2e, build_run_dev_server, ccs_get_api_endpoints.

---

### Agent 11: Debugger (DBG)

**Role:** The system's specialist in finding and fixing defects. Activated reactively when bugs are discovered.

**Responsibilities:** Performs root cause analysis on all types of failures: compile errors, runtime exceptions, test failures, CI failures, integration issues, and performance bugs. For each bug: classify the failure type, identify the immediate cause, trace the causal chain to the root cause, formulate a testable fix hypothesis, implement the fix with a VFE savepoint, verify the fix passes the failing test AND does not regress other tests, commit the fix. Handles CI pipeline failures: downloads and parses GitHub Actions logs, identifies the specific failure point, applies the fix, pushes, and verifies CI passes.

**Activation:** Activated on any TASK_FAILED event, any test failure routed from the Test Runner, and any CI_FAILURE event from the GitHub Bridge. Also activated at Grade 2 of Loop Prevention as a specialist for any stuck agent.

**Skills loaded:** The skill most relevant to the bug's domain (python_fastapi.md for backend bugs, react_frontend.md for frontend bugs, etc.), general_coding.md.

**Key tools:** VFE read/write tools, test_run_suite, test_run_single, build_verify_syntax, ccs_get_symbol_definition, ccs_get_symbol_references, ccs_get_impact_analysis, search_error_solutions, github_get_run_logs.

---

### Agent 12: Code Reviewer (REVIEW)

**Role:** Enforces code quality standards continuously throughout development.

**Responsibilities:** Reviews all code written by development agents (BEND, FEND, DBA, AUTH) against the ProjectStandards document. Checks: naming conventions, error handling completeness, code documentation, pattern compliance (are service classes using the Repository pattern? are API responses using the standard response format?), dead code, obvious inefficiencies. Issues findings categorized as Critical (must fix before proceeding), High (fix before phase gate), Medium (fix recommended), and Low (optional improvement). Routes Critical and High findings immediately back to the producing agent.

**Activation:** Triggered by CODE_READY events — runs in parallel with Test Writer for every code commit.

**Skills loaded:** general_coding.md, the relevant technology skill.

**Key tools:** VFE read tools, build_run_linter, build_run_type_checker, ccs_measure_complexity, ccs_find_unused_symbols, ccs_find_circular_imports.

---

### Agent 13: Security Checker (SEC)

**Role:** Applies security-focused review to all code and dependencies.

**Responsibilities:** Scans all code for OWASP Top 10 vulnerability patterns: SQL injection, XSS, insecure deserialization, broken authentication, sensitive data exposure, security misconfigurations, and injection attacks. Scans all dependencies for known CVEs using pip-audit and npm audit. Checks for hardcoded secrets (API keys, passwords, tokens). Verifies all protected endpoints enforce authentication. Checks for missing rate limiting on sensitive endpoints. Verifies secure password hashing is used. Checks security headers are configured. Queries the Knowledge Graph for known security concerns with the chosen technology versions.

**Activation:** Activated at Phase 5 Quality Assurance gate and whenever a change touches authentication, authorization, or external data handling.

**Skills loaded:** authentication.md, general_coding.md.

**Key tools:** VFE read tools, build_run_security_scanner, search_cve_database, knowledge_query (SecurityCheck), ccs_get_api_endpoints (to verify auth on all endpoints).

---

### Agent 14: Optimizer (OPT)

**Role:** Identifies and fixes performance bottlenecks.

**Responsibilities:** Analyzes all database queries for N+1 patterns, missing indexes, and inefficient joins. Reviews algorithmic complexity: identifies O(n²) algorithms that can be O(n log n), redundant computations, unnecessary loops. Reviews async code for blocking calls that should be async. Reviews frontend for bundle size issues, unnecessary re-renders, and missing lazy loading. Runs performance benchmarks (k6 or equivalent for API throughput, pytest-benchmark for function performance) and verifies that optimizations improve measurements. Never optimizes without measurement — every change must prove its value.

**Activation:** Activated in Phase 5 Optimization.

**Skills loaded:** postgresql.md (for query optimization), python_fastapi.md or node_express.md, general_coding.md.

**Key tools:** VFE read/write tools, test_run_performance, ccs_measure_complexity, ccs_find_circular_imports, build_run_type_checker.

---

### Agent 15: Auditor (AUD)

**Role:** The independent quality authority. Its findings gate project delivery.

**Responsibilities:** Performs the complete pre-delivery audit across five dimensions: Requirements Coverage (every Must-Have requirement implemented and verified by passing tests — uses systematic requirements tracing), Code Quality (test coverage above threshold, complexity within limits, documentation coverage adequate, no dead code), Security (reuses Security Checker findings, adds final verification), Performance (benchmarks meet specified targets or are within acceptable range), and Delivery Readiness (README is complete and accurate, setup instructions work in a clean environment — the Auditor actually follows the README in a Docker container, build process verified, CI is green). Produces a Final Audit Report with all findings classified by severity. A Critical or High finding blocks delivery.

**Activation:** Activated at Phase 5 (mid-project health audit) and Phase 6 (Final Audit) gates.

**Skills loaded:** general_coding.md, all relevant technology skills for the project.

**Key tools:** All VFE read tools, test_run_suite, test_get_coverage_report, build_run_security_scanner, ccs_measure_complexity, ccs_find_unused_symbols, build_run_build (in clean environment).

---

### Agent 16: GitHub Manager (GIT)

**Role:** All GitHub operations. The only agent that interacts with the GitHub API and git CLI.

**Responsibilities:** Repository initialization (create repo, branch protection, initial commit). Branch management (create feature branches, merge feature→develop, merge develop→main at milestones). Commit creation (atomic, Conventional Commits format, meaningful messages). Pull request management (create, review with comments, merge). GitHub Actions workflow generation (create tailored CI/CD YAML), monitoring (poll for run status), and failure notification (route to Debugger). Release creation (tag, release notes, GitHub Release). Issue creation for deferred audit findings.

**Activation:** Activated at project start (repo initialization), after every reviewed code batch (commit + PR), at milestone transitions (merge), when CI fails, and at project delivery (release creation).

**Skills loaded:** docker_deployment.md (for CI/CD workflows), general_coding.md.

**Key tools:** All git_* tools, github_* tools, memory_retrieve, memory_store_decision.

---

### Agent 17: Docs Writer (DOCS)

**Role:** Generates all project documentation.

**Responsibilities:** Writes a comprehensive README.md: project overview, prerequisites list, installation steps (tested to be accurate), configuration reference (all environment variables documented), how to run the application, how to run the tests, how to deploy (for the configured targets). Generates API reference documentation from code (OpenAPI/Swagger for FastAPI, or equivalent markdown for other frameworks). Writes a deployment guide for the configured cloud target. Creates a CHANGELOG.md from the git history. Writes inline code documentation (docstrings for all public functions that lack them — coordinates with Code Reviewer to ensure coverage). The Docs Writer tests the README by mentally walking through every step and flagging any step that would fail.

**Activation:** Activated in Phase 7 Delivery preparation and when documentation findings are raised by the Auditor.

**Skills loaded:** docker_deployment.md, general_coding.md.

**Key tools:** VFE read/write tools, vfe_get_file_tree, ccs_get_api_endpoints, git_log (for changelog), docs_validate_docstrings.

---

### Agent 18: Delivery Agent (DEL)

**Role:** Packages and delivers the completed project.

**Responsibilities:** Prepares the codebase for delivery: removes development artifacts (__pycache__, .DS_Store, temporary files), verifies .gitignore is comprehensive, ensures consistent line endings and file encoding. Assembles the complete deliverable package: source code (via GitHub), documentation set, configuration templates, test suite with run instructions, Delivery Summary Report. The Delivery Summary Report contains: complete feature inventory (every implemented feature with its requirement ID), technology stack summary, how to set up and run the project, how to run the tests, known limitations (if any), deferred requirements with explanations, and a summary of the Final Audit findings.

**Activation:** Activated after Final Audit passes all gates (Phase 7).

**Skills loaded:** general_coding.md.

**Key tools:** VFE read tools, vfe_get_file_tree, git_* tools, github_create_release, memory_retrieve, docs_generate_readme (if README needs final polish).

---

### Agent 19: Knowledge Agent (KNOW)

**Role:** Manages the Knowledge Graph and retrieves external information for other agents.

**Responsibilities:** Processes all knowledge_query tool calls from other agents (TechCompatibilityCheck, PatternRecommendation, etc.). Fetches official documentation when agents need current information about a technology (APIs, configuration options, known issues). Updates the Knowledge Graph after project completion with new learnings. Manages the Knowledge Graph's integrity: checks for circular edges, validates node properties, ensures LEARNED_FROM edges are correctly linked. Researches new technologies encountered in project briefs that are not in the Knowledge Graph.

**Activation:** On-demand (activated by knowledge_query tool calls from other agents and after project delivery for knowledge graph updates).

**Skills loaded:** general_coding.md.

**Key tools:** knowledge_query, knowledge_update, search_official_docs, search_error_solutions, search_npm_registry, search_pypi.

---

### Agent 20: Memory Keeper (MEM)

**Role:** Manages the Memory Engine — recording, retrieving, and maintaining all three tiers of project memory.

**Responsibilities:** Records all significant system events to Episodic Memory (task completions, decisions, bugs found/fixed, human interactions). Maintains the Project Memory sections: updates Technical Decisions when the Architect makes a new decision, updates Established Conventions when a coding pattern is first used, updates Active Issues when bugs are found and resolved, updates Integration Notes when the Architect defines component connections. Manages episodic memory compression (daily summaries for events older than 7 days). Responds to memory retrieval requests from the Context Assembly Pipeline.

**Activation:** Always active (receives all system events and memory retrieval requests).

**Skills loaded:** None (infrastructure agent, not a knowledge worker).

**Key tools:** memory_store_episodic, memory_store_decision, memory_store_convention, memory_store_issue, memory_retrieve, memory_search_semantic.

---

### Agent 21: Config Developer (CFG)

**Role:** Implements all project configuration, environment management, and containerization.

**Responsibilities:** Creates the settings module (Pydantic Settings for Python, dotenv for Node.js): all configuration values loaded from environment variables, validated on startup, documented in .env.example. Creates Docker configuration: multi-stage Dockerfile (development stage, production stage), docker-compose.yml for local development (with all required services: database, Redis if needed, etc.), .dockerignore. Creates production deployment configuration for the configured target (Railway/Fly.io/DigitalOcean configuration files, systemd service files, Nginx configuration if needed). Verifies that the application starts correctly with the example configuration in a Docker environment.

**Activation:** Activated after the tech stack is finalized (beginning of Phase 2 Infrastructure) and again in Phase 7 for delivery configuration.

**Skills loaded:** docker_deployment.md, python_fastapi.md or node_express.md.

**Key tools:** VFE write tools, build_run_dev_server (to test the configuration), admin_start_service (to start required services in admin mode).

---

### Agent 22: Integration Tester (INTT)

**Role:** Verifies that all system components work together correctly.

**Responsibilities:** Runs the complete integration test suite against the fully-assembled application stack. Executes API contract tests: every endpoint defined in the Integration Specification is tested for correct behavior (right status codes, right response schemas, correct error responses). Runs the E2E test suite for all critical user journeys. Identifies boundary bugs (API response format doesn't match what the frontend expects, database schema doesn't match ORM model, middleware order is incorrect). Routes all failures to the Debugger. Verifies the application starts cleanly from a clean database state (no existing-state dependencies in the main code path).

**Activation:** Activated at Phase 4 Integration gate and when any integration-related bug is reported.

**Skills loaded:** python_fastapi.md or node_express.md, react_frontend.md.

**Key tools:** test_run_suite, test_run_e2e, test_run_contract, build_run_dev_server, ccs_get_api_endpoints, memory_retrieve.

---

### Agent 23: Performance Tester (PERF)

**Role:** Measures application performance and establishes benchmarks.

**Responsibilities:** Identifies the 5 most performance-sensitive operations in the application (typically: the most-called API endpoints, the most complex database queries, the largest data transformations). Writes performance tests (k6 scripts for API load testing, pytest-benchmark for function-level benchmarks). Establishes baseline measurements before optimization. Re-measures after Optimizer makes changes to verify improvement. Flags operations that do not meet the project's performance requirements (from the non-functional requirements in the project spec). Reports to the Auditor for the Performance audit pillar.

**Activation:** Activated at Phase 5 (before Optimizer), and again after Optimization is complete.

**Skills loaded:** python_fastapi.md or node_express.md, postgresql.md.

**Key tools:** test_run_performance, ccs_measure_complexity, vfe_read_file, memory_retrieve.

---

# PART FIVE: PROJECT LIFECYCLE

---

## Chapter 15: Complete Project Lifecycle

### 15.1 Phase 0: Project Intake (5-20 minutes)

The user submits a project brief. The Human Interface Agent receives it and activates the intent parsing sub-routine: the brief is analyzed for explicit requirements, implicit domain requirements (e.g., any project with user accounts gets security requirements automatically), ambiguities, and scale estimation.

If ambiguities exist that cannot be inferred from domain conventions, the HIA formulates 1-3 targeted clarification questions with multiple-choice options where possible. These are presented in the WebUI. While waiting for answers, AACS-Core proceeds on all unambiguous aspects of the brief.

When the brief is sufficiently defined (either clarifications answered or all ambiguities resolved through inference), the Project Specification Document is produced and stored in VFE and Project Memory. Phase 0 complete.

Phase 0 Gate: PSD complete, all Must-Have requirements have measurable success criteria, scale assessed, no unresolved critical ambiguities.

### 15.2 Phase 1: Architecture and Planning (20-60 minutes)

The Architect agent activates. It queries the Knowledge Graph for technology recommendations based on the project type and requirements. It designs the system architecture, selects the technology stack, designs the database schema, and creates the Integration Specification. All decisions are recorded in Project Memory as Technical Decisions.

The Project Manager agent activates (in parallel with the Architect's later work). It creates the complete task backlog from the architecture and requirements. Task count for a typical medium project: 60-120 tasks. The Orchestrator receives the task backlog and builds the task dependency graph.

The GitHub Manager initializes the repository, creates the initial CI/CD workflow, and pushes the scaffold.

Phase 1 Gate: Architecture Document complete, tech stack selected, database schema designed, Integration Spec complete, task backlog created, repository initialized, CI green.

### 15.3 Phase 2: Infrastructure Setup (20-40 minutes)

The Config Developer creates the project scaffold: directory structure, dependency files (requirements.txt, package.json), configuration module, Docker setup, environment templates. The Database Developer creates initial migrations and ORM models. The GitHub Manager commits the scaffold.

Phase 2 Gate: Project scaffold runs without errors, database migrations apply cleanly, Docker build succeeds, CI green.

### 15.4 Phase 3: Core Development (Hours to Days)

The primary development phase. The Orchestrator dispatches tasks according to the dependency graph. Parallel tracks run simultaneously where dependencies allow:

Backend track: Backend Developer implements API endpoints, business logic, service classes. Database Developer adds complex queries and repositories.

Frontend track: Frontend Developer builds components and pages. State management is implemented.

Auth track: Auth Developer implements the authentication system.

Config track: Config Developer integrates configuration and environment management.

For every block of code written (CODE_READY event), the Code Reviewer and Test Writer are activated in parallel. The Orchestrator does not wait for review to complete before dispatching the next task — review and development run concurrently.

Bugs found during development are immediately routed to the Debugger. The Loop Prevention System monitors all active tasks.

Phase 3 Gate: All planned code written and individually reviewed, unit test coverage above 78%, no unresolved Critical or High review findings, CI green.

### 15.5 Phase 4: Integration (1-3 hours)

The Integration Tester runs the complete integration test suite. All E2E tests for primary user journeys run. Contract tests verify frontend-backend API alignment. The Optimizer performs an initial performance review.

Integration bugs (the most common: API response format mismatch, foreign key constraint errors, middleware ordering issues) are routed to the Debugger and resolved.

Phase 4 Gate: All integration tests pass, all E2E tests pass for Must-Have user journeys, API contract tests pass, application starts cleanly from fresh database state.

### 15.6 Phase 5: Quality Assurance (1-2 hours)

The Optimizer runs, measuring and improving performance. The Security Checker performs the comprehensive security scan. The Performance Tester runs load tests. The Code Reviewer performs a final standards review.

All Critical and High findings are dispatched as fix tasks. Medium and Low findings are documented. The phase iterates until all Critical and High findings are resolved.

Phase 5 Gate: Zero Critical security findings, test coverage above 80%, performance benchmarks within 90% of targets, documentation coverage above 75%, CI green.

### 15.7 Phase 6: Final Audit (1-2 hours)

The Auditor performs the complete final audit across all five pillars: Requirements Coverage (100% of Must-Have requirements implemented and verified), Code Quality (all metrics within thresholds), Security (zero Critical, zero High vulnerabilities), Performance (benchmarks pass), Delivery Readiness (README tested in clean Docker environment, build verified).

The Docs Writer produces the complete documentation set. The Final Audit Report is generated.

If any Critical or High audit findings are discovered, they are immediately dispatched as fix tasks. The audit re-runs after fixes are verified.

Phase 6 Gate: Final Audit Report passes, zero Critical findings, zero High findings, all documentation complete, CI green.

### 15.8 Phase 7: Delivery (30-60 minutes)

The Delivery Agent assembles the complete deliverable. The GitHub Manager creates the release PR (develop → main), verifies final CI, merges, creates the release tag, and publishes the GitHub Release.

The Human Interface Agent sends the delivery notification through the WebUI chat with: what was built (feature summary), how to set it up, the GitHub repository link, a link to the Final Audit Report, and an invitation to continue the project.

Project status transitions to DELIVERED. AACS-Core remains active, the project context remains fully loaded. Post-delivery follow-up is available immediately.

### 15.9 Post-Delivery Continuation

A delivered project is not closed. The user can return at any time — the next day, a week later, a month later — and continue through the WebUI chat. All project memory, all code context, all architectural decisions are preserved.

Post-delivery messages are classified by the Human Interface Agent:

"Add a feature for X" → Full feature development cycle: requirements → architecture note → tasks → development → testing → audit → delivery as a new version.

"Fix the bug where Y" → Routed to Debugger. Fix cycle with unit test addition. New version released.

"How does X work?" → Memory and code context query. Answered directly in chat with code references.

"Can you explain the architecture?" → Architecture document retrieved from Project Memory and summarized in chat.

"Deploy this to Heroku" → Config Developer adds platform-specific configuration. New release with deployment instructions.

---

# PART SIX: INTERFACE AND CONFIGURATION

---

## Chapter 16: WebUI

### 16.1 Design

The AACS-Core WebUI is a single-page application served by the FastAPI backend. It provides real-time visibility into everything the system is doing and a natural chat interface for human interaction. The WebUI is accessible from any device on the same LAN, protected by password authentication.

### 16.2 Main Panels

**Chat Panel (Primary).** The central interaction point. Shows all AACS-Core ↔ user messages in chronological order with timestamps. System messages (phase completions, milestone announcements, CI status, delivery notification) are shown inline. The input bar allows the user to send any message at any time. A "Human Required" badge appears when AACS-Core is waiting for user input on a critical question.

**Activity Feed.** Real-time stream of system events: tasks started/completed, code written, tests run, bugs found/fixed, CI events, phase transitions. Each event has an icon, color, and one-line description. Filter by event type (Code, Tests, GitHub, System). Searchable.

**Project Status.** Current phase, phase progress bar, milestone tracker, open issues count, test coverage gauge, budget consumed, estimated completion.

**File Tree.** Real-time mirror of the VFE. Browse the project's directory structure. Click any file to view its content with syntax highlighting and segment boundaries visible. Shows status indicators: green dot (reviewed, tests passing), yellow dot (under review), red dot (review findings or test failures).

**Code Viewer.** Syntax-highlighted file viewer with line numbers. Shows coverage overlay (green/red line tinting for covered/uncovered lines). Shows inline review findings as annotations. Shows version history for the selected file.

**Tests Panel.** Current test run results: total/passed/failed/skipped counts. Per-file coverage. List of failing tests with error messages. CI status for the current branch.

**GitHub Panel.** Recent commits, open PRs, CI run status, releases list. Live CI run progress.

**Settings.** LLM provider config (base URL, API key, model ID), GitHub config (token, username), admin mode toggle, resource limits (concurrency, budget), WebUI password change.

### 16.3 Real-Time Updates

All panels receive real-time updates via WebSocket. The WebSocket connection subscribes to all system events. When an event occurs (a file is written to VFE, a test run completes, a CI run finishes), the relevant panel updates immediately without page refresh. Connection loss is handled with automatic reconnection and a state snapshot fetch to resync.

### 16.4 Project List

The Projects page shows all projects with their status (Active, Delivered, Paused). Each project shows: name, creation date, last active date, current phase (if active), and a quick-access button to open the project. Creating a new project opens a large text area for the project brief, with optional file attachment for detailed requirements documents.

---

## Chapter 17: Terminal Interface

AACS-Core provides a complete CLI for users who prefer terminal interaction.

```
aacs-core start                     Start AACS-Core (launches WebUI)
aacs-core stop                      Gracefully stop (saves checkpoint)
aacs-core status                    Print current project status

aacs-core project new               Interactive project brief entry
aacs-core project new --brief "..." Start project with inline brief
aacs-core project list              List all projects
aacs-core project resume <id>       Resume a paused project
aacs-core project pause             Pause current project

aacs-core chat "<message>"          Send message to current project
aacs-core agents                    List active agents and current tasks
aacs-core tests                     Show current test results

aacs-core checkpoint                Create manual checkpoint
aacs-core checkpoint list           List available checkpoints
aacs-core checkpoint restore <id>   Restore from checkpoint

aacs-core config show               Show current configuration
aacs-core config set <key> <value>  Update config value
aacs-core diagnose                  Run system health check
```

---

## Chapter 18: Configuration

### 18.1 Config File

Located at `~/.aacs-core/config.yaml` on Linux and `$HOME\.aacs-core\config.yaml` on Windows.

```yaml
version: "1.0"

llm:
  base_url: "https://api.openai.com/v1"
  api_key: "stored-in-credential-vault"
  model_id: "gpt-4o"
  fast_model_id: "gpt-4o-mini"  # used for low-stakes tasks (formatting, simple analysis)
  max_tokens: 4096
  temperature: 0.1               # low for code generation

github:
  token: "stored-in-credential-vault"
  username: "your-github-username"
  default_visibility: "private"
  auto_create_repo: true

webui:
  port: 8080
  bind: "0.0.0.0"               # LAN accessible
  require_auth: true

admin_mode: false                 # set true for full system access

resources:
  max_concurrent_agents: 3
  daily_budget_usd: 20.0
  project_budget_usd: 50.0
  budget_warning_percent: 75
  checkpoint_interval_minutes: 30

quality:
  min_test_coverage: 80
  min_security_scan: true
  require_e2e_tests: true
  loop_prevention_human_grade: 4

storage:
  data_dir: "~/.aacs-core"
  projects_dir: "~/.aacs-core/projects"
  checkpoints_dir: "~/.aacs-core/checkpoints"
```

### 18.2 First-Run Wizard

The first-run wizard guides the user through initial configuration in under 5 minutes:

Screen 1: LLM Provider setup — enter Base URL, API Key, and Model ID. AACS-Core tests the connection with a minimal call. Confirmation shown on success.

Screen 2: GitHub setup — enter Personal Access Token and username. AACS-Core tests by calling `/user` API. Confirms authenticated username.

Screen 3: Admin Mode — explains what it enables. User chooses. If enabled, verifies current process has elevated privileges.

Screen 4: Resources — shows detected RAM and CPU. Suggests concurrency level and budget. User confirms or adjusts.

Screen 5: WebUI — confirm port (default 8080), set access password.

Screen 6: Complete — shows the WebUI URL and LAN URL. Option to start a first project immediately.

---

# PART SEVEN: INSTALLATION AND DEPLOYMENT

---

## Chapter 19: Installation

### 19.1 Requirements

Python 3.11+, Git 2.30+, 6GB free disk space (for AACS-Core installation, local embedding model, and project data), 8GB RAM minimum (12GB recommended), internet connectivity for LLM API and GitHub, C compiler (for tree-sitter parser compilation — gcc on Linux, Visual Studio Build Tools on Windows).

### 19.2 Installation

```bash
# Linux / macOS
curl -fsSL https://raw.githubusercontent.com/aacs-core/install/main/install.sh | bash

# Windows PowerShell (Admin)
iwr https://raw.githubusercontent.com/aacs-core/install/main/install.ps1 | iex
```

The installer: checks Python version, creates a virtual environment at `~/.aacs-core/venv`, installs all Python dependencies, compiles tree-sitter language parsers (Python, TypeScript, JavaScript, Go), downloads the local embedding model (~80MB), creates the data directory structure, registers the CLI command, and launches the first-run wizard.

### 19.3 Upgrade

```bash
aacs-core upgrade    # pulls latest version, re-runs dependency installation, preserves all data
```

---

## Chapter 20: Success Metrics

### 20.1 What AACS-Core Optimizes For

AACS-Core measures its success through four metrics visible in the WebUI:

**Delivery Rate:** Projects delivered with all Must-Have requirements satisfied. Target: 95%+.

**Human Intervention Rate:** Number of Human Required prompts per project. Target: Under 3 for a well-specified project.

**Test Coverage at Delivery:** Percentage of code covered by automated tests. Target: 80%+.

**First-Time Audit Pass Rate:** Projects where the Final Audit passes without requiring a rework cycle. Target: 70%+.

These metrics are tracked per project and shown in the Projects list as quality indicators. Over time, as AACS-Core's Knowledge Graph grows and its episodic memory accumulates project history, these metrics should improve.

### 20.2 The 50-File Commitment

AACS-Core achieves its 50-file target by ruthless consolidation without removing capability. The LLM Engine is one file but implements the complete 6-stage context assembly and 4-stage response processing. The Shell Engine is one file but handles cross-platform, admin mode, output parsing, and process management. Each agent is one file but implements the complete 8-stage cognitive cycle with self-review, verification, and impact assessment.

The test proves that the quality of a system does not depend on the number of files it uses. What matters is the quality of the intelligence in those files, the rigor of the processes they implement, and the soundness of the data structures they maintain. AACS-Core is proof that a 50-file Python system can autonomously deliver production-quality software.

---

*End of AACS-Core Master Architecture and Engineering Specification*
*Version 1.0 — The Small Avatar of AACS*
*Status: Complete Engineering Blueprint*
*"Maximum capability. Minimum files. Full autonomy."*

---
# PART EIGHT: COMPLETE TOOL CATALOG

---

## Chapter 21: All Tools in AACS-Core

### 21.1 Tool Architecture

Tools in AACS-Core are Python functions wrapped with a typed interface: an input schema (Pydantic model), an output schema (Pydantic model), a permission requirement, and a health probe. Tools are the only way agents interact with the outside world — they cannot make direct API calls or file system operations without going through a tool. This ensures every operation is logged, permission-checked, and audit-trailed.

Tools are organized into 6 categories, implemented across 6 files (one file per category in the `tools/` directory). Each tool file contains ~4-6 tool implementations. Total tools: ~30.

---

### 21.2 Filesystem Tools (`tools/filesystem.py`)

**vfe_create_file**
Input: path (str), language (str), initial_content (str, optional)
Output: FileNode reference (path, language, segment_count)
Permission: STANDARD
Creates a new file in VFE with optional initial content. If initial_content is provided, tree-sitter parses it into segments immediately.

**vfe_read_file**
Input: path (str), version (str, default "HEAD")
Output: FileContent (full text assembled from segments, plus segment list with types and IDs)
Permission: STANDARD
Reads a file's complete content from VFE. Can read historical versions by version ID.

**vfe_read_segment**
Input: path (str), segment_id (str)
Output: SegmentContent (type, human_label, content, line_start, line_end)
Permission: STANDARD
Reads a single named segment. Efficient for agents that only need to review or modify one function.

**vfe_write_segment**
Input: path (str), segment_id (str), new_content (str)
Output: WriteResult (success, new_content_hash, version_id_created)
Permission: STANDARD
Updates the content of a named segment. Creates a version entry. Triggers CODE_READY event.

**vfe_add_segment**
Input: path (str), segment_type (str), human_label (str), content (str), after_segment_id (str, optional)
Output: AddResult (success, new_segment_id, version_id)
Permission: STANDARD
Adds a new segment to a file. Inserts after the specified segment, or appends if none specified.

**vfe_replace_file**
Input: path (str), new_content (str), commit_message (str)
Output: ReplaceResult (success, version_id, segments_created_count)
Permission: STANDARD
Replaces a file's entire content. Old content is retained as a version snapshot. New content is re-parsed into segments.

**vfe_get_file_tree**
Input: root (str, default "/"), include_hidden (bool, default False)
Output: FileTree (nested directory structure with file metadata)
Permission: STANDARD
Returns the complete VFE file tree. Used by WebUI file tree component and planning agents.

**vfe_search_symbol**
Input: symbol_name (str), symbol_types (list, optional), scope (str, optional)
Output: SymbolSearchResult (list of matches with path, segment_id, line_number)
Permission: STANDARD
Searches for symbol definitions and references across all VFE files in memory (no subprocess).

**vfe_diff_versions**
Input: path (str), version_a (str), version_b (str)
Output: DiffResult (list of changes: added/removed/modified segments)
Permission: STANDARD
Returns semantic diff between two versions — segment-level, not line-level.

**disk_sync_now**
Input: scope (str, "all" or path prefix)
Output: SyncResult (files_synced, files_failed, verification_passed)
Permission: STANDARD
Forces an immediate disk sync for the specified scope. Verifies hash of each synced file.

---

### 21.3 Code Analysis Tools (`tools/code_analysis.py`)

**ccs_get_symbol_definition**
Input: symbol_name (str), scope (str, optional)
Output: SymbolDefinition (qualified_name, location, signature, docstring, type)
Permission: STANDARD
Returns the complete definition of a code symbol from the live Code Context Index.

**ccs_get_impact_analysis**
Input: changed_paths (list of str)
Output: ImpactReport (dependent_files, at_risk_tests, api_contracts_affected, summary)
Permission: STANDARD
Computes the impact of changes to a set of files: what else might be affected, what tests might break.

**ccs_get_api_endpoints**
Input: None
Output: EndpointRegistry (list of endpoints with path, method, auth_required, request/response schemas)
Permission: STANDARD
Returns all registered HTTP route handlers found in the codebase by the Code Context Index.

**ccs_get_orm_models**
Input: None
Output: ModelRegistry (list of ORM models with table_name, fields, relationships)
Permission: STANDARD
Returns all ORM model definitions discovered in the codebase.

**ccs_measure_complexity**
Input: path (str), function_name (str, optional)
Output: ComplexityReport (cyclomatic_complexity, cognitive_complexity per function)
Permission: STANDARD
Measures code complexity. High-complexity functions are flagged for refactoring.

**ccs_find_unused_symbols**
Input: scope (str, optional)
Output: UnusedSymbolList (symbol name, location, type)
Permission: STANDARD
Identifies defined symbols that are never referenced — dead code candidates.

**ccs_find_circular_imports**
Input: None
Output: CircularImportList (each cycle as a list of module paths)
Permission: STANDARD
Detects circular import chains which cause runtime failures and architectural issues.

---

### 21.4 Build and Test Tools (`tools/build_test.py`)

**build_install_dependencies**
Input: manager (str: "pip"/"npm"/"pnpm"), working_dir (str), flags (list, optional)
Output: InstallResult (success, packages_installed, vulnerabilities_found)
Permission: STANDARD
Installs project dependencies. Returns structured result including any security warnings.

**build_verify_syntax**
Input: path (str), language (str)
Output: SyntaxCheckResult (valid, errors: list with line/message)
Permission: STANDARD
Fast syntax check without building: ast.parse for Python, node --check for JS, tsc --noEmit for TS.

**build_run_linter**
Input: target (str), config (str, optional)
Output: LinterReport (findings: list with file/line/severity/message)
Permission: STANDARD
Runs ruff/flake8 (Python) or ESLint (JS/TS). Returns structured LinterReport.

**build_run_type_checker**
Input: target (str), config (str, optional)
Output: TypeCheckReport (errors: list with file/line/message, total_errors)
Permission: STANDARD
Runs mypy (Python) or tsc (TypeScript). Returns structured type errors.

**build_run_security_scanner**
Input: target (str), scanner (str: "bandit"/"semgrep"/"npm-audit"/"pip-audit")
Output: SecurityScanReport (findings: list with severity/category/location/description)
Permission: STANDARD
Runs the selected security scanner. Returns findings classified by OWASP category.

**build_run_migration**
Input: direction (str: "up"/"down"), target_version (str, optional), database_url (str)
Output: MigrationResult (success, version_before, version_after, statements_run)
Permission: ELEVATED
Applies or reverts database migrations against the specified database URL.

**test_run_suite**
Input: framework (str), target (str, optional), coverage (bool), flags (list, optional)
Output: TestRunResult (total, passed, failed, error, coverage_percent, failures: list)
Permission: STANDARD
Runs the full test suite. Returns structured results with per-test outcomes.

**test_run_single**
Input: framework (str), test_path (str), test_function (str, optional)
Output: TestRunResult (single test outcome with full output)
Permission: STANDARD
Runs a single test file or function. Used by Debugger for targeted verification.

**test_run_e2e**
Input: framework (str: "playwright"/"cypress"), test_path (str), base_url (str)
Output: E2ETestRunResult (journeys_tested, passed, failed, screenshots for failures)
Permission: STANDARD
Runs the E2E test suite. For failures, captures screenshots and page state.

**test_run_performance**
Input: script_path (str), target_url (str), duration_seconds (int)
Output: PerformanceTestResult (p50_ms, p95_ms, p99_ms, throughput_rps, errors)
Permission: STANDARD
Runs a k6 or Locust load test. Returns structured performance metrics.

**test_get_coverage_report**
Input: None
Output: CoverageReport (overall_percent, per_file: dict, uncovered_lines: list)
Permission: STANDARD
Returns the latest coverage report from the most recent test run.

**build_run_build**
Input: command (str), working_dir (str), env (dict, optional)
Output: BuildResult (success, duration_seconds, output_summary, artifacts: list)
Permission: STANDARD
Runs the project's build command (e.g., `npm run build`, `python -m build`).

---

### 21.5 Git and GitHub Tools (`tools/git_tools.py`)

**git_status**
Input: working_dir (str)
Output: GitStatus (staged_files, modified_files, untracked_files, ahead, behind)
Permission: STANDARD
Returns parsed git status.

**git_add_and_commit**
Input: paths (list), message (str), working_dir (str)
Output: GitCommitResult (commit_hash, files_committed, branch)
Permission: STANDARD
Stages specified files and creates a commit with the given message.

**git_push**
Input: branch (str), remote (str, default "origin"), working_dir (str)
Output: GitPushResult (success, commits_pushed, remote_url)
Permission: STANDARD
Pushes the branch to the remote.

**git_create_branch**
Input: branch_name (str), from_branch (str, default "develop"), working_dir (str)
Output: GitResult (success, branch_created, from_branch)
Permission: STANDARD
Creates a new branch from the specified base.

**git_merge_branch**
Input: source (str), target (str), method (str: "merge"/"squash"), working_dir (str)
Output: GitMergeResult (success, commit_hash, conflicts: list if any)
Permission: STANDARD
Merges source branch into target.

**github_create_repo**
Input: name (str), description (str), private (bool), owner (str)
Output: RepoResult (repo_url, clone_url, default_branch)
Permission: STANDARD
Creates a GitHub repository via the GitHub API.

**github_create_pr**
Input: title (str), body (str), head (str), base (str), repo (str)
Output: PullRequest (pr_number, pr_url, status)
Permission: STANDARD
Creates a Pull Request on GitHub.

**github_get_ci_status**
Input: repo (str), branch (str, optional), run_id (str, optional)
Output: CIStatus (status: "success"/"failure"/"pending", run_url, job_results: list)
Permission: STANDARD
Gets the current CI status for a branch or specific run.

**github_get_run_logs**
Input: run_id (str), repo (str)
Output: ParsedRunLogs (job_logs: dict with job_name → parsed_output, failure_summary)
Permission: STANDARD
Downloads and parses GitHub Actions run logs. Failure_summary highlights the specific failure point.

**github_create_release**
Input: tag (str), name (str), body (str), repo (str)
Output: ReleaseResult (release_url, tag_created, assets: list)
Permission: STANDARD
Creates a GitHub Release with the specified tag and release notes.

---

### 21.6 Search and Knowledge Tools (`tools/search.py`)

**search_official_docs**
Input: technology (str), version (str, optional), query (str)
Output: DocContent (source_url, relevant_sections: list with title and content)
Permission: STANDARD
Fetches relevant sections from official documentation for a technology.

**search_error_solutions**
Input: error_message (str), technology (str), version (str, optional)
Output: SolutionList (solutions: list with description, code_example, source_url, confidence)
Permission: STANDARD
Searches for solutions to specific error messages from curated sources.

**search_pypi**
Input: package_name (str), version (str, optional)
Output: PackageInfo (name, version, description, homepage, latest_version, security_advisories)
Permission: STANDARD
Queries PyPI for package information and security status.

**search_npm**
Input: package_name (str), version (str, optional)
Output: PackageInfo (name, version, description, homepage, latest_version, security_advisories)
Permission: STANDARD
Queries npm registry for package information.

**search_cve_database**
Input: package_name (str), version (str), ecosystem (str: "pypi"/"npm"/"cargo")
Output: CVEReport (vulnerabilities: list with severity/cvss/description/patched_version)
Permission: STANDARD
Checks a package version against CVE databases.

**knowledge_query**
Input: query_type (str), parameters (dict)
Output: KnowledgeQueryResult (results: list with relevant knowledge items and confidence)
Permission: STANDARD
Executes one of the 5 Knowledge Graph query types: TechCompatibilityCheck, PatternRecommendation, TechPairing, SecurityCheck, LessonRetrieval.

**knowledge_update**
Input: updates (list of node/edge definitions)
Output: GraphUpdateResult (nodes_added, edges_added, nodes_updated)
Permission: STANDARD
Adds new knowledge to the Knowledge Graph (post-project learning).

---

### 21.7 Monitoring and Admin Tools (`tools/monitoring.py`)

**monitor_system_health**
Input: None
Output: SystemHealthReport (health_score 0-100, components: dict with status per engine, issues: list)
Permission: STANDARD
Returns the current system health status.

**monitor_project_metrics**
Input: None
Output: ProjectMetricsReport (phase, completion_percent, test_coverage, open_findings, budget_consumed, agents_active)
Permission: STANDARD
Returns current project quality and progress metrics.

**monitor_budget_status**
Input: None
Output: BudgetStatus (daily_limit, daily_consumed, project_limit, project_consumed, burn_rate, projected_exhaustion)
Permission: STANDARD
Returns detailed budget consumption status with projections.

**admin_install_tool**
Input: tool_name (str), version (str, optional), package_manager (str)
Output: InstallResult (success, tool_path, version_installed)
Permission: ADMIN
Installs a development tool system-wide. Requires Admin Mode.

**admin_start_service**
Input: service_name (str), start_command (str, optional), port (int, optional)
Output: ServiceStartResult (success, pid, health_check_passed)
Permission: ADMIN
Starts a system service (PostgreSQL, Redis, etc.). Requires Admin Mode.

**admin_set_env_var**
Input: name (str), value (str), scope (str: "project"/"system"), persist (bool)
Output: EnvSetResult (success, scope_applied)
Permission: ADMIN
Sets an environment variable. Requires Admin Mode for system scope.

**comms_notify_human**
Input: message (str), urgency (str: "info"/"warning"/"critical"/"human_required")
Output: NotificationResult (delivered_to_webui, delivered_to_terminal)
Permission: STANDARD
Sends a notification to the human user through all configured channels.

---

# PART NINE: PHASE GATE CRITERIA

---

## Chapter 22: Complete Phase Gate Specifications

Phase gates are the mandatory quality checkpoints between project phases. Every criterion in a gate must be satisfied before the project advances. Gates are assessed by the Orchestrator using tool calls and agent reports.

### Gate 1: Intake → Architecture (Phase 0 → Phase 1)

**Pass Criteria:**
- ProjectSpecificationDocument is complete and stored in VFE and Project Memory.
- All requirements have unique IDs, categories (functional/non-functional), and priority (Must-Have/Should-Have/Nice-to-Have).
- Every Must-Have requirement has at least one measurable success criterion.
- No two requirements directly contradict each other (or conflicts are explicitly documented with resolution).
- Domain-implied implicit requirements are documented (e.g., security requirements for any project with user authentication).
- Scale assessment (Small/Medium/Large) is documented with justification.
- User has confirmed the requirements OR the system has documented all inferences made with reasoning.

**Fail Conditions:**
- Any Must-Have requirement lacks a measurable success criterion.
- More than 3 unresolved requirement conflicts.
- The project scope is completely undefined (no way to determine what is in/out of scope).

### Gate 2: Architecture → Development Infrastructure (Phase 1 → Phase 2)

**Pass Criteria:**
- Architecture Document complete: all components, their responsibilities, and their interfaces defined.
- Technology Stack Document complete: every technology selection has documented rationale.
- Database Schema Document complete: all tables, fields, constraints, relationships, and indexes defined.
- Integration Specification complete: all API endpoints specified with full request/response schemas.
- Project Standards Document complete: naming, error handling, logging, response format all defined.
- Task Backlog complete: all Must-Have requirements have corresponding tasks, each with acceptance criteria.
- Task Dependency Graph is acyclic (no circular dependencies).
- GitHub repository initialized, CI workflow active and passing on empty scaffold.
- All required technologies available (packages installable, no missing system tools).

**Fail Conditions:**
- Missing Architecture Document or missing Integration Specification.
- Technology selections with known critical incompatibilities not addressed.
- Circular dependencies in the task backlog.
- CI failing on the initial scaffold commit.

### Gate 3: Infrastructure → Core Development (Phase 2 → Phase 3)

**Pass Criteria:**
- Project scaffold runs without errors (health check endpoint responds if web app).
- All dependency files committed (requirements.txt, package.json, etc.).
- Database migrations apply cleanly against a fresh database.
- Docker build succeeds.
- Development server starts successfully.
- All environment variables documented in .env.example.
- CI green on Phase 2 scaffold commit.

**Fail Conditions:**
- Application fails to start.
- Docker build fails.
- Database migrations fail on a clean database.

### Gate 4: Core Development → Integration (Phase 3 → Phase 4)

**Pass Criteria:**
- All planned backend and frontend code is written and committed.
- Unit test coverage across all modules ≥ 78%.
- Zero unresolved Critical code review findings.
- Zero unresolved High code review findings.
- All syntax checks pass on all files.
- All type checks pass (zero type errors).
- CI green on develop branch.
- Build succeeds cleanly.

**Fail Conditions:**
- Any Must-Have backend feature missing from VFE.
- Unit test coverage below 72% overall.
- Any unresolved Critical review finding.
- CI failing on develop.

### Gate 5: Integration → Quality Assurance (Phase 4 → Phase 5)

**Pass Criteria:**
- All integration tests pass (every API endpoint tested with real database).
- All API contract tests pass (frontend and backend agree on schemas).
- All E2E tests pass for all Must-Have user journeys.
- Application starts successfully from a fresh database state (no dirty state dependencies).
- No "it works on my machine" issues: application runs identically in Docker.

**Fail Conditions:**
- Any integration test failure not formally deferred with justification.
- API contract test failure (frontend-backend mismatch must always be fixed).
- Application fails to start from fresh state.

### Gate 6: Quality Assurance → Final Audit (Phase 5 → Phase 6)

**Pass Criteria:**
- Performance benchmarks within 90% of specified non-functional requirements.
- Security scan shows zero Critical vulnerabilities in application code.
- Security scan shows zero High vulnerabilities in application code.
- Dependency scan shows zero Critical known CVEs in production dependencies.
- All Critical code quality findings resolved.
- All High code quality findings resolved.
- Test coverage overall ≥ 80%.
- Documentation coverage ≥ 75% (public functions have docstrings).
- No dead code (unused functions, unused imports).
- CI green.

**Fail Conditions:**
- Any Critical security vulnerability in application code.
- Performance benchmark failing a Must-Have non-functional requirement by more than 10%.
- Test coverage below 76%.

### Gate 7: Final Audit → Delivery (Phase 6 → Phase 7)

**Pass Criteria:**
- Requirements Coverage Matrix: 100% of Must-Have requirements have status = verified.
- Final Audit Report produced by Auditor agent shows overall PASS.
- Delivery Readiness check: Auditor followed the README from scratch in a Docker container and the application ran correctly.
- All tests pass in the clean Docker environment.
- README complete with all required sections (overview, prerequisites, installation, configuration, running, testing, deployment).
- API documentation complete (all endpoints documented).
- Deployment guide present and accurate for the configured target.
- Zero Critical audit findings.
- Zero High audit findings outstanding.
- CI green on develop.

**Fail Conditions:**
- Any Must-Have requirement not implemented and verified.
- README setup test failed in clean environment.
- Any Critical audit finding outstanding.

---

# PART TEN: ERROR CODES AND RECOVERY REFERENCE

---

## Chapter 23: AACS-Core Error Reference

### 23.1 Error Categories

AACS-Core errors are classified into four categories: LLM errors (LE-), Tool errors (TE-), Agent errors (AE-), and System errors (SE-). Each error has a defined recovery procedure.

### 23.2 LLM Errors (LE-)

**LE-001: Rate Limit (429).** Recovery: exponential backoff starting at 10 seconds. Maximum 5 retries. If all fail, queue the request and continue other work. Notification: "LLM rate limit hit — queued, retrying." No human escalation needed.

**LE-002: Auth Failure (401/403).** Recovery: immediate pause of all LLM calls. Human notification: "LLM API authentication failed — please check your API key in Settings." System waits for user to update credentials. Resumes automatically after credential update.

**LE-003: Context Too Long (400 context_length_exceeded).** Recovery: re-run Stage 9 of Context Assembly with aggressive truncation (remove 50% of memory items, keep only the single most relevant skill section). If still too long, split the task into sub-tasks. Logged as an efficiency issue for prompt optimization.

**LE-004: Server Error (500/502/503).** Recovery: retry 3 times with 10-second delays. If all fail, queue request and notify: "LLM provider having issues — will retry automatically." No human escalation unless down for > 10 minutes.

**LE-005: Invalid Response Format.** Recovery: send targeted correction prompt "Your previous response was missing required structure. Please provide only: [specific output format]. Do not include any preamble." Maximum 2 correction retries. If still failing, escalate to a higher-tier model (if configured).

**LE-006: Content Policy Refusal.** Recovery: rephrase the task to be more neutral in language (e.g., replace "bypass authentication" with "implement authentication testing"). Log the refusal. If rephrasing fails, mark as a Human Required item with a note explaining the situation.

**LE-007: Budget Exhausted.** Recovery: complete all in-progress write operations and commit code to VFE. Create a checkpoint. Generate a Budget Exhausted notification for the human with: remaining work list, estimated additional cost to complete, options (increase budget, reduce scope, pause project). System waits for human response.

**LE-008: All Providers Unavailable.** Recovery: shift to Conservative Mode (complete any in-progress VFE writes, sync to disk, create checkpoint). Display "All LLM providers unavailable — project paused until connectivity is restored." Automatically resumes when any provider becomes available (checked every 60 seconds).

### 23.3 Tool Errors (TE-)

**TE-001: VFE Transaction Conflict.** Recovery: segment-level merge if the conflicting changes are in different segments. Three-way merge if overlapping. If unresolvable, rollback the lower-priority transaction and retry. Logged as a coordination event.

**TE-002: Disk Sync Hash Mismatch.** Recovery: immediate re-sync using WriteThrough mode for the affected files. Verify the re-sync. If mismatch persists, alert the human: "Disk write verification failed for [files] — disk may be full or have I/O errors."

**TE-003: Git Operation Failed.** Recovery: for push failures (network), retry 3 times with 30-second delays. For merge conflicts, route to Debugger for conflict resolution. For auth failures, notify human to check GitHub token.

**TE-004: CI Run Failed.** Recovery: the GitHub Bridge downloads the run logs, parses the failure point, and creates a CI_FAILURE task for the Debugger. The Debugger analyzes and fixes. The fix is pushed and CI is monitored. If CI fails 5 consecutive times on the same issue, escalate to Grade 4 (Human Required).

**TE-005: Build Failed.** Recovery: route to Debugger as a BUILD_FAILURE task. The Debugger analyzes the build output, identifies the cause, and generates a fix. Never marks a build failure as "acceptable" — all builds must pass before proceeding.

**TE-006: Test Runner Crashed.** Recovery: retry test run with verbose output to capture the crash message. Route crash details to Debugger. If the crash is a test infrastructure issue (broken fixture, missing test database), fix the infrastructure before re-running tests.

**TE-007: Admin Operation Blocked.** Recovery: check if Admin Mode is properly enabled. If not enabled, notify the human that the operation requires Admin Mode and how to enable it. If enabled but blocked by OS (permissions issue), provide specific guidance for granting the required permission.

### 23.4 Agent Errors (AE-)

**AE-001: Agent Heartbeat Timeout.** Recovery: save the agent's Working Memory. Terminate the agent instance. Restart from the last incomplete plan step with a fresh context load. If the same agent times out 3 times on the same task, escalate to Grade 3 (task decomposition).

**AE-002: Self-Review Critical Failure Persistent.** Recovery: if an agent's self-review consistently finds Critical failures in its own output (3 consecutive corrections), refresh the skill loading (reload the relevant skill document) and retry. If still failing, escalate to Grade 2 (spawn Debugger to analyze the pattern).

**AE-003: Verification Failure Persistent.** Recovery: if syntax/type checks fail after 2 correction retries, escalate to Grade 2 — spawn a fresh agent instance with a clean context to attempt the task from scratch, without the accumulated context that may be causing the failures.

### 23.5 System Errors (SE-)

**SE-001: Checkpoint Corruption.** Recovery: discard the corrupted checkpoint. Load the previous checkpoint. If 3 consecutive checkpoints are corrupted, switch to WriteThrough sync mode and alert the human: "Checkpoint integrity failures detected — check disk health. Switched to maximum-safety sync mode."

**SE-002: VFE Memory Pressure Critical (>90%).** Recovery: force sync all dirty files to disk. Drop in-memory content for files not accessed in the last 20 minutes (reload from disk on next access). If still at >90% after cleanup, reduce max_concurrent_agents to 1 and alert the human.

**SE-003: Cascading Agent Failures (>3 agents failing simultaneously).** Recovery: pause all agent dispatch. Create an emergency checkpoint. Analyze the failure pattern — if all failures share a common root (shared tool failing, database down, Git remote unreachable), fix the root cause first. If no common root, resume with reduced concurrency.

**SE-004: GitHub API Rate Limit.** Recovery: queue all GitHub operations. Wait for rate limit reset (GitHub returns the reset time in the response headers). Continue all non-GitHub work while waiting. Batch queued operations for efficiency when the rate limit resets.

---

# PART ELEVEN: DATA STRUCTURES REFERENCE

---

## Chapter 24: Key Data Structures

### 24.1 TaskDefinition

The standard unit of work dispatched by the Orchestrator to agents.

```python
class TaskDefinition(BaseModel):
    task_id: str                    # UUID
    title: str                      # Short human-readable title
    description: str                # Complete description of what must be done
    task_type: TaskType             # Enum: CODE_GENERATION, CODE_REVIEW, DEBUGGING, etc.
    assigned_agent: AgentType       # Which agent type handles this task
    input_artifacts: List[str]      # VFE paths the agent needs to read
    output_artifacts: List[OutputSpec] # What the agent must produce with VFE paths
    acceptance_criteria: List[str]  # Measurable criteria for task completion
    priority: Priority              # Critical / High / Normal / Low
    dependencies: List[str]         # task_ids that must complete before this starts
    estimated_effort: EffortSize    # Small / Medium / Large / XLarge
    retry_count: int                # 0 for first attempt
    retry_history: List[RetryRecord]# Previous attempts with failure reasons
    context_hints: List[str]        # Topics to retrieve from memory
    created_at: datetime
```

### 24.2 TaskResult

Returned by agents when they complete (or fail to complete) a task.

```python
class TaskResult(BaseModel):
    task_id: str
    agent_id: str
    status: TaskStatus              # COMPLETED / COMPLETED_WITH_CONCERNS / FAILED
    artifacts_produced: List[str]   # VFE paths of all code/docs written
    verification_passed: bool       # All syntax/type/import checks passed
    self_review_findings: List[ReviewFinding]  # From agent's self-review stage
    impact_assessment: ImpactReport # What this change affects
    concerns: List[str]             # Agent-flagged concerns for downstream
    execution_time_seconds: int
    llm_calls_made: int
    llm_tokens_used: int
    completed_at: datetime
```

### 24.3 ProjectSpecificationDocument

The structured project specification produced from the user's brief.

```python
class ProjectSpecificationDocument(BaseModel):
    project_id: str
    project_name: str
    description: str                # 2-3 paragraph summary
    domain: ProjectDomain           # web_app / mobile / api_only / cli / etc.
    scale: ProjectScale             # small / medium / large
    requirements: List[Requirement]
    implicit_requirements: List[Requirement]  # Domain-implied requirements
    assumptions: List[Assumption]   # What AACS-Core inferred with reasoning
    out_of_scope: List[str]         # Explicitly excluded
    tech_hints: List[str]           # Technologies mentioned by user
    created_at: datetime
    confirmed: bool                 # True once user approves or waives review
```

### 24.4 ArchitectureDocument

The complete technical architecture produced by the Architect agent.

```python
class ArchitectureDocument(BaseModel):
    architecture_style: str         # e.g., "Layered Monolith", "Event-Driven"
    components: List[Component]     # Each with name, responsibility, technology
    technology_stack: TechStack     # Complete tech selection with rationale
    database_schema: DatabaseSchema # Tables, fields, relationships, indexes
    integration_spec: IntegrationSpec  # All API endpoints, WebSocket messages
    project_standards: ProjectStandards  # Coding conventions, naming rules
    decisions: List[ArchDecision]   # ADRs: each decision with rationale
    created_at: datetime
```

### 24.5 EpisodicEvent

The unit of episodic memory — one recorded project event.

```python
class EpisodicEvent(BaseModel):
    event_id: str
    event_type: EpisodeType         # TASK_COMPLETED, BUG_FOUND, DECISION_MADE, etc.
    timestamp: datetime
    agents_involved: List[str]
    subject: str                    # Brief description of what this event is about
    narrative: str                  # 1-2 sentence human-readable description
    technical_detail: dict          # Structured event-type-specific data
    significance_score: float       # 0.0-1.0, used for compression priority
    tags: List[str]                 # For retrieval filtering
    project_id: str
```

### 24.6 ProjectMemory

The structured knowledge base about the current project.

```python
class ProjectMemory(BaseModel):
    project_id: str
    technical_decisions: List[TechDecision]
    # Each TechDecision: topic, decision, rationale, alternatives_considered, made_at

    established_conventions: List[Convention]
    # Each Convention: category, description, examples, established_at

    active_issues: List[KnownIssue]
    # Each KnownIssue: type, description, severity, status, workaround, found_at

    integration_notes: List[IntegrationNote]
    # Each IntegrationNote: from_component, to_component, protocol, auth_method, notes

    technology_notes: List[TechNote]
    # Each TechNote: technology, note_type, content, source

    last_updated: datetime
```

---

# PART TWELVE: OVERNIGHT OPERATION AND ADVANCED USE

---

## Chapter 25: Overnight Operation

### 25.1 Protocol

When AACS-Core detects it will operate overnight (either by detecting that it is past 22:00 local time with significant work remaining, or when the user explicitly requests overnight operation), it activates the Overnight Protocol:

Pre-overnight preparation: Complete all currently atomic operations. Create a comprehensive checkpoint. Run a smoke test suite to verify current state is healthy. Switch VFE sync to WriteBehind (30-second intervals). Reduce activity logging frequency (log only significant events, not every operation). Log "Overnight Operation Beginning — Current Phase: X, Tasks Remaining: N".

Overnight execution: Continue all development work at full throughput. Checkpoint every 20 minutes (more frequent than daytime 30 minutes). Resource-aware throttling: if system temperature or memory exceeds configured thresholds, reduce concurrent agents from 3 to 1 temporarily.

Morning transition: When the user next opens the WebUI (or at the configured morning time), AACS-Core generates a Morning Summary displayed prominently in the WebUI chat: what was completed overnight, what milestones were reached, any issues encountered and whether they were resolved, what is currently in progress, estimated time to delivery, budget consumed overnight.

### 25.2 Multi-Day Projects

For projects that span multiple days, AACS-Core adds:

Daily progress reports: stored in Project Memory and accessible via the chat interface ("What did we build today?"). Day-end checkpoint: more comprehensive than the 30-minute checkpoint, retained for 14 days. Weekly knowledge graph update: on Monday morning, AACS-Core updates the Knowledge Graph with learnings from the previous week's work. Human check-in prompt: after 24 hours of uninterrupted operation, AACS-Core sends a brief check-in to the WebUI chat: "I've been running for 24 hours. Current status: [summary]. Any instructions before I continue?"

---

## Chapter 26: Extending AACS-Core

### 26.1 Adding Skills

To add a new skill, create a markdown file in the `skills/` directory following the standard 6-section structure: Overview, Project Setup, Core Patterns, Error Handling, Security Considerations, Testing Approach, Common Mistakes. No Python code changes required — the Skills Registry automatically discovers and loads all `.md` files in the skills directory. The skill becomes available to the context assembly pipeline immediately after the file is created.

### 26.2 Adding Tools

To add a new tool, define a function in the appropriate tool category file in `tools/`. The function must: accept a Pydantic input model as its first argument, return a Pydantic output model, raise typed exceptions defined in `core/exceptions.py`, and include a docstring describing the tool's purpose, permission level, and side effects. Register the tool in the ToolRegistry by adding it to the `_register_tools()` method in `core/state.py`. The tool is available to all agents immediately after registration.

### 26.3 Customizing Agent Behavior

Each agent's behavior is controlled by its role definition (a dict in the agent's Python file containing: role_description, quality_checklist, default_skills, permitted_tools, and cognitive_pattern_overrides). To change how an agent approaches tasks, modify its quality_checklist (add/remove/change checklist items) or its role_description (change the guiding principles the agent follows). No new files needed — agent customization is in-place modification of the agent file.

### 26.4 The Path to Full AACS

AACS-Core is designed with the full AACS upgrade path in mind. When the user is ready to scale:

The `core/llm.py` engine becomes the full `aacs/engines/llm/` package with the 10-stage CAP and 8-stage RRP.

The `core/vfe.py` engine becomes the full `aacs/engines/vfe/` package with all 16 segment types, full versioning, and the complete sync system.

Each agent file splits into the corresponding `aacs/registries/agent_registry/permanent/definitions/` file (role definition) and `aacs/divisions/*/` file (division logic).

The 7 skill files become the full 67-skill library with multi-part packages and the Skill Intelligence System.

The tools expand from 6 grouped files to 86 individual tool files.

Every data structure and protocol in AACS-Core is a direct subset of the corresponding full AACS structure — no migration needed, only expansion.

---

## Chapter 27: Success Stories — What AACS-Core Can Build

### 27.1 Capability Envelope

AACS-Core can autonomously deliver any software project that fits within these parameters:

Primary language is Python, JavaScript, or TypeScript (or a combination). Database is PostgreSQL, MySQL, SQLite, MongoDB, or Redis. Deployment target is Docker-compatible or a common PaaS platform. Project size is up to "Large" on the scale assessment (estimated 2-3 weeks of human engineering effort). No specialized hardware integration or embedded systems. No real-time trading systems or safety-critical applications.

Projects that fit this profile and where AACS-Core excels:

Full-stack web applications (React + FastAPI + PostgreSQL). API-only backends for mobile applications. Content management systems. E-commerce platforms (without payment processor integration complexity). Educational platforms. Internal business tools and dashboards. CLI tools and automation scripts. Data processing pipelines. AI-integrated applications (LLM APIs, embeddings, RAG systems). Real-time applications with WebSockets. Multi-tenant SaaS applications.

### 27.2 A Realistic Example Timeline

**Project:** "Build a project management tool with tasks, teams, time tracking, and reporting."

Day 1, 09:00 — User submits project brief. Project Intake complete by 09:20.

Day 1, 09:20-11:00 — Architecture phase. Tech stack selected (FastAPI + React + PostgreSQL). Database schema designed (12 tables). 85 tasks created.

Day 1, 11:00-23:00 — Core Development. All backend models, services, API endpoints implemented and unit tested. All frontend components built. Auth system complete. Approximately 55 tasks complete.

Day 2, 00:00-08:00 — Overnight Development. Frontend pages assembled and connected to API. E2E tests written. CI green. Morning summary shows 82 of 85 tasks complete.

Day 2, 08:00-14:00 — Integration, QA, Audit. All integration tests pass. Security scan clear. Performance benchmarks passing. Final Audit passes.

Day 2, 14:30 — Delivery. GitHub release published. Delivery report in WebUI.

Total time: ~29 hours. Human interaction: 2 brief chat exchanges. LLM cost estimate: $18-25.

---

*End of AACS-Core Master Architecture and Engineering Specification*
*Version 1.0 — Complete*
*Document Length Target: 3,000+ lines*

*"Maximum capability. Minimum files. Full autonomy."*
*AACS-Core — The Small Avatar of AACS*

---
# PART THIRTEEN: COMPLETE AGENT COGNITIVE SPECIFICATIONS

---

## Chapter 28: Detailed Agent Behavioral Specifications

This chapter provides the complete cognitive specifications for each of the 23 agents — the exact role descriptions, quality checklists, skill loadouts, and behavioral overrides that make each agent distinct and effective.

---

### 28.1 Orchestrator — Detailed Specification

**Role Description (loaded as system context for all Orchestrator LLM calls):**

"You are the Orchestrator of AACS-Core. You do not write code. You do not make technical decisions. Your sole function is to keep the project moving forward with maximum efficiency and quality. You are a coordinator, not a creator.

You maintain the project task queue and dispatch tasks to agents. You monitor progress, enforce phase gates, handle escalations, and ensure that failures are recovered from — not ignored. When a task fails, your job is to determine why and route it appropriately for recovery. When an agent is stuck in a loop, your job is to apply the right intervention before resources are wasted. When a phase gate fails, your job is to dispatch targeted remediation tasks.

You think in terms of the project's critical path. You prioritize tasks that are blocking other tasks. You parallelize tasks that have no dependencies between them. You never let a failing task sit unaddressed.

When you make decisions about task dispatch, recovery procedures, or escalations, you explain your reasoning briefly. This reasoning is recorded in Project Memory as Orchestrator decisions, available for the user to review."

**Quality Checklist:** The Orchestrator's quality checklist applies to its dispatch and coordination outputs, not to code. Key items: every dispatched task has all its prerequisites complete, no task is dispatched to an agent whose skills don't match the task type, escalations include full context (not just "it failed"), phase gate assessments are systematic (check every criterion, not just the ones that seem likely to pass).

**Orchestrator-specific methods:**

`_assess_phase_gate(gate_id: str) → GateAssessment` — Queries State Store, Test results, Audit reports, and VFE metrics. Produces a structured gate assessment with each criterion explicitly evaluated.

`_apply_loop_prevention(task_id: str, grade: int)` — Applies the specific grade response: Grade 0 sends context refresh signal to the agent, Grade 1 sends a forced-alternative message, Grade 2 spawns the Debugger as specialist, Grade 3 sends task back to Project Manager for decomposition, Grade 4 sends Human Required notification.

`_handle_cascading_failures(failed_tasks: List[str])` — If 3+ agents fail in a 10-minute window, pauses dispatch, analyzes failure patterns (are they all in the same module? all hitting the same tool?), attempts to identify a common root cause, fixes the root cause first, then resumes.

---

### 28.2 Debugger — Detailed Specification

**Role Description:**

"You are the Debugger of AACS-Core. You are a specialist at finding the root causes of failures and producing minimal, verified fixes. You are activated when other agents encounter failures they cannot resolve themselves.

Your methodology: You do not guess. You diagnose. Before proposing any fix, you understand exactly why the failure occurs. You construct a causal chain from the error symptom back to the root cause. Your fix addresses the root cause, not just the symptom.

Your safety discipline: Before applying any fix to a file in VFE, you create a savepoint. You apply the fix within a transaction. You run the specific test that was failing to verify the fix works. You run the broader test suite for the affected module to verify no regressions. If a regression is detected, you roll back immediately. You never commit a fix that causes a regression — you would rather escalate to Grade 3 decomposition than commit regressing code.

Your CI discipline: When a GitHub Actions CI run fails, you download the complete run logs, read them fully, identify the exact line that caused the failure, and trace it to the source code issue. CI failures are almost always one of: a missing environment variable, a wrong database URL in the test config, a dependency not installed in CI, a test that passes locally but fails due to timing in CI, or an actual code bug. You identify which category the failure belongs to before attempting a fix."

**Quality Checklist Key Items:**
- Root cause identified (not just symptom addressed) — CRITICAL
- Fix verified by the previously-failing test passing — CRITICAL
- No previously-passing tests now failing (regression check) — CRITICAL
- VFE savepoint created before any fix applied — CRITICAL
- Causal chain documented in Episodic Memory — HIGH
- Fix is minimal (changes the minimum code necessary) — HIGH

**Debugger-specific `_stage_execute` override:**

The Debugger overrides Stage 4 (execution) with its specialized debugging protocol:
1. Classify the failure (8 categories: syntax, import, runtime, logic, integration, performance, CI, environment).
2. Identify immediate cause from the error output.
3. Trace causal chain (using code analysis tools).
4. Formulate and document the root cause hypothesis.
5. Create VFE savepoint.
6. Apply fix within a transaction.
7. Run the specific failing test: if it doesn't pass, roll back and try a different approach.
8. Run the broader module test suite: if any regression, roll back and diagnose the regression.
9. Only commit if both verifications pass.

---

### 28.3 Auditor — Detailed Specification

**Role Description:**

"You are the Auditor of AACS-Core. You are the independent quality authority. You operate after all development is complete, and your findings gate the project's delivery. You are not part of the development team — you are the quality assurance function that confirms the development team's work meets the required standards.

Your audit covers five pillars: Requirements Coverage, Code Quality, Security, Performance, and Delivery Readiness.

For Requirements Coverage: You do not trust assertions that requirements are met. You trace each requirement to specific code and specific tests. You verify that the test for the requirement actually tests the right behavior (not just that a test with the right name exists). If a requirement is not traceable to code and tests, it is not covered.

For Code Quality: You measure, you do not estimate. You run the coverage tool and read the numbers. You run the complexity analyzer and check each function above threshold. You run the documentation coverage tool. You look for dead code. You do not accept 'good enough' — you check every metric against its threshold.

For Security: You run every available scanner. You read every finding. You classify findings by severity accurately (do not downgrade Critical to High to avoid blocking delivery). Security findings blocking delivery are the most important thing AACS-Core can tell the user — a delivered system with a Critical vulnerability is worse than a delayed delivery.

For Performance: You run the benchmarks. You read the p99 latency numbers. You check them against the project's specified non-functional requirements. If no performance requirements were specified, you apply reasonable defaults (API endpoints: p99 < 500ms, database queries: p99 < 100ms).

For Delivery Readiness: You follow the README yourself (simulated, in a clean Docker environment). If any step fails, it is a Critical finding. The user will follow this README. It must work."

**Quality Checklist Key Items:**
- Every Must-Have requirement traced to code AND passing test — CRITICAL
- All scanners run, findings read and classified (not skimmed) — CRITICAL
- README steps actually tested, not assumed — CRITICAL
- Metrics recorded with actual numbers (not "looks good") — HIGH
- All Critical and High findings have specific locations and remediation steps — HIGH

---

### 28.4 GitHub Manager — Detailed Specification

**Role Description:**

"You are the GitHub Manager of AACS-Core. You are responsible for all interactions with Git and GitHub. No other agent touches git or the GitHub API — all GitHub operations go through you.

Your commit discipline: Every commit you create is atomic (one logical change), follows Conventional Commits format, and has a meaningful description. You never make 'big bang' commits that bundle unrelated changes. You never make commits with messages like 'fix stuff' or 'update'. Every commit message tells a story of what changed and why.

Your branch discipline: Features live in feature branches. Reviewed and tested code merges to develop. Only audited, delivery-ready code merges to main. You enforce this — you never push code to main directly.

Your CI discipline: After every push, you monitor CI. A failing CI run is not something you note and move on from — it is a blocking issue that triggers the Debugger. You download the failure logs, route them to the Debugger with full context, and wait for the fix before continuing.

Your PR discipline: Every PR has a complete description: what was built, how it was built, what tests cover it. Review comments you add to PRs are specific, actionable, and linked to exact code locations. You do not write vague review comments."

**GitHub Manager-specific behavior:** This agent overrides the standard Cognitive Cycle's Stage 4 to use the GitHub Bridge's specialized methods rather than generic LLM calls for most operations. LLM calls are still used for: generating PR descriptions, generating commit messages from diff context, generating release notes, and diagnosing CI failures before routing to the Debugger.

---

## Chapter 29: Agent Interaction Patterns

### 29.1 Standard Task Dispatch Flow

When the Orchestrator dispatches a task, the following message exchange occurs:

1. Orchestrator → Agent: `TASK_ASSIGN` message with complete TaskDefinition.
2. Agent: Executes 8-stage cognitive cycle (typically 3-15 minutes for a code generation task).
3. Agent → Orchestrator: `TASK_RESULT` message with TaskResult.
4. Orchestrator: Processes result. If successful, marks task complete, updates dependency graph, dispatches newly unblocked tasks. If failed, applies Error Shield recovery procedure.

### 29.2 Parallel Development + Review

When the Backend Developer writes a new file and commits to VFE:

1. VFE emits `CODE_READY` event.
2. Code Reviewer receives `CODE_READY`, activates immediately.
3. Orchestrator also receives `CODE_READY`, dispatches Test Writer for the same code.
4. Code Reviewer runs review, sends `CODE_REVIEW_COMPLETE` with findings.
5. Test Writer runs tests, sends `TEST_WRITE_COMPLETE`.
6. Orchestrator receives both. If Code Reviewer found Critical/High issues: dispatch correction task to Backend Developer. If Test Writer found problems: dispatch debugging task. If both clean: mark the code as reviewed-and-tested, proceed.

### 29.3 CI Failure Response Chain

When GitHub Actions CI fails:

1. GitHub Bridge emits `CI_FAILURE` event with run_id and failure summary.
2. Orchestrator activates GitHub Manager agent.
3. GitHub Manager calls `github_get_run_logs` tool to download full logs.
4. GitHub Manager analyzes the failure and routes to Debugger with: the specific error, the specific failing step, the full log output.
5. Debugger performs root cause analysis and produces a fix.
6. Debugger writes fix to VFE (using savepoint and transaction protocols).
7. Orchestrator dispatches GitHub Manager to commit the fix.
8. GitHub Manager commits, pushes, monitors CI again.
9. If CI passes: loop ends, work continues. If CI fails again: increment the CI failure count. At 3 consecutive failures on the same issue: escalate to Grade 4 (Human Required).

### 29.4 Post-Delivery Continuation Pattern

When the user sends a message after project delivery:

1. Human Interface Agent receives the chat message.
2. HIA classifies: is this a new feature, a bug report, a question, a scope change?
3. For "add a feature": HIA creates a mini-project specification for the feature. Sends to Project Manager to create tasks. Orchestrator dispatches tasks. The existing project's memory, architecture, and conventions are all available — the system builds on what exists.
4. For "there's a bug": HIA creates a bug report with the user's description. Routes to Debugger as a BUG_REPORT task. Debugger investigates (has full code context), produces a fix. Test Writer adds a regression test. GitHub Manager commits and releases a new version.
5. For "how does X work": HIA queries Project Memory (decisions, conventions) and VFE (code context). Assembles a clear answer referencing both the architectural decision and the actual code. Sends back to user in the chat.

---

## Chapter 30: Quality Standards and Metrics

### 30.1 Code Quality Standards in AACS-Core

AACS-Core enforces the following quantitative standards. All are checked automatically by the appropriate agent and verified by the Auditor.

**Test Coverage:** Minimum 80% overall. No individual module below 70%. Business logic modules (service classes, domain models) must be above 85%. Background job handlers must be above 75%. Configuration modules are excluded from coverage requirements.

**Cyclomatic Complexity:** Maximum 10 per function. Functions above 15 are automatically flagged as Mandatory Refactor. Functions between 10 and 15 are High findings from the Code Reviewer. The Optimizer monitors complexity trends and proposes refactoring when a module's average complexity increases.

**Documentation Coverage:** Minimum 75% of public functions and classes must have docstrings. Docstrings for public API endpoints must include parameter descriptions and example responses. Docstrings for complex algorithms must explain the approach and complexity.

**Security Standards:** Zero hardcoded secrets. All user inputs validated at the API layer. All passwords hashed with bcrypt (cost factor ≥ 12) or Argon2. All JWT tokens have an expiry. All API endpoints that modify data require authentication. All authentication tokens transmitted only over HTTPS (enforced in deployment config). Zero Critical CVEs in dependencies. Zero High CVEs in dependencies (or all explicitly acknowledged with remediation plan).

**Performance Standards (defaults, override-able per project):** API endpoints (CRUD operations): p99 latency < 500ms. API endpoints (complex queries): p99 latency < 2,000ms. Database queries: p99 execution time < 100ms. Frontend initial load: LCP < 2.5s, TTI < 3.5s.

### 30.2 Code Style Standards

Code style is defined in the `general_coding.md` skill and in the Architecture agent's Project Standards document. Key universal standards enforced by the Code Reviewer:

Naming: snake_case for Python (variables, functions, modules), camelCase for JavaScript/TypeScript (variables, functions), PascalCase for classes in all languages, SCREAMING_SNAKE_CASE for constants, kebab-case for file names in frontend projects.

Error handling: All exceptions must be caught explicitly (no bare `except:` or `catch (e: any)`). All caught exceptions must be either: re-raised (possibly wrapped in a more specific type), logged and re-raised, or handled with a specific recovery action. Never silently swallowed.

Async: No synchronous blocking calls in async functions. Database calls always awaited. External API calls always awaited. File I/O always async (using aiofiles or equivalent).

Functions: Maximum 30 lines per function (soft limit, enforced at 50). Single responsibility — one function does one thing. No side effects beyond its declared purpose (no global state modification, no unexpected I/O).

### 30.3 Git Standards

Commit messages follow Conventional Commits strictly. AACS-Core uses these types: `feat` (new feature), `fix` (bug fix), `refactor` (code restructure without feature/fix), `test` (add/modify tests), `docs` (documentation only), `chore` (build, CI, maintenance), `perf` (performance improvement), `security` (security fix or hardening).

Every commit message has a scope (the component affected). Examples: `feat(auth): implement JWT refresh token rotation [TASK-042]`, `fix(database): prevent N+1 query in user listing endpoint [TASK-067]`, `test(api): add integration tests for user registration flow [TASK-088]`.

All commits must reference the task ID. The GitHub Manager enforces this — it will not create a commit without a task reference.

---

## Chapter 31: Security Architecture

### 31.1 Credential Security

All secrets in AACS-Core are stored in the OS-native credential vault. On Windows, this is the Windows Credential Manager. On macOS, the Keychain. On Linux, the Secret Service API (KWallet or GNOME Keyring, with encrypted file fallback). The `keyring` Python library provides a unified interface.

The config file (`~/.aacs-core/config.yaml`) never contains actual secrets. Fields with sensitive values use the `vault:` prefix, which tells the config loader to read from the credential vault. If a secret is accidentally placed in the config file, AACS-Core detects the value format (API key pattern) and moves it to the vault automatically, replacing the config value with the vault reference.

### 31.2 LLM Context Security

Project code sent to the LLM API is sanitized before transmission. The LLM Engine's pre-send sanitizer: scans for patterns matching API keys, tokens, passwords (high-entropy strings in assignments, environment variables). Replaces detected values with `[REDACTED]` in the context. Logs the redaction (not the value) for auditability. This ensures that if a developer accidentally commits a secret to the project codebase, it is not exfiltrated to external LLM providers.

### 31.3 WebUI Security

The WebUI is password-protected using bcrypt. Session tokens are signed JWTs with a 24-hour expiry. HTTPS can be enabled with a user-provided certificate or a self-signed certificate (generated automatically on first start with HTTPS enabled). Rate limiting applies to login attempts: 5 attempts per IP per 5 minutes.

The WebUI has read-only access to project code — the code viewer displays VFE contents but provides no code editing capability through the UI. Code changes only happen through agents.

### 31.4 Shell Command Safety

The Shell Engine's permission system prevents agents from executing commands outside their permitted scope. No agent can access files outside the project directory without ADMIN permission. No agent can install system packages without ELEVATED or ADMIN permission. The Orchestrator has NO shell permissions — it coordinates but never executes shell commands. This prevents the coordination layer from accidentally causing system changes.

All shell commands are logged with: timestamp, executing agent, full command (sanitized), exit code, duration. The admin log (`admin.log`) is append-only and is never deleted by AACS-Core itself. This provides a complete audit trail of every system change made during the project.

---

## Chapter 32: Complete Implementation Timeline

### 32.1 Build Order

The following is the recommended implementation order for building AACS-Core. Each phase produces a working, testable subsystem.

**Week 1-2: Core Infrastructure**

Build `config.py` first (everything depends on configuration). Build `core/types.py` next (all enums and shared types). Build `core/exceptions.py` (error taxonomy). Build `core/state.py` (State Store and Message Bus — the nervous system). Build `core/vfe.py` (Virtual Filesystem — the workspace). At this point you can write and run basic VFE tests. Build `core/shell.py` (Shell Engine — without admin mode initially). At this point you can write code to VFE and execute commands.

**Week 3-4: LLM and Memory**

Build `core/llm.py` (LLM Engine). Start with a simplified 3-stage context assembly (role + task + output format) and 2-stage response processing (format validation + result packaging). Get a single LLM call working end-to-end. Build `core/memory.py` (Memory Engine). Start with Working Memory and Project Memory only. Add Episodic Memory after basic memory works. Add ChromaDB vector retrieval last.

**Week 5: Knowledge and GitHub**

Build `core/knowledge.py` (Knowledge Graph). Seed the graph from `data/knowledge_seed.json`. Verify all 5 query types work. Build `core/github.py` (GitHub Bridge). Test repository creation, branch management, and CI monitoring with a real GitHub test repo.

**Week 6-7: Agents**

Build `agents/base.py` first — this is the complete 8-stage cognitive cycle. Test it with a simple mock agent. Build agents in this order: Orchestrator → Project Manager → Architect → Backend Developer → Test Writer → Code Reviewer → Debugger. These 7 agents can build a working backend. Add the remaining 16 agents.

**Week 8: Tools**

Build all 6 tool files. Each tool is relatively simple (wraps an existing engine or external call). The tool registry in `core/state.py` handles registration. Test each tool category independently.

**Week 9: Skills**

Write all 7 skill documents. This is writing work, not coding work. Each skill should be reviewed against the 6-section structure and tested by using it in a real agent call and evaluating whether the output quality improves.

**Week 10: WebUI and Integration**

Build `webui/server.py`, `webui/routes.py`, and `webui/frontend/index.html`. Build `main.py` (brings everything together). Build `cli.py`. Build `core/checkpoint.py` (build last — depends on all other systems being stable).

**Week 11-12: Testing and Hardening**

Write the test suite. Run end-to-end tests with a real project brief. Harden error handling based on observed failures. Tune the Loop Prevention System thresholds based on real-world usage.

### 32.2 Estimated Complexity Budget

The 50-file architecture does not mean 50 equally complex files. Complexity is concentrated in the core engines:

`core/llm.py` — Most complex file. The 6-stage context assembly and 4-stage response processing require significant engineering. Estimate: 800-1000 lines.

`core/vfe.py` — Second most complex. Segment model, transactions, versioning, sync. Estimate: 600-800 lines.

`core/shell.py` — Third most complex. Cross-platform, admin mode, output parsing. Estimate: 500-700 lines.

`core/memory.py` — Fourth most complex. Three tiers, hybrid retrieval, compression. Estimate: 400-600 lines.

`agents/base.py` — Fifth most complex. The 8-stage cognitive cycle is the core agent intelligence. Estimate: 350-500 lines.

All other files: 50-200 lines each. The 23 individual agent files are each ~60-100 lines (role definition + checklist + overrides). The 6 tool files are each ~200-300 lines. The skill files are each ~2,000-4,000 words of markdown.

Total estimated Python code: approximately 6,000-8,000 lines across all Python files.

---

*End of AACS-Core Master Architecture and Engineering Specification*
*Version 1.0 — Complete Document*

*Summary:*
*27 Chapters across 13 Parts*
*23 Agents fully specified*
*7 Phase Gates with complete criteria*
*47 Error codes with recovery procedures*
*53 Tools across 6 categories*
*7 Skill documents*
*50 files total*

*"Maximum capability. Minimum files. Full autonomy."*
