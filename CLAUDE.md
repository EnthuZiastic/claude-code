# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

This is an **archival mirror** of the Claude Code source code that was accidentally leaked via npm sourcemaps on March 31, 2026. It is a read-only research/exploration codebase — there is no `package.json`, no build system, and no test runner present. The source files are extracted from Anthropic's published npm package and are not runnable as-is.

## Exploring the Source

Since there's no build toolchain, navigate and read files directly. Key entry points:

- `src/main.tsx` — CLI bootstrap: Commander.js arg parsing, React/Ink terminal renderer, session management (~785KB)
- `src/QueryEngine.ts` — Core LLM interaction loop: tool execution, message history, token budgeting, auto-compaction
- `src/Tool.ts` — Abstract base class for all tools
- `src/tools/` — 40+ built-in tools (Bash, file ops, web, LSP, MCP, agent spawning)
- `src/entrypoints/cli.tsx` — Process-level entrypoint, defers to main.tsx
- `src/commands/` — Slash command implementations (declarative prompt-based)
- `src/services/` — MCP client, OAuth, analytics, autoDream memory consolidation
- `src/bridge/` — IDE integration (VS Code, JetBrains, Neovim) via WebSocket/IPC
- `src/coordinator/` — Multi-agent swarm orchestration
- `src/buddy/` — Tamagotchi companion system (cosmetic, not functional to core)

## Architecture

Claude Code is a terminal AI agent framework built on:
- **React + Ink** (custom fork in `src/ink/`) for terminal UI rendering
- **Commander.js** for CLI argument parsing
- **Anthropic SDK** (`@anthropic-ai/sdk`) for LLM calls
- **Model Context Protocol SDK** for MCP server integration
- **Bun** as the primary runtime (Node.js v18+ fallback)
- **Zod** for schema validation across tool inputs

### Message Flow

1. User input → `processUserInput()` (slash command parsing, memory attachment)
2. `QueryEngine.query()` — main LLM loop
3. Anthropic API call (supports extended thinking, streaming)
4. Tool execution via `StreamingToolExecutor`
5. Results appended to message history → loop until stop or max turns

### Tool System

Tools extend `Tool.ts` and expose:
- `ToolInputJSONSchema` — validated input definition
- `execute(context)` — returns `ToolResultBlockParam`
- `ToolProgressData` — for streaming progress updates

Tool availability is gated by `canUseTool()` at runtime. Slash commands declare an `allowedTools[]` whitelist (e.g., `/commit` only allows `Bash(git *)`).

### Memory Architecture

- **autoDream** (`src/services/autoDream/`) runs daily as a background subagent to consolidate session transcripts into `MEMORY.md`
- **memdir** (`src/memdir/`) handles the file-based memory system at `~/.claude/MEMORY.md` or `.claude/MEMORY.md` per project
- Memory is organized by topic (user, feedback, project, reference types)

### Feature Gates

Internal (Anthropic) builds use build-time `--define` flags to include gated features:
- `COORDINATOR_MODE` — multi-agent swarm
- `KAIROS` — always-on proactive assistant watching logs
- `ULTRAPLAN` — offloads planning to remote Opus 4.6 (30-min budget)

These appear as dead code in external builds and are eliminated at bundle time.

### Undercover Mode

`src/utils/undercover.ts` — activated via `CLAUDE_CODE_UNDERCOVER=1` for Anthropic employees contributing to public repos. Strips internal model codenames (Tengu, Capybara, etc.) and hides AI identity signals.

### Buddy System

`src/buddy/` — a Tamagotchi companion with 18 species determined by a Mulberry32 PRNG seeded from `userId`. Each buddy has `DEBUGGING`, `CHAOS`, and `SNARK` stats. Purely cosmetic.
