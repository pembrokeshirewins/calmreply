# CalmReply vNext — Design Document

**Date:** 2026-02-20
**Status:** Approved
**Source spec:** calmreply-vnext-spec.md
**North Star:** Increase Send Rate (% of sessions where user actually sends)

---

## Product Principle

CalmReply is a **communication safety layer** for people who freeze, shut down, or spiral. Not "AI that writes messages."

---

## Rollout Strategy

3 waves, ordered by impact and dependency:

| Wave | Features | Goal |
|------|----------|------|
| **1 — Core UX** | Progressive disclosure layout, collapsible context field, Delay Shield, Short Reply Only | Improve experience for every existing user immediately |
| **2 — Interpret Mode** | New /webhook/calmreply-interpret, 3-section output, "Draft a reply" transition | New capability: understand messages, not just reply |
| **3 — Behavioural** | Boundary Presets, "Seen Active Elsewhere", "I sent it" tracking, weekly summary | Retention and engagement |

---

## Wave 1 Design (Build Now)

### Default Screen Layout — Progressive Disclosure

**Visible by default:**
- Large textarea (existing)
- "+ Add a note (optional)" collapsible field
- Mode toggle: [Reply Mode] [Interpret Mode]
- CTA button: "Help me respond" / "Help me understand this"

**Behind "Options" expander:**
- Situation dropdown (default: Avoiding)
- Tone pills (Softer / Firmer)
- Boundary dropdown (Wave 3 — hidden for now)
- Short reply only checkbox

**localStorage persistence:**
- Options expander: collapsed/expanded state remembered
- Context field: collapsed/expanded state remembered
- Auto-promote to expanded once user has typed in context field

### Collapsible Context Field

- **Default:** Collapsed, shows "+ Add a note (optional)"
- **Expanded label (Reply mode):** "Add a note (optional)"
- **Expanded label (Interpret mode):** "Anything you want me to consider when interpreting this? (optional)"
- **localStorage key:** `calmreply_context_expanded` (boolean)
- **Auto-promote:** If user ever submits with text in the context field, set `calmreply_context_used` = true, keep expanded by default going forward
- **Never call it "context" in the UI** — too much like work

### Textarea Placeholder Adapts by Mode

- **Reply mode:** "Paste the message you've been avoiding, or describe who you need to reach out to"
- **Interpret mode:** "Paste the message you want help understanding"

### CTA Button Adapts by Mode

- **Reply mode:** "Help me respond"
- **Interpret mode:** "Help me understand this"

### Delay Shield (Prompt Enhancement)

Activates automatically when situation = "Avoiding". No new UI control.

**Prompt addition to Avoiding node:**
```
DELAY SHIELD:
When the user has been avoiding a reply, include a brief opener that:
- Acknowledges the delay in 1 short sentence max
- Never grovels, never stacks apologies
- Creates forward momentum immediately

Use ONE of these intent patterns (adapt to context, never copy verbatim):
- Acknowledge-and-pivot: "Thanks for your patience — here's where I'm at."
- Normalise-the-gap: "I wanted to give this proper thought before replying."
- Skip-it-entirely: If the delay is ambiguous or short, just reply naturally.

HARD RULE: The opener must be ≤15 words. The rest of the reply is the substance.
```

**With Short Reply active:** Delay acknowledgment is 1 clause within the single sentence, not a separate opener.

### Short Reply Only

**UI:** Checkbox in Options expander: "Short reply only"

**Prompt modifier (appended to all situation prompts when active):**
```
SHORT REPLY MODE ACTIVE:
- Maximum 40 words. Hard cap 60 if absolutely unavoidable.
- One paragraph only. No line breaks.
- Must still contain: acknowledgment + clear action or ask.
- No filler. No "I hope this finds you well." No sign-offs unless the input had one.
```

### Mode Toggle (Wave 1 — Frontend Only)

The toggle is visible and functional in Wave 1, but Interpret Mode shows a placeholder message: "Interpret Mode coming soon. For now, use Reply Mode to get a response you can send."

This lets us ship the UI layout without blocking on the Interpret workflow.

---

## Wave 2 Design (Build Later)

### Interpret Mode Backend

- **New n8n workflow:** POST /webhook/calmreply-interpret
- **Model:** GPT-4o (not mini — interpretation needs stronger reasoning)
- **Single node** with structured 3-section output

### Interpret Mode Output Structure

```
What this message is doing
• [2-4 bullets: intent, emotional tone, no diagnosis]

How to read this (pick the one that fits)
• [3 neutral interpretations — not catastrophic, not dismissive]

Suggested next move
→ [One recommended action]

  [ Draft a reply → ]
```

### "Draft a Reply" Transition

- Switches to Reply mode
- Preserves pasted message and any note
- Pre-selects inferred situation type
- Generates immediately (one click from understanding to action)

### Hard Rules for Interpret Prompt

- No therapy language
- No "you should get help"
- No moral judgement
- No diagnosis assumptions
- Probability language for interpretations ("It may be...", "One possibility...")

---

## Wave 3 Design (Build Later)

### Boundary Presets

Dropdown in Options: Gentle boundary / Direct boundary / Clarify expectations / Ask for more time / Close the loop

Output includes: one clear boundary sentence + one next step. No over-explaining.

Visible when situation is: complaint, scope, awkward, avoiding, general. Hidden for ghosted, invoice.

### "Seen Active Elsewhere" Variant

New scenario option in dropdown: "They've been active elsewhere / left me on read"

Generates non-accusatory check-in. No "I saw you online." Assumes message may have been missed. Offers an easy out.

### "I Sent It" Tracking

- Button appears after generation: "I sent it"
- On click: "Nice. That counts." confirmation
- localStorage counters:
  - `calmreply_sent_log`: JSON map of ISO date → count
  - `calmreply_streak`: consecutive days with ≥1 sent
  - `calmreply_last_sent_date`: ISO date
- No message content stored

### Weekly Summary

On first use each week: "Last week: 4 hard replies sent. That's progress."

No leaderboard. No badges. No fireworks.

- `calmreply_weekly_last_shown`: ISO week ID

---

## Backend Summary

| Endpoint | Wave | Purpose |
|----------|------|---------|
| POST /webhook/calmreply | Existing | Reply generation (7 situation nodes) |
| POST /webhook/calmreply-interpret | Wave 2 | Interpret mode (single GPT-4o node) |
| POST /webhook/calmreply-email | Existing | Email capture → MailerLite |
| POST /webhook/calmreply-track | Existing | Analytics → Google Sheets |

---

## What Stays the Same

- Card-based design, colours, font stack
- Email capture section
- Footer ("Built by a paramedic with AuDHD...")
- Coaching note display for Avoiding type
- Response disclaimer
- Feedback widget
- All existing prompt nodes (v5.1) — enhanced, not replaced
- Vercel deployment from /public

---

## Acceptance Criteria (Wave 1)

- [ ] Default view: textarea + mode toggle + CTA only
- [ ] Options expander works, state persisted in localStorage
- [ ] Context field collapsible, label adapts by mode, state persisted
- [ ] Context auto-promotes to expanded after first use
- [ ] Mode toggle visible, Interpret shows "coming soon" placeholder
- [ ] CTA text adapts by mode
- [ ] Textarea placeholder adapts by mode
- [ ] Short Reply checkbox in Options, enforces ≤60 word output
- [ ] Delay Shield active for Avoiding type, opener ≤15 words
- [ ] Delay Shield + Short Reply interact correctly (woven, not prepended)
- [ ] No regressions: all 7 situation types still work
- [ ] Mobile usable at 375px
- [ ] Copy to `public/index.html` and deploy
