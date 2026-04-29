# KreativeLand Ecosystem - Master Agent Instructions

## How to Address the User
- Address the user as **KL**
- KL is building an educational technology ecosystem for ESL/EFL learners, primarily targeting Chinese students preparing for the zhongkao (high school entrance exam) in Hangzhou
- Be concise and direct. KL is technical and prefers action over lengthy explanations

---

## Quest Registry

Multiple quests run concurrently across the KreativeLand ecosystem. Each quest is responsible for a specific area. **Every quest must maintain its own status section in its own AGENTS.md and keep it current.**

| Quest Name | Workspace Folder | AGENTS.md | Responsibility |
|------------|-----------------|-----------|----------------|
| **Qoder Admin** | KreativeLand | This file | KreativeLand portal, cross-quest coordination, Scott/office PC, infrastructure |
| **VocabVista** | VocabVista | `../VocabVista/AGENTS.md` | VocabVista backend API, vocabulary database, word list generation, story generation |
| **VocabVista Product Manager** | VocabVista | `../VocabVista/AGENTS.md` | Roadmap, UI/UX design, feature specs for all VocabVista-connected apps |
| **Little Reader Storybook** | LittleReader | `../LittleReader/AGENTS.md` | LittleReader frontend app - serving stories to younger children |
| **English Prep** | EnglishExamPrep | `../EnglishExamPrep/AGENTS.md` | EnglishExamPrep frontend app (英考通) - English exam practice for KL's 15yo daughter |
| **ZhongkaoPrep** | ZhongkaoPrep | `../ZhongkaoPrep/AGENTS.md` | ZhongkaoPrep frontend app (中考冲刺) - Multi-subject zhongkao exam practice (math, science) |
| **Little Learner** | LittleLearner | `../LittleLearner/AGENTS.md` | LittleLearner early education app (reading, music, math) for infants/toddlers |
| **Little Learner PM** | LittleLearner | `../LittleLearner/AGENTS.md` | Roadmap, UI/UX design, feature specs for LittleLearner product line |

