# CalmReply Launch Plan — First Users

**Date:** 2026-02-18
**Goal:** Get CalmReply in front of real users, collect feedback, build email list
**Budget:** $0
**Timeline:** 7 days (this week + next Monday)
**Owner:** Adrian Swales (solo)

---

## Before You Read This: Three Hard Questions

**1. What does "working" mean in 7 days?**
You need a number. Without one, you'll either feel great about 12 visitors or feel crushed by 500 who didn't convert. Proposed targets:

| Metric | Target (7 days) | How to track |
|--------|-----------------|--------------|
| Unique visitors | 500+ | Vercel Analytics (free, add today) |
| Responses generated | 50+ | n8n execution count |
| Email signups | 20+ | MailerLite subscriber count |
| Qualitative feedback | 5+ comments | Reddit threads + email replies |

**2. What breaks under load?**
You've tested with 12 automated requests. What happens with 200 real users in 6 hours? Potential failure points:

- **n8n Cloud free tier limits** — check your execution quota NOW. If you're on free, you get 2,500 executions/month. A successful Reddit post could burn through that in a day.
- **GPT-4o-mini rate limits** — you're calling OpenAI via n8n. What's your API spend cap?
- **No error monitoring** — if the webhook goes down at 2am during your Reddit post's peak, you won't know until morning.

**3. What if the Complaint node embarrasses you?**
The blame leakage is a known issue. If someone pastes an aggressive complaint and the tool responds with "I take full responsibility" — that's a bad first impression. Options:
- (a) Ship as-is, accept the risk (current plan)
- (b) Add a disclaimer below responses: "Always review before sending"
- (c) Temporarily hide the Complaint type for launch week (reduces risk, reduces feature set)

**Recommendation:** Option (b). Add a one-liner below the output. Takes 5 minutes. Protects you.

---

## The Strategy: Personal Story + Niche Communities

Your unfair advantage isn't the tech. It's the story. A paramedic with AuDHD who built a tool because he kept freezing on difficult messages — that's the kind of thing Reddit upvotes to the front page.

The plan: **Lead with the story, not the product.**

---

## Phase 1: Pre-Launch (Today — 2-3 hours)

### 1.1 Add Basic Analytics
You're flying blind without visitor tracking. Vercel has free analytics — enable it now.

**Action:** Go to Vercel dashboard → Project Settings → Analytics → Enable Web Analytics.
Then add the script tag to `index.html`:
```html
<script defer src="/_vercel/insights/script.js"></script>
```

### 1.2 Add Response Disclaimer
Below the output area, add a subtle line:
> "Always review and personalise before sending."

This protects you from the Complaint node blame issue and sets the right expectation (assistive tool, not autopilot).

### 1.3 Check n8n Execution Limits
Log into aswales.app.n8n.cloud → Settings → Usage. Note:
- Current plan (free/starter/pro?)
- Monthly execution limit
- Current usage this month

If you're on free tier (2,500/month), a viral Reddit post will kill your backend. Consider upgrading to Starter for launch week ($24/month — your only cost).

### 1.4 Prepare Your Reddit Account
- Check your Reddit account age and karma. Subreddits like r/ADHD and r/SideProject have minimum thresholds.
- If your account is new or low-karma, spend 2-3 days commenting genuinely in r/ADHD and r/freelance before posting.
- Do NOT create a throwaway for this. Use your real account.

### 1.5 Deploy Pre-Launch Updates
After adding analytics + disclaimer:
- Edit `index.html` → copy to `public/index.html` → git push → Vercel auto-deploys.

---

## Phase 2: Launch Week Schedule

### Day 1 (Wednesday) — Primary Launch: r/ADHD

**Why Wednesday:** Highest Reddit engagement is Tue-Thu. Morning US time catches both UK evening and US daytime.

**Post Title:**
> "I built a free response generator for difficult emails — because I kept freezing (AuDHD paramedic here)"

