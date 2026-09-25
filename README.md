# git-grep-first ⚡
### Fixing the Windows AI Agent Shell Trap: Preventing Context Poisoning with Zero External Dependencies

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/mrblackman/git-grep-first-for-windows-ai-agent/pulls)

**Author:** Mustafa KILINC ([@mrblackman](https://github.com/mrblackman))  
**Repository:** [github.com/mrblackman/git-grep-first-for-windows-ai-agent](https://github.com/mrblackman/git-grep-first-for-windows-ai-agent)  

---

## 🚀 The Instant Fix

No complex configurations or external packages needed. **Copy and paste the prompt below into your AI coding assistant (Cursor, Claude Code, Antigravity, Copilot, Cline, or Windsurf):**

```text
From now on, whenever you search for code, functions, symbols, or text inside this repository on Windows:

1. NEVER use PowerShell's 'Select-String' or 'Get-ChildItem -Recurse'.
2. ALWAYS prefer your built-in search tool if available. If a shell search is needed:
   - For exact symbols and strings:
     git -c core.quotepath=false grep -n -I -F -e '<query>' -- ':!*.min.js' ':!*.map' .
   - For newly created, unstaged, or untracked files:
     git -c core.quotepath=false grep --untracked -n -I -F -e '<query>' .

Rationale: 'Select-String' is blind to .gitignore, crawls node_modules/bin/obj, and can dump
tens of thousands of minified tokens into your context window in a single lookup.
'git grep' searches only relevant tracked files and respects .gitignore, preventing context bloat.

Please write this rule into this project's agent instructions file
(e.g., AGENTS.md, CLAUDE.md, .cursor/rules/search.mdc, or .github/copilot-instructions.md).
```

*Your agent will lock in this search policy and stop traversing build artifacts.*

---

## 💥 The Problem: The Windows `Select-String` Trap

When operating on Windows (`Shell: powershell`), modern LLMs (Claude, GPT-4o, Gemini) fall into a pre-trained assumption:
> *"Windows does not have native grep. Use PowerShell's `Select-String` (or `Get-ChildItem -Recurse | Select-String`) to avoid 'command not found' errors."*

In modern software repositories, this creates several regressions:

```text
Agent runs: Get-ChildItem -Recurse | Select-String 'handleAuth'
   │
   ├── ❌ .gitignore Blindness: Traverses node_modules/, bin/, obj/, .next/, dist/
   ├── ❌ Context Poisoning: Can dump tens of thousands of minified tokens into the prompt
   ├── ❌ Attention Dilution: Agent quality degrades as the context fills with irrelevant code
   └── ❌ CLI Freeze: .NET CLR MatchInfo allocation can stall the terminal on large repos
```

In multi-agent architectures (where an Orchestrator coordinates multiple subagents executing many searches), this is not just a minor delay — it is **systemic context poisoning** that degrades the agent's reasoning quality.

> **A note on scope:** `git grep` searches *tracked* files — essentially the union of your committed and staged files. Files matching `.gitignore` are excluded because they were never tracked in the first place. Forcibly-added files (`git add -f`) *will* appear. If you need to search outside a Git repository, use `git grep --no-index --exclude-standard` or fall back to `ripgrep`.

---

## 🔍 Code Search Hierarchy

`git-grep-first` does not claim to replace AST parsers or vector embeddings. It solves a specific **Tool-Selection Failure** in shell execution. AI coding agents should follow this priority order:

| Priority | Search Type | Recommended Approach |
| :---: | :--- | :--- |
| **1 — Built-in first** | Any search | Use the agent's native/built-in search tool when available (Cursor search, VS Code search, etc.) — these are already scoped and optimized. |
| **2 — Exact / Symbol** | Function names, classes, constants, error codes | `git -c core.quotepath=false grep -n -I -F -e '<query>' .` |
| **3 — Literal + exclusions** | Searches in repos with large committed assets | Add exclusion pathspecs: `-- ':!*.min.js' ':!*.map' ':!*.lock'` |
| **4 — Pattern / Regex** | Multi-keyword patterns, regex structures | `git grep -E -n '<regex>'` |
| **5 — Untracked files** | New files created by agent before `git add` | `git grep --untracked -n -I -F -e '<query>' .` |
| **6 — Semantic / Conceptual** | *"Where does the auth token get generated?"* | Semantic index / LSP / MCP (`grepai`, vector search) — classical grep cannot deduce conceptual intent. |

---

## ⚠️ Known Limitations & Edge Cases

Understanding these edge cases prevents misuse:

**PowerShell quoting (critical):**  
Always use **single quotes** in PowerShell for the query. Double quotes expand variables:
```powershell
# ✅ Correct — single quotes, no variable expansion
git grep -n -F -e 'handleAuth'

# ❌ Wrong — $scope, $this, $_ will be expanded or empty
git grep -n "$scope"
```
On `cmd.exe`, use double quotes instead: `git grep -n -F -e "handleAuth"`.

**Literal vs. regex:**  
`git grep` defaults to **BRE regex** mode. For exact string matching (e.g., `arr[0]`), always include `-F`:
```powershell
git grep -n -F -e 'arr[0]'   # literal: finds exactly arr[0]
git grep -n    -e 'arr[0]'   # regex:   finds arr0 (not what you want)
```

**git is required:**  
`git grep` only works inside a Git repository. If you are outside a repo:
```powershell
# Option 1: search non-git directory
git grep --no-index --exclude-standard -n -F -e '<query>' .

# Option 2: fall back to ripgrep if available
rg -n '<query>'
```

**Large committed assets:**  
`git grep` searches *tracked* files — including any large files that were committed (e.g., `wwwroot/lib/**/*.min.js`). If your repo tracks minified bundles, exclude them explicitly:
```powershell
git grep -n -F -e '<query>' -- ':!*.min.js' ':!*.map' ':!*.lock' .
```

**`-O` flag opens external programs — never auto-allow:**  
`git grep -O` passes results to an external command. Do not include this in any auto-approved command allowlist.

**UTF-16 files:**  
`git grep` does not match inside UTF-16LE encoded files (common when Windows tools like PowerShell 5.1 write output with `>`). For these, `ripgrep` or `Select-String` is more reliable.

**Submodules:**  
Add `--recurse-submodules` if your repo uses Git submodules.

---

## 🛡️ Why Not Just Use `ripgrep` (`rg`)?

`ripgrep` is excellent, but:

1. **Zero External Dependencies:** `git.exe` is pre-installed on any machine using a Git-based AI coding assistant. `rg` requires separate installation.
2. **Zero Setup Friction:** Autonomous agents operating in restricted sandboxes cannot install binaries without prompting the user.

> **However:** Many hosted AI coding tools (Cursor, VS Code, Claude Code) already bundle `rg` internally. In those environments, the built-in search tool is the right first choice. `git grep` is the reliable fallback when no built-in tool is available and you need shell-level search.

---

## 🛠️ Agent Instruction Snippets (Copy-Paste Ready)

Add the appropriate snippet to your project's instruction file. All snippets use single-quoted queries for PowerShell safety.

### `AGENTS.md` / `CLAUDE.md`

```markdown
## Search Policy (Windows / PowerShell)

1. Prefer the built-in search tool when available.
2. For shell-level searches inside this Git repository:
   - Exact match:   git -c core.quotepath=false grep -n -I -F -e '<query>' .
   - With exclusions: git grep -n -I -F -e '<query>' -- ':!*.min.js' ':!*.map' .
   - Regex pattern: git grep -n -E -e '<pattern>' .
   - Untracked files: git grep --untracked -n -I -F -e '<query>' .
3. NEVER use: Get-ChildItem -Recurse | Select-String (blind to .gitignore)
4. Outside a Git repo: git grep --no-index --exclude-standard -n -F -e '<query>' .
5. For UTF-16 files or semantic queries: use ripgrep (rg) or a semantic index.
```

### `.cursor/rules/search-policy.mdc`

```markdown
---
description: Enforce git grep on Windows to prevent context poisoning
globs: ["**/*"]
alwaysApply: true
---
## Search Policy
- Built-in search first, then shell.
- Shell search: `git -c core.quotepath=false grep -n -I -F -e '<query>' .`
- Untracked: `git grep --untracked -n -I -F -e '<query>' .`
- Never: recursive Select-String pipelines.
```

### `.github/copilot-instructions.md`

```markdown
## Code Search (Windows)
Use `git grep` for shell-level searches. Never use `Get-ChildItem -Recurse | Select-String`.
Example: `git -c core.quotepath=false grep -n -I -F -e 'handleAuth' .`
```

---

## 📜 License

Released under the **[MIT License](LICENSE)**.

---

## ☕ Support

If this helped your workflow, consider buying me a coffee.

[!["Buy Me A Coffee"](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://buymeacoffee.com/mrblackman)