### Quest Workspace Paths
All projects: `D:\Projects\`

---

## Cross-Quest Coordination Protocol

Tasks across quests are often **interrelated and co-dependent**. Follow these rules:

1. **Keep your AGENTS.md updated** - after completing any significant task, update your quest's status section in your AGENTS.md immediately. Other quests rely on this to know what's happening
2. **Check other quests before starting dependent work** - if your task depends on another quest's output (e.g., Little Reader needs a VocabVista API endpoint), read that quest's AGENTS.md to check current status
3. **Document API contracts and shared interfaces** - if you create or change something another quest consumes, document it clearly in your status section
4. **Qoder Admin has cross-project access** - the Qoder Admin quest can read and modify files in any project folder as needed for coordination, git housekeeping, and ecosystem-wide maintenance. Other quests should not modify files outside their own folder unless KL explicitly asks
5. **Flag blockers** - if you're blocked on another quest's work, note it in your status section so KL and the other quest can see it
6. **Reference the shared setup doc** - `../qoder-fresh-setup-info.md` has ecosystem-wide setup info, Microsoft To Do integration, and production architecture

### Handoff & Spec Protocol (MANDATORY)

When you receive a handoff document (`HANDOFF-*.md`, `docs/HANDOFF-*.md`) or a spec document (`SPEC-*.md`, `docs/SPEC-*.md`), or instructions referencing a previous quest's work:

1. **Read the handoff document thoroughly** before doing anything else
2. **Execute the handoff tasks** as instructed
3. **IMMEDIATELY update your Quest Status section in AGENTS.md** after completing the handoff work. This is NOT optional. The quest that created the handoff is relying on seeing your status update to know the work was done
4. **Include in your status update:**
   - What was completed from the handoff
   - What was not completed and why
   - Any new information discovered (e.g., "handoff said X wasn't done, but it was already done")
   - Current blockers if any
5. **Do this BEFORE moving on to other work.** The AGENTS.md update is part of the handoff task, not a separate task to do later
6. **Mark the corresponding To Do task as (DONE).** If the handoff came from a To Do task, append "(DONE)" to the task title via Graph API. This is mandatory -- do NOT skip this step
7. **Tell KL you have updated AGENTS.md and marked the task (DONE).** After updating your status section and the To Do task, explicitly tell KL that you have done both. Do not wait for KL to ask

### Git Identity
Use this identity for all commits across the ecosystem:
- **Name:** `KL`
- **Email:** `kl@kreativeland.com`

Set per-repo (already configured on all projects):
```bash
git config user.name "KL"
git config user.email "kl@kreativeland.com"
```

### Git Repository Setup (IMPORTANT - OneDrive Safe)

**Problem:** OneDrive corrupts/deletes `.git` directories because it aggressively syncs the small binary files inside them. This has destroyed git repos across all projects multiple times.

**Solution:** All `.git` directories are stored OUTSIDE OneDrive at `D:\GitRepos\`. Each project folder in OneDrive has a `.git` **file** (not directory) that points to the external location.

| Project | .git file content | GitHub Remote |
|---------|------------------|---------------|
| VocabVista | `gitdir: D:/GitRepos/VocabVista.git` | https://github.com/Oohyeah99/VocabVista.git |
| EnglishExamPrep | `gitdir: D:/GitRepos/EnglishExamPrep.git` | https://github.com/Oohyeah99/EnglishExamPrep.git |
| ZhongkaoPrep | `gitdir: D:/GitRepos/ZhongkaoPrep.git` | https://github.com/Oohyeah99/ZhongkaoPrep.git |
| LittleReader | `gitdir: D:/GitRepos/LittleReader.git` | https://github.com/Oohyeah99/LittleReader.git |
| KreativeLand | `gitdir: D:/GitRepos/KreativeLand.git` | https://github.com/Oohyeah99/KreativeLand.git |
| LittleLearner | `gitdir: D:/GitRepos/LittleLearner.git` | https://github.com/Oohyeah99/LittleLearner.git |
| AIPlayground | `gitdir: D:/GitRepos/AIPlayground.git` | https://github.com/Oohyeah99/AIPlayground.git |

**Rules for all quests:**
1. **NEVER delete or modify the `.git` file** in the project folder. It's a tiny text file, not a directory
2. **If git stops working** (e.g., "not a git repository"), check if the `.git` file still exists and contains the correct `gitdir:` path. If OneDrive deleted it, recreate it
3. **To init a new project with this pattern:**
   ```bash
   git init --separate-git-dir="D:/GitRepos/ProjectName.git"
   git remote add origin https://github.com/Oohyeah99/ProjectName.git
   git config user.name "KL"
   git config user.email "kl@kreativeland.com"
   ```
4. **Always use HTTP/1.1 for push** (GFW blocks HTTP/2 to GitHub):
   ```bash
   git -c http.version=HTTP/1.1 -c http.postBuffer=524288000 push origin master
   ```
5. **Credential popup prevention:** Global git config has `credential.guiPrompt=false` and `credential.interactive=never`. Credentials are cached by Git Credential Manager. If a push fails with auth error, the cached OAuth token may have expired -- re-run the push and GCM will refresh it silently
6. **If push fails with connection reset / timeout (GFW):** KL has a proxy that is usually ON but sometimes OFF. Tell KL: "Push failed -- likely GFW. Can you check your proxy is on?" and retry after confirmation. Do NOT silently retry more than once

### Local Laptop Ollama -- DO NOT USE

**CRITICAL:** KL's laptop (`localhost:11434`) MUST NOT be used to run Ollama unless KL explicitly asks for it first. This is a hard constraint.

- **No automated tasks** should use local Ollama
- **No apps** should default to localhost Ollama without explicit user opt-in
- **No background jobs** (word categorization, batch processing, etc.) should run on local Ollama
- **All LLM inference** should use the office 4090 GPU (`100.113.43.60:11434`), NVIDIA NIM cloud, or OpenAI API
- The only exception is when KL is actively developing/testing and explicitly asks to use local Ollama

**Reason:** Local Ollama runs on KL's laptop CPU (Intel Core Ultra X9 + 32GB RAM) and impacts system performance. It should only be used when KL specifically needs it for development or testing purposes.

---

## KreativeLand Ecosystem - All Projects

| Project | Type | Tech Stack | Live URL | Status |
|---------|------|------------|----------|--------|
| **VocabVista** | Full-stack API + Admin UI | Python FastAPI + React 18 + Vite 6 | Backend on Azure VM | Active - central platform |
| **EnglishExamPrep** | React SPA | React 19, Vite 8, TypeScript 6, Tailwind v4 | https://english.kreativeland.com | Active |
| **ZhongkaoPrep** | React SPA | React 19, Vite 8, TypeScript 6, Tailwind v4, KaTeX | https://exam.kreativeland.com | Active - v2.0 live |
| **LittleReader** | React SPA | React 18, Vite 6, Tailwind 3 | https://reader.kreativeland.com | Active |
| **VocabLearner** | React SPA | React 18, Vite 6, Tailwind 3 | https://vocab.kreativeland.com | **Deprecated** - files kept as archive |
| **KreativeLand** | Static portfolio hub | HTML/CSS/JS | https://kreativeland.com | Active |
| **PatternPhonics** | Interactive tool | HTML/CSS/JS | https://phonics.kreativeland.com | Active |
| **OllamaChat** | Local chat UI | HTML/CSS/JS | Local only | Active |
| **AIPlayground** | AI multi-provider interface | Express 5, vanilla JS, Ollama + OpenAI + Gemini | ai.kreativeland.com (pending) | Active - v1.1 |
| **LittleLearner** | Early education platform | TBD | Not deployed | Active - new project |
| **todoMCP** | MCP service | TypeScript/Node | npm: @jhirono/todomcp | Active |

### How Projects Interconnect
```
                    +-----------------+
                    |  KreativeLand   |  (portfolio hub - links to all apps)
                    +--------+--------+
                             |
     +---------------+-------+-------+------------------+
     |               |               |                  |
