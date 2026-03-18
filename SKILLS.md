# Skills — cheat sheet

## Mỗi skill làm gì

| Skill | Dùng khi | Không dùng khi |
|-------|----------|----------------|
| `/explore-repo` | Mở project cũ, quên structure, trace entry point | Đã nhớ rõ code |
| `/fix-issue <mô tả>` | Fix bug, thêm feature nhỏ — tự scope + test | Refactor lớn, chưa rõ yêu cầu |
| `/review-safe <file>` | Audit bảo mật/logic trước khi merge, read-only | Muốn sửa luôn |
| `/git-helper <mode>` | Đặt tên branch, viết commit, changelog, PR desc | Thao tác git phức tạp (rebase, conflict) |
| `/docker-ops <args>` | Check status, build, update container, cleanup | Orchestration (k8s, swarm) |
| `/mcp-research <query>` | Tra API spec, DB schema, docs từ GitHub/internal | Câu hỏi về code trong repo hiện tại |
| `/yolo` | Chạy tới cùng không hỏi, snapshot trước | Khi cần review từng bước |
| `/mornin` | Đầu buổi — load memory, check git + container, brief next action | Giữa chừng công việc |
| `/call-it-a-day` | Wrap-up session, lưu memory, tắt máy | Giữa chừng công việc |

---

## Daily flow

### Buổi sáng — mở project cũ
```
/mornin              → load memory + git + container, brief next action (1 lệnh thay 3)
```
hoặc thủ công:
```
/explore-repo        → nhớ lại stack, entry point, run/test command
/docker-ops          → container nào chạy, cái nào chết
/git-helper branch   → đang ở đâu, diff với main thế nào
```

### Làm việc
```
/git-helper branch feat/xxx   → tên branch mới
/fix-issue <mô tả>            → fix + viết test
/review-safe <file>           → self-review trước commit
/git-helper commit            → commit message từ staged changes
```

### Cần tra cứu ngoài repo
```
/mcp-research <câu hỏi>       → API endpoint, schema, config key
```

### Ship
```
/docker-ops update <service>        → rebuild + recreate
/git-helper pr                      → PR title + summary + risk
/git-helper changelog v1.0..v1.1   → release notes
```

---

## 80% ngày thường

```
/explore-repo        → nhớ lại project
/fix-issue           → làm việc
/git-helper commit   → commit
/docker-ops update   → deploy
```
