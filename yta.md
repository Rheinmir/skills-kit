---
description: Yes-to-all executor — snapshot → execute → rollback ready. No questions asked until done.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash, TodoWrite, Agent
---

Task: $ARGUMENTS

## PHASE 0 — SNAPSHOT (bắt buộc, trước mọi thứ)

Tạo snapshot để có thể rollback ngay nếu cần. Chạy song song:

```bash
# Detect repo type + snapshot
if git rev-parse --git-dir > /dev/null 2>&1; then
  IS_GIT=true
else
  IS_GIT=false
fi
```

**Nếu là git repo:**
```bash
git add -A
git stash push -m "yta-snapshot-$(date +%Y%m%d-%H%M%S)" --include-untracked
SNAPSHOT_REF=$(git stash list | head -1 | awk '{print $1}' | tr -d ':')
echo "SNAPSHOT: $SNAPSHOT_REF"
```
> Nếu không có gì để stash (working tree clean), ghi nhận commit hiện tại:
```bash
SNAPSHOT_REF=$(git rev-parse --short HEAD)
echo "SNAPSHOT: commit $SNAPSHOT_REF (tree already clean)"
```

**Nếu KHÔNG phải git repo:**
```bash
SNAP_DIR="$HOME/.claude/backups/yta-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$SNAP_DIR"
cp -R . "$SNAP_DIR/" 2>/dev/null || true
echo "SNAPSHOT: $SNAP_DIR"
```

In ra dòng này TRƯỚC khi làm bất cứ điều gì:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SNAPSHOT TAKEN: <ref hoặc path>
ROLLBACK:       <lệnh rollback — xem bên dưới>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Lệnh rollback chuẩn bị sẵn:
- Git stash: `git checkout -- . && git stash pop <ref>`
- Git clean commit: `git reset --hard <commit-sha>`
- Non-git: `cp -R <SNAP_DIR>/. .`

---

## PHASE 1 — EXECUTE (không hỏi, không dừng)

**Nguyên tắc bất di bất dịch trong skill này:**
- KHÔNG hỏi xác nhận bất kỳ bước nào
- KHÔNG dừng để clarify — tự suy luận intent từ context
- Nếu có nhiều cách → chọn cách đơn giản nhất, ghi chú lý do ở cuối
- Nếu gặp lỗi → tự diagnose và fix, không báo giữa chừng
- KHÔNG amend commit cũ — chỉ tạo commit mới nếu cần
- Chạy hết task tới khi có output hoàn chỉnh

Dùng TodoWrite để track progress. Mark done ngay khi xong từng bước.

Thực hiện task: **$ARGUMENTS**

---

## PHASE 2 — DONE REPORT

Khi task hoàn tất, in summary theo format:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
YTA DONE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TASK:      <task đã làm>
OUTPUT:    <files đã thay đổi / tạo mới / kết quả>
DECISIONS: <lựa chọn tự động quan trọng + lý do>
ROLLBACK:  <lệnh copy/paste để undo toàn bộ>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Không prose thừa. Không "Would you like me to...".
