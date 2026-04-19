---
name: SPEC
description: Skilflow product specification — active reference before every build session. v2.3 — April 19, 2026
version: v1.13
status: production
llm: claude
owner: 
tags: []
last_updated: April 19, 2026
---

# SKILFLOW — Product Specification

**Version:** v2.3
**Created:** March 30, 2026
**Author:** David Oliver Moreau
**Status:** Active — reference before every build session
**Last Updated:** April 19, 2026

---

## TABLE OF CONTENTS

1. Product Overview & Positioning
2. Design System
3. Architecture
4. User Tiers & Interaction Model
5. Editor Interaction Model
6. API Contract
7. Data Models
8. MCP Tools
9. Feature List
10. Open Items
11. Roadmap

---

## 1. PRODUCT OVERVIEW & POSITIONING

### What is Skilflow?

Skilflow is a skill management platform for AI assistants. It provides a centralized place to create, version, validate, and publish AI skills (structured markdown instruction files) across multiple LLM platforms — without requiring users to touch Git, terminals, or markdown directly.

### Core Positioning

**Skilflow is infrastructure, not a destination for power users.**

Power users never need to open Skilflow. They work in Claude, Claude Code, and GitHub. Skilflow runs silently in the background — syncing, versioning, making skills available to the team.

Skilflow IS a destination for everyone else — the 90% who need guidance, templates, and a simple way to use what the power users built.

### Three User Tiers

**The creator (power user ~10%)**
- Builds and refines skills from scratch
- Works via MCP in Claude, GitHub push, Claude Code
- Publishes skills to the org library
- Reviews suggestions from others
- Never needs to open the Skilflow UI unless managing the team

**The collaborator (intermediate ~30%)**
- Forks org or public templates
- Customizes with guided builder
- Contributes improvements back via "Suggest an edit"
- Uses the full editor but doesn't build from scratch

**The consumer (~60%)**
- Opens a skill, clicks "Open in Claude", gets results
- Never touches the editor or markdown
- Sees consumer mode: one big CTA, no complexity
- Can suggest edits via lightweight textarea flow

### Design Principles

1. **Infrastructure for power users** — invisible, syncs automatically, no extra steps
2. **Destination for everyone else** — simple, guided, one-click
3. **Non-technical users first** — no terminal, no JSON, no markdown required
4. **Promotion gates** — explicit draft → staging → production workflow
5. **Shared ownership** — skill owners manage content, platform manages deployment
6. **One-click everything** — never more than 3 steps for consumers
7. **Archive before delete** — never permanently remove without explicit confirmation

**The "Sharon test"** — before shipping any feature: could a non-technical, design-savvy person use this without instructions? If no, simplify it.

---

## 2. DESIGN SYSTEM

### Brand

**Domain:** skilflow.ai
**Tagline:** Teach AI once. Let your whole team benefit.

**Logo mark — C3 at -10° (FINAL)**
Three horizontal bars, decreasing width and opacity, each tilted -10°. Mark background: #7c3aed. Bars: white.

**Wordmark:** Sora 700, 15px. Dark bg: "Skil" = #f1f5f9, "flow" = #a78bfa. Light bg: "Skil" = #0f172a, "flow" = #6d28d9.
**Favicon:** #7c3aed background, white bars.

### Brand Colors

```css
--brand:       #7c3aed
--brand-light: #a78bfa  /* dark mode */
--brand-light: #6d28d9  /* light mode */
--brand-dim:   rgba(124,58,237,0.12)
```

