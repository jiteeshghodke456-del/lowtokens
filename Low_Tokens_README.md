# Low Tokens 🧠⚡

> *A context optimization engine that makes small local code models reason over repositories thousands of times larger than their context window — with a visual dashboard, graph-state memory, and zero VRAM bloat.*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Status: Prototype](https://img.shields.io/badge/Status-Prototype-blue.svg)]()

---

## 🚨 The Problem: The Context Bottleneck

Local coding models (like Qwen2.5-Coder, DeepSeek-Coder, etc.) have improved dramatically, but they suffer from one critical limitation: **They cannot understand large repositories.**

Current Retrieval-Augmented Generation (RAG) approaches treat codebases like flat text documents, fetching the "Top 20" most semantically similar files. But software is an **executable graph**. Dumping whole files wastes tokens, hides the real root cause, and breaks the context window.

Standard RAG sends 10,000–50,000 tokens to the LLM per query. Low Tokens sends **500–1,000 tokens** — with equal or better answer quality.

---

## 💡 Our Solution: Verification-Guided Context Optimization (VGCO)

**Low Tokens** is not an offline coding assistant. It is a **context optimization engine**.

Instead of increasing model size or dumping whole files, Low Tokens searches for the **minimum sufficient context** needed for reasoning. It extracts only the exact execution pathways (using ASTs, Call Graphs, and Dependency Graphs) to create a tightly packed payload.

```
Minimize(Context Size)
Subject To: Verification Success
```

### How it works:
1. **Repository Analysis:** Scans the repo to build a structural index (AST, call graphs).
2. **Context Optimization:** Traces the exact execution path of a user's query.
3. **Local Reasoning:** Feeds a hyper-compact context (e.g., 3 functions, 1 config, 1 failing test) to the LLM. The KV cache holds only the selected function bodies + a 2-line session summary — never the full chat history.
4. **Adaptive Expansion:** If the LLM's generated patch fails verification (linting, tests), the engine traces the new error, mathematically expands the context graph to include missing nodes, and retries.

---

## 🏗️ High-Level Architecture

```text
                 Developer
                     │
           Natural Language Query
                     │
                     ▼
         Repository Analysis Engine
         (Tree-sitter, Git, Build)
                     │
                     ▼
       Unified Program Representation
        (AST + Call Graph + Dep Graph)
                     │
                     ▼
     Verification-Guided Context Optimizer
                     │
                     ▼
   ┌─────────────────────────────────────┐
   │   Graph-State Memory (System RAM)   │
   │   Visited nodes stored as Python    │
   │   objects — zero VRAM growth        │
   └─────────────┬───────────────────────┘
                 │
                 ▼
          Initial Compact Context
          (~500–1000 tokens max)
                 │
                 ▼
           Code LLM Engine
           (KV cache: nodes
            + 2-line summary
            + query only)
                 │
                 ▼
        Verification & Validation ──────┐
                     │                  │
                 (Success)          (Failure)
                     │                  │
                     ▼                  ▼
              Return Patch       Adaptive Expansion
                     │           (add missing nodes,
                     │            retry loop)
                     ▼
           Web Dashboard Output
           (Visual Graph + Diff + Stats)
```

---

## 🚀 Features

### Core Engine
- **Privacy-First:** Source code never leaves your machine unless you explicitly route it to a cloud provider.
- **Offline-First:** Designed to function completely without internet access.
- **Incremental Indexing:** Updates only affected structural metadata when files change.
- **Model Agnostic:** Works seamlessly with small local models or powerful cloud APIs.

### Intelligence & Memory
- **🗣️ Conversational Graph Memory:** Multi-turn sessions with zero VRAM growth. The engine stores visited graph nodes as Python objects in system RAM — not raw conversation text in the LLM context. Each new turn gets a fresh, compact KV cache with only the relevant nodes + a 2-line summary of previous turns. VRAM stays flat across the entire session.
- **💾 Smart Caching:** Repeated or similar queries are served instantly from a graph cache, keyed by query hash + repo state. No re-indexing, no re-inference.

### Modes
- **📝 Explain Mode:** Traces the call graph for any function or feature and returns a plain-English walkthrough of the execution path — with exact file and line references. Perfect for onboarding to a new codebase.
- **🔀 PR Review Mode:** Point at a GitHub PR diff → the engine traces only the changed functions + their callers and callees → delivers a focused code review with the full execution context a reviewer would normally miss.
- **🔐 Secrets & Security Scanner:** While tracing the graph, flags hardcoded credentials, exposed API keys, and insecure patterns encountered along any execution path. Free security audit on every query.

### Developer Experience
- **🌐 Visual Graph Explorer:** A local web dashboard (at `localhost:8765`) that shows exactly which nodes were traced, why they were included, and how they connect. Users can manually add or remove nodes before inference. Builds trust and transparency — unlike black-box RAG tools.
- **🖱️ One-Click Refactor:** Right-click any function in the IDE → "Low Tokens: Refactor" → the engine traces the full dependency graph of that function and returns a clean, dependency-aware refactored version. No query writing needed.
- **📊 Session Dashboard:** At the end of each session, shows a real-time report of token savings, context reduction ratio, patches applied, and test results — proving the value of graph-guided context with actual numbers.
- **⚙️ `.lowtoken.yaml` Config:** A per-project settings file that stores language, test command, model preference, and exclusion rules. Commit it once to the repo and every teammate gets the same Low Tokens setup automatically. No flags needed on every command.
- **🧙 `low-tokens init` Wizard:** A one-time setup command that auto-detects your project's language, framework, and test runner; builds the structural index; and creates the `.lowtoken.yaml` config — all in under 10 seconds.

---

## 🖥️ Interfaces

Low Tokens is built in three layers, in order of priority:

| Layer | Interface | Purpose |
|---|---|---|
| 1 | **CLI** | Primary interface — fast, scriptable, testable |
| 2 | **Web Dashboard** | Visual graph explorer, session stats, diff viewer |
| 3 | **IDE Plugin** | VS Code extension (Phase 2, post-hackathon) |

The CLI and Web Dashboard are built for the current prototype. The IDE plugin shares the same backend and will be added as a `.vsix` VS Code extension post-hackathon.

---

## ⚡ Usage

```bash
# First-time setup (auto-detects language, framework, builds index)
low-tokens init

# Ask a question about a bug
low-tokens ask "why is the login failing for inactive users?"

# Get a plain-English explanation of a feature
low-tokens explain "how does the payment flow work?"

# Refactor a specific function
low-tokens refactor auth.py:validate_token

# Review a PR for issues
low-tokens review --pr 42

# Scan for secrets and insecure patterns
low-tokens scan --secrets

# Open the visual web dashboard
low-tokens dashboard
```

---

## 💻 Environment & Supported Models

### Hardware Optimization
Low Tokens is aggressively optimized for mid-to-high-end developer laptops. It has been tested and tuned for environments running hardware such as the **Intel Core i7-13650HX, NVIDIA RTX 5050, and 32GB RAM**, which provides the perfect balance of CPU threading for graph analysis and GPU VRAM for holding a quantized local model entirely in memory.

**VRAM stays constant regardless of conversation length.** Graph nodes (the "memory" between turns) are stored as Python objects in system RAM, not in the LLM's KV cache. Each inference turn receives a fresh, compact prompt of ~500–1,000 tokens — never the full conversation history.

### Supported LLMs
* **Local (On-Device):** Qwen2.5-Coder, DeepSeek-Coder, Gemma, Phi.
* **Cloud API:** Google Gemini Pro, GPT-4 (Drastically reduces token costs and "lost-in-the-middle" hallucinations by sending only the exact execution path).

---

## 📊 Evaluation Metrics

We evaluate Low Tokens as a systems engineering project:
- **Context Reduction Ratio:** Amount of repo context eliminated before inference.
- **Token Reduction:** Decrease in prompt size vs. file-based retrieval (target: >90% reduction).
- **Bug Localization Accuracy:** Precision in finding relevant AST nodes.
- **Patch Success Rate:** Percentage of generated patches that pass the verification loop.
- **Session Token Savings:** Cumulative tokens saved over a full multi-turn session vs. standard RAG.
- **Cache Hit Rate:** Percentage of queries served instantly from the graph cache.

---

### Authors
**Advait Anil Malwade**, **Jiteesh Ghodke**, **Vanshika Dekate**, **Nishigandha Waykar**

Built for high-speed, local-first repository reasoning.
