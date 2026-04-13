---
name: slimdoli-ai
description: "Full briefing for the Slimdoli personal AI stack — architecture, URLs, credentials, status, GitHub sync, and roadmap for the self-hosted Open WebUI + LiteLLM + Claude API setup on OVH VPS"
---

**Content Source**: Slimdoli AI Stack build sessions, March 2026
**Skill Owner**: David Oliver Moreau
**Version**: v1.2 | **Updated**: March 30, 2026

# SLIMDOLI AI STACK

---

## PURPOSE

This skill provides full context for the Slimdoli personal AI stack. Load it at the start of any conversation involving infrastructure, configuration, troubleshooting, or feature additions.

**When to use this skill:**
- Troubleshooting the stack
- Adding new features or integrations
- Onboarding someone to manage the infrastructure
- Planning the Mitel enterprise proposal
- Any task involving the VPS, Open WebUI, LiteLLM, skills, or MCP server

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

Mitel Marketing MCP → https://ai.slimdoli.com/mcp/sse → Nginx → FastMCP (port 8000)
                                                                          ↓
                                                                     BigQuery
                                                              (mitel-ga4-to-bigquery)
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

**SSH access:**
```bash
# From personal Mac (SSH key configured)
ssh -o ServerAliveInterval=60 -i ~/.ssh/id_ed25519 ubuntu@vps-e9b78f25.vps.ovh.net

# From work Mac (password auth — no SSH key)
ssh ubuntu@vps-e9b78f25.vps.ovh.net
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
| GCP Console | https://console.cloud.google.com | Project: mitel-ga4-to-bigquery |

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

**Key commands:**
```bash
# Start all
sudo docker compose -f ~/docker-compose.yml up -d

# Stop all
sudo docker compose -f ~/docker-compose.yml down

# Force recreate (after config changes)
sudo docker compose -f ~/docker-compose.yml up -d --force-recreate

# Update Open WebUI
sudo docker compose -f ~/docker-compose.yml pull open-webui
sudo docker compose -f ~/docker-compose.yml up -d --force-recreate open-webui

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

## LITELLM CONFIGURATION

**Config file:** `/home/ubuntu/litellm-config.yaml`

**Models:**
- `claude-sonnet-4-6` → anthropic/claude-sonnet-4-6
- `claude-opus-4-6` → anthropic/claude-opus-4-6
- `claude-haiku-4-5` → anthropic/claude-haiku-4-5-20251001

**Master key:** `sk-litellm-master`

**Open WebUI API:**
- Base URL: `https://ai.slimdoli.com`
- API Key: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6Ijk1ZjgzMWU0LTk0ODQtNDJlYi1iZDZmLTQyM2JjZDQyY2I2MCIsImV4cCI6MTc3NzE5NTI4OSwianRpIjoiNWU0OWJhM2EtMTIwMy00NDIwLWFkNjctM2I0OWVlODA0MmU2In0._w5XcDoRgjva6HqY6is7Iel-NBgRYbq1-RQJ4emzm3E`

**Virtual keys:**

| User | Key | Budget | Models |
|---|---|---|---|
| David (davoliver@gmail.com) | sk-FCicbFgx8Vs5HO_jvQ6u_A | $50/month | All 3 |
| Sharon (sliminny@gmail.com) | sk-fcrMKk4irobEGg79Anea8w | $20/month | Sonnet + Haiku |
| Michael (michael.lamb@mitel.com) | sk-5a-EGNXSuZphcy0V_XM07A | $10/month | Sonnet + Haiku |
| David Mitel (david.oliver@mitel.com) | sk-KhK3vClNa6SPPEqss2ZySw | $10/month | Sonnet + Haiku |

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
- `/etc/nginx/sites-available/api.slimdoli.com`

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
- Type: MCP
- URL: `https://mcp.zapier.com/api/v1/connect`
- Auth: Bearer token (personal Zapier account)
- Connected: Gmail, Google Calendar, GA4, Asana

### Mitel Marketing MCP
- Admin Panel → Settings → Integrations → Manage Tool Servers
- Type: MCP
- URL: `https://ai.slimdoli.com/mcp/sse`
- MCP ID: `mitel_marketing_mcp`
- Auth: None
- Purpose: BigQuery analytics tools for Mitel Marketing team (Open WebUI MCP transport)

