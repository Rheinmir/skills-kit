---
description: Bắt đầu buổi sáng — load memory, check trạng thái repo + container, brief việc cần làm
allowed-tools: Read, Glob, Grep, Bash, TodoWrite
---

Người dùng vừa mở phiên làm việc mới. Thực hiện theo thứ tự:

**1. Thu thập context (parallel)**
- Đọc `/Users/giatran/.claude/memory/MEMORY.md` để lấy index memory
- Đọc các file memory liên quan đến project hiện tại (project_*.md, feedback_*.md)
- Chạy `git log --oneline -5` để xem commits gần nhất
- Chạy `git status --short` để xem files đang thay đổi
- Chạy `git branch --show-current` để biết đang ở branch nào
- Chạy `git log origin/HEAD..HEAD --oneline 2>/dev/null` để xem commits chưa push
- Chạy `docker ps --format "table {{.Names}}\t{{.Status}}" 2>/dev/null` để check container

**2. Output ra màn hình theo format:**

```
=== GOOD MORNIN ===

PROJECT:      <tên project + working dir>
BRANCH:       <branch hiện tại> | <X commits chưa push nếu có>
CONTAINER:    <tên + trạng thái từng container, hoặc "không có container nào chạy">

ĐANG DỞ:     <tính năng / bug / task chưa xong từ phiên trước — lấy từ memory>
NEXT ACTION: <việc cụ thể cần làm NGAY — actionable, không phải danh sách>
GOTCHAS:     <context quan trọng không được quên — edge case, blocker, quyết định thiết kế>

FILES DỞ:   <files đang thay đổi chưa commit nếu có, kèm ghi chú ngắn nếu nhớ>
```

**3. Gợi ý tiếp theo** (1-2 dòng, không hỏi):
- Nếu có commits chưa push → nhắc push
- Nếu container down → nhắc `docker-ops`
- Nếu có files uncommitted → nhắc review hoặc commit

**4. Kết thúc bằng 1 dòng:**
> Sẵn sàng. Bắt đầu từ: **<NEXT ACTION tóm tắt cực ngắn>**

Không hỏi thêm. Không prose thừa. Chỉ output theo format trên.
