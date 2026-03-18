---
description: CI/CD helper — push git để trigger pipeline, report luồng từ file triển khai có trong repo
allowed-tools: Bash, Read, Glob, Grep
---

Người dùng muốn push code và/hoặc hiểu luồng CI/CD của repo hiện tại.

**Detect mode từ input:**
- Nếu input bắt đầu bằng `f` hoặc `force` → **FORCE MODE**: push ngay không hỏi, report sau
- Không có prefix → **NORMAL MODE**: hỏi xác nhận trước khi push

**Thực hiện theo thứ tự:**

**1. Scan file triển khai (parallel)**

Tìm tất cả file CI/CD phổ biến:
- `Jenkinsfile` hoặc `Jenkinsfile.*`
- `.github/workflows/*.yml`
- `.gitlab-ci.yml`
- `Dockerfile`, `docker-compose*.yml`
- `.circleci/config.yml`
- `bitbucket-pipelines.yml`
- `azure-pipelines.yml`
- `cloudbuild.yaml`
- `fly.toml`, `render.yaml`, `railway.toml`

**2. Đọc và phân tích file tìm được**

Với mỗi file tìm được, đọc nội dung và extract:
- Các **stages/jobs** theo thứ tự
- **Trigger** (push branch nào, PR, manual, schedule)
- **Environment** target (dev/staging/prod)
- **Commands** chính (build, test, deploy)

**3. Output report theo format:**

```
=== CI/CD PIPELINE REPORT ===

FILE:      <tên file tìm thấy>
PLATFORM:  <Jenkins / GitHub Actions / GitLab CI / ...>
TRIGGER:   <push to main / PR / manual / ...>

STAGES:
  1. <stage name> → <lệnh chính>
  2. <stage name> → <lệnh chính>
  ...

ENV TARGETS: <dev / staging / prod>
DEPLOY TO:   <server / k8s / docker / cloud service>
GOTCHAS:     <biến env cần set, secret cần có, điều kiện đặc biệt>
```

Nếu không tìm thấy file CI/CD nào → báo rõ và gợi ý platform phù hợp dựa trên stack repo.

**4. Kiểm tra git status**

```bash
git status --short
git log origin/HEAD..HEAD --oneline 2>/dev/null
```

Hiển thị:
- Files chưa commit
- Commits chưa push

**5. Push theo mode**

**NORMAL MODE** — hỏi trước:
- Nếu có commits chưa push → hỏi: `Có N commit chưa push lên <branch>. Push để trigger pipeline không?`
- Nếu user confirm → `git push`
- Nếu còn file chưa commit → hỏi có muốn commit trước không

**FORCE MODE** (`f` prefix) — không hỏi:
- Nếu có file chưa commit → `git add -A && git commit -m "chore: force deploy"` rồi push
- Nếu chỉ có commits chưa push → `git push` ngay
- Sau khi push xong mới output report và kết quả

**Không tự push mà không hỏi — trừ khi FORCE MODE.**
