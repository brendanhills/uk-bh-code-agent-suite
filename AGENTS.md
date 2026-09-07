# Global Rules

- **Bug Reporting & Management Protocol**:
  * **CRITICAL HARD CONSTRAINT**: Code modifications, edits, and bug fixes are **EXCLUSIVELY PERMITTED when the user explicitly invokes the `/fix_bug` command**. Commands like `/bug`, `/fr`, `/list_bugs`, `/triage_bug`, `/bug_plan`, and `/bug_review` MUST ONLY inspect codebase files or record metadata into `.agents/bugs.json` / `.agents/bug_plan.md`—they MUST NEVER modify source code, edit implementation files, or attempt to resolve the reported issue.
  * **Test Execution Constraint**: Running automated tests (e.g., `pytest`, test harnesses) is **EXCLUSIVELY PERMITTED during `/triage_bug` (for root-cause investigation), `/fix_bug` (for reproduction and fix verification), and `/bug_review` (for regression checks)**. Running tests during `/bug`, `/fr`, `/list_bugs`, or `/bug_plan` is **STRICTLY PROHIBITED**.
  * **`/bug` Command**: When the user runs `/bug` or reports an issue, do NOT try to fix it. Record description, date, and workspace context into `<workspace-root>/.agents/bugs.json` using the next sequential integer ID from the single shared registry (`id = max(existing_ids) + 1`, e.g. 4, not BUG-4), with `"reporter"` auto-detected from `git config user.name` (fallback `$USER`), `"date_reported": "YYYY-MM-DD"`, empty `"date_triaged"`, `"date_resolved"`, `"priority"`, `"impact"`, `"risk"`, and `"phase"` fields (`""`) and status `"Reported"`. Evaluate context: if the report is explicitly a feature request, set `"type"` to `"FR"`, otherwise set `"type"` to `"Bug"`. Do NOT write code edits, run tests, or attempt fixes.
  * **`/fr` Command**: When the user runs `/fr` or submits a feature request, do NOT implement it immediately. Record into `<workspace-root>/.agents/bugs.json` using the next sequential integer ID from the single shared registry (`id = max(existing_ids) + 1`, e.g. 4, not FR-2 or BUG-4), setting `"type"` to `"FR"`, `"reporter"` auto-detected from `git config user.name` (fallback `$USER`), `"date_reported"` to current date, status to `"Reported"`, and empty date_triaged/date_resolved/priority/impact/risk/phase fields. Bugs and FRs share the exact same ID sequence.
  * **Plugin & Skill Deduplication**: Skills bundled inside `.agents/plugins/<plugin_name>/skills/` MUST NOT be duplicated in `.agents/skills/` to prevent redundant tool registration in system prompts.
  * **Definitions**:
    - **`type`**: Classification attribute (`"Bug"` vs `"FR"`).
    - **`reporter`**: Author name auto-detected from `git config user.name` or `$USER`.
    - **`impact`**: Measures *how much of the system is affected* by the item (technical scope and severity).
    - **`priority`**: Represents the *order in which we should work on them* (resolution scheduling).
    - **`risk`**: Assesses *implementation risk / potential for regressions* (`"Low"`, `"Medium"`, `"High"`).
    - **Lifecycle Dates**: `"date_reported"` (created), `"date_triaged"` (investigated/reviewed), `"date_resolved"` (fixed/closed).
  * **`/list_bugs` Command**: Read `<workspace-root>/.agents/bugs.json` from the active workspace and render a clear, structured markdown table displaying all recorded items, their type (`Bug` vs `FR`), reporter, priority, impact, risk, status, assigned phase, and reported/resolved dates. Supports type and reporter filters (e.g. `/list_bugs fr` or `/list_bugs my`).
  * **`/triage_bug` Command Protocol**:
    - **With Argument (`/triage_bug #1`)**: Look up the specified bug ID inside `<workspace-root>/.agents/bugs.json`. Investigate the cause, formulate a solution plan, and assess the bug's `priority` (`"P0"`-`"P3"`), `impact` (`"Critical"`-`"Low"`), and `risk` (`"Low"`-`"High"`). Document cause, plan, priority, impact, risk, and set `"date_triaged"` to current date, updating status to `"Investigated"`. Do NOT execute the fix.
    - **Without Argument (`/triage_bug`)**: Read `<workspace-root>/.agents/bugs.json` and identify all bugs that have not yet been triaged (where `priority` or `impact` are empty `""`). Triage each untriaged bug, updating database records and setting `"date_triaged"` to current date without executing fixes.
  * **`/bug_plan` Command Protocol**:
    - **Modes**: Standard (`/bug_plan`) balances priority vs implementation risk. Demo/Launch mode (`/bug_plan demo` or `/bug_plan launch`) prioritizes low-risk quick wins and stability polish ahead of upcoming releases.
    - **Execution**: Automatically triages any untriaged bugs first, groups open bugs into implementation phases (Phase 1 to Phase 4), updates `<workspace-root>/.agents/bugs.json` with assigned `risk` and `phase` fields, writes a detailed roadmap to `<workspace-root>/.agents/bug_plan.md`, and renders a summary table in chat.
  * **`/bug_review` Command Protocol**:
    - **Execution**: Audits active bugs (for obsolescence from architectural changes) and recently closed bugs (for regressions). Inspects git history and codebase changes, runs targeted unit tests for candidate bugs, updates `.agents/bugs.json` status (`"Obsolete"`, `"Reopened"`, `"Fix Verified"`), sets `"date_triaged"` to current date with rationale in `"review_notes"`, and outputs a summary table.
  * **`/fix_bug` Command Protocol**:
    - **With Argument (`/fix_bug #1`)**: Look up the bug ID inside `.agents/bugs.json`. Formulate a clear design plan. The **first task in implementing the fix must be to develop an automated reproduction unit test** that fails specifically due to this bug. Verify only that targeted test fails, then write code modifications required to resolve the bug. Verify that the reproduction unit test (and existing tests) pass successfully, set `"date_resolved"` to current date, and update status to `"Fix Implemented"`.
    - **Without Argument (`/fix_bug`)**: Read `.agents/bugs.json`, identify all remaining open/unverified bugs (status not `"Fix Verified"`), sort chronologically by priority (`P0` -> `P1` -> `P2` -> `P3`), and present a recommended sequence of resolution.

