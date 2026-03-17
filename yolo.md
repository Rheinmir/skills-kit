---
description: Alias của /yta — yes-to-all, snapshot trước, chạy tới cùng không hỏi
allowed-tools: Read, Edit, Write, Glob, Grep, Bash, TodoWrite, Agent
---

Task: $ARGUMENTS

Đây là alias của skill `/yta`. Thực thi hoàn toàn theo logic của `/yta`:

## PHASE 0 — SNAPSHOT

```bash
if git rev-parse --git-dir > /dev/null 2>&1; then
  git add -A
  git stash push -m "yolo-snapshot-$(date +%Y%m%d-%H%M%S)" --include-untracked 2>/dev/null \
    && SNAP=$(git stash list | head -1 | cut -d: -f1) \
    || SNAP=$(git rev-parse --short HEAD)
else
  SNAP="$HOME/.claude/backups/yolo-$(date +%Y%m%d-%H%M%S)"
  mkdir -p "$SNAP" && cp -R . "$SNAP/" 2>/dev/null
fi
echo "SNAPSHOT: $SNAP"
```

In ngay:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SNAPSHOT: <ref>   ROLLBACK: git stash pop <ref>  |  cp -R <path>/. .
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## PHASE 1 — EXECUTE

Không hỏi. Không dừng. Tự chọn approach tốt nhất. Fix lỗi tự động. Chạy tới khi có output hoàn chỉnh.

**$ARGUMENTS**

## PHASE 2 — DONE REPORT

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
YOLO DONE
TASK:      <task>
OUTPUT:    <files / kết quả>
DECISIONS: <auto-choices + lý do>
ROLLBACK:  <lệnh copy/paste>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
