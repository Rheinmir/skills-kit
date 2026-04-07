---
name: find
description: Laser-targeted search for a specific symbol, feature, or concept in the codebase. Token-efficient — stops the moment it has enough to answer. Use when user asks "where is X", "find the auth handler", "which file handles /checkout", "where is User model defined". Do NOT use for full repo mapping (use explore-repo for that).
allowed-tools: Read, Glob, Grep, Bash
---

Find: **$ARGUMENTS** in the codebase.

**CRITICAL RULES:**
- STOP immediately when you have enough to answer. Do not continue pipeline steps.
- Read snippets only — never full files unless unavoidable.
- Report tokens saved vs a full-file approach.
- Parallel tool calls within each step.

---

## STEP 0 — Bootstrap from memory (FREE, always run first)

Run in parallel:
```bash
# 1. Read project memory docs (cheap, high-signal)
cat AGENTS.md 2>/dev/null | head -60
cat be/AGENTS.md 2>/dev/null | head -40
cat fe/AGENTS.md 2>/dev/null | head -40

# 2. Read any KI or memory files
ls .gemini/ 2>/dev/null
cat PROJECT.md 2>/dev/null | head -40
```

**Goal:** Extract subsystem ownership map from AGENTS.md tables.
- Which directory owns the feature/domain being searched?
- Narrow search space from "entire repo" → "1-2 directories"

→ If memory clearly names the file/module → jump to STEP 3 (read snippet). Skip STEP 1 and STEP 2.

---

## STEP 1 — .graph-agent SQL lookup (if index exists)

```bash
test -f .graph-agent/index.db && echo EXISTS
```

If EXISTS → run targeted SQL queries in parallel (do NOT read any source files yet):

```sql
-- Find symbol by name (exact or fuzzy)
SELECT s.name, s.kind, s.line_start, f.path
FROM symbols s JOIN files f ON s.file_id = f.id
WHERE s.name LIKE '%<keyword>%'
ORDER BY length(s.name) ASC LIMIT 15;

-- Find files matching keyword in path
SELECT path, language FROM files
WHERE path LIKE '%<keyword>%'
ORDER BY length(path) ASC LIMIT 10;

-- Find what calls a symbol (callers)
SELECT e.src_symbol, f.path, e.line
FROM edges e JOIN files f ON e.src_file_id = f.id
WHERE e.dst_symbol LIKE '%<keyword>%' AND e.kind = 'CALLS'
LIMIT 10;

-- Find what a symbol imports (deps of a file)
SELECT e.dst_file_path FROM edges e
JOIN files f ON e.src_file_id = f.id
WHERE f.path LIKE '%<keyword>%' AND e.kind = 'IMPORTS'
LIMIT 15;
```

Shell command form:
```bash
DB=".graph-agent/index.db"
sqlite3 "$DB" "SELECT s.name, s.kind, s.line_start, f.path FROM symbols s JOIN files f ON s.file_id=f.id WHERE s.name LIKE '%<keyword>%' ORDER BY length(s.name) ASC LIMIT 15;"
sqlite3 "$DB" "SELECT path, language FROM files WHERE path LIKE '%<keyword>%' ORDER BY length(path) ASC LIMIT 10;"
```

→ If graph returns file + line → go to STEP 3 (read snippet at that line). Skip STEP 2.
→ If graph returns multiple candidates → parallel read snippets of top 3.

---

## STEP 2 — Targeted Grep (fallback, no graph)

Run in parallel — use the narrowed directory from STEP 0, NOT full repo:

```bash
# Symbol/function definition
grep -rn "func <keyword>\|def <keyword>\|function <keyword>\|class <keyword>\|const <keyword> =\|<keyword>:" \
  <narrowed_dir>/ \
  --include="*.go" --include="*.ts" --include="*.tsx" --include="*.py" --include="*.js" \
  --exclude-dir=node_modules --exclude-dir=.git --exclude-dir=dist \
  -l  # ← list files only first, NOT content (saves tokens)

# Route/endpoint
grep -rn "\"/<keyword>\|'/<keyword>\|/<keyword>" \
  <narrowed_dir>/ \
  --include="*.go" --include="*.ts" --exclude-dir=node_modules \
  -l
```

**Two-pass:** First `-l` to get file list (1-2 lines each). Then grep only those files with `-A 5 -B 3`.

→ Do NOT grep the full repo if AGENTS.md already narrowed to a subdirectory.
→ Stop after first file confirms the answer.

---

## STEP 3 — Read snippet only (never full file)

Once you have file + approximate line:

```bash
# Read ±20 lines around the match
sed -n '<start>,<end>p' <file_path>

# Or use grep with context
grep -n -A 15 -B 5 "<keyword>" <file_path>
```

**Hard limits:**
- Max 40 lines per snippet
- Max 2 files read per search
- If function spans >40 lines → read signature + first 10 lines only

---

## STEP 4 — Filename search (if steps above missed)

```bash
find . -type f \( -iname "*<keyword>*" \) \
  | grep -v node_modules | grep -v .git | grep -v dist | grep -v .next \
  | head -10
```

Then `head -30 <matched_file>` to confirm.

---

## STEP 5 — Full grep fallback (expensive, last resort)

Only if all above failed. Log: `[STEP 0-4 failed, falling back to full grep]`

```bash
grep -rn "<keyword>" . \
  --include="*.go" --include="*.ts" --include="*.tsx" --include="*.py" \
  --exclude-dir=node_modules --exclude-dir=.git --exclude-dir=dist --exclude-dir=.next \
  | head -30
```

Read only the top 3 matches with `sed -n` snippets.

---

## OUTPUT (minimal, always this format)

```
FOUND: `<file_path>:<line>`
VIA:   Step <N> [memory|graph|grep|filename|full-grep]

<snippet — max 15 lines>

RELATED (if any):
- `<other_file>:<line>` — <one-line role>

REPORT (target ≤70 words, may exceed only when needed to express full meaning — keep tight):
- kind: <function | method | class | route | config | type | ...>
- name: <exact name>
- purpose: <what it does in one phrase>
- called by: <who uses it, or "entry point">
- depends on: <key deps if relevant>
- side effects: <mutations / API calls / DB writes if any>
- notes: <non-obvious gotcha or context>
```

No prose. No explanation outside the block unless user asks. If not found after all steps: `NOT FOUND after 5 steps. Try: <suggested grep command>`.
