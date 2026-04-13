---
name: skilflow-ux
description: Complete UX/UI design system for Skilflow — the AI skill management platform. Load this skill for any design work. v1.4 adds loading animation spec, drag and drop spec, role badge colors, and teams/projects component patterns.
version: v1.4
status: production
llm: claude
owner: David Oliver Moreau
tags: [skilflow, ux, design-system, brand, components, layout]
---

# SKILFLOW UX SKILL — Complete Design System v1.4
**Owner:** David Oliver Moreau
**Last Updated:** April 12, 2026
**Status:** Production

---

## CRITICAL RULES — Read Before Generating Any UI

1. **Dark theme default** unless explicitly asked for light — default to OS preference
2. **Inter font always** for UI — Sora for wordmark and headings only
3. **Purple only** (#7c3aed / #a78bfa dark, #6d28d9 light) — teal is RETIRED as brand color
4. **Exact pixel values** — do not approximate spacing, sizes, or border-radius
5. **Interactive HTML always** — never static mockups, never ASCII diagrams
6. **Copy the CSS block exactly** from Section 7 — do not rewrite variables
7. **No creative deviation** unless explicitly asked to explore alternatives
8. **Personality C always** — noise texture + gradient mesh on every prototype
9. **If unsure, ask** — never guess at layout or component placement

---

## 1. BRAND IDENTITY

### Logo Mark — FINAL (C3 at -10°)

```html
<svg width="28" height="28" viewBox="0 0 28 28" xmlns="http://www.w3.org/2000/svg">
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
/* Dark mode */
--brand:       #7c3aed;
--brand-light: #a78bfa;
--brand-dim:   rgba(124,58,237,0.12);

/* Light mode */
--brand:       #7c3aed;
--brand-light: #6d28d9;
--brand-dim:   rgba(109,40,217,0.08);
```

### Semantic (unchanged)
```css
--success: #10b981;  /* production, diff-add */
--warning: #f59e0b;  /* comments, staging */
--danger:  #ef4444;  /* errors, diff-remove */
```

### Dark Theme
```css
--bg: #0a0f1e; --surface: #111827; --surface2: #1e2840;
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
Admin:  bg rgba(245,158,11,0.12)   text #f59e0b
Editor: bg rgba(16,185,129,0.12)   text #10b981
Viewer: bg rgba(148,163,184,0.08)  text #94a3b8
Owner:  bg rgba(124,58,237,0.12)   text #a78bfa
```

---

## 3. PERSONALITY C VISUAL TREATMENT (mandatory on all prototypes)

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
.btn-primary:hover { transform: translateY(-1px); box-shadow: 0 4px 20px rgba(124,58,237,0.45); }
```

### Card Hover
```css
.skill-card:hover { border-color: rgba(124,58,237,0.4); transform: translateY(-2px); }
```

### Active Section (editor)
```css
.sec-block.active { border-left: 2px solid #7c3aed; background: rgba(124,58,237,0.05); animation: glow 2s ease-in-out infinite; }
@keyframes glow { 0%,100%{box-shadow:0 0 12px rgba(124,58,237,0.15)} 50%{box-shadow:0 0 28px rgba(124,58,237,0.4)} }
@keyframes pulse { 0%,100%{opacity:0.4;transform:scale(1)} 50%{opacity:0.8;transform:scale(1.1)} }
```

---

## 4. LOADING ANIMATION SPEC

Full-screen takeover. Used for page load and sync. Clean, honest — no shimmer, no glow.

### Visual
- Background: #090c14 + subtle purple radial gradient
- Skilflow logo + wordmark centered at top
- One step at a time: label (16px 600 #f1f5f9) + sub-label (12px #4b6082)
- Bar: 14px tall, border-radius 7px, bg #1a2236, fill #7c3aed
- Percentage left: 11px 700 #a78bfa
- Step counter right: 11px #4b6082

### Bar behavior
- Fills 0% → 88% over ~1400ms, then resets and loops to next step
- Never hits 100% until API returns
- On API return: jumps to 100%, turns green #10b981
- Done state after 600ms delay

### Done state — conditional
- **Nothing changed:** green checkmark + "All done · 86 unchanged" only
- **Something changed:** checkmark + summary row + two columns:
  - New: "+ skill-name" green, max 5 then "+N more"
  - Updated: "↑ skill-name (v1.1→v1.2)" purple, max 5

### CSS needed
```css
@keyframes fadeIn { from{opacity:0;transform:translateY(6px)} to{opacity:1;transform:translateY(0)} }
@keyframes checkPop { 0%{transform:scale(0)} 60%{transform:scale(1.3)} 100%{transform:scale(1)} }
```

---

## 5. DRAG AND DROP UPLOAD

Drop zone above skill grid. Supports .md files, single or multiple.

### States
1. **Idle** — dashed border (#2d3f5c), purple icon, "Drop .md files here · or click to browse"
2. **Drag over** — purple border (2px solid #7c3aed), pulsing, file count badge
3. **Processing** — each file: icon + name + spinner → green checkmark
4. **Result** — summary line + skill cards with "Open →" button

All imports land as draft. Files without frontmatter → document type.

---

## 6. TYPOGRAPHY

Load: Sora (600/700/800) + Inter (400/500/600/700) + JetBrains Mono (400/500)

| Context | Font | Size | Weight |
|---------|------|------|--------|
| Wordmark | Sora | 15px | 700 |
| Hero | Sora | clamp(44px,6.5vw,80px) | 800 |
| Card title | Sora | 17-20px | 600 |
| Body | Inter | 14-16px | 400 |
| UI labels | Inter | 13px | 500 |
| Editor content | JetBrains Mono | 13px | 400 |
| Badges | Inter | 10px | 700 |

---

## 7. COMPLETE CSS BOILERPLATE

```css
@import url('https://fonts.googleapis.com/css2?family=Sora:wght@600;700;800&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{--brand:#7c3aed;--brand-light:#a78bfa;--brand-dim:rgba(124,58,237,0.12)}
[data-theme="dark"]{--bg:#0a0f1e;--surface:#111827;--surface2:#1e2840;--border:#1e2d45;--border2:#2d3f5c;--text1:#f1f5f9;--text2:#94a3b8;--text3:#4b6082;--brand-light:#a78bfa;--brand-dim:rgba(124,58,237,0.12)}
[data-theme="light"]{--bg:#f8fafc;--surface:#ffffff;--surface2:#f1f5f9;--border:#e2e8f0;--border2:#cbd5e1;--text1:#0f172a;--text2:#1e293b;--text3:#64748b;--brand-light:#6d28d9;--brand-dim:rgba(109,40,217,0.08)}
@keyframes glow{0%,100%{box-shadow:0 0 12px rgba(124,58,237,0.15)}50%{box-shadow:0 0 28px rgba(124,58,237,0.4)}}
@keyframes pulse{0%,100%{opacity:0.4;transform:scale(1)}50%{opacity:0.8;transform:scale(1.1)}}
@keyframes fadeIn{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:translateY(0)}}
@keyframes checkPop{0%{transform:scale(0)}60%{transform:scale(1.3)}100%{transform:scale(1)}}
html{scroll-behavior:smooth}
body{font-family:'Inter',system-ui,sans-serif;background:var(--bg);color:var(--text1);-webkit-font-smoothing:antialiased;position:relative}
body::before{content:'';position:fixed;inset:0;background-image:url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.035'/%3E%3C/svg%3E");pointer-events:none;z-index:0;opacity:0.4}
body::after{content:'';position:fixed;inset:0;background:radial-gradient(ellipse 60% 40% at 20% 10%,rgba(124,58,237,0.07) 0%,transparent 60%),radial-gradient(ellipse 40% 30% at 80% 80%,rgba(109,40,217,0.05) 0%,transparent 60%),radial-gradient(ellipse 50% 50% at 50% 50%,rgba(124,58,237,0.03) 0%,transparent 70%);pointer-events:none;z-index:0}
```

Default to OS preference:
```javascript
document.body.setAttribute('data-theme', window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
```

---

## 8. COMPONENT SPECS

### Nav (52px)
Frosted glass. Left: logo + wordmark. Center (when in team context): team switcher pill. Right: theme toggle + avatar.

### Dashboard Cards
Border-radius 12px. Hover: purple border + glow + translateY(-2px). Grid gap 12-16px.

### Role Badges
```
padding: 2px 8px · border-radius: 4px · font-size: 10px · font-weight: 700 · text-transform: uppercase
```

### Editor Layout
Left sidebar 160-200px · Main editor flex 1 · Right panel 260-300px.
Editor content: JetBrains Mono 13px, line-height 1.8.

### Teams & Projects
- Team cards: 12px radius, hover purple glow + shimmer top line
- Project cards: same pattern, slightly smaller (role badge top right)
- Member rows: avatar initials circle + name + email + role badge + ⋯ menu

---

## 9. DESIGN DECISIONS

| Decision | Chosen | Why |
|----------|--------|-----|
| Brand | Purple #7c3aed | Replaces teal April 12 — distinctive, Stripe-adjacent |
| Personality | Noise + mesh + glows | Depth without layout changes |
| Theme default | OS preference | Respects system setting |
| Editor AI | Text selection tooltip | Natural, Google Docs-style |
| Loading animation | Solid fill bar, one at a time | Clean, honest — no fake shimmer tricks |
| Done state | Conditional detail | Only show changes if something actually changed |
| Drag and drop | Always-visible drop zone above grid | Reduces friction for imports |
| Version field | Display only, never editable | Auto-incremented — manual editing causes collaboration issues |
| Model field | Hidden for documents | Only relevant for skill files loaded into AI |

---

## 10. LIVE URLS

| Service | URL |
|---------|-----|
| App | https://app.skilflow.ai |
| API | https://api.skilflow.ai (8002) |
| MCP | https://mcp.skilflow.ai (8003) |
| Landing | https://skilflow.ai |
| Server | vps-e9b78f25.vps.ovh.net |

---

*skilflow-ux · v1.4 · April 12, 2026 · David Oliver Moreau*