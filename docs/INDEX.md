# Claude Code — Project Knowledge Base

> Comprehensive index of the leaked Claude Code source (Anthropic's AI coding CLI).
> Extracted from the npm sourcemap published March 31, 2026.
> This is a read-only research mirror — no build system is present.

---

## Table of Contents

1. [Repository Overview](#1-repository-overview)
2. [Core Systems](#2-core-systems)
3. [Tool Ecosystem](#3-tool-ecosystem)
4. [Services Layer](#4-services-layer)
5. [State Management](#5-state-management)
6. [Permission System](#6-permission-system)
7. [Message Flow](#7-message-flow)
8. [Memory Architecture](#8-memory-architecture)
9. [CLI Commands](#9-cli-commands)
10. [UI & Rendering](#10-ui--rendering)
11. [Multi-Agent & Coordination](#11-multi-agent--coordination)
12. [IDE Bridge](#12-ide-bridge)
13. [Notable Subsystems](#13-notable-subsystems)
14. [Startup & Performance](#14-startup--performance)
15. [Extension Points](#15-extension-points)
16. [Directory Map](#16-directory-map)

---

## 1. Repository Overview

| Property | Value |
|---|---|
| Language | TypeScript / TSX |
| Runtime | Bun v1.0+ (Node.js v18+ fallback) |
| UI Framework | React + Ink (custom fork) |
| CLI Framework | Commander.js |
| LLM SDK | `@anthropic-ai/sdk` |
| Protocol | Model Context Protocol (`@modelcontextprotocol/sdk`) |
| Schema Validation | Zod |
| Architecture | Monolithic single-binary CLI with modular subsystems |

This is not a simple CLI — it is a **full agentic framework** with terminal rendering, 40+ tools, multi-agent orchestration, memory consolidation, plugin system, and deep IDE integration.

---

## 2. Core Systems

### `src/main.tsx` — CLI Entrypoint (~4,700 LOC)
- Commander.js argument parsing and fast-path exits (`--version`, `--dump-system-prompt`)
- React/Ink terminal renderer initialization
- Session management, startup profiling (`profileCheckpoint()`)
- Parallel prefetch coordination: MDM config, keychain, GrowthBook feature flags
- Feature-gated module loading (`feature('COORDINATOR_MODE')`, etc.)

### `src/QueryEngine.ts` — LLM Orchestration (~1,300 LOC)
- Core API query loop: sends messages, receives streaming responses
- Manages tool execution queue and result injection
- Token budget tracking and extended context (1M token) support
- Auto-compaction triggers when context approaches limits
- Cost tracking per query

### `src/query.ts` — Query Processing (~1,700 LOC)
- Tool result processing pipeline
- Message normalization (normalizes content blocks across API versions)
- Compaction: summarizes old messages to free context budget
- Reactive compaction triggers based on token thresholds

### `src/Tool.ts` — Tool Abstraction (~792 LOC)
- Abstract base class all tools extend
- Defines `ToolInputJSONSchema`, `execute(context)`, `ToolResultBlockParam`
- `PermissionContext` passed to every tool call
- `ToolProgressData` for streaming progress updates
- `canUseTool()` — runtime gating based on permission mode and command context

### `src/Task.ts` — Task State Machine (~125 LOC)
- 7 task types: `pending`, `in_progress`, `completed`, `failed`, `cancelled`, `paused`, `waiting`
- Task lifecycle with parent/child relationships
- Used by TaskCreate/TaskUpdate/TaskList tools for in-session work tracking

### `src/tools.ts` — Tool Registry
- Central export of all 43 built-in tools
- Applies feature flags to conditionally include gated tools
- Order matters: determines tool priority in system prompt

### `src/commands.ts` — Command Registry
- Maps slash command names to command definitions
- Declarative: each command has `type`, `allowedTools[]`, `getPromptForCommand()`

---

## 3. Tool Ecosystem

All tools live in `src/tools/` and extend `Tool.ts`.

### File Operations
| Tool | File | Purpose |
|---|---|---|
| `FileReadTool` | `FileReadTool.ts` | Read files; supports pagination for large files |
| `FileEditTool` | `FileEditTool.ts` | Exact string replacement with uniqueness enforcement |
| `FileWriteTool` | `FileWriteTool.ts` | Write/overwrite files; requires prior read |
| `GlobTool` | `GlobTool.ts` | Pattern-based file discovery, sorted by mtime |
| `GrepTool` | `GrepTool.ts` | ripgrep-backed content search with full regex |
| `NotebookEditTool` | `NotebookEditTool.ts` | Jupyter notebook cell editing |

### Execution
| Tool | Purpose |
|---|---|
| `BashTool` | Shell command execution with 2-min timeout |
| `PowerShellTool` | Windows PowerShell execution |
| `REPLTool` | Interactive REPL with persistent state |
| `WorkflowTool` | Structured multi-step workflow execution |
| `SleepTool` | Delay execution (used in agent loops) |

### Information & Search
| Tool | Purpose |
|---|---|
| `WebFetchTool` | HTTP fetch with HTML→markdown conversion |
| `WebSearchTool` | Web search (Brave/Bing integration) |
| `LSPTool` | Language server: go-to-def, hover, diagnostics |

### Task Management
| Tool | Purpose |
|---|---|
| `TaskCreateTool` | Create tracked tasks with subtasks |
| `TaskGetTool` | Read task state |
| `TaskUpdateTool` | Transition task status |
| `TaskListTool` | List all session tasks |
| `TaskOutputTool` | Stream task output |
| `TaskStopTool` | Cancel running tasks |

### Multi-Agent & AI
| Tool | Purpose |
|---|---|
| `AgentTool` | Spawn sub-agents with isolated context |
| `SkillTool` | Execute slash commands as tools |
| `SendMessageTool` | Send messages to other agents |
| `AskUserQuestion` | Prompt user for clarification (blocking) |
| `BriefTool` | Summarize context for handoffs |

### MCP Integration
| Tool | Purpose |
|---|---|
| `MCPTool` | Execute tools on connected MCP servers |
| `ListMcpResourcesTool` | List MCP server resources |
| `ReadMcpResourceTool` | Read MCP resource content |

### System & Configuration
| Tool | Purpose |
|---|---|
| `EnterPlanMode` / `ExitPlanMode` | Toggle plan mode (no tool execution) |
| `EnterWorktree` / `ExitWorktree` | Switch git worktree context |
| `TodoWriteTool` | Write todo lists |
| `CronCreate` / `CronList` / `CronDelete` | Schedule recurring agent runs |
| `RemoteTriggerTool` | Trigger remote agent executions |

---

## 4. Services Layer

All services live in `src/services/`.

### API (`services/api/`)
- `claude.ts` — Anthropic API client with retry logic, streaming, and auth headers
- `bootstrap.ts` — Server config fetch on startup (model list, feature flags)
- `logging.ts` — Request/response logging with PII scrubbing

### MCP (`services/mcp/`)
- `McpClient.ts` — MCP server lifecycle management (start, health-check, restart)
- `McpAuth.ts` — OAuth flow for MCP servers requiring authentication
- `McpTransport.ts` — stdio / SSE / WebSocket transport abstraction
- `McpPermissions.ts` — Per-server tool permission management
- Supports official servers (filesystem, git, etc.) and community servers

### Memory (`services/memory/`)
- `sessionMemory.ts` — In-session context extraction and summarization
- `teamSync.ts` — Team-shared memory synchronization
- Feeds into `autoDream` for durable persistence

### Auto-Compaction (`services/compaction/`)
- `autoCompact.ts` — Triggered when token budget exceeds threshold
- `reactiveCompact.ts` — Real-time compaction during long tool chains
- Summarizes oldest messages while preserving tool results and key context

### autoDream (`services/autoDream/`)
- Background subagent that runs on 24-hour cadence
- Orient → Gather → Consolidate → Prune lifecycle
- Reads MEMORY.md, finds new signals from daily logs, updates durable memory files
- Runs `/dream` command as a forked subagent with isolated context

### Analytics (`services/analytics/`)
- GrowthBook feature flag evaluation (A/B testing, rollouts)
- Anthropic telemetry event logging (opt-out supported)
- Privacy-respecting metrics collection

### Plugins (`services/plugins/`)
- Plugin discovery from `~/.claude/plugins/`
- Plugin tool merging into main tool registry
- Plugin lifecycle management

### Other Services
- `lsp/` — Language server protocol client and diagnostics cache
- `oauth/` — Multi-provider OAuth (GitHub, Google, Anthropic)
- `limits/` — Rate limiting and quota management (policy limits)
- `mdm/` — Mobile device management config reading (enterprise)

---

## 5. State Management

### `src/state/AppStateStore.ts`
- Central `DeepImmutable<AppState>` — mutations are compile-time forbidden
- Zustand-like store pattern (no Redux, no React Context dispatch)
- Subscription mechanism for React component updates
- Key state slices:
  - `messages` — full conversation history
  - `tools` — registered tool instances
  - `permissionMode` — current permission level
  - `session` — active session metadata
  - `mcpServers` — connected MCP server states
  - `tasks` — in-session task tree

---

## 6. Permission System

### Three Permission Modes
| Mode | Behavior |
|---|---|
| `default` | Prompt user for potentially dangerous tools |
| `always-ask` | Prompt for every tool execution |
| `bypass` | Trust all tools (never prompt) |

### Rule Types (highest priority first)
1. **deny** — always block, no override
2. **allow** — always permit without prompt
3. **ask** — prompt user each time

### Rule Matching
- Pattern matching with glob support (e.g., `Bash(git *)`)
- MCP server-prefixed rules (e.g., `mcp__filesystem__read_file`)
- Content-based path restrictions (e.g., allow only within project root)
- `alwaysAllowRules` per command (e.g., `/commit` only allows `Bash(git *)`)

---

## 7. Message Flow

```
User Input
    │
    ▼
processUserInput()
    │  • slash command detection
    │  • memory attachment
    │  • tool context injection
    ▼
QueryEngine.query()
    │
    ▼
Anthropic API (streaming)
    │  • extended thinking (beta)
    │  • tool_use blocks detected
    ▼
StreamingToolExecutor
    │  • canUseTool() permission check
    │  • permission prompt if needed
    │  • parallel tool execution
    │  • ToolProgressData streaming
    ▼
Tool Results
    │  • normalized to ToolResultBlockParam
    │  • appended to message history
    ▼
Loop or Stop
    │  • stop_reason == 'end_turn' → done
    │  • max_turns exceeded → done
    │  • auto-compaction if needed
    ▼
Persist + Render
    • session transcript written to disk
    • React/Ink re-render with new state
```

---

## 8. Memory Architecture

### File Structure
```
~/.claude/
├── MEMORY.md              # Global persistent memory index
├── memory/                # Memory files by topic
│   ├── user_profile.md
│   ├── feedback_*.md
│   └── project_*.md
└── projects/<hash>/       # Per-project memory
    └── MEMORY.md
```

### memdir (`src/memdir/`)
- Reads/writes MEMORY.md and linked memory files
- Types: `user`, `feedback`, `project`, `reference`
- Each memory has frontmatter: `name`, `description`, `type`
- MEMORY.md is a 200-line-max index (truncated beyond that)

### autoDream Consolidation
- Runs daily as background subagent
- Reads session transcripts from `~/.claude/projects/<hash>/sessions/`
- Extracts durable signals (corrections, confirmed patterns, project decisions)
- Updates/creates memory files, prunes stale entries

### Session Memory (`services/memory/sessionMemory.ts`)
- In-session extraction of notable events
- Feeds into end-of-session consolidation
- Team sync for shared memory across team members

---

## 9. CLI Commands

101 slash commands defined in `src/commands/`. Key categories:

### Session Management
`/session`, `/resume`, `/rewind`, `/clear`, `/compact`, `/branch`

### Configuration
`/config`, `/keybindings`, `/theme`, `/login`, `/logout`, `/upgrade`, `/doctor`

### Agents & Orchestration
`/agents`, `/status`, `/monitor`, `/brief`

### Code Intelligence
`/review`, `/diff`, `/files`, `/cost`, `/plan`, `/memory`, `/thinkback`

### AI Features
`/commit`, `/pr`, `/debug`, `/explain`, `/fix`, `/test`

### Meta
`/help`, `/skills`, `/mcp`, `/init`, `/permissions`

Each command is declarative:
```typescript
{
  type: 'prompt',
  name: 'commit',
  allowedTools: ['Bash(git *)'],
  getPromptForCommand: (args) => `...`
}
```

---

## 10. UI & Rendering

### Ink (`src/ink/`)
- Custom fork of [Ink](https://github.com/vadimdemedes/ink) (React for CLIs)
- 43 files — heavily modified for Claude Code's needs
- Handles TTY size changes, mouse events, focus management
- Patches to support streaming partial renders

### Components (`src/components/`)
- 150+ React components for terminal UI
- Key components: `MessageView`, `ToolCallView`, `ProgressBar`, `InputBox`, `ThinkingBlock`
- `outputStyles/` — ANSI color themes and formatting rules

### Vim Mode (`src/vim/`)
- Full vi key binding emulation for the input box
- Normal/insert/visual modes
- Multi-key sequences (e.g., `dd`, `yy`, `p`)

### Voice Mode (`src/voice/`)
- Speech-to-text input using system APIs
- Transcription piped into the normal input flow

---

## 11. Multi-Agent & Coordination

### AgentTool (`src/tools/AgentTool.ts`)
- Spawns sub-agents with isolated message history
- Sub-agent gets a subset of parent tools
- Results returned as a single message to parent
- Supports background agents (non-blocking)

### Coordinator Mode (`src/coordinator/`) — feature-gated
- Multi-agent swarm orchestration
- Assigns tasks to specialized sub-agents
- Aggregates results with deduplication
- Only available in internal Anthropic builds

### KAIROS — feature-gated
- "Always-on" proactive assistant mode
- Watches application logs and system events
- Triggers autonomous actions without user input
- Internal Anthropic builds only

### ULTRAPLAN — feature-gated
- Offloads complex planning to remote Opus 4.6 session
- 30-minute budget for deep analysis
- Returns structured plan injected into main session

---

## 12. IDE Bridge

Located in `src/bridge/`.

### Integration Targets
- **VS Code** — via extension IPC
- **JetBrains** — via plugin WebSocket
- **Neovim** — via RPC channel

### Key Files
- `bridgeMain.ts` — Bridge lifecycle and message routing
- `replBridge.ts` — REPL execution model for IDE-initiated queries
- `remoteBridgeCore.ts` — WebSocket server for remote sessions (claude.ai/code web)

### Protocol
- JSON messages over stdio/WebSocket
- Bidirectional: IDE → Claude (send input, context), Claude → IDE (file edits, diagnostics)
- Auth via Anthropic session token

---

## 13. Notable Subsystems

### Buddy System (`src/buddy/`)
A Tamagotchi-style companion, purely cosmetic.
- 18 species from Common (*Pebblecrab*) to Legendary (*Nebulynx*)
- Deterministic Gacha: Mulberry32 PRNG seeded from `userId` — same user always gets same buddy
- Each buddy has stats: `DEBUGGING`, `CHAOS`, `SNARK`
- "Soul" description generated by Claude at first encounter
- Displayed in terminal as ASCII art

### Undercover Mode (`src/utils/undercover.ts`)
For Anthropic employees contributing to public repos. Activated via `CLAUDE_CODE_UNDERCOVER=1`.
- Strips internal model codenames from outputs (Tengu, Capybara, etc.)
- Blocks internal repo URLs and project names
- Removes signals that the assistant is an AI
- Confirms `Tengu` is Claude Code's internal codename

### Feature Gates
Build-time dead code elimination via `feature()` calls:
```typescript
const mod = feature('COORDINATOR_MODE') ? require('./coordinator') : null
```
- External npm build: only default features compiled in
- Internal Anthropic build: all features including KAIROS, ULTRAPLAN, Coordinator

### Migrations (`src/migrations/`)
- Data migration system for `~/.claude/` config changes between versions
- Run automatically on startup when version mismatch detected

### upstreamproxy (`src/upstreamproxy/`)
- HTTP/HTTPS proxy support for enterprise environments
- Intercepts Anthropic API calls and routes through corporate proxy
- Respects `HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY` env vars

---

## 14. Startup & Performance

### Target: ~135ms to first render

Parallel prefetch at process start (before heavy imports):
1. `startMdmRawRead()` — enterprise MDM config
2. `startKeychainPrefetch()` — OAuth tokens
3. GrowthBook feature flags — A/B test assignments

`profileCheckpoint()` sprinkled throughout to measure critical path timing.

### Fast-Path Exits (zero module loading)
- `--version` — read from package.json, print, exit
- `--dump-system-prompt` — render prompt, exit
- `--daemon-worker=<kind>` — supervisor process, skip UI

### Lazy Loading Pattern
```typescript
// Break circular deps + defer cost
const getTeammateUtils = () => require('./utils/teammate.js')
```
Dynamic requires used heavily to avoid circular dependencies and defer module initialization.

### System Prompt Caching
- System prompt rendered once per session
- Cached with hash; only regenerated on config change
- Reduces per-query overhead

---

## 15. Extension Points

### 1. MCP Servers
Configure in `~/.claude/mcp.json` or project `.mcp.json`. Any MCP-compatible server's tools become available as `mcp__<server>__<tool>`.

### 2. Plugins
Drop plugin packages in `~/.claude/plugins/`. Plugins export additional tools merged into the tool registry at startup.

### 3. Skills (Slash Commands)
Custom slash commands via `~/.claude/commands/*.md`. Frontmatter defines tool allowlist and prompt template.

### 4. Hooks
Query lifecycle hooks for customization (pre/post tool execution, message filtering). Configured in `~/.claude/settings.json`.

### 5. CLAUDE.md
Project-level instructions loaded into system prompt at startup. Overrides default behavior for a specific repo.

---

## 16. Directory Map

```
src/
├── main.tsx                  CLI bootstrap + React/Ink entry
├── QueryEngine.ts            LLM loop + tool orchestration
├── query.ts                  Message processing + compaction
├── Tool.ts                   Tool base class + permission types
├── Task.ts                   Task state machine
├── tools.ts                  Tool registry
├── commands.ts               Slash command registry
│
├── tools/                    43 built-in tool implementations
├── commands/                 101 slash command definitions
├── components/               150+ React terminal UI components
├── screens/                  Full-screen UI views (welcome, setup)
├── hooks/                    React hooks for state/events
│
├── services/
│   ├── api/                  Anthropic API client + auth
│   ├── mcp/                  MCP server lifecycle + transport
│   ├── memory/               Session memory extraction + team sync
│   ├── compaction/           Auto-compaction logic
│   ├── autoDream/            Daily memory consolidation
│   ├── analytics/            GrowthBook + telemetry
│   ├── plugins/              Plugin discovery + merging
│   ├── lsp/                  Language server client
│   └── oauth/                Multi-provider OAuth
│
├── state/                    AppStateStore (DeepImmutable)
├── context/                  React context providers
├── coordinator/              Multi-agent swarm (feature-gated)
├── bridge/                   IDE integration (VS Code, JetBrains, Neovim)
├── buddy/                    Tamagotchi companion system
├── memdir/                   File-based memory CRUD
├── migrations/               ~/.claude config migrations
├── ink/                      Custom React/Ink terminal renderer (43 files)
├── outputStyles/             ANSI color themes
├── vim/                      Vi key binding emulation
├── voice/                    Speech-to-text input
├── upstreamproxy/            HTTP proxy for enterprise
├── utils/                    60+ utility modules
├── schemas/                  Zod schemas for config/API types
├── types/                    Shared TypeScript type definitions
├── keybindings/              Keyboard shortcut system
├── entrypoints/              Process entrypoints (cli.tsx, daemon)
├── bootstrap/                Startup sequence helpers
├── remote/                   Remote session (claude.ai/code web)
├── server/                   Local HTTP server (API mode)
├── native-ts/                Native binary bindings (Bun-specific)
├── skills/                   Built-in skill implementations
├── tasks/                    Task management utilities
├── assistant/                Assistant persona configuration
├── query/                    Query configuration types
├── cli/                      CLI argument type definitions
├── cost-tracker.ts           Per-session cost accumulator
├── costHook.ts               Cost reporting hook
├── history.ts                Conversation history persistence
├── setup.ts                  First-run setup wizard
├── dialogLaunchers.tsx       Modal dialog system
├── interactiveHelpers.tsx    Shared interactive UI helpers
├── projectOnboardingState.ts Per-project onboarding tracking
└── replLauncher.tsx          REPL session launcher
```
