# Reddit Post Coach — Design Document

**Date:** 2026-02-19
**Status:** Approved
**Purpose:** Internal marketing tool for writing, scoring, and scheduling CalmReply Reddit launch posts

---

## What It Is

A single HTML file opened locally in the browser. Guides Adrian through writing Reddit posts for the CalmReply launch campaign by:

1. Showing what to write next (schedule)
2. Providing sub-specific cheat sheets (from intelligence doc)
3. Scoring drafts against each sub's rules (client-side)
4. Generating full drafts from rough notes via AI (n8n + GPT-4o)
5. Saving finished posts to Notion for copy-pasting when on shift

**This is NOT part of CalmReply the product.** It's a separate internal tool with its own n8n workflow.

---

## Architecture

```
Single HTML file (local, open in browser)
    |
    |-- Client-side: schedule, cheat sheets, scoring engine, localStorage memory
    |
    |-- n8n webhook 1: POST /calmreply-reddit-coach
    |     GPT-4o generates draft from notes + sub blueprint + style memory
    |
    |-- n8n webhook 2: POST /calmreply-reddit-save
    |     Creates entry in Notion "Reddit Posts" database
    |     Returns Notion page URL
```

**n8n workflow:** Completely separate from the CalmReply product workflow. Own webhook paths, own OpenAI nodes.

---

## Screen 1: Schedule Dashboard

Shows all 8 posts in chronological order as cards.

**Each card displays:**
- Subreddit name
- Scheduled date and time (GMT)
- Status pill: Upcoming / Draft Started / Ready / Posted
- Post angle (one-line summary)

**Behaviour:**
- Next upcoming post highlighted with "Write this one next"
- Status persisted in localStorage
- Tap a card → navigates to Screen 2 for that sub

**Schedule data (baked into HTML):**

| # | Day | Time (GMT) | Subreddit | Angle |
|---|-----|-----------|-----------|-------|
| 1 | Tue 24 Feb | 11:30 AM | r/ADHD | Question (avoidance stories) |
| 2 | Fri 27 Feb | 8:00 PM | r/AutisticAdults | Question (the freeze) |
| 3 | Mon 2 Mar | 2:30 PM | r/autism | Question (blank page vs words) |
| 4 | Thu 5 Mar | 4:00 PM | r/AutisticWithADHD | AuDHD conflict |
| 5 | Wed 12 Mar | 5:30 PM | r/aspergers | Observation (processing pattern) |
| 6 | Fri 13 Mar | 5:00 PM | r/adhdwomen | Question (shame spiral) |
| 7 | Wed 18 Mar | 5:30 PM | r/neurodivergent | The cost of reply paralysis |
| 8 | Wed 19 Mar | 1:00 PM | r/socialanxiety | Question (the freeze) |

---

## Screen 2: Write & Score

**Header area:**
- Subreddit name, date/time, link strategy reminder
- Word count target (e.g. "Aim for 200-350 words")

**Collapsible Cheat Sheet panel:**
- Key points to hit (numbered beats)
- Proven hooks (title templates)
- Kill signals (what NOT to do)
- DO/DON'T from intelligence doc
- Comment tone guidance

All data baked into JS objects per subreddit.

**Input area:**
- Title input field
- Post body textarea
- Live word count (turns amber outside target range, red if far outside)

**Action buttons:**
- "Score My Draft" → runs client-side scorer, shows Screen 3 inline
- "Do It For Me" → sends to n8n, returns draft into textarea for editing
- "Save to Notion" → sends to n8n, shows "Saved" + Notion link

---

## Screen 3: Score Report (inline below textarea)

Appears after "Score My Draft" is pressed. Checklist format.

**Scoring categories:**

| Category | Weight | Checks |
|----------|--------|--------|
| Title quality | 25% | No promotional language ("free", "tool", "built", "check out"), leads with feeling/question, reasonable length |
| Kill signal scan | 25% | Sub-specific banned phrases via regex. Examples: AI mentions anywhere, "neurodivergent" in r/aspergers, clinical jargon in r/socialanxiety, any URL in post body |
| Word count | 15% | Within sub's target range |
| Structure hits | 20% | Contains question? First-person "I" framing? Paramedic/AuDHD context where required by sub? |
| Link discipline | 15% | No URLs in body, no "calm-reply"/"calm reply" in body |

**Score display:**
- 8-10: "Ready to post" (green)
- 5-7: "Getting there" (amber) + specific fix list
- 1-4: "Needs work" (red) + specific fix list

