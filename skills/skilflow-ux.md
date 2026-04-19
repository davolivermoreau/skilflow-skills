---
name: skilflow-ux
description: Complete UX/UI design system for Skilflow. v1.5 — adds consumer mode, template store, guided builder, org template library, owner dashboard, pre-auth dark mode, connect page, invite flow patterns.
version: v1.5
status: production
llm: claude
owner: David Oliver Moreau
tags: [skilflow, ux, design-system, brand, components, layout]
last_updated: April 19, 2026
---

# SKILFLOW UX SKILL — Complete Design System v1.5

**Owner:** David Oliver Moreau
**Last Updated:** April 19, 2026
**Status:** Production

---

## CRITICAL RULES

1. **Dark theme default** — OS preference for in-app, forced dark for pre-auth pages
2. **Inter font always** for UI — Sora for wordmark and headings only
3. **Purple only** (#7c3aed / #a78bfa dark, #6d28d9 light) — teal retired as brand color
4. **Exact pixel values** — do not approximate spacing, sizes, border-radius
5. **Interactive HTML always** — never static mockups, never ASCII diagrams
6. **Copy the CSS block exactly** from Section 7 — do not rewrite variables
7. **No creative deviation** unless explicitly asked to explore alternatives
8. **Personality C always** — noise texture + gradient mesh on every prototype
9. **If unsure, ask** — never guess at layout or component placement

---

## 1. BRAND IDENTITY

### Logo Mark — FINAL (C3 at -10°)

```html
<svg width="28" height="28" viewBox="0 0 28 28">
  <rect width="28" height="28" rx="6" fill="#7c3aed"/>
  <rect x="4" y="7" width="20" height="4.5" rx="2.25" fill="white" transform="rotate(-10, 14, 9.25)"/>
  <rect x="4" y="14" width="14" height="4.5" rx="2.25" fill="white" opacity="0.55" transform="rotate(-10, 11, 16.25)"/>
  <rect x="4" y="21" width="8" height="4.5" rx="2.25" fill="white" opacity="0.22" transform="rotate(-10, 8, 23.25)"/>
</svg>
```

**Wordmark — dark bg:** "Skil" = #f1f5f9, "flow" = #a78bfa, Sora 700 15px
**Wordmark — light bg:** "Skil" = #0f172a, "flow" = #6d28d9

---

## 2. COLOR SYSTEM

### Brand (Purple)
```css
--brand:       #7c3aed
--brand-light: #a78bfa  /* dark */
--brand-light: #6d28d9  /* light */
--brand-dim:   rgba(124,58,237,0.12)
```

### Semantic
```css
--success: #10b981   /* production, done states, org templates */
--warning: #f59e0b   /* staging, comments, pending */
--danger:  #ef4444   /* errors, delete, suspended */
```

### Dark Theme
```css
--bg: #090c14; --surface: #111827; --surface2: #1e2840;
--border: #1e2d45; --border2: #2d3f5c;
--text1: #f1f5f9; --text2: #94a3b8; --text3: #4b6082;
```

### Light Theme (WCAG AA)
```css
--bg: #f8fafc; --surface: #ffffff; --surface2: #f1f5f9;
--border: #e2e8f0; --border2: #cbd5e1;
--text1: #0f172a; --text2: #1e293b; --text3: #64748b;
```

### Role Colors
```
Owner:  bg rgba(124,58,237,0.12)  text #a78bfa
Admin:  bg rgba(245,158,11,0.12)  text #f59e0b
Editor: bg rgba(16,185,129,0.12)  text #10b981
Viewer: bg rgba(148,163,184,0.08) text #94a3b8
```

### Template Badge Colors
```
Public template:  bg rgba(124,58,237,0.1)  text #a78bfa  (purple)
Org template:     bg rgba(16,185,129,0.1)  text #10b981  (teal/green)
```

---

## 3. PERSONALITY C (mandatory on all prototypes)

### Noise Texture
```css
body::before {
  content:''; position:fixed; inset:0; pointer-events:none; z-index:0; opacity:0.4;
  background-image:url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.035'/%3E%3C/svg%3E");
}
```

### Gradient Mesh
```css
body::after {
  content:''; position:fixed; inset:0; pointer-events:none; z-index:0;
  background:
    radial-gradient(ellipse 60% 40% at 20% 10%, rgba(124,58,237,0.07) 0%, transparent 60%),
    radial-gradient(ellipse 40% 30% at 80% 80%, rgba(109,40,217,0.05) 0%, transparent 60%),
    radial-gradient(ellipse 50% 50% at 50% 50%, rgba(124,58,237,0.03) 0%, transparent 70%);
}
```

### Nav
```css
.nav {
  background: rgba(9,12,20,0.85);
  backdrop-filter: blur(20px);
  box-shadow: 0 1px 0 rgba(124,58,237,0.2), 0 4px 24px rgba(0,0,0,0.3);
}
```

### Primary Button
```css
.btn-primary {
  background: linear-gradient(135deg, #7c3aed 0%, #6d28d9 100%);
  box-shadow: 0 2px 12px rgba(124,58,237,0.3);
}
.btn-primary:hover { transform: translateY(-1px); }
```

---

## 4. LOADING ANIMATION

Full-screen takeover. Clean, honest — no shimmer.

- Background: #090c14 + purple radial gradient
- Logo + wordmark centered
- One step label (16px 600 #f1f5f9) + sub-label (12px #4b6082)
- Bar: 14px tall, border-radius 7px, bg #1a2236, fill #7c3aed
- Fills 0→88% per step, jumps to 100% green on API return

**Admin pages use red (#ef4444) bar** — signals owner/admin context.

---

## 5. SKELETON LOADERS

Used instead of empty states while data loads. Never show "No X yet" while loading.

```css
@keyframes skeletonPulse {
  0%, 100% { opacity: 0.4; }
  50% { opacity: 0.8; }
}
.skeleton {
  background: #1e2d45;
  border-radius: 4px;
  animation: skeletonPulse 1.5s ease infinite;
}
```

**Stat cards:** 4 gray pulsing blocks, same dimensions as real cards
**List rows:** circle (32px avatar) + two lines (120px + 80px)
**Minimum display:** 400ms (prevent flash)

---

## 6. DRAG AND DROP UPLOAD

Drop zone above skill grid. .md files only.

States: Idle → Drag over (purple border) → Processing (spinner per file) → Result (summary + cards)
All imports land as draft.

---

## 7. TYPOGRAPHY

| Context | Font | Size | Weight |
|---------|------|------|--------|
| Wordmark | Sora | 15px | 700 |
| Hero | Sora | clamp(44px,6.5vw,80px) | 800 |
| Card title | Sora | 17-20px | 600 |
| Body | Inter | 14-16px | 400 |
| UI labels | Inter | 13px | 500 |
| Editor | JetBrains Mono | 13px | 400 |
| Badges | Inter | 10px | 700 |

---

## 8. COMPLETE CSS BOILERPLATE

```css
@import url('https://fonts.googleapis.com/css2?family=Sora:wght@600;700;800&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{--brand:#7c3aed;--brand-light:#a78bfa;--brand-dim:rgba(124,58,237,0.12)}
[data-theme="dark"]{--bg:#090c14;--surface:#111827;--surface2:#1e2840;--border:#1e2d45;--border2:#2d3f5c;--text1:#f1f5f9;--text2:#94a3b8;--text3:#4b6082}
[data-theme="light"]{--bg:#f8fafc;--surface:#ffffff;--surface2:#f1f5f9;--border:#e2e8f0;--border2:#cbd5e1;--text1:#0f172a;--text2:#1e293b;--text3:#64748b}
@keyframes glow{0%,100%{box-shadow:0 0 12px rgba(124,58,237,0.15)}50%{box-shadow:0 0 28px rgba(124,58,237,0.4)}}
@keyframes pulse{0%,100%{opacity:0.4;transform:scale(1)}50%{opacity:0.8;transform:scale(1.1)}}
@keyframes fadeIn{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:translateY(0)}}
@keyframes checkPop{0%{transform:scale(0)}60%{transform:scale(1.3)}100%{transform:scale(1)}}
@keyframes skeletonPulse{0%,100%{opacity:0.4}50%{opacity:0.8}}
body{font-family:'Inter',system-ui,sans-serif;background:var(--bg);color:var(--text1);-webkit-font-smoothing:antialiased}
```

Default to OS preference:
```javascript
document.body.setAttribute('data-theme',
  window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
);
```

Force dark for pre-auth pages (login, signup, join, invite, auth/confirm, welcome):
```javascript
useEffect(() => {
  document.documentElement.setAttribute('data-theme', 'dark');
  return () => document.documentElement.removeAttribute('data-theme');
}, []);
```

---

## 9. COMPONENT SPECS

### Nav (52px)
Frosted glass. Left: logo + wordmark + breadcrumb if in team context. Right: theme toggle + avatar.
Owner badge: red pill "OWNER" next to wordmark on /owner pages.

### Role Selector (pill style — NOT dropdown)
Three pills side by side: Admin · Editor · Viewer
Active: background var(--brand), color #fff, border-radius 7px
Inactive: transparent, color var(--text2), hover text1
Used in: all invite modals, member management

### Skill Cards
Border-radius 12px. Hover: purple border + glow + translateY(-2px).
Status badges: green=production, amber=staging, gray=draft, muted=archived.
Template badge: teal "Org template" or purple "Template" top right.

### Consumer Mode Skill View
Single column, max-width 680px, centered.
Hero CTA card: prominent, purple border, big "Open in Claude" button.
"What this skill does": collapsed expandable.
Footer: "Suggest an edit" + "Leave a comment" + "Used by X members".
No editor, no markdown visible.

### Owner Dashboard (/owner)
Dark bg #090c14 forced. Red loading bar. Skeleton loaders on all sections.
Org list: Manage button → /owner/organizations/:id (full page, not modal).
Action buttons: Suspend (ghost) + Delete (red outline).

### Owner Org Detail (/owner/organizations/:id)
Full page with tabs: Overview / Members / Projects / Invitations / Token usage.
Breadcrumb: Admin › Organizations › [Org name].
Suspend + Delete buttons top right.
Members: avatar + name + email + joined + role + Remove button.
Invitations: pending (Resend + Cancel) + accepted (Dismiss) + Clear all.

---

## 10. TEMPLATE STORE PATTERNS

### Public template store (/templates)
No auth to browse, auth to fork.
Category colors: purple=Communication, green=Marketing, blue=Product, amber=Sales, red=Operations.
Cards: icon + category badge + name + description + fork count + "Use template →".
On fork → guided builder at /templates/:name/build.

### Org template library (/templates/org)
Auth required. Shows templates with template_scope = 'org' for current team.
Teal badge "Org template" + "Made by [author]" + "Used by X members".
Tab switcher: "Skilflow templates" | "[Org name] templates".

### Guided skill builder (/templates/:name/build)
Three-panel layout:
- Left (220px): section navigator — green=done, purple=current, gray=upcoming + progress bar
- Center: section editor — AI explanation card + context prompt + content textarea
- Right (280px): AI suggestion panel (hidden until generated)
- Bottom: Prev/Next navigation + "Finish & save"

AI explanation card: purple left border, Skilflow logo, 2-3 sentences on why this section exists.
Context prompt: textarea "Tell us about your [context]" + "Generate customized version →" button.
Right panel: generated content in monospace + "Apply this version" + "Keep original".

### Share as template flow
Three-dot menu → "Share as template" → modal:
Option A: Share with org (immediate, teal badge)
Option B: Submit to public store (pending review, notifies davoliver@gmail.com)

---

## 11. INVITE & ONBOARDING FLOW

### /join/:slug page (pre-auth, forced dark)
Personal message in quote card (amber left border, italic).
Team name + role + "Accept & create account" button.
Confidentiality agreement checkbox (pending).

### /auth/confirm page (pre-auth, forced dark)
Handles token_hash + type=signup from confirmation email.
Success → checks sessionStorage for pending invite → redirects accordingly.
Error → branded error page with "Request new confirmation" link.

### /welcome/:team_id (pre-auth, forced dark)
Team name + role card + 3 value pillars + CTA "Explore your team's skills →".

### /connect page (auth required)
3-step guided MCP setup:
Step 1: Open Claude.ai settings → numbered sub-steps with chevrons
Step 2: Copy connector URL (https://mcp.skilflow.ai/mcp) with copy button + amber warning
Step 3: Test with "List my Skilflow skills" prompt
Done state: green checkmark + 3 example prompts
Trouble state: 3 common issues with fixes

Step indicator: segmented pill row (purple=active, green=completed, muted=upcoming).

---

## 12. EMAIL TEMPLATES

All dark themed. Consistent structure:
- Background: #090c14
- Card: #111827, border #1e2d45, border-radius 14px
- Purple gradient top accent line (3px)
- Logo mark + wordmark
- Skilflow footer with link to skilflow.ai

Approval email: green accent line (success context).

---

## 13. DESIGN DECISIONS

| Decision | Chosen | Why |
|----------|--------|-----|
| Brand | Purple #7c3aed | Replaces teal — distinctive, Stripe-adjacent |
| Pre-auth theme | Forced dark | Consistent from invite email to product |
| Role selector | Pills not dropdown | Native dropdown breaks dark theme |
| Delete flow | Archive first, type DELETE | Prevent accidental data loss |
| Manage org | Full page not lightbox | Too much content for a modal |
| Template badge | Teal for org, purple for public | Visual distinction between scopes |
| Consumer mode | Single CTA, no editor | 60% of users just want to use skills |
| Loading | Solid bar, honest timing | No fake shimmer |
| Admin loading | Red bar | Signals privileged context |

---

## 14. LIVE URLS

| Service | URL |
|---------|-----|
| App | https://app.skilflow.ai |
| API | https://api.skilflow.ai (8002) |
| MCP | https://mcp.skilflow.ai (8003) |
| Landing | https://skilflow.ai |
| Connect | https://app.skilflow.ai/connect |
| Templates | https://app.skilflow.ai/templates |
| Owner | https://app.skilflow.ai/owner |
| Privacy | https://app.skilflow.ai/privacy |
| Server | vps-e9b78f25.vps.ovh.net |

---

*skilflow-ux · v1.5 · April 19, 2026 · David Oliver Moreau*
