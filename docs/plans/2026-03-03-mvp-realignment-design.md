# CalmReply MVP Realignment — Personal-First Single Prompt

**Date:** 2026-03-03
**Status:** Approved
**Evidence:** 88 F5Bot Reddit triage executions, 24 high-relevance posts, ntfy push notification analysis
**North Star:** Match the tool to the pain people actually express on Reddit

---

## Problem

CalmReply's 7 situation types (Avoiding, Complaint, Ghosted, Invoice, Scope, Awkward, General) are 100% business-focused. F5Bot intelligence data from 88 Reddit alerts shows:

| Trigger context | Count (high-relevance) | % |
|----------------|----------------------|---|
| Relationship (partner, dating) | 10 | 42% |
| Social (friends, acquaintances) | 7 | 29% |
| Family (parents, in-laws) | 3 | 12.5% |
| Work | 3 | 12.5% |

**75% of the freeze pain is personal.** The product only addresses the 12.5%.

Additionally, 42% of high-relevance posts got freeze archetype `none` — the AI couldn't match them to any existing CalmReply category. The pain doesn't fit neat buckets.

### Highest-pain scenarios from real Reddit data

1. "How do I respond to my mom when she sends guilt-tripping texts" — Pain: 7, Shame: 6 (r/Advice, r/raisedbynarcissists)
2. "Is my relationship romantically dead?" — Pain: 7 (r/relationship_advice)
3. "Dating with autistic burnout + relational PTSD" — Pain: 7 (r/autism)
4. "I told a friend I saw her husband with another woman" — Pain: 7 (r/Marriage)
5. "My friend wants to know what I think of her new man" — Pain: 6 (r/Advice)
6. "How do I respond to my mother-in-law's boyfriend on Zoom" — Pain: 6 (r/Cantonese)
7. "I really need lessons" (scared to respond at work) — Pain: 6 (r/workplace_bullying)
8. "I don't know if I'm weak or exhausted from my job" — Pain: 6 (r/offmychest)

---

## Design

### 1. Frontend Changes

**Remove the situation dropdown entirely.** People in freeze can't self-categorise. The AI reads the message and figures it out.

**Options expander now contains:**
- Three tone pills: **Softer** (default) / **Firmer** / **Protective**
- Short Reply Only checkbox

**Tone definitions:**
- **Softer** — warm, conciliatory, bridges the gap. For when you want to maintain the relationship.
- **Firmer** — direct, boundaried, no padding. For when you need to be clear.
- **Protective** — firm but compassionate. Holds your ground without escalating. For manipulative, guilt-tripping, or toxic dynamics where you need to protect yourself.

**Positioning (personal-first):**
- Title: "CalmReply — Handles the conversations you've been avoiding" (stays)
- Tagline: "Handles the conversations you've been avoiding." (stays)
- Textarea placeholder: "Paste the message you're stuck on, or describe what you need to say"
- Context field label: "Anything else I should know? (optional)"
- CTA: "Help me respond" (stays)

**Output area:**
- Reply text (copyable)
- Separator line `---`
- Coaching note on every response (not copied): names the pattern and explains the freeze

**Everything else stays:** progressive disclosure, mode toggle (Interpret Mode placeholder), copy button, feedback widget, email capture, footer.

### 2. Backend / n8n Architecture

**Current:** Webhook → Switch (on situationType) → 7 OpenAI nodes → Set → Respond

**New:** Webhook → 1 OpenAI node → Set → Respond

The Switch node and all 7 situation-specific OpenAI nodes get replaced by a single node.

**Webhook payload simplifies:**
```json
// Old
{ "situationType": "avoiding", "userMessage": "...", "tone": "softer" }

// New
{ "userMessage": "...", "tone": "softer" }
```

`situationType` removed from the frontend. `tone` now accepts: `softer`, `firmer`, `protective`.

**Coaching note format** (same separator as current Avoiding type):
```
[reply text]
---
[coaching note]
```

The frontend already splits on `\n---\n` and excludes the coaching note from copy. The only change: this now happens for every response, not just Avoiding.

**Delay Shield** becomes part of the single prompt's logic. If the AI detects avoidance (from message content or context field), it applies the ≤15-word opener rule.

### 3. The Prompt

One comprehensive prompt replacing all 7. Structure:

**Role and mission:**
- Communication safety layer for people who freeze, shut down, or spiral
- User pastes a message they received (or describes a situation). Draft a reply they can copy and send.
- Personal-first: family, relationships, friends, work, social — any difficult conversation

