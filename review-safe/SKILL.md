---
name: review-safe
description: Safe code audit — read-only review, no edits, findings only
allowed-tools: Read, Glob, Grep
---

Review target: $ARGUMENTS

**Audit only. Do not fix. Do not edit.**

Run these checks in parallel across target files:

1. **Security** — SQL/cmd/XSS injection, auth bypass, hardcoded secrets, unsafe deserialization, missing input validation at system boundaries
2. **Logic** — off-by-one, null/nil dereference, race conditions, silent error swallowing, wrong operator precedence
3. **Correctness** — does implementation match intent? missing edge cases? assumptions that will break?
4. **Dependencies** — unused imports, circular deps, unpinned versions in security-sensitive libs

**Format each finding:**

```
[CRITICAL|HIGH|MEDIUM|LOW] file:line
Issue: <one line — what is wrong>
Why:   <why this matters — exploit path or failure mode>
Fix:   <direction, not code>
```

Rules:
- No praise, no summary, no "overall the code looks..."
- Only findings that matter — skip style, skip nitpicks
- If $ARGUMENTS is a PR/diff: read changed lines first, then expand context only as needed
- MEDIUM and below: only report if actionable
