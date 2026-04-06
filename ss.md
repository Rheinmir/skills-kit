---
name: ss
description: Alias của /snapshot — snapshot code tại thời điểm hiện tại, rollback ngay nếu cần
allowed-tools: Bash
---

## SNAPSHOT — Backup code ngay bây giờ

```bash
# Detect git hay không
if git rev-parse --git-dir > /dev/null 2>&1; then
  IS_GIT=true
else
  IS_GIT=false
fi

PROJECT_NAME=$(basename "$(pwd)" | tr ' ' '_')
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
BACKUP_BASE="$HOME/.gemini/antigravity/backups/snapshots/$PROJECT_NAME"

if [ "$IS_GIT" = true ]; then
  # Git repo: stash nếu dirty, ghi commit nếu clean
  if ! git diff --quiet || ! git diff --cached --quiet || [ -n "$(git ls-files --others --exclude-standard)" ]; then
    SNAP_REF=$(git stash push -m "snapshot-$TIMESTAMP" --include-untracked 2>&1 | tail -1)
    STASH_ID=$(git stash list | head -1 | cut -d: -f1)
    ROLLBACK_CMD="git stash pop $STASH_ID"
    SNAP_INFO="git stash: $STASH_ID (snapshot-$TIMESTAMP)"
  else
    SNAP_REF=$(git rev-parse --short HEAD)
    ROLLBACK_CMD="git reset --hard $SNAP_REF"
    SNAP_INFO="git commit: $SNAP_REF (tree clean, no stash needed)"
  fi
else
  # Non-git: rsync với --link-dest để hardlink file không đổi (zero duplicate)
  mkdir -p "$BACKUP_BASE"
  SNAP_DIR="$BACKUP_BASE/$TIMESTAMP"
  LATEST_LINK="$BACKUP_BASE/latest"

  if [ -d "$LATEST_LINK" ]; then
    rsync -a --link-dest="$LATEST_LINK" --exclude='.git' --exclude='node_modules' --exclude='__pycache__' --exclude='*.pyc' --exclude='.next' --exclude='dist' --exclude='tmp' . "$SNAP_DIR/"
  else
    rsync -a --exclude='.git' --exclude='node_modules' --exclude='__pycache__' --exclude='*.pyc' --exclude='.next' --exclude='dist' --exclude='tmp' . "$SNAP_DIR/"
  fi

  # Cập nhật symlink latest
  ln -sfn "$SNAP_DIR" "$LATEST_LINK"

  # Đếm số snapshots
  SNAP_COUNT=$(ls -1 "$BACKUP_BASE" | grep -v '^latest$' | wc -l | tr -d ' ')
  SNAP_SIZE=$(du -sh "$SNAP_DIR" 2>/dev/null | cut -f1)
  ROLLBACK_CMD="rsync -a --delete \"$SNAP_DIR/\" ."
  SNAP_INFO="$SNAP_DIR ($SNAP_SIZE, tổng $SNAP_COUNT snapshot)"
fi

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "SNAPSHOT TAKEN"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "PROJECT:  $PROJECT_NAME"
echo "TIME:     $TIMESTAMP"
echo "SNAPSHOT: $SNAP_INFO"
echo ""
echo "ROLLBACK:"
echo "  $ROLLBACK_CMD"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

Sau khi chạy, in lại output trên cho user thấy ngay.
