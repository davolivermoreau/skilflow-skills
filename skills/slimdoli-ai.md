---
name: slimdoli-ai
description: Full briefing for the Slimdoli personal AI stack — architecture, URLs, credentials, status, GitHub sync, upgrade procedures, and roadmap for the self-hosted Open WebUI + LiteLLM + Claude API setup on OVH VPS
version: v1.1
status: production
llm: claude
owner: 
tags: []
last_updated: April 25, 2026
---

**Content Source**: Slimdoli AI Stack build sessions, March 2026; v0.9.2 upgrade session, April 25, 2026
**Skill Owner**: David Oliver Moreau
**Version**: v1.2 | **Updated**: April 25, 2026

# SLIMDOLI AI STACK

---

## PURPOSE

This skill provides full context for the Slimdoli personal AI stack. Load it at the start of any conversation involving infrastructure, configuration, troubleshooting, or feature additions.

**When to use this skill:**
- Troubleshooting the stack
- Adding new features or integrations
- Onboarding someone to manage the infrastructure
- Planning the Mitel enterprise proposal
- Any task involving the VPS, Open WebUI, LiteLLM, or skills

---

## ARCHITECTURE

```
Your Mac (skill files)
    ↓ git add / commit / push
GitHub (github.com/davolivermoreau/skills-hub) — private repo
    ↓ GitHub Actions triggers automatically
Open WebUI (https://ai.slimdoli.com) — skills synced via API

User (browser/PWA) → https://ai.slimdoli.com → Nginx → Open WebUI → LiteLLM → Claude API
                                                                          ↓
                                                                     PostgreSQL
                                                                  (usage tracking)

Admin dashboard → https://admin.slimdoli.com/ui → Nginx → LiteLLM UI
```

---

## SERVER

| Item | Value |
|---|---|
| Provider | OVH Cloud |
| Plan | VPS-1 |
| OS | Ubuntu 25.10 |
| vCores | 4 |
| RAM | 8GB |
| Storage | 75GB SSD |
| Datacenter | Strasbourg (SBG), France |
| IPv4 | 51.91.126.68 |
| Hostname | vps-e9b78f25.vps.ovh.net |

**SSH access from Mac:**
```bash
ssh -o ServerAliveInterval=60 -i ~/.ssh/id_ed25519 ubuntu@vps-e9b78f25.vps.ovh.net
```

---

## URLS & ACCESS

| Service | URL | Credentials |
|---|---|---|
| Open WebUI | https://ai.slimdoli.com | Admin account |
| LiteLLM Dashboard | https://admin.slimdoli.com/ui | admin / Admin1234 |
| GitHub Skills Repo | https://github.com/davolivermoreau/skills-hub | Private |
| GitHub Actions | https://github.com/davolivermoreau/skills-hub/actions | Monitor sync |
| OVH Control Panel | https://ovhcloud.com | davoliver@gmail.com |
| GoDaddy DNS | https://godaddy.com | davoliver@gmail.com |
| Anthropic Console | https://console.anthropic.com | Personal account |

**DNS:** Both subdomains point to 51.91.126.68 via GoDaddy A records (TTL 600).
**SSL:** Let's Encrypt via Certbot, auto-renewal configured.
**SEO:** X-Robots-Tag noindex/nofollow on all pages. robots.txt Disallow: /

---

## DOCKER COMPOSE STACK

**File location:** `/home/ubuntu/docker-compose.yml`

| Container | Image | Port | Purpose |
|---|---|---|---|
| ubuntu-open-webui-1 | ghcr.io/open-webui/open-webui:main | 3000 | Chat interface |
| ubuntu-litellm-1 | ghcr.io/berriai/litellm:main-stable | 4000 | API proxy + usage tracking |
| ubuntu-postgres-1 | postgres:16 | 5432 | LiteLLM database |

**Current Open WebUI version:** v0.9.2 (as of April 25, 2026)

