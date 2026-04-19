# Docker Workshop — Starter Repo

In this workshop, You'll **containerize a Node.js + Express
web app backed by Postgres**, orchestrate it with **docker-compose**, and wire
up a **GitHub Actions CI pipeline** that builds your image, scans it with
**Trivy** (Image Scanning tools), and pushes it to **Docker Hub**.

The app code under `app/` is **complete**. Your job is to fill in the
`#TODO(...)` markers in the infrastructure files. Every TODO maps to a
checklist item in the Self-Verification section below — tick them off as you go.

---

## Self-Verification Checklist (10 items)

Tick each box as you complete it. The workshop is **done** when all 10 are ticked.

- [ ] **AC-01:** `docker build -t workshop-app .` exits with status 0 locally.
- [ ] **AC-02:** Dockerfile uses a multi-stage build (`grep -c '^FROM' Dockerfile` = 2).
- [ ] **AC-03:** `.dockerignore` excludes at least `node_modules`, `.git`, and `.env`.
- [ ] **AC-04:** Dockerfile contains a `HEALTHCHECK` instruction.
- [ ] **AC-05:** `docker compose up -d` brings up two healthy containers within 60 seconds.
- [ ] **AC-06:** `web` waits for `db` health (`docker compose config` shows `condition: service_healthy`).
- [ ] **AC-07:** `.github/workflows/ci.yml` triggers on`push` to `main`.
- [ ] **AC-08:** The CI step `scan` uses `aquasecurity/trivy-action` with `severity: CRITICAL` and `exit-code: '1'`, and reports **0 CRITICAL** findings.
- [ ] **AC-09:** The CI step `push` pushes two tags: `:latest` and `:${{ github.sha }}`.
- [ ] **AC-10:** The `build-scan-push` job goes green on your fork after Docker Hub secrets are configured.

---

## Step-by-Step Walkthrough

> **TODO convention:** look for `# TODO(step-N<letter>)` (or `// TODO(step-N<letter>)`)
> comments inside the target file. Each step below tells you which file and which
> TODOs to complete.

### Step 1 — Fork and clone

1. Fork this repo to your own GitHub account.
2. Clone your fork locally and `cd` into it.
3. Copy `.env.example` to `.env`.

### Step 2 — Look at the app (no changes)

Open `app/src/index.js`. Notice the three routes:
- `GET /` → `{ ok: true }`
- `GET /health` → 200 if it can reach Postgres, 503 otherwise (the Dockerfile HEALTHCHECK hits this)
- `GET /tasks` → list from the `tasks` table, seeded by `app/db/init.sql`

You will **not** modify any file under `app/`.

### Step 3 — `.dockerignore`

Open `.dockerignore` and complete `TODO(step-3)`. Required entries:

**Verify (AC-03):**
```bash
grep -E '^(node_modules|\.git|\.env)$' .dockerignore
```
Should print three matching lines.

### Step 4 — Multi-stage Dockerfile

Open `Dockerfile` and complete `TODO(step-4a)` through `TODO(step-4g)` (7 markers).
Pay special attention to **step-4f** — the HEALTHCHECK uses a **Node one-liner**
because `node:20.11-slim` does **not** include `curl` or `wget`.

### Step 5 — docker-compose.yml

Open `docker-compose.yml` and complete `TODO(step-5a)` through `TODO(step-5h)` (8 markers).
The key wiring: `web` `depends_on` `db` with `condition: service_healthy`, so the
app doesn't try to connect before Postgres is ready.

**Verify (AC-05, AC-06):**
```bash
docker compose up -d
docker compose ps                 
docker compose config | grep -A1 depends_on
```

Smoke test (also confirms AC-04 at runtime):
```bash
docker inspect --format='{{.State.Health.Status}}' \
  $(docker compose ps -q web)            # should print: healthy
curl -fsS http://localhost:3000/health   # should print {"status":"ok"}
curl -fsS http://localhost:3000/tasks    # should list the seed task
```

### Step 6 — GitHub Actions CI

Open `.github/workflows/ci.yml` and complete `TODO(step-6a)` through `TODO(step-6e)` (5 markers).
Delete the `workflow_dispatch:` placeholder trigger after you add the real ones in step-6a.

**Add secrets to your fork** *before* pushing:
- GitHub → your fork → **Settings → Secrets and variables → Actions → New repository secret**
- Add `DOCKERHUB_USERNAME` (your Docker Hub username)
- Add `DOCKERHUB_TOKEN` (the Personal Access Token from Prerequisites)

**Verify (AC-07..AC-10):**
```bash
git add .
git commit -m "complete docker workshop"
git push
# Then open the "Actions" tab on your fork and watch the job run.
```

When `build-scan-push` goes green, your image is live on Docker Hub at
`docker.io/<your-username>/workshop-app:latest` and `:${{ github.sha }}`.

---

## File Map

```
.
├── README.md                    # this file
├── .gitignore
├── .dockerignore                # 1 TODO
├── .env.example
├── Dockerfile                   # 7 TODOs (step-4a..g)
├── docker-compose.yml           # 8 TODOs (step-5a..h)
├── .github/workflows/ci.yml     # 5 TODOs (step-6a..e)
└── app/
    ├── package.json             # complete — do not edit
    ├── package-lock.json        # complete — do not edit
    ├── db/init.sql              # complete — do not edit
    └── src/
        ├── index.js             # complete — do not edit
        └── db.js                # complete — do not edit
```

Good luck! 

<img width="343" height="193" alt="image" src="https://github.com/user-attachments/assets/b98eb5cf-e82b-4dfd-8902-bcc31c594969" />