### Mitel Analytics REST
- Admin Panel → Settings → Integrations → Manage Tool Servers
- Type: OpenAPI
- URL: `https://api.slimdoli.com`
- OpenAPI spec: `https://api.slimdoli.com/openapi.json`
- Auth: None
- Purpose: Direct REST queries to same BigQuery tools — reliable fallback, also usable outside Open WebUI

**MCP server files:** `/home/ubuntu/mitel-marketing-mcp/`
**Service:** `mitel-mcp` (systemd)

```bash
# MCP server management
sudo systemctl status mitel-mcp
sudo systemctl restart mitel-mcp
sudo journalctl -u mitel-mcp -n 20 --no-pager

# Activate venv for manual testing
cd ~/mitel-marketing-mcp
source venv/bin/activate
python3 server.py
```

**BigQuery connection:**
- Project: `mitel-ga4-to-bigquery`
- Service account: `mitel-marketing-mcp@mitel-ga4-to-bigquery.iam.gserviceaccount.com`
- Credentials: `/home/ubuntu/mitel-marketing-mcp/credentials.json`
- Roles: BigQuery Data Viewer + BigQuery Job User
- Datasets: `analytics_373858783` (raw), `analytics_processed` (pre-aggregated)

**Processed tables:**
| Table | Rows | Purpose |
|---|---|---|
| `daily_page_performance` | ~72,820 | Page metrics by date |
| `daily_utm_performance` | ~42,168 | Campaign/UTM metrics |
| `session_attribution` | ~319,077 | Session-level attribution |
| `form_submissions` | ~4,826 | Lead gen form completions |

**MCP tools:**
- `bigquery_list_tables` — list tables in a dataset
- `bigquery_describe_table` — schema + field descriptions
- `bigquery_run_query` — raw SQL (power users only)
- `bigquery_get_page_performance` — page metrics by date range
- `bigquery_get_utm_performance` — campaign performance
- `bigquery_get_daily_metrics` — top-level daily metrics
- `bigquery_get_form_conversions` — form type breakdown

---

## OPEN WEBUI MODELS

| Model | Base | Visibility | Purpose |
|---|---|---|---|
| Claude Sonnet | claude-sonnet-4-6 | Public | General use |
| Claude Opus | claude-opus-4-6 | Public | Complex tasks |
| Claude Haiku | claude-haiku-4-5 | Public | Fast/cheap |
| Mitel Marketing Assistant | Claude Sonnet | Public | Marketing with skills |
| Mitel UX Assistant | Claude Sonnet | Public | UX/wireframes |
| Mitel Analytics Assistant | Claude Sonnet | Public | Analytics + BigQuery via MCP |

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

- [ ] LiteLLM /ui redirect from admin.slimdoli.com root
- [ ] Personal assistant system prompt
- [ ] Docker Compose auto-start on VPS reboot
- [ ] Tailscale — deferred, revisit later
- [ ] Mitel pilot proposal with cost comparison
- [ ] LiteLLM + VPN + SSO for Mitel production
- [ ] Fix intermittent "Failed to connect to MCP server" warning in Open WebUI
- [ ] Add session_attribution pre-built tool to Mitel Marketing MCP
- [ ] Fix daily_metrics_historical table name reference in server.py

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
- **FastMCP SSE over FastAPI** — Open WebUI requires MCP type (SSE) for tools to appear in model builder; FastAPI OpenAPI spec doesn't integrate as tools
- **MCP on same VPS** — lightweight process, shares nginx, credentials stay server-side

---

## VERSION HISTORY

### v1.2 (March 30, 2026)
- Added Mitel Marketing MCP section under Integrations (full setup details, tools, BigQuery tables)
- Added MCP server to architecture diagram
- Added Mitel Analytics Assistant to Open WebUI models table
- Added GCP Console to URLs & Access
- Updated SSH section to document work Mac (password auth) vs personal Mac (key auth)
- Updated pending items — removed BigQuery MCP (completed), added 3 new items
- Added FastMCP SSE and MCP on same VPS to Key Decisions

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
*Last Updated: March 30, 2026*
*Status: Production*