**Key commands:**
```bash
# Start all
sudo docker compose -f ~/docker-compose.yml up -d

# Stop all
sudo docker compose -f ~/docker-compose.yml down

# Force recreate (after config changes)
sudo docker compose -f ~/docker-compose.yml up -d --force-recreate

# Logs
sudo docker logs ubuntu-open-webui-1 --tail 50
sudo docker logs ubuntu-litellm-1 --tail 50

# Status
sudo docker ps
```

**Docker volumes:**
- `open-webui` (external: true) — Open WebUI data, users, chats, settings
- `ubuntu_postgres_data` — LiteLLM database

⚠️ The open-webui volume is declared as `external: true` in docker-compose.yml to preserve data across stack recreations.

---

## OPEN WEBUI UPGRADE PROCEDURE

⚠️ **Major version upgrades may include database schema migrations.** v0.9.0 (April 2026) introduced schema changes. Always back up before any upgrade that crosses a major version boundary.

⚠️ **Postgres user gotcha:** The default `postgres` role does NOT exist in `ubuntu-postgres-1`. The container is provisioned with `POSTGRES_USER=litellm` / `POSTGRES_DB=litellm`. Always use `-U litellm` for `pg_dumpall`, `psql`, etc. To verify env vars: `sudo docker exec ubuntu-postgres-1 env | grep -i postgres`.

### Step 1 — Check current version
```bash
sudo docker exec ubuntu-open-webui-1 cat /app/package.json | grep version
```

### Step 2 — Backup (mandatory before major version crossings)
```bash
mkdir -p /home/ubuntu/backups

# Open WebUI volume (users, chats, settings, skills metadata)
sudo docker run --rm \
  -v open-webui:/data \
  -v /home/ubuntu/backups:/backup \
  alpine tar czf /backup/open-webui-$(date +%Y%m%d-%H%M).tar.gz -C /data .

# LiteLLM Postgres dump (note: -U litellm, NOT -U postgres)
sudo docker exec ubuntu-postgres-1 pg_dumpall -U litellm > /home/ubuntu/backups/litellm-pg-$(date +%Y%m%d-%H%M).sql

ls -lh /home/ubuntu/backups/
```

Expected sizes: volume backup typically 500MB–2GB depending on chat history; pg_dumpall typically 5–20MB.

### Step 3 — Pull and recreate
```bash
cd /home/ubuntu
sudo docker compose pull open-webui
sudo docker compose up -d --force-recreate open-webui
```

This recreates only the open-webui container. LiteLLM and Postgres remain untouched.

### Step 4 — Watch migration logs
```bash
sudo docker logs -f ubuntu-open-webui-1
```

Wait for either:
- The v0.9.x banner showing the new version, OR
- Steady HTTP 200 responses to `/api/version` and `/api/config`

Press Ctrl+C to exit once stable. Abort if you see `ERROR` or `Traceback` lines.

### Step 5 — Verify
```bash
sudo docker exec ubuntu-open-webui-1 cat /app/package.json | grep version
curl -I https://ai.slimdoli.com
sudo docker ps   # all 3 containers should show healthy
```

Then in browser: log in, confirm chats are present, confirm skills load in Workspace → Tools, send a test message to a Claude model.

### Rollback (if needed)
```bash
# Stop the broken container
sudo docker compose down open-webui

# Restore the volume from backup
sudo docker run --rm \
  -v open-webui:/data \
  -v /home/ubuntu/backups:/backup \
  alpine sh -c "rm -rf /data/* && tar xzf /backup/open-webui-YYYYMMDD-HHMM.tar.gz -C /data"

# Pin to the previous working version in docker-compose.yml
# Change: ghcr.io/open-webui/open-webui:main
# To:     ghcr.io/open-webui/open-webui:v0.8.12  (or whatever the prior working version was)

sudo docker compose up -d open-webui
```

### Backup retention
Keep upgrade backups for at least 1 week after a successful upgrade. Once stable:
```bash
ls -lh /home/ubuntu/backups/
rm /home/ubuntu/backups/open-webui-YYYYMMDD-HHMM.tar.gz
rm /home/ubuntu/backups/litellm-pg-YYYYMMDD-HHMM.sql
```

---

## LITELLM CONFIGURATION