**Post Body (adapt, don't copy verbatim):**

Core beats to hit:
1. **Who you are** — paramedic, AuDHD, UK-based
2. **The problem** — freezing on difficult messages. Drafting 10 versions, sending none.
3. **What you built** — paste the message, pick situation type, choose tone, get a response in 30 seconds
4. **What it's NOT** — not a chatbot, not therapy, no signup, no tracking
5. **The ask** — "I'd love feedback. What breaks? What's missing?"
6. **Link** — in comments, not the post title (avoids spam filters)

**Critical rules:**
- Post the link to calm-reply.com as a COMMENT, not in the post body
- Respond to every single comment for 24 hours
- Don't be defensive about criticism
- Don't mention AI/GPT unless asked directly (focus on the outcome, not the tech)

**What to monitor:**
- Upvotes in first hour (>10 = gaining traction, <5 = might get buried)
- Comments (engagement > upvotes for learning)
- Vercel Analytics for traffic spike
- n8n execution count for actual usage
- MailerLite for signups

### Day 1 (same day) — Secondary: Indie Hackers

Post a "building in public" story on Indie Hackers. Different angle:
- Focus on the BUILD, not just the product
- Share: "2-week build. Single HTML file. n8n + GPT-4o-mini backend. $0 cost."
- Indie Hackers audience cares about the journey, not just the tool

### Day 2 (Thursday) — r/SideProject

**Post Title:**
> "I built a communication tool for people who freeze on difficult emails [free, no signup]"

This sub loves:
- Demo videos (record a 30-second screen capture of using the tool)
- Tech stack breakdowns
- Honest limitations

If you can record a quick screen recording (even with your phone), it'll outperform a text-only post 3:1.

### Day 3 (Friday) — r/freelance + Hacker News

**r/freelance angle:** "Freelancers — stop losing clients because you can't respond to difficult emails"

Position it as a business tool, not a neurodivergent tool. Different audience, different framing. Same product.

**Hacker News (Show HN):**

Title: `Show HN: CalmReply – Free communication assistant for difficult messages`

HN is high-risk, high-reward:
- If it hits the front page: 10K-80K visitors in 24 hours
- If it doesn't: 100-500 visitors
- The audience will scrutinise your tech choices. Be ready to explain why GPT-4o-mini, why n8n, why single HTML file.

### Day 5-7 (Following Week) — Neurodivergent Subs

Stagger across 3 days:
- Monday: r/autism
- Wednesday: r/neurodivergent
- Friday: r/aspergers (if previous posts went well)

**IMPORTANT:** Each post must be unique. Don't copy-paste. Tailor the story to each community's culture.

---

## Phase 3: Parallel Submissions (Low-Effort, Do Today)

Submit to these platforms in one 30-minute batch. They're passive — submit and forget:

| Platform | What to submit | Time needed |
|----------|---------------|-------------|
| BetaList | Product listing | 10 min |
| Product Hunt (upcoming) | Schedule a launch | 15 min |
| OpenHunts | Product listing | 5 min |

Product Hunt tip: Don't launch the same day as Reddit. Save it for the following week when you've incorporated user feedback.

---

## Channel-by-Channel Summary

| Channel | Audience | Angle | Day |
|---------|----------|-------|-----|
| r/ADHD | Neurodivergent (2M) | Personal story: "I freeze on emails" | Wed (Day 1) |
| Indie Hackers | Solo founders | Build-in-public journey | Wed (Day 1) |
| r/SideProject | Makers (622K) | Demo + tech stack | Thu (Day 2) |
| r/freelance | Freelancers (250K+) | Business tool: "stop losing clients" | Fri (Day 3) |
| Hacker News | Developers | Show HN: tech + problem | Fri (Day 3) |
| r/autism | Neurodivergent (477K) | Accessibility tool | Mon (Day 5) |
| r/neurodivergent | Neurodivergent | "Built for us, by us" | Wed (Day 7) |
| Product Hunt | Tech early adopters | Polished launch | Week 2 |

---

## What to Measure

### Daily check (takes 2 minutes):
1. **Vercel Analytics** → unique visitors, top referrers
2. **n8n Cloud** → execution count (are people actually generating responses?)
3. **MailerLite** → new subscribers

### After 7 days, answer these questions:
1. Which channel sent the most traffic?
2. What % of visitors actually generated a response? (visitors vs. executions)
3. Which situation type was used most? (check n8n execution logs if possible)
4. What feedback did you get? (list the top 3 things people said)
5. Did anyone use it more than once? (you can't tell yet — this is a v2 problem)

---

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| n8n hits execution limit | HIGH if Reddit post goes viral | Site appears broken | Check limits today. Upgrade plan if needed. |
| Complaint node generates embarrassing response | MEDIUM | Negative first impression | Add "review before sending" disclaimer |
| Reddit post gets removed by mods | LOW-MEDIUM | Wasted effort | Read sub rules. Don't put link in title. |
| Low engagement (post gets buried) | MEDIUM | Demoralising but not fatal | This is expected for most posts. Don't bet everything on one. |
| Harsh technical criticism on HN | HIGH (it's HN) | Ego hit, but useful feedback | Prepare mentally. Respond gracefully. |
| No one signs up for email list | MEDIUM | Missed long-term value | The CTA is subtle — consider making it more prominent for launch |

---

## Post-Launch: What Comes Next

This plan gets you your first 500 visitors and first real feedback. After that, you'll need to decide:

1. **Is this worth continuing?** (Did people actually use it? Did they come back?)
2. **What do users want next?** (More situation types? Different tones? Save history?)
3. **Is there a business here?** (Can you charge for premium features? What would people pay for?)

Don't answer these now. Launch first, learn, then decide.

---

## Today's Checklist

- [ ] Enable Vercel Analytics
- [ ] Add "Always review and personalise before sending" disclaimer
- [ ] Check n8n execution limits and plan
- [ ] Check Reddit account age/karma
- [ ] Deploy updated frontend
- [ ] Submit to BetaList + OpenHunts
- [ ] Draft r/ADHD post (don't publish yet — sleep on it)

---

*"The best launch strategy is the one you actually execute."*
