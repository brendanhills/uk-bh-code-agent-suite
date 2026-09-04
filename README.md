# Code Agent Suite for Jetski & Antigravity

A unified developer extension, diagnostics, and harness suite for **Jetski** and **Antigravity** that deploys shared agent skills, Conductor, plugins, lifecycle hooks, prompt heuristics, diagnostics, and global execution rules.

---

## 🚀 Quick Start (Setup on a New Machine)

To install all plugins, skills, hooks, prompt heuristics, and global rules on any new Cloudtop, Chromebook, or developer machine:

```bash
# Global installation (recommended for all Jetski & Antigravity sessions)
./install.sh

# Or install locally scoped to a specific workspace
./install.sh --local /path/to/workspace
```

### What `./install.sh` Configures
- **Plugins**: Symlinks `.agents/plugins/` (and Conductor) to `~/.gemini/config/plugins/`.
- **Conductor**: Full, self-contained Conductor engine and track workflow skills.
- **Standalone Skills**: Deploys progressive disclosure skills to `~/.gemini/config/skills/`.
- **Global Rules**: Installs all `.md` rules from `.agents/rules/` into `~/.gemini/config/agents/rules/` and creates discovery symlinks in `~/.gemini/config/rules/`.
- **Direct File Editing Mandate**: Automatically verifies and appends the native tool editing mandate to `~/.gemini/config/AGENTS.md`.
- **Hooks & Scripts**: Deploys pre/post invocation hooks and diagnostic scripts into `~/.gemini/config/agents/`.
- **Prompt Heuristics**: Synchronizes core prompt guidelines and metrics.
- **Maintenance & Diagnostics**: `maintenance/fix_antigravity.sh` and `update_projects.py` to diagnose hanging language servers and auto-sync GCP project configurations.

---

## 🛡️ Direct File Editing & Anti-Scripting Mandate

To ensure deterministic edits, prevent approval popup spam, and eliminate buggy scratch scripts, all agents operating under this harness adhere to the **Direct File Editing Policy**:

### 1. Hard Prohibitions
- ❌ **No Shell File Manipulation**: `sed`, `awk`, `perl -pi`, `cat <<EOF >`, `echo "..." >`, and shell redirections are strictly banned for file reading or editing.
- ❌ **No Temporary File Scripts**: Agents must not create temporary Python or Bash scripts in `scratch/`, `/tmp`, or workspace directories simply to parse, search, or edit files.
- ❌ **No Inline Script One-Liners**: `python3 -c "..."` and `bash -c "..."` are prohibited for inspecting or modifying data.

### 2. Mandatory Native Tool Mapping Matrix

| Operational Goal | Required Tool | Prohibited Fallback |
| :--- | :--- | :--- |
| **Inspect / Read File Content** | `view_file` | `cat`, `head`, `tail`, Python file read scripts |
| **Search File Content / Code** | `grep_search` / `code_search` | `grep`, `rg`, `find | xargs` scripts |
| **List Directory Contents** | `list_dir` / `find_by_name` | `ls`, `find`, `tree` |
| **Localized Code / Text Edits** | `replace_file_content` | `sed -i`, `awk`, Python scripts |
| **New File Creation / Full Rewrites** | `write_to_file` (`Overwrite: true`) | `cat <<EOF >`, `echo >` |
| **Jupyter Notebook Edits (`.ipynb`)** | `notebook_edit` | `nbformat` / raw JSON scripts |

### 3. Edit Failure & Retry Protocol
If `replace_file_content` fails due to whitespace or matching discrepancies:
1. Re-read the target line range using `view_file` to obtain the verbatim ground truth.
2. Re-execute `replace_file_content` using the exact line chunk.
3. For large rewrites or structural changes, use `write_to_file` with `Overwrite: true`.
4. Agents must never fall back to terminal scripts or `sed` on edit errors.

---

## 📂 Repository Structure

```
custom_harness/
├── .agents/
│   ├── hooks/                   # Lifecycle hooks (e.g. check_prompt_heuristics.py)
│   ├── plugins/                 # Packaged plugins (bug_management, spec_drift)
│   ├── rules/                   # Universal agent rule definitions
│   │   ├── direct_file_editing_standards.md   # Anti-scripting & direct edit rules
│   │   ├── zero_approval_spam_standards.md    # Sandbox approval optimization
│   │   ├── agent_safety_and_batch_standards.md
│   │   ├── cloudtop_git_velocity.md
│   │   ├── pragmatic_testing_standards.md
│   │   └── standalone_customization_architecture.md
│   ├── scripts/                 # Maintenance, diagnostic, and metrics CLI utilities
│   ├── skills/                  # Standalone progressive disclosure skills
│   ├── prompt_heuristics.md     # Heuristics guidelines for prompt evaluation
│   └── hooks.json               # Hook definitions
├── conductor/                   # Conductor workflow extension submodule
├── install.sh                   # Universal global & local installer script
└── README.md                    # This documentation
```

---

## 🛠️ Single `dev` Branch & Subfolder Tagging Workflow

This repository follows the multi-project mono-branch standard on `dev`:
- Work is committed directly on the `dev` branch.
- Milestone checkpoints use subfolder-prefixed annotated tags: `custom_harness/checkpoint-YYYYMMDD-HHMM`.
- To checkpoint progress: run the `checkpoint` command to update tracking files, create the tag, and push upstream (`git push origin dev --tags`).

---

## 📋 TODO & Roadmap

### TODO: Shift-Left Security & Fast Quality Gates
Prevent late-stage security and testing failures (e.g. CodeQL CWE-22 path traversal alerts and sluggish CI pipelines) by shifting detection directly into local developer and agent editing loops:

1. **Shared Zero-Dependency Security Primitives (`security_utils.py`)**:
   - Provide standard library `pathlib.Path`-based security utilities (`safe_join`, `sanitize_slug`, `validate_safe_path`) preventing CWE-22 (Path Traversal) and null-byte injection across Python projects.
   - Maintain zero external runtime dependencies (`uv` / `pip` not required).
2. **Turnkey Pre-Commit Security Hooks (`.pre-commit-config.yaml`)**:
   - Package a pre-commit template incorporating:
     - `bandit`: Fast Python AST security scanning (<1s) for common CWE vulnerabilities.
     - `semgrep`: Local OWASP Top 10 and CodeQL parity scanning (~2s) to flag path traversal, command injection, and secret leakage before push.
3. **Test Suite Optimization (Mocking Exponential Backoff Sleep)**:
   - Provide standard mock fixtures for API/network retry loops (e.g., Gemini AI exponential backoff) to eliminate real `time.sleep` delays during test runs, reducing test execution times from 90+ seconds to <5 seconds.
4. **Agent Security & Quality Rule (`.agents/rules/shift_left_security_standards.md`)**:
   - Codify mandatory rules for agents: enforce canonical containment checks for dynamic path access, mandate fast mock-driven retry tests, and verify local linter passes prior to staging commits.