**Config file:** `/home/ubuntu/litellm-config.yaml`

**Models:**
- `claude-sonnet-4-6` → anthropic/claude-sonnet-4-6
- `claude-opus-4-6` → anthropic/claude-opus-4-6
- `claude-haiku-4-5` → anthropic/claude-haiku-4-5-20251001

**Master key:** `sk-litellm-master`

**Postgres credentials** (provisioned via env vars in docker-compose.yml):
- `POSTGRES_USER=litellm`
- `POSTGRES_PASSWORD=litellm2026`
- `POSTGRES_DB=litellm`

**Virtual keys:**

| User | Key | Budget | Models |
|---|---|---|---|
| David (davoliver@gmail.com) | sk-FCicbFgx8Vs5HO_jvQ6u_A | $50/month | All 3 |
| Sharon (sliminny@gmail.com) | sk-fcrMKk4irobEGg79Anea8w | $20/month | Sonnet + Haiku |
| Michael (michael.lamb@mitel.com) | sk-5a-EGNXSuZphcy0V_XM07A | $10/month | Sonnet + Haiku |

**LiteLLM Team:** Personal ($50/month budget)

**Create virtual key via API:**
```bash
curl -X POST http://localhost:4000/key/generate \
  -H "Authorization: Bearer sk-litellm-master" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "USER_ID",
    "key_alias": "user-key",
    "models": ["claude-sonnet-4-6"],
    "max_budget": 10,
    "budget_duration": "30d"
  }'
```

---

## NGINX CONFIGURATION

**Config files:**
- `/etc/nginx/sites-available/ai.slimdoli.com`
- `/etc/nginx/sites-available/admin.slimdoli.com`

```bash
sudo nginx -t                          # Test config
sudo systemctl restart nginx           # Restart
sudo tail -20 /var/log/nginx/error.log # Check errors
sudo certbot renew --dry-run           # Test SSL renewal
```

---

## INTEGRATIONS

### Brave Search
- Admin Panel → Settings → Web Search
- Engine: Brave, free tier (2,000 queries/month)

### Zapier MCP
- Admin Panel → Settings → Integrations → Manage Tool Servers
- URL: `https://mcp.zapier.com/api/v1/connect`
- Auth: Bearer token (personal Zapier account)
- Connected: Gmail, Google Calendar, GA4, Asana

---

## OPEN WEBUI MODELS

| Model | Base | Visibility | Purpose |
|---|---|---|---|
| Claude Sonnet | claude-sonnet-4-6 | Public | General use |
| Claude Opus | claude-opus-4-6 | Public | Complex tasks |
| Claude Haiku | claude-haiku-4-5 | Public | Fast/cheap |
| Mitel Marketing Assistant | Claude Sonnet | Public | Marketing with skills |
| Mitel UX Assistant | Claude Sonnet | Public | UX/wireframes |

---

## SKILLS LIBRARY

**38 skills** in Open WebUI (Workspace → Tools), auto-synced from GitHub.

**Source folders on work Mac:**
- `~/Documents/Skill Hub Staging/Mains Skills Hub`
- `~/Documents/Skill Hub Staging/Personal-Team-Skills`

**GitHub repo:** https://github.com/davolivermoreau/skills-hub (private)

**Auto-sync flow:**
```
Edit skill on Mac → git commit + push → GitHub Actions → Open WebUI API
```

**Sync script:** `~/Documents/Skill Hub Staging/upload-to-openwebui.py`

**Manual sync (if needed):**
```bash
cd ~/Documents/Skill\ Hub\ Staging
source venv/bin/activate
python3 upload-to-openwebui.py
```

**Primary file rule:** picks `skill-name.md`, excludes `skill.md`, versioned files, `.txt`, `.zip`.

**Skill update workflow:**
```bash
# After editing skill files:
cd ~/Documents/Skill\ Hub\ Staging
git add .
git commit -m "Update skill-name v1.x — description"
git push
# Monitor: https://github.com/davolivermoreau/skills-hub/actions
```

