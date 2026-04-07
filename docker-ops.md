---
name: docker-ops
description: Docker ops — check status, build, update/restart containers, cleanup
allowed-tools: Bash, Glob, Read
---

Task: $ARGUMENTS

**Step 1: SNAPSHOT — run in parallel, always first:**
```
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}\t{{.Image}}"
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.Size}}\t{{.CreatedSince}}"
docker system df
```

**Step 2: DETECT mode from $ARGUMENTS + snapshot results:**

| Mode | Trigger |
|------|---------|
| `status` | no args, or "check", "ps", "list" |
| `build` | **"build", "b"** — execute immediately, no confirmation |
| `update` | "update", "restart", "recreate", container name given |
| `logs` | "log", "tail", container name given |
| `cleanup` | "clean", "prune", "remove stopped" |
| `compose` | docker-compose.yml detected or "compose" in args |

**Shorthand aliases (execute immediately, no confirmation prompt):**
- `build` / `b` → MODE: build
- `b <service>` → build only that service's image (e.g. `b frontend`)
- `b <service> --no-cache` → build without cache

---

**MODE: status**
- Output from snapshot, no extra commands
- Flag: containers Exited >1h, images >1GB, disk usage >80%

**MODE: build** ← triggered by "build" or "b", runs immediately without asking
- Glob for `Dockerfile*` in working directory to find build targets
- If service name given (e.g. `b frontend`): build only that image
- If no service: build ALL images found via Dockerfile* glob
- `docker build -t <name>:latest <context>` — use `--no-cache` only if explicitly requested
- Then recreate the container if it was running: stop → rm → run with same flags
- After: `docker ps --filter name=<container>` to confirm
- Output: `[OK] <image> — <size>, rebuilt in <time>`

**MODE: update** (rebuild + recreate running container)
- Detect if compose or standalone:
  - Compose: `docker compose up -d --build <service>` (or all if no service named)
  - Standalone: `docker build -t <image> . && docker stop <container> && docker rm <container> && docker run -d --name <container> <same flags as old container>`
- Get old container flags: `docker inspect <container> --format '{{json .HostConfig}}'` before stopping
- Confirm: `docker ps --filter name=<container>`

**MODE: logs**
- `docker logs --tail 100 -f <container>` — stop after 5s if no error pattern
- If error found: show last 50 lines around error

**MODE: cleanup**
- Run in sequence (safe order):
  1. `docker container prune -f`
  2. `docker image prune -f`
  3. `docker volume prune -f` — **only if args explicitly say "volumes"**
  4. `docker network prune -f`
- Show reclaimed space after each step

**MODE: compose**
- Read `docker-compose.yml` (or specified file) first
- Map services to containers via `docker compose ps`
- Suggest: `up -d`, `up -d --build <svc>`, `restart <svc>`, `pull && up -d`

---

**Hard rules:**
- Always snapshot before acting — never assume container/image state
- Never `docker rm -f` a running container without confirming first
- Never prune volumes unless user explicitly says so
- `--no-cache` only when user asks — default uses cache for speed
- If compose file exists and task targets a service: prefer compose over raw docker
- Output format after each action: `[OK|WARN|ERR] <container/image> — <one-line status>`
