---
description: Fix issue + write tests — input: issue number, description, or file:line
allowed-tools: Read, Edit, Write, Glob, Grep, Bash, TodoWrite
---

Task: $ARGUMENTS

**Execute in strict order. No shortcuts. No scope creep.**

1. **SCOPE** — Parse the task. Grep/Glob to find all affected files. Write todos before touching code.

2. **READ** — Read every file you'll modify. Trace the call path to the bug/feature site. Understand existing patterns before writing new ones.

3. **FIX** — Minimal change only. No refactors, no style cleanup, no unrelated improvements. Comments only where logic is genuinely non-obvious.

4. **TEST** — Write tests that would have caught this issue. Follow existing test file conventions (location, naming, imports). Run the test suite.

5. **VERIFY** — Confirm fix passes + no regressions. If a test fails: diagnose root cause. Do NOT retry the same approach. Do NOT use `--no-verify` or skip hooks.

**Hard rules:**
- Read before every edit
- If scope is ambiguous → stop and ask, don't guess
- One logical change per session — don't bundle unrelated fixes
- If existing tests contradict your fix, investigate before overriding
