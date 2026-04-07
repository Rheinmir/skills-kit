---
name: call-it-a-day
description: Wrap-up session — tóm lược việc đang làm, lưu memory, sẵn sàng tắt máy
allowed-tools: Read, Glob, Grep, Bash, Write, TodoWrite
---

Người dùng sắp kết thúc phiên làm việc. Thực hiện theo thứ tự:

**1. Thu thập context hiện tại (parallel)**
- Đọc `CHANGES.md` hoặc `CHANGELOG.md` nếu có trong working directory, không có thì tạo mới
- Chạy `git log --oneline -10` để xem commits gần nhất
- Chạy `git status --short` để xem files đang thay đổi
- Chạy `git log origin/HEAD..HEAD --oneline 2>/dev/null` để kiểm tra commits chưa push
- Đọc memory index tại `/Users/giatran/.gemini/antigravity/memory/MEMORY.md` nếu tồn tại

**2. Tổng hợp — output ra màn hình theo format:**

```
=== CALL IT A DAY ===

ĐANG LÀM:     <tính năng / bug / task đang dở — 1-2 câu>
TIẾN ĐỘ:      <đã xong gì, còn lại gì>
FILES ĐÃ SỬA: <danh sách file thay đổi chưa commit nếu có>
CHƯA PUSH:    <commits đã commit nhưng chưa push lên remote — nếu có thì hỏi user có muốn push không>
NEXT SESSION: <việc cần làm đầu tiên khi mở lại — cụ thể, actionable>
GOTCHAS:      <context quan trọng không được quên — edge case, quyết định thiết kế, blocker>
```

**3. Lưu memory** (nếu có thông tin mới đáng nhớ cho phiên sau):
- Viết/cập nhật file memory phù hợp tại `/Users/giatran/.gemini/antigravity/memory/`
- Cập nhật `/Users/giatran/.gemini/antigravity/memory/MEMORY.md` index
- Ưu tiên lưu: quyết định thiết kế quan trọng, context dự án, feedback từ user

**4. Kết thúc bằng 1 dòng:**
> Máy sẵn sàng tắt. Phiên sau bắt đầu từ: **<NEXT SESSION tóm tắt cực ngắn>**

Không hỏi thêm. Không prose thừa. Chỉ output theo format trên.image.png