**WHO IS WHO rule (from v5):**
- The user is the RECIPIENT. You draft FROM the user BACK TO the sender.

**Auto-detection (replaces the Switch):**
- Silently analyse: who sent it (parent, partner, friend, colleague, stranger, ex), the relationship dynamic (manipulative, loving, professional, passive-aggressive, controlling), communication channel, emotional temperature, power imbalance

**Tone modifier (injected from frontend):**
- Softer: warm, bridges the gap, ≤200 words
- Firmer: direct, boundaried, ≤100 words hard limit
- Protective: firm but compassionate, names boundary without attacking, ≤150 words

**Core rules (carried and expanded from v5):**
- British English throughout
- No placeholders — copy and send without editing
- No grovelling, no over-apologising
- Register matching (mirror their greeting style)
- Blame guard: don't accept fault you don't know about
- **Boundary guard** (new): for Protective tone, include one clear boundary statement. Don't over-explain or justify.
- **Manipulation detection** (new): if the message uses guilt, gaslighting, emotional blackmail, or DARVO, the reply doesn't engage with the manipulation. Responds to the legitimate content only.

**Delay Shield (folded in):**
- If context suggests avoidance: ≤15-word opener, no stacking apologies
- With Short Reply active: delay acknowledgment woven into the single sentence

**Short Reply mode (from v5.2):**
- ≤40 words, hard cap 60. One paragraph. Acknowledgment + clear action.

**Coaching note (new — universal):**
After `\n---\n`, write a coaching note (2-4 sentences) that:
- Names the communication pattern (guilt-tripping, boundary testing, stonewalling, emotional dumping, passive aggression, etc.)
- Briefly explains why this message might trigger freeze/avoidance
- Validates without diagnosing. No therapy language, no "you should get help", no moral judgement
- Example: "This message shifts responsibility for their feelings onto you — that's guilt-tripping. It's designed to make 'no' feel cruel, which is why you froze. Your reply holds the boundary without taking the bait."

### 4. What Gets Cut

**Situation types removed (all 7):**
- Avoiding → folded into auto-detection + Delay Shield logic
- Complaint / De-escalation → handled by auto-detection
- Ghosted Follow-Up → handled by auto-detection
- Invoice Chase → 0 matches in 88 Reddit alerts
- Scope Pushback → 0 matches in 88 Reddit alerts
- Awkward or Difficult Message → auto-detection handles this
- General Communication Help → the whole tool is now this

**n8n nodes removed:** Switch node + 7 OpenAI nodes
**n8n nodes added:** 1 OpenAI node with unified prompt

**Frontend elements removed:** Situation dropdown, `situationType` in webhook payload

**NOT cut:** Context field, Short Reply checkbox, tone selector (expanded to 3), mode toggle, copy button, feedback widget, email capture, disclaimer, progressive disclosure layout.

**Risk mitigation:** The 7 existing prompts have rules worth preserving (blame guard, register matching, 3-case detection for ghosted). These aren't lost — they're folded into the single prompt's rule set.

### 5. Success Criteria

**Coaching note relevance:**
- Pastes a guilt-tripping text from parent → coaching note names it ("guilt-tripping", "emotional manipulation"), not generic ("this is a difficult message")
- Test against real Reddit scenarios from ntfy alerts

**Protective tone works:**
- Reply in Protective tone to a manipulative message holds a boundary without attacking, submitting, or over-explaining
- Sendable without feeling cruel OR like a pushover

**Auto-detection accuracy:**
- Without a dropdown, AI correctly distinguishes: family manipulation, relationship conflict, friend honesty dilemma, workplace intimidation, professional request
- Test with the 8 highest-pain Reddit scenarios

**No regression on professional use cases:**
- Client complaint or late invoice still gets a usable reply
- Handled as part of auto-detection

**Metrics (if measurable):**
- Copy rate stays same or improves
- Fewer thumbs-down on personal scenarios
- F5Bot intelligence: `suggested_action` aligns with what the tool now does

---

## What Stays the Same

- Single HTML file, no frameworks
- n8n Cloud backend, GPT-4o-mini
- Vercel hosting from /public
- Progressive disclosure layout (Wave 1)
- Interpret Mode placeholder (Wave 2 future)
- Card-based design, colours, font stack
- Email capture, footer, disclaimer
- `calmreply_` localStorage prefix
- `[SHORT REPLY MODE]` and `[USER CONTEXT: ...]` client-side markers
