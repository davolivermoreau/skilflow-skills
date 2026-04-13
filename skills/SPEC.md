---
name: SPEC
description: Skilflow product specification — active reference before every build session. v2.2
version: v1.12
status: production
llm: claude
owner: 
tags: []
---

# SKILFLOW — Product Specification

**Version:** v2.2
**Created:** March 30, 2026
**Author:** David Oliver Moreau
**Last Updated:** April 12, 2026
**Status:** Active — reference before every build session

---

## WHAT CHANGED IN v2.2

- Full-screen loading animation spec added (page load + sync)
- Sync done state — conditional detail (only shown if changes occurred)
- Drag and drop file upload spec added
- Welcome screen + first-time dashboard spec added
- Theme OS preference fix spec added
- Analytics events + GTM installed
- Backend role enforcement shipped
- Workspace isolation shipped
- Version lock shipped
- Email invitations + accept flow shipped

---

## 1. PRODUCT OVERVIEW

Skilflow is a skill management platform for AI assistants. Centralized place to create, version, validate, and publish AI skills across multiple LLM platforms — without touching Git, terminals, or markdown directly.

**The "Sharon test"** — could a non-technical, design-savvy person use this without instructions? If no, simplify it.

**Design Principles:** Non-technical users first · Platform intelligence · Promotion gates · Shared ownership · No single point of failure · LLM-aware authoring · One-click everything

---

## 2. DESIGN SYSTEM

See `skilflow-ux` skill (v1.4) for full design system reference.

**Brand summary:**
- Logo: C3 at -10°, #7c3aed background, white bars
- Wordmark: Sora 700. Dark: #f1f5f9 / #a78bfa. Light: #0f172a / #6d28d9
- Brand: --brand #7c3aed, --brand-light #a78bfa (dark) / #6d28d9 (light)
- Personality C: noise texture + gradient mesh + frosted nav + card glows + gradient button
- Theme: defaults to OS preference (prefers-color-scheme), toggle cycles light/dark/system

---

## 3. ARCHITECTURE

**Stack:** React 18 · FastAPI Python 3.13 · Supabase PostgreSQL · FastMCP

**URLs:**
- app.skilflow.ai — React frontend
- api.skilflow.ai — FastAPI port 8002 (skilflow.service)
- mcp.skilflow.ai — FastMCP port 8003 (skilflow-mcp.service)
- skilflow.ai — Landing page
- Server: OVH VPS vps-e9b78f25.vps.ovh.net (Ubuntu 25.10)

**MCP Server (LIVE April 12, 2026):**
- Transport: Streamable HTTP. Auth: OAuth 2.1 + PKCE + API key fallback
- Tools: list · get · save · update · publish · search
- GitHub write-back: MCP updates commit to GitHub automatically
- Branded connector manifest: /.well-known/mcp-manifest.json
- Logs: `sudo journalctl -u skilflow-mcp -f`

**Backend:** skilflow.service port 8002. Nginx proxy timeouts 300s.

**Universal Markdown Sync:** All .md files. With frontmatter → skill. Without → document. .skilflowignore → skipped.

---

## 4. TEAMS & PROJECTS MODEL

**Concept:**
```
User
├── Personal skills (no project, owner only)
└── Teams (one or many)
    └── Projects (one or many per team)
        └── Skills (one skill can belong to multiple projects)
```

**Roles (per project):**
- Admin — full access, manage members, publish to production, approve changes
- Editor — create/edit skills, publish to staging, comment
- Viewer — read only, comment only

**Key tables:** teams · team_members · projects · project_members · skill_projects · team_invitations

**Email invitations:** Resend integration. Branded HTML email. Accept flow at /invite/:token. Redirects to /welcome/:team_id after first accept.

---

## 5. EDITOR INTERACTION MODEL

**Text selection tooltip:** Select text → Explain / Optimize / AI Editor / Comment.
**Right panel:** Mode switcher tabs. Close button always visible.
**Comment badges:** Amber pill on section headings. Clicking opens Comment tab.
**AI Assistant:** POST /skills/ai-assistant · claude-haiku-4-5-20251001 · max_tokens 1000

---

## 6. LOADING ANIMATION SPEC

### Full-screen loader (page load + sync)

Used in two moments: initial dashboard page load, and when sync runs.

