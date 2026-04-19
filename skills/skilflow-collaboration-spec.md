---
name: skilflow-collaboration-spec
description: Collaboration spec — org template library, consumer mode, guided builder, activation system, skill governance. v1.1
version: v1.1
status: production
llm: claude
owner: 
tags: [skilflow, collaboration, spec, roadmap]
last_updated: April 19, 2026
---

# Skilflow Collaboration Spec

**Version:** v1.1
**Last Updated:** April 19, 2026

Full specification for collaboration features — org template library, consumer mode, activation, and skill governance.

---

## 1. USER TIERS

| Tier | % of users | How they use Skilflow |
|------|-----------|----------------------|
| Creator (power user) | ~10% | MCP in Claude, GitHub push, Claude Code |
| Collaborator | ~30% | Fork templates, guided builder, suggest edits |
| Consumer | ~60% | Consumer mode — one click "Open in Claude" |

---

## 2. ORG TEMPLATE LIBRARY

Skills can be promoted to org templates by editors and admins.

### Flow
1. Editor builds a skill they're happy with
2. Three-dot menu → "Share as template"
3. Modal: Share with org / Submit to public store
4. Org members see it in /templates/org with teal "Org template" badge

### Database
```sql
ALTER TABLE skills
ADD COLUMN is_template boolean DEFAULT false,
ADD COLUMN template_scope text DEFAULT null,
-- values: org | public | pending_public | null
ADD COLUMN template_forked_from text DEFAULT null;
```

### MCP tool
```
skilflow_share_as_template(name, scope)
```

Lets power users share from Claude:
"Share my brand-voice skill as an org template"

---

## 3. CONSUMER MODE

Viewers see a simplified skill view — no editor, no markdown.

### Layout
- Skill name + description + status badge + version
- **Big "Open in Claude" CTA** — primary action
- Activation prompt copy block
- "What this skill does" — collapsed expandable
- "Suggest an edit" + "Leave a comment" + "Used by X members"

### Suggest an edit
Lightweight textarea modal → sends notification to skill admins.
No direct edit access for viewers — maintains quality control.

### Role routing
```
role = viewer   → ConsumerSkillView
role = editor+  → full SkillEditor
```

Admins see "Switch to consumer view" preview link.

---

## 4. GUIDED SKILL BUILDER

Launches when forking a template. Walks user through each section.

### Panels
- Left: section navigator (green=done, purple=current, gray=upcoming)
- Center: AI explanation + context prompt + section editor
- Right: AI suggestion panel (slides in on generate)

### AI generation
```
POST /templates/generate-section
{ skill_name, section_title, section_content, user_context }
→ { generated_content }
Model: claude-haiku-4-5-20251001, max_tokens 800
```

System prompt: rewrite the section to fit the user's context, keep same structure.

### Section parsing
Parse skill content by ## headings. Each ## = one step.

### Finish flow
Saves compiled skill as draft → redirects to full editor → toast "Ready to review and publish"

---

## 5. ACTIVATION SYSTEM

### Dashboard checklist
Shows for users < 7 days old. Dismissable for 30 days.

Steps:
1. Create your first skill
2. Connect Claude.ai → /connect
3. Connect GitHub → /settings/github
4. Invite a team member

Progress: "2 of 4 steps complete"

### Activation email sequence

**Day 1 (24h)** — if Claude not connected:
Subject: "Connect Skilflow to Claude in 3 minutes"
CTA: → /connect

**Day 3 (72h)** — if no skill created:
Subject: "Your first skill takes less than 5 minutes"
CTA: → create skill

**Day 7** — if GitHub not connected:
Subject: "Keep your skills versioned with GitHub"
CTA: → /settings/github

### Tracking table
```sql
CREATE TABLE user_activation (
  user_id uuid primary key,
  skill_created boolean default false,
  claude_connected boolean default false,
  github_connected boolean default false,
  team_member_invited boolean default false,
  checklist_dismissed_at timestamptz,
  updated_at timestamptz default now()
);
```

---

## 6. SKILL GOVERNANCE

### Archive-first policy
Skills are never permanently deleted without going through archive first.

Flow:
1. Three-dot → Archive → immediate, toast with 5s Undo
2. Three-dot → Delete permanently (only visible if archived)
3. Modal: type "DELETE" to confirm
4. Removes from all projects + GitHub

### Org template retirement
Admins can retire org templates:
- Template removed from /templates/org
- Existing forks unaffected
- Users who forked get a notification: "The template you forked has been retired"

---

## 7. CROSS-ORG SKILL MOVEMENT

Users with multiple orgs can move skills between them.

"Move to another project" shows all projects grouped by org:
```
SkilFlow
├── Main Project (current) — greyed out
└── [other projects]

SlimDoli
└── SlimDoli AI  ← selectable
```

On move: asks "Also remove from current project?" — user decides.

---

*skilflow-collaboration-spec · v1.1 · April 19, 2026*