**Commit message format:**
- New: `"Add skill-name v1.0 — description"`
- Update: `"Update skill-name v1.x — description"`
- Fix: `"Fix skill-name v1.x — description"`

---

## USERS

| User | Role (Open WebUI) | Role (LiteLLM) |
|---|---|---|
| David (davoliver@gmail.com) | Admin | Admin |
| Sharon (sliminny@gmail.com) | User | Internal (View Only) |
| Michael (michael.lamb@mitel.com) | User | Internal (View Only) |

---

## PWA INSTALLATION

- **Mac Safari:** File → Add to Dock
- **Mac Chrome:** Address bar install icon → Install
- **iPhone:** Safari → Share → Add to Home Screen

---

## COST TRACKING

Current spend (March 2026): ~$0.06 (testing only)
- Per-user tracking via LiteLLM virtual keys
- Dashboard: https://admin.slimdoli.com/ui

---

## PENDING ITEMS

- [ ] BigQuery Python MCP tool — copy service account JSON to VPS, build tool
- [ ] LiteLLM /ui redirect from admin.slimdoli.com root
- [ ] Personal assistant system prompt
- [ ] Docker Compose auto-start on VPS reboot
- [ ] Tailscale — deferred, revisit later
- [ ] Mitel pilot proposal with cost comparison
- [ ] LiteLLM + VPN + SSO for Mitel production
- [ ] Consider pinning Open WebUI to a specific version tag (currently `:main`) for Phase 2 to make updates deliberate

---

## ROADMAP

**Phase 1 (current):** Personal POC — David + Sharon + Michael
**Phase 2:** Mitel marketing pilot — 5-10 power users
**Phase 3:** Mitel rollout — 100 users with VPN + SSO + LiteLLM budgets

**Cost comparison for Mitel proposal:**
- Claude.ai Team (100 users): ~$3,000/month
- This stack (100 users): ~$200-500/month API + ~$20/month VPS

---

## KEY DECISIONS & RATIONALE

- **OVH France datacenter** — data residency in EU, relevant for future GDPR compliance
- **LiteLLM proxy** — enables per-user token tracking and budget limits (essential for enterprise)
- **Open WebUI over OpenClaw** — multi-user, browser-based, MCP support, skills/knowledge system
- **Claude API over Ollama** — VPS too small for quality local models; quality > privacy for this use case
- **external: true on open-webui volume** — prevents data loss when recreating Docker Compose stack
- **Separate personal/work setup** — personal API key, separate from Mitel Claude Enterprise
- **GitHub Actions for sync** — push to main triggers automatic skill upload to Open WebUI
- **`:main` tag on Open WebUI image** — convenient for Phase 1; should pin to specific version tag for Phase 2 onward to avoid surprise upgrades

---

## VERSION HISTORY

### v1.2 (April 25, 2026)
- Added comprehensive Open WebUI Upgrade Procedure section with backup, recreate, verify, and rollback steps
- Documented Postgres user gotcha (`POSTGRES_USER=litellm`, NOT `postgres` — `pg_dumpall -U postgres` will fail with "role does not exist")
- Added permanent v0.9.0 schema migration warning for future major upgrades
- Recorded successful upgrade: Open WebUI v0.8.12 → v0.9.2 (April 25, 2026, ~5 sec downtime, no data loss)
- Added "Current Open WebUI version" reference under Docker Compose Stack
- Added `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` to LiteLLM Configuration
- Added pending item: consider pinning Open WebUI to specific version tag for Phase 2

### v1.1 (March 29, 2026)
- Added GitHub Skills Repo section with auto-sync flow
- Added GitHub Actions sync workflow documentation
- Added git commit workflow and commit message format
- Updated skills count to 38
- Removed GitHub sync from pending items (completed)
- Updated architecture diagram to show GitHub sync layer

### v1.0 (March 29, 2026)
- Initial production release
- Full stack documentation post-build
- Covers architecture, credentials, Docker, Nginx, LiteLLM, integrations, skills, roadmap

---

*Skill ID: slimdoli-ai*
*Version: v1.2*
*Last Updated: April 25, 2026*
*Status: Draft (pending promotion to Production)*
