# git-grep-first ⚡
### Eliminating the Windows AI Agent Shell Trap: Slashes Token Bloat by 99% and Eliminates Context Poisoning

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-mrblackman-yellow.svg?style=flat&logo=buy-me-a-coffee)](https://buymeacoffee.com/mrblackman)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)]()
[![Tested on](https://img.shields.io/badge/Tested%20on-Cursor%20%7C%20Claude%20Code%20%7C%20Antigravity%20%7C%20Copilot%20%7C%20Cline-orange.svg)]()

**Author:** Mustafa KILINC ([@mrblackman](https://github.com/mrblackman))  
**Repository:** [github.com/mrblackman/git-grep-first-for-windows-ai-agent](https://github.com/mrblackman/git-grep-first-for-windows-ai-agent)  

---

## 🚀 The 10-Second Instant Fix

You don't need complex configurations or external packages. **Just copy and paste the prompt below directly into your AI coding assistant (Cursor, Claude Code, Antigravity, Copilot, Cline, or Windsurf):**

```text
From now on, whenever you search for code, functions, symbols, or text inside this repository on Windows:

1. NEVER use PowerShell's 'Select-String' or 'Get-ChildItem -Recurse'.
2. ALWAYS use:
   git grep -n "<query>"
3. For newly created, unstaged, or untracked files, use:
   git grep --untracked -n "<query>"

Rationale: 'Select-String' is blind to .gitignore, crawls node_modules/bin/obj, and dumps 40,000+ minified tokens into your context window. 'git grep' is instant, respects .gitignore, and searches tracked and untracked files with zero token bloat.

Please permanently write this rule into this project's agent instructions file (e.g., .cursorrules, CLAUDE.md, AGENTS.md, or .clinerules depending on our environment).
```

*That's it. Your agent will permanently configure its search policy, prioritize `git grep`, and immediately stop freezing your terminal.*

---

## 💥 The Problem: The Windows `Select-String` Trap

When operating on Windows (`Shell: powershell`), modern LLMs (Claude 3.5 Sonnet, GPT-4o, Gemini 2.0) fall into an instinctive pre-trained assumption:
> *"Windows does not have native grep. Use PowerShell's `Select-String` (or `Get-ChildItem -Recurse | Select-String`) to avoid 'command not found' errors."*

In modern software repositories, this creates five catastrophic regressions:

```text
Agent runs: Get-ChildItem -Recurse | Select-String "handleAuth"
   │
   ├── ❌ .gitignore Blindness: Traverses node_modules/, bin/, obj/, .next/, dist/
   ├── ❌ Context Poisoning: Dumps 40,000+ characters of minified vendor code into prompt
   ├── ❌ Token Explosion: Burns $0.50 - $1.00+ in quota on a single routine lookup
   ├── ❌ Attention Dilution: Agent hallucinates and forgets your original prompt
   └── ❌ CLI Freeze: .NET CLR MatchInfo allocation stalls terminal for 15+ seconds
```

In multi-agent architectures (where an Orchestrator coordinates 5–10 subagents executing dozens of searches), this is not just a minor delay—it is **systemic context poisoning** that destroys the agent's reasoning capability.

---

## 📊 Benchmark: `Select-String` vs. `git grep -n`

We measured real-world searches inside an enterprise ASP.NET Core + Tailwind + Node.js codebase on Windows 10:

| Metric | PowerShell `Select-String` | `git grep -n "<query>"` | Real-World Impact |
| :--- | :--- | :--- | :--- |
| **Lookup Time** | **14.8 seconds** | **0.038 seconds (38 ms)** | **~400x Faster** (Instant) |
| **Token Ingestion** | **38,400 tokens** | **112 tokens** | **99.7% Token Reduction** |
| **`.gitignore` Awareness**| ❌ No (crawls build junk) | ✅ Yes (source code only) | Zero Context Pollution |
| **Memory / GC Pressure** | Allocates heavy `MatchInfo` objects | Native C string streaming buffer | Zero CLI lockups or UI freezes |
| **Output Ergonomics** | Verbose table, prone to CLI trunc | Universal `file:line: code` format | Surgical diffs & edits |
| **Security Permissions** | Piped commands trigger prompts | Atomic read-only command | Safely auto-allowable |

> **Note on Methodology:** Benchmarked on Windows 10 Pro (NVMe SSD) inside an enterprise repository containing ~1,800 source files alongside ~42,000 vendor/build files in `node_modules/` and `bin/obj`. While exact multiples vary by repo size and storage hardware, the architectural protection against context poisoning is universal.

---

## 🔍 Code Search Taxonomy: Where `git grep` Fits

`git-grep-first` does not claim to replace AST parsers or vector embeddings. It solves the critical **Tool-Selection Failure** in shell execution. AI coding agents should classify searches across three distinct tiers:

| Tier | Search Type | Primary Command / Tool | Why? |
| :--- | :--- | :--- | :--- |
| **Tier 1: Exact / Symbol** | Function names, classes, constants, error codes | `git grep -n "<query>"` | Zero context poisoning, sub-50ms latency, 100% native. |
| **Tier 2: Pattern / Regex** | Multi-keyword patterns, regex structures | `git grep -E -n "<regex>"` | Fast C-level regex execution without loading files into RAM. |
| **Tier 2b: Untracked Files**| New files created by agent before `git add` | `git grep --untracked -n "<query>"` | **Zero external dependencies:** Scans newly created files while still honoring `.gitignore`! |
| **Tier 3: Semantic / Conceptual** | *"Where does user auth token get generated?"* | Semantic Index / LSP / MCP (`grepai`, `zvec-grep`) | Classical grep cannot deduce conceptual intent. Semantic embeddings or LSP call graphs take over. |

---

## 🛡️ Why Not Just Use `ripgrep` (`rg`)?

`ripgrep` is an outstanding tool, but in autonomous AI coding setups:
1. **Zero External Dependencies:** `git.exe` is **100% guaranteed** to be pre-installed and available in `PATH` for anyone using an AI coding assistant. `rg` requires manual installation (`winget install BurntSushi.ripgrep.MSVC` or Scoop/Choco).
2. **Zero Setup Friction:** Autonomous agents operating in restricted sandboxes cannot install binaries or alter system paths without prompting the user.
3. **Native `--untracked` Flag:** A common misconception is that `git grep` only searches committed files. In reality, `git grep --untracked -n` searches both tracked and unstaged working-tree files while continuing to exclude `.gitignore` entries.

---

## 🛠️ Enterprise Agent Search Policy (Copy-Paste)

For production multi-agent workflows, add this formal search policy to your repository's root instructions (`AGENTS.md`, `.cursorrules`, or `CLAUDE.md`):

```markdown
# Agent Search Policy (Windows)

When searching this repository:
1. NEVER use recursive PowerShell pipelines (`Get-ChildItem -Recurse`, `Select-String`).
2. For exact symbols, identifiers, and strings: ALWAYS use `git grep -n "<query>"`.
3. For newly created or unstaged files: ALWAYS use `git grep --untracked -n "<query>"`.
4. For regex patterns: Use `git grep -E -n "<pattern>"`.
5. Strictly avoid traversing: `node_modules/`, `bin/`, `obj/`, `dist/`, `.next/`.
6. Prefer narrow, targeted searches before broad queries to protect context window hygiene.
```

---

## 🧠 Why Does `git grep` Always Work?

Because **100% of developers using AI coding assistants are working inside a Git repository**, and `git.exe` is already installed, configured, and accessible on the Windows system `PATH`. 

Steering the model toward `git grep` transforms Windows from a sluggish, token-burning second-class citizen into a lightning-fast, production-grade AI coding environment.

---

## ☕ Support

If this 1-prompt fix saved your workflow from terminal freezes, slashed your token bills, or made your Windows AI agent 400x faster, consider buying me a coffee!

[!["Buy Me A Coffee"](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://buymeacoffee.com/mrblackman)

---

## 📜 License & Citation

Released under the **[MIT License](LICENSE)**.

```bibtex
@misc{kilinc2026gitgrepfirst,
  author = {Mustafa KILINC (@mrblackman)},
  title = {git-grep-first: Eliminating the Windows AI Agent Shell Trap and Context Poisoning},
  year = {2026},
  publisher = {GitHub},
  howpublished = {\url{https://github.com/mrblackman/git-grep-first-for-windows-ai-agent}}
}
```