**Visual:**
- Full screen takeover, background #090c14
- Subtle purple radial gradient background
- Skilflow logo + wordmark centered at top
- One step visible at a time: label (16px 600) + sub-label (12px #4b6082)
- Single progress bar: 14px tall, border-radius 7px, #1a2236 bg, #7c3aed fill
- Percentage below bar left: 11px 700 #a78bfa
- Step counter below bar right: 11px #4b6082

**Animation logic:**
- Bar fills 0% → 88% over ~1400ms, then loops to next step
- Never reaches 100% until API actually returns
- When API returns → bar jumps to 100%, turns green #10b981
- Done state appears after 600ms

**Page load steps:**
1. Connecting to your library — Authenticating with GitHub and Supabase
2. Loading your skills — Fetching skills from your workspace
3. Checking for updates — Comparing local and remote versions
4. Getting ready — Preparing your dashboard

**Sync steps:**
1. Authenticating with GitHub — Verifying your personal access token
2. Fetching skill files — Reading .md files from your repository
3. Comparing records — Checking which files changed since last sync
4. Updating changed skills — Writing new content and bumping versions
5. Indexing documents — Tagging .md files without frontmatter
6. Committing write-backs — Pushing MCP changes back to GitHub

### Done state — conditional

**If nothing changed (new=0, updated=0):**
- Green checkmark + "All done · 86 unchanged"
- "View your skills →" button only

**If something changed:**
- Green checkmark + "Sync complete"
- Summary: "3 new · 12 updated · 86 unchanged"
- Two columns (only shown if items exist):
  - New: "+ skill-name" in green, max 5 then "+N more"
  - Updated: "↑ skill-name (v1.1→v1.2)" in purple, max 5
- "View your skills →" button

**Backend sync response must include:**
```json
{
  "new_skills": ["skill-name-1"],
  "updated_skills": [{"name": "skill-name", "old_version": "v1.1", "new_version": "v1.2"}],
  "unchanged_count": 86
}
```

---

## 7. DRAG AND DROP UPLOAD

Drop zone above skill grid, always visible. Supports single or multiple .md files.

**States:**
1. Idle — dashed border, "Drop .md files here to import · or click to browse"
2. Drag over — purple border pulses, shows file count badge
3. Processing — each file shows with spinner → checkmark as frontmatter is read
4. Result — summary line + each imported skill as card with "Open →"

All imported skills land as draft. Files without valid frontmatter import as documents.

---

## 8. ONBOARDING — NEW USER EXPERIENCE

### Welcome page (/welcome/:token)
Shown once after accepting invite. localStorage flag: `skilflow_welcomed_{team_id}`.

- Logo + "You're in" eyebrow
- "Welcome to [Team Name]"
- Inviter context + role card (icon + role name + badge + what they can do)
- 3-pillar explainer: Skills / Projects / Publish
- CTA: "Explore your team's skills →"

### First-time dashboard
- Dismissable welcome banner (localStorage: `skilflow_banner_dismissed_{team_id}`)
- Personal skills empty state with dashed card + CTA
- Team context switcher pill in nav

---

## 9. THEME SYSTEM

Three options: Light / Dark / Use device setting (default).

```javascript
const savedTheme = localStorage.getItem('skilflow-theme');
if (savedTheme && savedTheme !== 'system') {
  document.documentElement.setAttribute('data-theme', savedTheme);
} else {
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
  document.documentElement.setAttribute('data-theme', prefersDark ? 'dark' : 'light');
}
```

OS change listener active when theme = 'system'. Settings → Preferences has 3-option selector.

---

## 10. ANALYTICS & TRACKING

**GTM:** GTM-N4NN73RR installed in React app index.html (head + body noscript).

**Internal analytics table:**
```sql
analytics_events(id, user_id, event_type, skill_name, metadata, created_at)
event_type: skill_viewed | skill_edited | skill_published | mcp_tool_called | user_login
```

**Analytics page (/analytics):** Active users 7d/30d · top skills · MCP call count · recent activity feed.

---

## 11. DATA MODELS

**skills:** id, name, description, content, version, status, llm, owner_id, file_type, source_path, sync_status, updated_at

**comments:** id, skill_name, author_id, body, parent_id, created_at

**skill_versions:** id, skill_name, version, content, change_note, changed_by, source, git_commit, created_at

**teams/projects:** see Section 4

**analytics_events:** id, user_id, event_type, skill_name, metadata, created_at

**skill_reviews (Phase 2):** id, skill_name, proposed_by, proposed_content, base_version, status, review_note, reviewed_by, reviewed_at, created_at

---

## 12. FEATURE LIST

### Shipped (April 12, 2026)
- [x] Auth: email/password, GitHub SSO, Google SSO
- [x] Dashboard: grid + list, filters, search, sort, stats, sync badges
- [x] Editor: markdown, metadata, history, comments, validate, publish
- [x] AI editor: text selection tooltip, mode switcher (Explain/Optimize/AI Editor/Comment)
- [x] Comment count badges + comments wired to API
- [x] Skill creation assistant (Claude Haiku, guided)
- [x] Universal markdown sync + .skilflowignore
- [x] MCP server — OAuth 2.1, 6 tools, GitHub write-back, branded manifest
- [x] Purple rebrand + Personality C
- [x] Theme defaults to OS preference + system option in Settings
- [x] Teams + Projects + Roles (UI + backend)
- [x] Backend role enforcement
- [x] Workspace isolation
- [x] Version lock (409 conflict detection)
- [x] Email invitations + accept flow (Resend)
- [x] Welcome screen + first-time dashboard
- [x] Analytics events + /analytics dashboard
- [x] GTM-N4NN73RR installed

### In progress / next session
- [ ] Full-screen loading animation (page load + sync)
- [ ] Sync done state with conditional detail
- [ ] Drag and drop file upload
- [ ] Backend sync response with skill lists

### Phase 1.5
- [ ] Browser extension (Chrome Manifest V3)
- [ ] Skill conflict detection
- [ ] AI publish notes
- [ ] Skill ownership + approval flow
- [ ] Content moderation middleware
- [ ] Testing suite (pytest + Playwright)

### Phase 2+
- [ ] Skill marketplace
- [ ] Skill analytics + impact scoring
- [ ] Publish project (batch)
- [ ] Skill localization
- [ ] Zapier + Slack integrations

---

## 13. OPEN ITEMS

| Item | Priority | Notes |
|------|----------|-------|
| Loading animation | High | Claude Code prompt ready |
| Drag and drop upload | High | Prototype approved |
| Sync done state detail | High | Backend response change needed |
| Content moderation | Medium | FastAPI middleware |
| Testing Layer 1 | Medium | pytest |
| Dashboard sort fix | Medium | Default updated_at DESC |

---

## 14. ROADMAP

| Priority | Feature | Phase | Status |
|----------|---------|-------|--------|
| 1 | MCP server | 1.5 | ✅ Live |
| 2 | Teams + Projects + Roles | 1.5 | ✅ Shipped |
| 3 | Loading animation | 1.5 | Next session |
| 4 | Browser extension | 1.5 | Planned |
| 5 | Skill conflict detection | 1.5 | Planned |
| 6 | Skill ownership + approval | 2 | Planned |
| 7 | Skill analytics | 2 | Planned |
| 8 | Skill marketplace | 2 | Planned |
| 9 | Team branching | 3 | Planned |
| 10 | Localization | 3 | Planned |

---

## Session Notes — April 12, 2026

**Massive shipping day. Everything below landed in one session.**

**Infrastructure:**
- MCP OAuth 2.1 fully working (3 bug fixes)
- GitHub write-back from MCP — updates commit automatically
- Branded connector manifest + favicon
- Nginx 300s timeout fix

**Design:**
- Purple rebrand (#7c3aed) across app + landing
- Personality C: noise, mesh, glows, frosted nav, gradient button
- Theme defaults to OS preference
- skilflow-ux v1.3 → v1.4 via MCP

**Features shipped:**
- Universal MD sync + .skilflowignore
- Comments fully wired
- Teams + Projects + Roles (full stack)
- Workspace isolation
- Backend role enforcement
- Version lock (409)
- Email invitations + accept flow
- Welcome screen + first-time dashboard
- Analytics events + dashboard
- GTM-N4NN73RR

**Prototypes approved:**
- Full-screen loading animation (solid bar, one step at a time, loops until API returns)
- Drag and drop upload (idle → drag over → processing → result)
- Sync done state (conditional detail only if changes occurred)

**Pending for next session:**
- Implement loading animation (Claude Code prompt ready)
- Implement drag and drop
- Backend sync response with skill name lists
- Dashboard sort fix (updated_at DESC)

---

*SPEC · v2.2 · April 12, 2026 · David Oliver Moreau*