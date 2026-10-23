# Omni-Workbench & Omni-Toolkit — Long-Horizon Multi-Agent Architecture

**Omni-Toolkit** and **Omni-Workbench** form a unified multi-agent orchestration framework, cross-project semantic memory engine, remote GPU sandbox bridge, and native **Rust / Tauri v2** desktop control tower (`Omni Workbench.exe`, `~3.4 MB`) + IDE Model Context Protocol (MCP) sidecar.

> **Repository Status:** The source repositories (`efeyener0/omni-toolkit` and `efeyener0/omni-workbench`) are kept private (with read-only **GitFront** inspection links provided on the CV). This page summarizes the core systems architecture.

---

## 1. System Architecture Overview

```text
Omni Ecosystem
├── Omni-Toolkit (Python 3.12 CLI & Orchestration Core)
│   ├── scripts/cl.py          # 2-tier Claude Code runner with Static-Prefix Routing & --diet mode
│   ├── scripts/cx.py          # 2-tier Codex CLI runner with disjoint file-ownership locks (--owns)
│   ├── scripts/ag.py          # Antigravity SDK & remote Google Colab Linux GPU VM agent bridge
│   └── scripts/mem.py         # Persistent SQLite FTS5 (BM25) cross-project memory, todos & ledgers
└── Omni-Workbench (Native Tauri v2 Desktop App + IDE Sidecar + Cloud Manager)
    ├── server.py & ui.html    # Shared control-tower backend, 256K/1M context gauge & ConPTY terminal
    ├── src-tauri/             # Native Rust Tauri v2 desktop wrapper (~3.4 MB executable)
    └── tools/pm.py            # 3-Tier hybrid cloud backup & zero-copy NTFS junction asset manager
```

---

## 2. Core Engineering Mechanisms

### 2.1. Static-Prefix Prompt Routing & `--diet` Context Compression (`61%` Reduction)
Long-horizon agentic workflows frequently suffer from context-window bloat and prompt-prefix invalidation:
- **Invariant Head Prefix:** `cl.py`, `cx.py`, and `ag.py` keep `STATIC_GLOBAL_SYSTEM_PROMPT` and `STATIC_WORKER_PREFIX` 100% byte-invariant at the head of every request. Dynamic task state—persona (`Lisa`, `Cameron`, `Maya`, `Kiri`, `Miriam`, `Mia`, `Vera`), canary verification tokens, `--owns` write globs, git commit trailers, and opt-in `--memory` slices—is strictly **tail-appended**.
- **Schema-Stripped Diet Mode (`--diet`):** For high-frequency implementation and scouting subagents, `--diet` strips non-essential slash commands, session persistence overhead, and unused MCP tool schemas, compressing the base context footprint by **61% (from `~28K` tokens down to `~10.9K` tokens)**.

### 2.2. 2-Tier Role-Specialized Co-Agent Hierarchy & Disjoint File Ownership (`--owns`)
To scale autonomous software engineering across **20+ repositories** without race conditions or merge conflicts, work is partitioned **by disjoint file ownership rather than overlapping functional tasks**:
- **Coordinator (`Lisa` / `Cameron`):** Freezes mathematical specifications, header/ABI contracts, and schema invariants before dispatching background workers; performs single-point integration builds and test execution.
- **Disjoint-Shard Builders (`Maya` / `Kiri`):** Own strict glob-locked file shards (`--owns "src/module_a/**"`); forbidden from modifying outside their shard or running concurrent conflicting builds.
- **Principal Read-Only Subtractive Critic (`Miriam`):** Audits mathematical invariants, edge cases, and dead code (*"The Agent Explores, The Maintainer Subtracts"*).
- **Build & Render Harness Verifier (`Vera`) & Fast Scout (`Mia`):** Diagnose compiler/test/render outcomes to exact `file:line` root causes.

### 2.3. Persistent Cross-Project Memory (`mem.py` — SQLite FTS5 `BM25`)
Eliminates cross-session amnesia and epistemic drift over multi-month R&D lifecycles:
- Separates **`active`** vs. **`legacy`** repositories in `projects.json` with structured Markdown project cards, persistent cross-project todos, immutable decision ledgers (`LEDGER`), and an **SQLite FTS5 (`BM25`)** full-text search index over `facts.jsonl`.
- Enforces **strict opt-in worker injection**: project memory is never blindly dumped into subagent prompts; only targeted slices (`--mem-project <slug>` / `--mem-query <q>`) are appended at the prompt tail when required.

### 2.4. Remote Google Colab GPU Sandbox Bridge (`ag.py colab-agent`) & Zero-Copy Cloud (`pm.py`)
- **Remote GPU Agent Execution:** Dispatches containerized agent and training workloads onto **Google Colab Linux GPU VMs** via ADC authentication with in-memory DPAPI credential injection (`omni_vault`) and guaranteed `finally:` VM teardown.
- **Zero-Copy Asset Management (`pm.py`):** Offloads multi-gigabyte binary/3D/render directories to cloud-synced storage via Windows NTFS directory junctions (`mklink /J`), reducing local SSD footprint for cold assets to **0 bytes** while preserving transparent filesystem paths.