- **Two-Tier Architecture & Workspace Scoping Protocol**:
  * **Repository Detection**: Always determine the Git repository root of the active workspace (`git rev-parse --show-toplevel`).
  * **Standalone Repositories (Tier 2)** (`~/dev/apps/*`, `~/dev/toolkits/*`, `~/dev/demos/*`, `~/dev/team/*`):
    - Independent repositories (`uk-bh-project-dash`, `uk-bh-cch-demos`, `uk-bh-code-agent-suite`, `uk-bh-csiro-demos`, `uk-bh-healthdirect-demos`, etc.) each have their own `origin` remote, root `README.md`, `Resume.md`, and default branch (typically `main` or active `feat/*`, `demo/*`, `arch/*`).
    - **Active Branch Integrity**: Stay on the project's current active branch. **NEVER switch to `dev` or assume the branch is `dev`** unless the project specifically uses a `dev` branch.
    - **Documentation Scoping**: When asked to update the project README or documentation, update the project's own root `README.md` describing that specific project. NEVER describe a standalone project as "UK BH Experiments Monorepo" or confuse it with the monorepo.
    - **Git Pushing**: Push strictly to the project repository's own `origin` remote on the current branch (`git push origin <branch>`). NEVER commit or push standalone project changes into `uk-bh-experiments`.
  * **Monorepo Scratchpad (Tier 1)** (`~/dev/experiments/uk-bh-experiments`):
    - Reserved strictly for rapid prototyping, spikes, and scratchpad experiments on branch `dev`.
    - Project description is "Google Cloud CE Experiments & Scratchpad (`uk-bh-experiments`)".
    - Uses subfolder-scoped checkpoint tags on `dev` (`<subfolder>/checkpoint-YYYYMMDD-HHMM`).

- **checkpoint**: When requested with "checkpoint" (or when you say "checkpoint" or "Finish for the day" or "finish for the day"):
  1. Identify the current Git repository and active branch.
  2. Update the local project root `README.md` and `Resume.md` (compaction summary), and track active status.
  3. Check for any untracked project source files/directories in the active workspace (confirming `.gitignore` is clean).
  4. Stage and commit all relevant modified and untracked project files with a descriptive message.
  5. Push the current branch to its remote repository (`git push origin <branch>`).

- **Tagging Protocol**:
  1. **Tier 2 Standalone Repositories**: Create annotated Git tags for milestones (`git tag -a "<tag_name>" -m "<descriptive message>"`), e.g. `v1.0`, `demo-canberra`, `checkpoint-YYYYMMDD-HHMM`. Push with `git push origin <branch> --tags`.
  2. **Tier 1 Monorepo (`uk-bh-experiments`)**: Use subfolder-prefixed taxonomy on `dev`: `<subfolder>/<category>-<identifier>` (e.g. `<subfolder>/checkpoint-YYYYMMDD-HHMM`, `<subfolder>/bugfix-<id>`). Push with `git push origin dev --tags`.