+----v----+  +-------v-------+  +----v-----+  +--------v------+
|ZhongKao |  |EnglishExamPrep|  |LittleRead|  |PatternPhonics |
|(中考冲刺)|  | (英考通)      |  |(story gen)|  |   (phonics)   |
+----+----+  +-------+-------+  +----+-----+  +---------------+
     |               |               |
     +---------------+---------------+
                     |
                +----v-----------+
                |   VocabVista   |
                | (central API + |
                | vocab database)|
                +----------------+
```

- **LittleReader** uses VocabVista's story generation and user vocabulary endpoints
- **EnglishExamPrep** has its own OpenAI integration but shares vocabulary concepts
- **ZhongkaoPrep** uses VocabVista user system for cross-app user management
- **PatternPhonics** is standalone (no API dependency)
- **OllamaChat** is standalone (local Ollama connection)

---

## Microsoft To Do Integration (todoMCP)

Agents can read and manage tasks in Microsoft To Do via the MCP protocol. See `../qoder-fresh-setup-info.md` for full setup instructions, Graph API fallback, and troubleshooting.

### Configuration
The MCP server is configured in `.mcp.json` at the project root. If the mstodo MCP server is not available in the session, use the Microsoft Graph API directly with the access token in `../todoMCP/tokens.json`.

### Task List Mapping
| Quest | To Do List Name |
|-------|-----------------|
| Qoder Admin | "Qoder PM (Quest)" |
| VocabVista | "Vocab Vista (Quest)" |
| VocabVista Product Manager | "Vocab Vista (Quest)" |
| Little Reader Storybook | "LR Storybooks (Quest)" |
| English Prep | "English Prep App (Quest)" |
| ZhongkaoPrep | "ZK Prep (Quest)" |
| Little Learner | "Learner Website (Quest)" |
| Little Learner PM | "Learner Website (Quest)" |
| **Scott (human assistant)** | "For Scott from Qoder Quests" |

### Assigning Tasks to Scott (Human Assistant)

Scott is KL's office assistant who manages the office PC (hp-pc1, 100.113.43.60) with the Nvidia 4090 GPU. Any quest can create tasks for Scott when physical/manual intervention is needed on the office machine (e.g., installing software, rebooting, checking hardware).

**To Do List:** "For Scott from Qoder Quests"
**List ID:** `AQMkADAwATM3ZmYAZS04OABjMS1mOTg2LTAwAi0wMAoALgAAA2g773rSOQhIms6qpVd3BLgBAPPHOT4NZ0tCnZHsl_2wI-EACZ7j-cgAAAA=`

**How to create a task for Scott:**
```bash
TOKEN=$(python3 -c "import json; f=open('D:/OneDrive/AI/Qoder Workspace/todoMCP/tokens.json'); d=json.load(f); print(d['accessToken'])")
LIST_ID="AQMkADAwATM3ZmYAZS04OABjMS1mOTg2LTAwAi0wMAoALgAAA2g773rSOQhIms6qpVd3BLgBAPPHOT4NZ0tCnZHsl_2wI-EACZ7j-cgAAAA="
# Write task body to a temp JSON file, then:
curl -s -X POST "https://graph.microsoft.com/v1.0/me/todo/lists/${LIST_ID}/tasks" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d @task_body.json
```

**Rules for Scott tasks:**
1. Always include clear step-by-step instructions in the task body -- Scott is non-technical
2. Reference the setup doc (`../office-computer-setup-for-scott.md`) when applicable
3. Set `"importance": "high"` for urgent tasks
4. Ask Scott to send results/screenshots back to "Master KL" for verification
5. Scott's setup document: `D:\Projects\office-computer-setup-for-scott.md`

### Task Workflow Rules
1. **Only check the relevant task list** for this project's quest. Do not scan other lists unless KL explicitly asks
2. **Only check the task list when KL asks** - triggers include: "todo", "check task list", "task list", or similar phrases. When triggered, immediately fetch all tasks from the relevant list and **start executing them right away -- do NOT ask KL for confirmation or permission to proceed**. KL asking you to check the list IS the instruction to do the work
3. **Always read the notes** (body content) of each task and check for **attachments** (images, files). These contain important context and instructions
4. **Never mark tasks as completed** in To Do. Instead, append "(DONE)" to the task title when finished. Only KL marks tasks as complete
5. **Skip draft tasks** - ignore any task whose title starts with "dft", "DFT", or "DRAFT"
6. **Starred/high-importance tasks first** - do these before anything else, notify KL when done, then automatically proceed with the rest
7. **Work top-to-bottom** - unless it's more efficient to do otherwise, process tasks in the order they appear (top of list first)
8. **After finishing a starred task**, let KL know it's done, then continue with the next task automatically

### Handling Expired Access Tokens (IMPORTANT)

Microsoft Graph API access tokens expire after ~1 hour. When using the Graph API fallback (curl/python), you will get HTTP 401 errors when the token expires. **Do NOT ask KL to re-authenticate.** Instead, use the refresh token to get a new access token programmatically:

```python
import json, urllib.request, urllib.parse