Semantic: green (#10b981) production, amber (#f59e0b) staging/comments, red (#ef4444) errors.

### Theme

Defaults to OS preference. Toggle stored in localStorage.
**Pre-auth pages force dark mode** — login, signup, join, invite, auth/confirm, welcome pages all enforce dark regardless of OS preference.

---

## 3. ARCHITECTURE

### Stack

```
Frontend:  React 18, React Router v6, Supabase JS, axios
Backend:   FastAPI, Python 3.13, uvicorn, Anthropic SDK, GitPython
Database:  Supabase (PostgreSQL) — Pro plan
Email:     Resend (noreply@skilflow.ai)
Analytics: GA4 (G-E1L57CNXM2) via GTM (GTM-N4NN73RR)
MCP:       FastMCP Python server

URLs:
  app.skilflow.ai   → React frontend (nginx static)
  api.skilflow.ai   → FastAPI port 8002 (systemd skilflow.service)
  mcp.skilflow.ai   → FastMCP port 8003 (systemd skilflow-mcp.service)
  skilflow.ai       → Landing page (static HTML)
  Server: OVH VPS vps-e9b78f25.vps.ovh.net (Ubuntu)
```

### Infrastructure

- **Supabase:** Pro plan — custom SMTP via Resend enabled
- **Email:** Resend, domain skilflow.ai, SPF + DKIM + DMARC configured in GoDaddy
- **GA4:** Consent Mode v2 — analytics_storage denied by default until user accepts cookie banner
- **GTM:** Initialization - All Pages trigger (Consent Mode compatible)
- **Cookie consent:** Custom banner, localStorage key `skilflow_cookie_consent`

### MCP Server — LIVE

**Transport:** Streamable HTTP
**Auth:** OAuth 2.1 + PKCE, with API key fallback (`sk-skilflow-*`)
**URL:** https://mcp.skilflow.ai/mcp

**Tools (7):**
- `skilflow_list_skills` — list skills, filter by status
- `skilflow_get_skill` — get full content by name
- `skilflow_save_skill` — create new skill
- `skilflow_update_skill` — update existing skill
- `skilflow_publish_skill` — promote to staging/production
- `skilflow_search_skills` — search by keyword
- `skilflow_share_as_template` — share skill as org or public template (pending)

**Ops:**
- Logs: `sudo journalctl -u skilflow-mcp -f`
- Restart: `sudo systemctl restart skilflow-mcp`

### Email Templates (Resend / Supabase)

All dark-themed, branded. Templates in Supabase Auth:
- Confirm signup — uses `{{ .SiteURL }}/auth/confirm?token_hash={{ .TokenHash }}&type=signup`
- Reset password
- Magic link
- Invite user

Custom emails via Resend:
- Owner invite org — branded dark template
- Access request notification — branded dark template
- Post-approval welcome — green accent, quick-start steps
- Feedback notification — type/rating/message to davoliver@gmail.com
- Activation sequence — Day 1 (Claude), Day 3 (first skill), Day 7 (GitHub)

---

## 4. USER TIERS & INTERACTION MODEL

### Role System

| Role | Can do |
|------|--------|
| Owner | Everything + team management + billing |
| Admin | Create/edit/publish skills, manage members, create org templates |
| Editor | Create/edit/publish skills, suggest templates |
| Viewer | View skills in consumer mode, suggest edits, leave comments |

### Consumer Mode (Viewer role)

When a Viewer opens a skill page they see a simplified view:
- Skill name + one-line description
- Status badge + version
- **Big "Open in Claude" CTA** — most prominent element
- Activation prompt copy block
- "What this skill does" — collapsed expandable
- "Suggest an edit" + "Leave a comment" footer
- "Used by X team members" count

No editor, no markdown, no version history visible by default.

### Skill View Mode Toggle

Admins can preview consumer mode via "Switch to consumer view" link.

---

## 5. EDITOR INTERACTION MODEL

### Text Selection Tooltip

Select any text → floating tooltip:
- **Explain** — what this section does
- **Optimize** — AI suggestions with diff
- **AI Editor** — chat to rewrite
- **Comment** — add team note

### Right Panel

Mode tabs: Explain / Optimize / AI Editor / Comment

### Skill Actions (three-dot menu)

1. Edit
2. Move to another project (cross-org supported)
3. Share as template (editor/admin only)
4. Publish / Change status
5. Archive → toast with 5s Undo
6. ─── divider ───
7. Delete permanently (only if archived) → requires typing DELETE

### Archive-first policy

Never permanently delete without:
1. Skill must be archived first
2. Delete permanently option only appears after archiving
3. Confirmation modal requiring user to type "DELETE"
4. Toast with undo for archive action

---

## 6. API CONTRACT

All requests to `https://api.skilflow.ai`. Auth via Bearer token.

### Skills
```
GET/POST/PUT/DELETE /skills
POST /skills/{name}/publish
POST /skills/{name}/validate
GET  /skills/{name}/history
POST /skills/ai-assistant
POST /skills/create/chat
```

### Sync
```
POST /sync/github
POST /sync/skills/{name}/exclude
GET  /sync/settings/excluded-files
```

### Teams & Projects
```
GET/POST /teams
GET/PUT/DELETE /teams/:id
GET/POST /teams/:id/projects
GET/POST /teams/:id/members
POST /teams/:id/members/invite
DELETE /teams/:id/members/:user_id
PUT /projects/:id
GET /projects/all  ← cross-org project list
```

### Invitations
```
POST /owner/invite-org
GET  /join/:slug (public)
POST /join/:slug/accept
GET  /invite/:token (public)
POST /invite/:token/accept
```

### Owner (davoliver@gmail.com only)
```
GET  /owner/stats
GET  /owner/organizations
GET  /owner/organizations/:id
GET  /owner/organizations/:id/members
GET  /owner/organizations/:id/projects
GET  /owner/organizations/:id/invitations
GET  /owner/organizations/:id/tokens
DELETE /owner/organizations/:id/members/:user_id
DELETE /owner/organizations/:id/invitations/:inv_id
POST /owner/organizations/:id/invitations/:inv_id/dismiss
POST /owner/organizations/:id/invitations/:inv_id/resend
PUT  /owner/organizations/:id/tokens/limit
POST /owner/organizations/:id/suspend
DELETE /owner/organizations/:id
GET  /owner/access-requests
POST /owner/access-requests/:id/approve
POST /owner/access-requests/:id/reject
POST /owner/access-requests/:id/dismiss
GET  /owner/feedback
```

### Templates
```
GET  /templates (public)
POST /templates/:name/fork
POST /templates/org/:name/fork
POST /templates/generate-section
GET  /templates/org  ← org-scoped templates
```

### Misc
```
POST /feedback
POST /access-requests (public)
```

---

## 7. DATA MODELS

### skills table
```sql
id, name, content, description, status, version,
file_type,          -- skill | document
source_path,        -- GitHub path
sync_status,        -- new | updated | null
sync_status_at,
is_template,        -- boolean, default false
template_scope,     -- org | public | pending_public | null
template_forked_from, -- source skill name if forked
source,             -- template | manual | github | mcp
created_at, updated_at
```

### teams
```sql
id, name, description, status,  -- active | suspended
created_at
```

### team_members
```sql
team_id, user_id, role,  -- owner | admin | editor | viewer
created_at
```

### team_invitations
```sql
id, team_id, email, role, token, vanity_slug,
custom_message, accepted, expires_at, created_at
```

### access_requests
```sql
id, full_name, email, company, use_case,
status,  -- pending | approved | rejected | dismissed
created_at
```

### feedback
```sql
id, user_id, type,  -- bug | feature | general
rating, message, created_at
```

### template_forks
```sql
id, skill_name, user_id, forked_at
```

### user_activation
```sql
user_id, skill_created, claude_connected,
github_connected, team_member_invited,
checklist_dismissed_at, updated_at
```

### oauth_tokens
```sql
id, user_id, access_token, expires_at, created_at
```

---

## 8. MCP TOOLS

### Current tools (7)

```
skilflow_list_skills(status?)
skilflow_get_skill(name)
skilflow_save_skill(name, content, description, status)
skilflow_update_skill(name, content?, description?, status?)
skilflow_publish_skill(name, target)
skilflow_search_skills(query)
skilflow_share_as_template(name, scope)  ← pending
```

### skilflow_share_as_template

```python
skilflow_share_as_template(name: str, scope: str)
# scope: 'org' | 'public'
# Sets is_template=true, template_scope=scope
# Returns: { skill_name, scope, message }
```

Power users can share templates directly from Claude:
"Share my brand-voice skill as an org template"

---

## 9. FEATURE LIST

### Shipped (April 2026)

**Auth & Onboarding**
- [x] Email/password auth via Supabase
- [x] Email confirmation via /auth/confirm (PKCE flow)
- [x] Vanity invite URLs: app.skilflow.ai/join/:slug
- [x] Custom personal message on invite page
- [x] Welcome screen after invite accept
- [x] Activation checklist on dashboard
- [x] Getting started checklist (dismissable)
- [x] Duplicate email prevention on invites
- [x] Pre-auth pages forced to dark mode
- [x] Confidentiality agreement gate (pending — spec ready)

**Dashboard**
- [x] Grid + list view, filters, search, sort
- [x] Stats: production / staging / draft counts
- [x] Sync badges: New / Updated
- [x] Team context switcher
- [x] Dashboard filters simplified: All / Production / Staging / Draft
- [x] Skill cards with category, version, author, project tag

**Editor**
- [x] Markdown editor with metadata panel
- [x] Version history
- [x] Text selection tooltip (Explain / Optimize / AI Editor / Comment)
- [x] Right panel mode switcher
- [x] Comment count badges on section headings
- [x] Inline project rename
- [x] Archive-first delete flow with typed confirmation
- [x] Cross-org "Move to project" (all orgs shown grouped)

**Teams & Projects**
- [x] Team management (/teams, /teams/:id)
- [x] Project management with skill list
- [x] Member management with role badges
- [x] Invite member with role pill selector (Admin/Editor/Viewer)
- [x] Archived skills hidden from project views
- [x] Consumer mode for Viewer role (pending Claude Code)

**MCP**
- [x] OAuth 2.1 + PKCE fully working
- [x] 6 tools live and callable from Claude.ai
- [x] /connect page — 3-step guided setup at app.skilflow.ai/connect
- [x] Trouble state with common issue fixes

**Owner Dashboard (/owner)**
- [x] Platform stats: orgs, users, skills, API spend
- [x] Access requests with approve/reject/dismiss
- [x] Organizations list with suspend/delete/manage
- [x] Full org detail page at /owner/organizations/:id
- [x] Tabs: Overview / Members / Projects / Invitations / Token usage
- [x] Skeleton loaders on all owner pages
- [x] Owner not added as visible member to managed orgs

**Template Store**
- [x] Public /templates page (browseable without auth)
- [x] 8 starter templates live in production
- [x] Category filters: Communication / Marketing / Product / Sales / Operations
- [x] Fork flow → guided builder
- [x] Guided skill builder — section-by-section with AI explanation
- [x] AI context prompt → generates customized version via Haiku
- [x] Org template library at /templates/org (pending Claude Code)
- [x] "Share as template" in skill menu (pending Claude Code)
- [x] Templates nav link

**Infrastructure**
- [x] GitHub sync with .skilflowignore
- [x] Universal markdown sync (all .md files)
- [x] GA4 + GTM live with Consent Mode v2
- [x] Cookie consent banner (GDPR/CCPA)
- [x] Privacy policy at /privacy
- [x] Feedback widget (purple floating button)
- [x] DNS: SPF + DKIM + DMARC configured
- [x] Resend email sending confirmed working
- [x] Activation email sequence (pending Claude Code)

### Pending

- [ ] skilflow_share_as_template MCP tool
- [ ] Org template library (/templates/org)
- [ ] Consumer mode for Viewer role
- [ ] Activation email sequence (Day 1/3/7)
- [ ] Confidentiality agreement gate on invite accept
- [ ] Comments backend wired to UI
- [ ] Drag-and-drop file upload
- [ ] Content moderation middleware
- [ ] Testing suite (pytest)

---

## 10. OPEN ITEMS

| Item | Priority | Notes |
|------|----------|-------|
| Consumer mode | High | Viewer role sees simplified skill view |
| Org template library | High | Before Pearl demo |
| Share as template | High | MCP tool + UI action |
| Activation emails | High | Day 1/3/7 nudge sequence |
| Confidentiality gate | High | Before any external invite |
| Comments wiring | Medium | Backend exists, frontend pending |
| Drag and drop | Medium | Prototype approved |
| Content moderation | Low | FastAPI middleware |
| Testing Layer 1 | Low | pytest, all endpoints |

---

## 11. ROADMAP

| Priority | Feature | Phase | Status |
|----------|---------|-------|--------|
| 1 | Org template library | 1.5 | Building |
| 2 | Consumer mode | 1.5 | Building |
| 3 | Activation email sequence | 1.5 | Pending |
| 4 | Confidentiality gate | 1.5 | Pending |
| 5 | Google Drive integration | 2 | Planned |
| 6 | SharePoint integration | 2 | Planned |
| 7 | Browser extension | 2 | Planned |
| 8 | Skill analytics + usage tracking | 2 | Planned |
| 9 | Public skill template store v2 (with ratings) | 3 | Planned |
| 10 | Skill marketplace (living frameworks) | 3 | Planned |
| 11 | Team branching + RBAC | 3 | Planned |

---

## SESSION NOTES — April 13-19, 2026

**Alpha launch prep:**
- First external user onboarded (product director, Netflix/Disney+ background)
- Pearl (hellopearl.com) demo scheduled — COO Ben + marketing lead Sébastien
- Email confirmation flow debugged — root cause was localhost:3000 in Supabase Site URL
- Vanity invite URL bug fixed — axios auth interceptor was cancelling public requests
- DNS configured for email deliverability (SPF/DKIM/DMARC)
- Owner dashboard rebuilt as full-page management at /owner/organizations/:id

**Product decisions:**
- Skilflow is infrastructure for power users, destination for everyone else
- Three user tiers: creator / collaborator / consumer
- Consumer mode: single CTA "Open in Claude", no editor visible
- Template store: public generic templates + org-level curated templates
- Guided skill builder: section-by-section with AI context → customized version
- Archive-first policy: never delete without explicit typed confirmation
- Cross-org skill movement supported

**8 starter templates created:**
professional-email-tone, executive-summary, meeting-notes,
content-brief-generator, user-story-writer, feature-prioritization,
cold-outreach-email, sop-writer

**3 Pearl demo skills created:**
pearl-brand-voice, pearl-client-communication, pearl-proposal-writer

---

*Document: SPEC.md · Version: v2.3 · April 19, 2026 · David Oliver Moreau*
