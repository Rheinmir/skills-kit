---
name: git-helper
description: Git helper — branch name, commit message, changelog, PR summary
allowed-tools: Bash
---

Task: $ARGUMENTS

**Detect mode from $ARGUMENTS: branch | commit | changelog | pr. Default: commit.**

Run only what the mode needs. Parallel git calls where independent.

---

**MODE: branch**
- `git log main..HEAD --oneline` + `git diff main --stat`
- Output: `<type>/<kebab-slug>` (max 50 chars, no ticket prefix unless given)
- Types: feat, fix, chore, refactor, docs, test

**MODE: commit**
- Run in parallel: `git diff --cached --stat` + `git diff --cached` (first 150 lines) + `git log -5 --oneline`
- If nothing staged: `git diff --stat HEAD` + `git diff HEAD` (first 150 lines)
- Output: conventional commit — `<type>(<scope>): <subject>` then blank line then body bullets (what changed + why, not how)
- Subject: ≤72 chars, imperative mood, no period

**MODE: changelog**
- Args must include range e.g. `v1.2..v1.3` or `HEAD~20..HEAD`
- `git log <range> --oneline --no-merges`
- Group by: Features / Fixes / Chores — skip if empty group
- Format: `- <what changed> (<short-sha>)`
- Omit: style, typo, lock-file-only, dependency bumps unless breaking

**MODE: pr**
- Run in parallel: `git log main..HEAD --oneline` + `git diff main --stat` + `git diff main` (first 200 lines)
- Output:
```
TITLE:   <50 chars, imperative>
SUMMARY: <2–4 bullets — what + why, not how>
TEST:    <how to verify — commands or manual steps>
RISK:    <breaking changes, migrations, flag deps — or "none">
```

**Hard rules:**
- No prose outside the output format
- Never read working-tree files — git commands only
- If range/base branch is ambiguous: ask before running changelog/pr
- Truncate diff reads at 200 lines — enough for intent, not full patch