**Each failed check shows a specific fix suggestion:**
- "Word count is 312 — this sub works best at 50-200. Cut it down."
- "Title contains 'I built' — rewrite to lead with a question or feeling."
- "Post body contains a URL — move it to your first comment."

---

## "Do It For Me" — AI Generation

### n8n Webhook 1: `/calmreply-reddit-coach`

**Request from HTML:**
```json
{
  "subreddit": "autism",
  "userNotes": "rough notes / bullet points",
  "title": "optional - if already written",
  "previousPosts": [
    { "subreddit": "ADHD", "body": "text of finished post 1" },
    { "subreddit": "AutisticAdults", "body": "text of finished post 2" }
  ]
}
```

**n8n flow:**
```
Webhook
  → Switch (on body.subreddit)
    → 8x OpenAI nodes (GPT-4o, one per sub)
      → Set node (format response)
        → Respond to Webhook
```

**Each OpenAI node's system prompt contains:**
1. Sub-specific rules from intelligence doc (blueprint, hooks, kill signals, word count, tone, DO/DON'T)
2. Previous finished posts as style examples (from the request payload)
3. Instruction to match Adrian's tone, phrasing, and personality

**Response:**
```json
{
  "title": "suggested title",
  "body": "full draft post",
  "firstComment": "suggested first comment text, or null if link strategy is wait-to-be-asked",
  "notes": "brief explanation of choices made"
}
```

Draft lands in the textarea for editing. User can re-score, edit further, then save.

### Memory System

- Finished posts stored in **localStorage** as an array of `{ subreddit, body }` objects
- When a post is saved to Notion OR manually marked as done, its text is added to the memory
- All stored posts are sent with every "Do it for me" request as `previousPosts`
- First post has zero examples — AI relies on sub blueprint only
- By post 3-4, the AI has enough examples to match Adrian's voice
- No extra database or webhook needed — localStorage is the cache, Notion is the archive

---

## Save to Notion

### n8n Webhook 2: `/calmreply-reddit-save`

**Request from HTML:**
```json
{
  "subreddit": "autism",
  "scheduledDate": "2026-03-02T14:30:00Z",
  "title": "Does anyone else find the blank page harder than the actual conversation?",
  "body": "the full post text",
  "firstComment": "the first comment text or null",
  "score": 8,
  "linkStrategy": "only-if-asked"
}
```

**n8n flow:**
```
Webhook → Notion: Create Database Item → Respond to Webhook
```

**Notion "Reddit Posts" database (inside Reddit Launch HQ page):**

| Property | Type | Values |
|----------|------|--------|
| Post Title | Title | — |
| Subreddit | Select | r/ADHD, r/autism, r/AutisticAdults, r/AutisticWithADHD, r/aspergers, r/adhdwomen, r/neurodivergent, r/socialanxiety |
| Status | Select | Draft, Ready, Posted |
| Scheduled | Date | from schedule data |
| Post Body | Rich Text | — |
| First Comment | Rich Text | — |
| Link Strategy | Select | first-comment, only-if-asked |
| Score | Number | 1-10 |

**Response:**
```json
{
  "notionUrl": "https://notion.so/page-id"
}
```

HTML shows "Saved" with clickable link to the Notion page.

**Workflow when on shift:** Open Notion on phone → Reddit Posts database → filter by "Ready" → copy post body → paste into Reddit → mark as "Posted."

---

## Data Baked Into HTML

All intelligence data lives as JS objects in the HTML file. No external file dependencies.

**Per-subreddit object contains:**
- `name`, `members`, `scheduledDate`, `scheduledTime`
- `wordCountMin`, `wordCountMax`
- `linkStrategy`: "first-comment" | "only-if-asked"
- `peakDays`, `peakHours`
- `keyPoints`: array of numbered beats to hit
- `provenHooks`: array of title templates
- `killSignals`: array of regex patterns + descriptions
- `doList`: array of DO items
- `dontList`: array of DON'T items
- `commentTone`: string description
- `angle`: one-line summary
- `flair`: suggested flair

---

## Visual Design

Same design language as CalmReply and the reddit-audit.py HTML output:
- Soft blue-grey background, white cards, muted teal accent
- System font stack, 16px+ body text
- Mobile-responsive (usable on phone if needed, designed for desktop)
- CSS custom properties matching CalmReply: `--bg`, `--card`, `--accent`, `--text-light`

---

## What This Tool Does NOT Do

- Post to Reddit (Reddit blocks n8n IPs; you paste manually)
- Replace your voice (you write or heavily edit every post)
- Track Reddit engagement (use F5Bot + manual checking)
- Connect to CalmReply the product in any way