# Read current tokens
with open('D:/OneDrive/AI/Qoder Workspace/todoMCP/tokens.json') as f:
    tokens = json.load(f)

# Refresh using OAuth2 refresh_token grant
data = urllib.parse.urlencode({
    'client_id': 'b555eb09-6e28-4d34-b125-a184c9cc9dfa',
    'client_secret': 'REDACTED_SEE_ENVIRONMENT',
    'refresh_token': tokens['refreshToken'],
    'grant_type': 'refresh_token',
    'scope': 'Tasks.Read Tasks.ReadWrite Tasks.Read.Shared Tasks.ReadWrite.Shared'
}).encode()

req = urllib.request.Request(
    'https://login.microsoftonline.com/consumers/oauth2/v2.0/token',
    data=data,
    headers={'Content-Type': 'application/x-www-form-urlencoded'}
)

resp = json.loads(urllib.request.urlopen(req).read())

# Save new tokens (MUST save - the refresh token rotates)
new_tokens = {
    'accessToken': resp['access_token'],
    'refreshToken': resp.get('refresh_token', tokens['refreshToken'])
}
with open('D:/OneDrive/AI/Qoder Workspace/todoMCP/tokens.json', 'w') as f:
    json.dump(new_tokens, f, indent=2)
```

**Key points:**
- The refresh token is long-lived (90 days) but **rotates** -- each refresh returns a new refresh token. Always save it back to `tokens.json`
- If the refresh token itself has expired (90 days without use), then KL must re-authenticate manually via the auth server flow (see `../qoder-fresh-setup-info.md` Section 1, Step 4)
- The MCP server (`todo-index.ts`) handles token refresh automatically when loaded. The above is only needed for the Graph API fallback (curl/python)
- **Always try refreshing before asking KL to re-authenticate.** The manual re-auth flow requires opening a browser and is disruptive
- **Python 3.14 note:** f-strings with dictionary key access (e.g., `f"{d['key']}"`) cause SyntaxError. Use string concatenation or temp variables instead. Also set `sys.stdout.reconfigure(encoding='utf-8')` for Chinese text output

---

## Agent Workflow Rules

1. **Read this file first** when starting any quest in this workspace
2. **Check the Quest Registry** - know which quests exist and what they're doing. Read other AGENTS.md files when your work depends on theirs
3. **Keep your quest status updated** - after significant work, update your quest's status section below
4. **Qoder Admin can modify files in any project folder** - for coordination, git housekeeping, and ecosystem-wide maintenance
5. **Coordinate via To Do** - check Microsoft To Do for assigned tasks and update progress
6. **Use preview for browser links** - when KL needs to visit a URL (localhost dev server, auth pages, etc.), always use `run_preview` to open it in the Qoder preview panel instead of asking KL to open a browser manually
7. **Auto-show created/updated content** - if you create an MD file, update a website, or modify a page that can be previewed, automatically open it in the preview panel so KL can see the result immediately. Exception: skip if you already opened a preview within the last 15 minutes
8. **Report deployment status after every task** - when you finish a task that modifies a deployed app, clearly state whether the changes have been deployed to production or are local-only
9. **Update `new-computer-setup.md` when installing software** - if you install any new software, CLI tool, runtime, or dependency on KL's machine (not per-project npm/pip installs, but system-level tools), update `../new-computer-setup.md` to include it

---

## KreativeLand Portal

The Qoder Admin quest owns the KreativeLand portal website.

**Live URL:** https://kreativeland.com
**Git remote:** https://github.com/Oohyeah99/KreativeLand.git
**Deploy:** `npx vercel --prod --yes` (from this folder)

### Portal Files
| File | Purpose |
|------|---------|
| `index.html` | Main portal page - 6 project cards, hero, about section |
| `style.css` | Full design system - dark theme, Linear-inspired, CSS custom properties |
| `ecosystem-diagram.html` | Interactive ecosystem architecture diagram |

**Tech:** Pure HTML/CSS/JS, no build step. Inter font via Google Fonts. Dark theme with ambient glow effects, card grid layout, responsive.

---

## Quest Status: Qoder Admin

**Quest:** Qoder Admin
**Scope:** KreativeLand portal, cross-quest coordination, Scott/office PC liaison, ecosystem infrastructure
**To Do list:** "Qoder PM (Quest)"

### Current Focus
- Media generation pipeline: ComfyUI + Flux (image) + HunyuanVideo (video) -- **Phase 2 COMPLETE (2026-04-20).** Full REST API live: `POST /api/apps/stories/{id}/generate-media` (triggers background pipeline), `GET /api/apps/stories/{id}/media-status` (poll progress + image URLs), `GET /api/comfyui/status` (health check). Scene prompts via Ollama (gemma4:26b with gemma4:e2b CPU fallback for VRAM contention). Images served at `/media/{story_id}/{index}.png`. New files: `schemas_media.py`, `services/media_gen.py`, `routers/media.py`. LittleReader can now use media generation.
- OpenClaw configuration on office PC (Phase 3 & 4) -- to be done later
- **Ask Little Learner forum pipeline (2026-04-17):**
  - Task 1 (Forum Export): DONE. `scripts/forum/export_posts.py` -- P0 categories exported: 1,316 topics (cat 18 Reading) + 426 topics (cat 29 Music) = 1,742 topics, 11,093 posts in 28 min. Output: `data/forum/{cat_id}_{slug}/topic_{id}.json`. Run `--priority P1` for additional categories
  - Task 2 (Embedding): BUILT. `scripts/forum/embed_chunks.py` -- Chunks posts (~500 tokens), embeds via nomic-embed-text on office 4090, loads to pgvector. **Ready to run after Task 3**
  - Task 3 (pgvector Setup): BUILT. `scripts/forum/setup_pgvector.py` -- Creates `forum_embeddings` table + HNSW index. **Prereq: `sudo apt install postgresql-{ver}-pgvector` on Azure VM**
  - Task 4 (RAG API): BUILT. `backend/routers/forum_knowledge.py` -- `POST /api/forum/ask` with X-LLM-Key auth. Embed question -> vector search -> LLM synthesis (qwen3.5:35b) -> answer + sources. Mounted in main.py (conditional import)
  - **Next steps:** (1) Install pgvector on Azure VM, (2) Run `setup_pgvector.py`, (3) Run `embed_chunks.py`, (4) Deploy updated backend, (5) Test RAG endpoint end-to-end
  - Spec: `../LittleLearner/docs/SPEC-forum-ai-pipeline.md`

### Completed
- **VocabVista VM migration (2026-04-29):** Full VocabVista backend migrated from old Azure VM (20.205.61.171) to new VM (20.255.60.68). Installed PostgreSQL 16, Python 3.12 venv, FastAPI + all dependencies. Restored database (15,011 words, 4 users, 6,786 vocab entries). Created systemd service + nginx reverse proxy with Let's Encrypt SSL. All API endpoints verified working at https://api.kreativeland.com. Kaley's EnglishExamPrep data confirmed accessible.
- **Workspace migration from OneDrive to D:\Projects\ (2026-04-29):** Verified all 7 projects at `D:\Projects\` with working git (external `.git` at `D:\GitRepos\`). Fixed hardcoded OneDrive paths in LittleLearner forum scripts (5 files) and VocabVista database utilities (3 files). Old OneDrive copies not deleted per KL request. All commits pushed to GitHub.
- **New computer migration (2026-04-23):** Fixed all path references from `C:\Users\KL\OneDrive\` to `D:\OneDrive\` across 14 files. Migrated all 7 git repos to `D:\GitRepos\` with external .git pattern. Restructured Quest Registry (Qoder Admin moved to KreativeLand workspace, VocabVista PM moved to VocabVista workspace).
- **LittleLearner workspace set up (2026-04-16):** Renamed Clone LittleLerner.kids to LittleLearner. AGENTS.md created with full To Do rules, shared infra docs, Scott task list. Added to Quest Registry. Portal card added ("Coming Soon" with teal accent).
- **Favicon and app icons created (2026-04-16):** Purple diamond icon matching portal branding. favicon.ico, SVG, PNGs (16/32/180/192/512), web manifest, theme-color meta. Deployed to kreativeland.com.
- **Scott task list set up (2026-04-16):** "For Scott from Qoder Quests" To Do list configured. First task created: Phase 5 ComfyUI installation. AGENTS.md updated so all quests can assign tasks to Scott.
- **GLM-4.7-Flash installed (2026-04-16):** 29.9B params (MoE, 3B active), 19GB, by Zhipu AI. Now available on office PC at `http://100.113.43.60:11434`.
- **Media generation spec written (2026-04-16):** `docs/SPEC-media-generation-pipeline.md` -- Flux for images, HunyuanVideo 1.5 for video, ComfyUI as headless server on office PC. Scott's setup doc updated with Phase 5 (ComfyUI installation).
- **Office PC Ollama access LIVE (2026-04-16):** Tailscale + Ollama verified working from KL's laptop. Endpoint: `http://100.113.43.60:11434`. Available models: gemma4:26b, qwen3.5:35b, gpt-oss:20b, qwen3-coder:30b, qwen3.5:9b, gemma4:e2b, nomic-embed-text (+ glm-4.7-flash once download completes). Any quest can use this for free local LLM inference. Models can be pulled/deleted remotely via `POST /api/pull` and `DELETE /api/delete`.
- Recreated master AGENTS.md with full ecosystem documentation
- Fixed .mcp.json path for new computer
- Re-authenticated Microsoft To Do (OAuth tokens refreshed)
- Verified To Do access: tasks, notes (body content), attachments all working
- Created qoder-fresh-setup-info.md with full setup guide
- Set up Quest Registry and cross-quest coordination protocol
- AGENTS.md files created for all quest workspaces
- Rewrote Scott's office computer setup instructions (Tailscale + Ollama + OpenClaw) -- old docs had wrong commands
- **Infrastructure verification (PM handoff Priority 1-3):**
  - Azure VM LIVE: `api.kreativeland.com` returning `{"status":"ok","version":"2.0.0"}` (IP: 20.205.61.171)
  - reader.kreativeland.com -- 200 OK (LittleReader deployed)
  - vocab.kreativeland.com -- 200 OK (VocabLearner deployed)
  - phonics.kreativeland.com -- 200 OK (PatternPhonics deployed)
  - english.kreativeland.com -- English Prep (英考通), custom domain on Vercel
  - kreativeland.com -- unreachable from Qoder shell (China GFW), KL confirms accessible via browser

