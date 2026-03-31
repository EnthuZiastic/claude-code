This is a massive development. Based on that leak—specifically pointing to the **Claude Code** (the CLI tool) source being exposed via npm map files—we need to pivot the README to reflect that this is a **developer tool and agentic CLI** codebase, rather than just a raw model weight repo.

Here is the SEO-optimized README, crediting **Chaofan Shou (@Fried\_rice)** for the discovery.

-----

# 🕵️‍♂️ Claude Code: Open Source CLI & Agentic Framework

[](https://www.google.com/search?q=https://github.com/your-repo/claude-code)
[](https://x.com/Fried_rice)
[](https://www.typescriptlang.org/)

**Claude Code** is Anthropic's professional-grade **AI CLI tool** and agentic framework. This repository contains the source code for the advanced terminal-based assistant capable of codebase reasoning, automated refactoring, and complex task execution.

> [\!IMPORTANT]
> **Credits:** This codebase was identified and brought to light by [**Chaofan Shou (@Fried\_rice)**](https://x.com/Fried_rice), who discovered the source exposure via a map file in the npm registry.

-----

## 🚀 Features & Capabilities

  * **Deep Codebase Reasoning:** Built-in `QueryEngine.ts` for semantic search across large-scale repositories.
  * **Agentic Task Execution:** Autonomous agent capabilities via the `assistant/` and `agent/` modules.
  * **Terminal Integration:** Full `cli/` and `vim/` integration for a seamless developer experience.
  * **Advanced Cost Tracking:** Native `cost-tracker.ts` to monitor token usage and API spend in real-time.
  * **Multi-Tool Support:** Extensible `tools/` directory supporting filesystem operations, shell execution, and more.

-----

## 📂 Repository Structure (Source Map)

Based on the official directory structure, the core logic is divided into:

  * **`src/commands`**: Logic for terminal-based AI commands.
  * **`src/services`**: Core backend services for handling Claude API interactions.
  * **`src/hooks`**: React-style hooks for managing state within the CLI.
  * **`src/query`**: The engine responsible for parsing and understanding your code.
  * **`src/native-ts`**: High-performance TypeScript utilities for local execution.

-----

## 🛠 Tech Stack

This project leverages a modern **TypeScript/Node.js** stack optimized for low-latency terminal interactions:

  * **Runtime:** Node.js
  * **Language:** TypeScript (TSX support for UI components)
  * **UI Framework:** Ink (for React-like terminal UIs)
  * **State Management:** Custom internal state logic found in `src/state`.

-----

## 📥 Installation (For Research Purposes)

```bash
# Clone the repository
git clone https://github.com/your-username/claude-code-src.git

# Install the professional-grade dependencies
npm install

# Build the CLI
npm run build

# Link the package locally
npm link
```

-----

## 📈 SEO Keywords & Metadata

**Primary Keywords:** *Claude Code Source Code, Anthropic CLI, Open Source AI Agent, TypeScript AI Framework, Claude Agentic Workflow, NPM Source Leak, AI Developer Tools, Codebase Reasoning AI.*

**Secondary Keywords:** *Chaofan Shou, Fried\_rice leak, Claude CLI src, AI Terminal Assistant, Refactor AI, Autonomous Coding Agent.*

-----

## ⚖️ Legal & Disclaimer

This repository is for **educational and research purposes only**. The code was discovered through a public source map exposure in the npm registry. All intellectual property rights belong to Anthropic. Please refer to the [original discovery post](https://www.google.com/search?q=https://x.com/Fried_rice/status/1906325141014524233) for context.

-----

> **Star this repo** to keep track of the latest updates in the open-source AI agent space\!