### Infrastructure Status (verified 2026-04-29)
| Service | URL | Status |
|---------|-----|--------|
| VocabVista API | api.kreativeland.com | **LIVE (New Azure VM 20.255.60.68, migrated 2026-04-29)** |
| LittleReader | reader.kreativeland.com | LIVE (Vercel) |
| VocabLearner | vocab.kreativeland.com | LIVE (Vercel) |
| PatternPhonics | phonics.kreativeland.com | LIVE (Vercel) |
| EnglishExamPrep | english.kreativeland.com | LIVE (Vercel) |
| ZhongkaoPrep | exam.kreativeland.com | LIVE (Vercel) |
| KreativeLand Portal | kreativeland.com | LIVE (KL confirmed, blocked by GFW from Qoder shell) |

### Azure VM Deployment
- **New VM (production):** 20.255.60.68 (public), Ubuntu 24.04, PostgreSQL 16, nginx reverse proxy, Let's Encrypt SSL
- **Old VM (reference):** 20.205.61.171 (public) / 100.87.60.60 (Tailscale) — still running, still accessible via Tailscale
- **SSH:** `azureuser` with password `?3198yXKb84f` — **no SSH keys on current laptop** (old laptop had them, new GalaxyBook2 does not)
- **Backend path:** `/opt/vocabvista/backend/`, service: `vocabvista.service` (systemd)
- **Deploy method:** Copy code to `/opt/vocabvista/backend/`, restart via `sudo systemctl restart vocabvista.service`
- **Nginx config:** `/etc/nginx/sites-available/api.kreativeland.com` — reverse proxy to 127.0.0.1:8002
- **Database:** PostgreSQL 16 at `localhost:5432`, db=`vocabvista`, user=`vocabvista`
- **GFW note:** Public IP SSH may fail from this laptop (GFW/proxy blocks it). **Use Tailscale IP for the old VM** (`100.87.60.60`) -- confirmed working via paramiko

### Pending
- Office computer: waiting for Scott to complete Tailscale + Ollama setup
- KreativeLand portal frontend updates (as needed)
- Ongoing cross-quest coordination
