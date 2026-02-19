# Reddit Post Coach — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single HTML file that schedules, coaches, scores, and saves Reddit posts for CalmReply's launch campaign.

**Architecture:** Single HTML file with inline CSS/JS, talking to two n8n webhooks (one for GPT-4o draft generation, one for Notion save). All subreddit intelligence data baked into JS objects. Scoring runs client-side. State persisted in localStorage.

**Tech Stack:** HTML/CSS/JS (no frameworks), n8n Cloud (webhooks + OpenAI + Notion nodes), GPT-4o, Notion API via n8n.

**Design doc:** `docs/plans/2026-02-19-reddit-post-coach-design.md`

---

## Task 1: HTML Skeleton + CSS + Screen Switching

**Files:**
- Create: `reddit-post-coach.html`

**Step 1: Create the HTML file with all CSS and screen-switching JS**

The file needs three screen divs (`#schedule`, `#writer`, `#score-report`), a shared header, and CSS matching CalmReply's design language. Only one screen is visible at a time.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Reddit Post Coach — CalmReply</title>
  <style>
    /* Use CalmReply's exact CSS custom properties */
    :root {
      --bg: #f0f4f7;
      --card: #ffffff;
      --accent: #5a9ea6;
      --accent-hover: #4a8a91;
      --text: #2c3e50;
      --text-light: #6b7c8a;
      --border: #dce4ea;
      --success: #27ae60;
      --warning: #e67e22;
      --error: #c0625a;
      --radius: 12px;
      --shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
    }

    /* Same base styles as CalmReply index.html */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    html { font-size: 16px; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
      background: var(--bg); color: var(--text); line-height: 1.6;
      min-height: 100vh; padding: 0 16px 48px;
    }

    /* Header */
    header { text-align: center; padding: 32px 0 8px; }
    header h1 { font-size: 1.5rem; font-weight: 700; letter-spacing: -0.02em; }
    header p { font-size: 0.9rem; color: var(--text-light); margin-top: 4px; }

    /* Screen visibility */
    .screen { display: none; }
    .screen.active { display: block; }

    /* Card (reused everywhere) */
    .card {
      background: var(--card); border-radius: var(--radius); box-shadow: var(--shadow);
      max-width: 640px; margin: 16px auto; padding: 24px;
    }
  </style>
</head>
<body>
  <header>
    <h1>Reddit Post Coach</h1>
    <p>CalmReply launch campaign</p>
  </header>

  <div id="schedule" class="screen active"></div>
  <div id="writer" class="screen"></div>

  <script>
    function showScreen(id) {
      document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
      document.getElementById(id).classList.add('active');
      window.scrollTo(0, 0);
    }
  </script>
</body>
</html>
```

**Step 2: Open in browser to verify**

Open `reddit-post-coach.html` in Chrome. Should see the header "Reddit Post Coach" on a blue-grey background. No errors in console.

**Step 3: Commit**

```bash
git add reddit-post-coach.html
git commit -m "feat: reddit post coach — HTML skeleton with CSS and screen switching"
```

---

## Task 2: Subreddit Intelligence Data

**Files:**
- Modify: `reddit-post-coach.html`

**Step 1: Add the SUBS data object inside the `<script>` tag**

This is the single source of truth for everything the tool knows. All data comes from `docs/planning/reddit-subreddit-intelligence.md` and `reddit-draft-posts.md`.

```javascript
const SUBS = [
  {
    id: 'ADHD',
    name: 'r/ADHD',
    members: '2.2M',
    scheduledDate: '2026-02-24T11:30:00Z',
    angle: 'Question (avoidance stories)',
    flair: 'Seeking Empathy',
    wordCountMin: 200, wordCountMax: 400,
    linkStrategy: 'first-comment',
    keyPoints: [
      'Open with a relatable ADHD avoidance question — invite stories, not advice',
      'Paramedic context — the novelty/purpose match, the everything-else freeze',
      'Specific funny examples of procrastination from your life',
      'The ghosting dread — physical anxiety description (thump in the chest)',
      'Punchline: your most unhinged avoidance was building a tool to avoid the avoidance'
    ],
    provenHooks: [
      '"What\'s the most unhinged way you\'ve avoided a task"',
      '"I [embarrassing ADHD thing] and here\'s what happened"',
      '"Does anyone else [specific struggle]?"'
    ],
    killSignals: [
      { pattern: /\b(AI|GPT|artificial intelligence|chatbot|language model)\b/i, message: 'AI mention detected — immediate suspicion in ND subs' },
      { pattern: /\b(check out|try this|download|sign up|visit)\b/i, message: 'Promotional language detected' },
      { pattern: /https?:\/\/|www\./i, message: 'URL in post body — links go in first comment only' },
      { pattern: /calm[\s-]?reply/i, message: 'Product name in post body — let them ask' }
    ],
    doList: [
      'Follow r/adhdwomen patterns but gender-neutral framing',
      'Personal story, first person, vulnerability > polish',
      'Show the struggle before any hint of a win',
      'End with a question that invites sharing'
    ],
    dontList: [
      'Don\'t mention building a tool in the title',
      'Don\'t use marketing language',
      'Don\'t post generic advice without lived experience',
      'Expect high removal risk — strict moderation (2.2M members)'
    ],
    commentTone: 'Casual, personal, empathetic. Match the energy of the community.',
    structureChecks: {
      requiresQuestion: true,
      requiresFirstPerson: true,
      requiresParamedic: false,
      requiresAuDHD: false
    }
  },
  {
    id: 'AutisticAdults',
    name: 'r/AutisticAdults',
    members: '~102K',
    scheduledDate: '2026-02-27T20:00:00Z',
    angle: 'Question (the freeze)',
    flair: '',
    wordCountMin: 50, wordCountMax: 200,
    linkStrategy: 'only-if-asked',
    keyPoints: [
      'The freeze moment — specific adult communication version: work emails, invoices, complaints',
      'Paramedic contrast — one sentence only: can handle emergencies, can\'t handle a passive-aggressive email',
      'The blank page insight — the freeze breaks when you have something to react to. Describe as a realisation, not a product pitch',
      'End with a question: "Does anyone else get this?" or "What do you do when this happens?"'
    ],
    provenHooks: [
      '"Does anyone else open a message, go completely blank, and then avoid it for a week?"',
      '"The freeze isn\'t about not knowing what to say"'
    ],
    killSignals: [
      { pattern: /\b(AI|GPT|artificial intelligence|chatbot|language model)\b/i, message: 'AI mention detected' },
      { pattern: /\b(check out|try this|download|sign up|visit)\b/i, message: 'Promotional language detected' },
      { pattern: /https?:\/\/|www\./i, message: 'URL in post body — links go in comment only if asked' },
      { pattern: /calm[\s-]?reply/i, message: 'Product name in post body' },
      { pattern: /\b(I built|I made|I created|free tool|my tool)\b/i, message: 'Tool/building language — don\'t mention building anything in the title or body' }
    ],
    doList: [
      'Keep SHORT — 50-200 words, punchy, no long stories',
      'Frame around adult communication challenges (workplace, business, emails)',
      'Describe the experience, not the tool',
      'End with a question'
    ],
    dontList: [
      'Don\'t mention building anything in the title',
      'Don\'t go over 200 words',
      'Don\'t describe how the tool works — just describe the experience'
    ],
    commentTone: 'Practical, matter-of-fact, supportive.',
    structureChecks: {
      requiresQuestion: true,
      requiresFirstPerson: true,
      requiresParamedic: false,
      requiresAuDHD: false
    }
  },
  {
    id: 'autism',
    name: 'r/autism',
    members: '~492K',
    scheduledDate: '2026-03-02T14:30:00Z',
    angle: 'Question (blank page vs words)',
    flair: 'Discussion',
    wordCountMin: 200, wordCountMax: 350,
    linkStrategy: 'only-if-asked',
    keyPoints: [
      'Open with a question about the autistic processing experience — cognitive load of processing tone, intent, and response simultaneously',
      'Paramedic context — clinical emergencies are protocol-driven (clear rules). Written communication has no protocol. Frame as a processing problem, not emotional',
      'The blank page vs editing distinction — "I can edit a draft in minutes but I can\'t start from nothing. The freeze is about initiation, not words"',
      'Mention masking exhaustion — the freeze is worse after a full day of masking',
      'What do others do? — genuine question, this sub values shared coping strategies',
      'Don\'t describe the tool in the post — describe the insight and let them ask'
    ],
    provenHooks: [
      '"Does anyone else find the blank page harder than the actual conversation?"',
      '"Reply paralysis: is it the words or the starting that\'s the problem?"'
    ],
    killSignals: [
      { pattern: /\b(AI|GPT|artificial intelligence|chatbot|language model)\b/i, message: 'AI mention detected' },
      { pattern: /\b(check out|try this|download|sign up|visit)\b/i, message: 'Promotional language detected' },
      { pattern: /https?:\/\/|www\./i, message: 'URL in post body' },
      { pattern: /calm[\s-]?reply/i, message: 'Product name in post body' },
      { pattern: /\bneurodivergent\b/i, message: 'Say "autistic" not "neurodivergent" — r/autism cares about specificity' },
      { pattern: /\b(I built|I made|I created|free tool|my tool)\b/i, message: 'Tool/building language detected' }
    ],
    doList: [
      'Say "autistic" not "neurodivergent" — this sub cares about specificity',
      'Frame around the specific autistic processing experience (not just anxiety)',
      'Reference masking exhaustion',
      'Use appropriate flair',
      'Acknowledge the range of support needs'
    ],
    dontList: [
      'Don\'t be preachy or prescriptive',
      'Don\'t oversimplify the experience',
      'Don\'t ignore that some autistic people have the opposite problem (impulsive replies)',
      'Don\'t treat autism as a quirky personality trait'
    ],
    commentTone: 'Mix of intellectual engagement and personal sharing. More diverse than other subs.',
    structureChecks: {
      requiresQuestion: true,
      requiresFirstPerson: true,
      requiresParamedic: true,
      requiresAuDHD: false
    }
  },
  {
    id: 'AutisticWithADHD',
    name: 'r/AutisticWithADHD',
    members: '~81K',
    scheduledDate: '2026-03-05T16:00:00Z',
    angle: 'The AuDHD communication conflict',
    flair: 'General Discussion',
    wordCountMin: 400, wordCountMax: 600,
    linkStrategy: 'first-comment',
    keyPoints: [
      'Name the AuDHD communication trap — ADHD wants to fire off an impulsive reply, autism needs to process for days. One leads to saying the wrong thing, the other to ghosting. You lose either way',
      'Paramedic examples — real specific situations. The shift handover email, the colleague message left for a week. Specific, not abstract',
      'What you tried that DIDN\'T work — templates? Scripts? Asking friends? Forcing yourself? This sub wants the full journey',
      'The blank page realisation — how you figured out the freeze was about initiation. Frame as a discovery',
      'What you ended up building — describe it functionally (paste, pick type, get starting point, edit, send). Don\'t name it. Don\'t link it',
      'RSD disclaimer — "be brutal, the RSD will hit later but that\'s future me\'s problem" energy',
      'Open question — "Does the freeze work this way for you? Same combo of wanting to reply and not being able to start?"'
    ],
    provenHooks: [
      '"The AuDHD communication trap — my ADHD wants to reply NOW, my autism needs 3 days, and neither works"',
      '"AuDHD and the impossible message"'
    ],
    killSignals: [
      { pattern: /\b(AI|GPT|artificial intelligence|chatbot|language model)\b/i, message: 'AI mention detected' },
      { pattern: /\b(check out|try this|download|sign up|visit)\b/i, message: 'Promotional language detected' },
      { pattern: /https?:\/\/|www\./i, message: 'URL in post body' },
      { pattern: /calm[\s-]?reply/i, message: 'Product name in post body' }
    ],
    doList: [
      'This is YOUR community — be fully authentic',
      'Go long (400-600 words). This sub rewards thorough, thoughtful posts',
      'Address the specific AuDHD push-pull, not just autism or just ADHD',
      'Include what didn\'t work before describing what did',
      'Be funny where it comes naturally'
    ],
    dontList: [
      'Don\'t rush this one — it should be your longest, most personal post',
      'Don\'t only address autism OR ADHD — the intersection is the whole point',
      'Don\'t make it about the tool — make it about the AuDHD experience that led to the tool'
    ],
    commentTone: 'Warm, detailed, personal. "Fellow AuDHD humans" energy. This community welcomes thorough sharing.',
    structureChecks: {
      requiresQuestion: true,
      requiresFirstPerson: true,
      requiresParamedic: true,
      requiresAuDHD: true
    }
  },
  {
    id: 'aspergers',
    name: 'r/aspergers',
    members: '~175K',
    scheduledDate: '2026-03-12T17:30:00Z',
    angle: 'Observation (processing pattern)',
    flair: '',
    wordCountMin: 200, wordCountMax: 350,
    linkStrategy: 'only-if-asked',
    keyPoints: [
      'Open with an observation, not a feeling — "I\'ve noticed a pattern..." or "I\'ve been thinking about why..."',
      'Describe the mechanism — what happens cognitively: processing load of reading tone + inferring intent + choosing register + predicting response + managing emotional regulation, simultaneously. Frame as a systems problem',
      'The blank page hypothesis — present as a testable observation: "When I have a rough starting point, the freeze breaks. The bottleneck seems to be initiation, not composition." Let them debate',
      'Brief personal context — paramedic, on the spectrum. Factual, not emotional. One or two sentences',
      'What you built, mechanically — "input a message, select a category, receive a draft, edit, send." This sub wants to know HOW it works',
      'Invite analytical feedback — "Does this match your processing pattern? Is initiation the actual bottleneck, or something else?"'
    ],
    provenHooks: [
      '"I\'ve noticed a pattern in how I process difficult messages — the freeze isn\'t about the words"',
      '"Observation: the barrier to replying isn\'t finding the right words — it\'s starting from nothing"'
    ],
    killSignals: [
      { pattern: /\b(AI|GPT|artificial intelligence|chatbot|language model)\b/i, message: 'AI mention detected' },
      { pattern: /\b(check out|try this|download|sign up|visit)\b/i, message: 'Promotional language detected' },
      { pattern: /https?:\/\/|www\./i, message: 'URL in post body' },
      { pattern: /calm[\s-]?reply/i, message: 'Product name in post body' },
      { pattern: /\bneurodivergent\b/i, message: 'Don\'t use "neurodivergent" — say Asperger\'s or autistic' },
      { pattern: /\bfree tool\b/i, message: '"Free tool" is product language — describe it functionally instead' },
      { pattern: /\bI felt\b/i, message: 'This sub prefers "I noticed" or "I observed" over emotional framing' }
    ],
    doList: [
      'Frame analytically — this is a debate club, not a support group',
      'Lead with mechanism, not emotion — "I noticed..." not "I felt..."',
      'Use structured format (What / Why / How)',
      'Be honest about AI if asked — this sub already discusses AI',
      'Invite debate about the cognitive pattern'
    ],
    dontList: [
      'Don\'t use emotional hooks as the primary framing',
      'Don\'t say "free tool" or "looking for feedback" in the title — product language',
      'Don\'t be paternalistic ("helping people like us")',
      'Don\'t use soft/emotional language as the main hook'
    ],
    commentTone: 'Matter-of-fact, analytical, concise. Users describe things factually, not emotionally. Disagreement is normal here.',
    structureChecks: {
      requiresQuestion: true,
      requiresFirstPerson: true,
      requiresParamedic: true,
      requiresAuDHD: false
    }
  },
  {
    id: 'adhdwomen',
    name: 'r/adhdwomen',
    members: '~538K',
    scheduledDate: '2026-03-13T17:00:00Z',
    angle: 'Question (shame spiral)',
    flair: 'Emotional Regulation',
    wordCountMin: 300, wordCountMax: 400,
    linkStrategy: 'only-if-asked',
    keyPoints: [
      'Personal hook — a SPECIFIC freeze moment. "Last Tuesday I got an email from [whoever] and I opened it, felt my chest tighten, and closed it. Then I did that every day for a week." Physical feelings matter here',
      'The shame spiral — longer you leave it, worse the anxiety, harder it gets, more you hate yourself for not being able to do basic communication',
      'You\'re AuDHD and a paramedic — can run a cardiac arrest but can\'t reply to an email. The contrast lands',
      'What you tried that didn\'t work — forcing yourself, writing drafts at 3am, asking someone else',
      'The blank page insight — frame as a personal discovery, NOT a tool pitch',
      '"Does anyone else experience this?" — end with invitation for shared experience. DO NOT say "I built something" or "I found a tool"'
    ],
    provenHooks: [
      '"Does anyone else get the message freeze where you open it, panic, close it, and repeat for a week?"',
      '"I\'m so tired of the shame spiral from not replying to messages"',
      '"I [embarrassing ADHD thing] and here\'s what happened"',
      '"Does anyone else [specific struggle that sounds niche but is universal]?"'
    ],
    killSignals: [
      { pattern: /\b(AI|GPT|artificial intelligence|chatbot|language model)\b/i, message: 'CRITICAL: r/adhdwomen has an explicit mod post BANNING AI output (2,163 upvotes)' },
      { pattern: /\b(check out|try this|download|sign up|visit|app|website)\b/i, message: 'CRITICAL: This sub is hypervigilant about stealth advertising' },
      { pattern: /https?:\/\/|www\./i, message: 'CRITICAL: No URLs in post body. Link ONLY if someone asks in comments' },
      { pattern: /calm[\s-]?reply/i, message: 'CRITICAL: Product name must NEVER appear in post body' },
      { pattern: /\b(I built|I made|I created|I found a tool|free tool|my tool|a tool)\b/i, message: 'CRITICAL: Do NOT mention building or finding a tool. This post is PURELY a personal story' },
      { pattern: /\bfree\b/i, message: '"Free" implies product — remove it' }
    ],
    doList: [
      'This post is PURELY a personal story and a question — no tool, no solution',
      'Tell a personal story about freezing on a difficult message',
      'Describe the physical feeling (chest tightens, stomach drops)',
      'Mention you\'re AuDHD and a paramedic',
      'Let the community ask what you do about it'
    ],
    dontList: [
      'DO NOT mention building anything',
      'DO NOT mention a tool, an app, or a website',
      'DO NOT link in the post OR in an unprompted comment',
      'DO NOT say "I found something that helps" — that\'s a red flag here',
      'DO NOT use the word "free" (implies product)',
      'DO NOT post on a Saturday'
    ],
    commentTone: 'Warm, personal, starts with "omg same". Lots of emojis and exclamation marks. Tips shared casually, not prescribed. "I\'m proud of you" energy.',
    structureChecks: {
      requiresQuestion: true,
      requiresFirstPerson: true,
      requiresParamedic: true,
      requiresAuDHD: true
    }
  },
  {
    id: 'neurodivergent',
    name: 'r/neurodivergent',
    members: '~14K',
    scheduledDate: '2026-03-18T17:30:00Z',
    angle: 'The cost of reply paralysis',
    flair: '',
    wordCountMin: 150, wordCountMax: 200,
    linkStrategy: 'first-comment',
    keyPoints: [
      'The cost of reply paralysis — not just inconvenience. Lost friendships. Damaged work relationships. People thinking you don\'t care when you care too much',
      'Brief context — AuDHD paramedic, one sentence',
      'The blank page insight — one or two sentences',
      'You built something for yourself — one sentence. Don\'t describe features',
      'Open question — "Does reply paralysis hit anyone else this hard? What do you do about it?"'
    ],
    provenHooks: [
      '"Reply paralysis has cost me friendships and I\'m tired of it"',
      '"Anyone else lose relationships because you literally can\'t make yourself reply to messages?"'
    ],
    killSignals: [
      { pattern: /\b(AI|GPT|artificial intelligence|chatbot|language model)\b/i, message: 'AI mention detected' },
      { pattern: /\b(check out|try this|download|sign up|visit)\b/i, message: 'Promotional language detected' },
      { pattern: /https?:\/\/|www\./i, message: 'URL in post body' },
      { pattern: /calm[\s-]?reply/i, message: 'Product name in post body' },
      { pattern: /\b(free tool|communication tool)\b/i, message: 'Product Hunt language — describe it as "a small thing I made" not "a communication tool"' }
    ],
    doList: [
      'Community-first: "built for us, by us" framing but through sharing, not announcing',
      'Keep concise — 150-200 words',
      'Lead with the cost/pain, not the solution'
    ],
    dontList: [
      'Don\'t list features',
      'Don\'t say "free" multiple times',
      'Don\'t make the title about the tool'
    ],
    commentTone: 'Warm, inclusive, community-oriented.',
    structureChecks: {
      requiresQuestion: true,
      requiresFirstPerson: true,
      requiresParamedic: false,
      requiresAuDHD: true
    }
  },
  {
    id: 'socialanxiety',
    name: 'r/socialanxiety',
    members: '~460K',
    scheduledDate: '2026-03-19T13:00:00Z',
    angle: 'Question (the freeze)',
    flair: 'Discussion',
    wordCountMin: 150, wordCountMax: 250,
    linkStrategy: 'only-if-asked',
    keyPoints: [
      'The universal freeze — describe WITHOUT any ND language. No masking, stimming, RSD, AuDHD. Just: "You open a message, your brain locks up, you close it. Repeat for days"',
      'The shame compound — longer you leave it, worse it gets because now you have to explain the delay AND reply. Two sentences max',
      '"I recently figured something out" — the blank page insight. Keep it vague. "The freeze isn\'t about the words — it\'s about starting from nothing"',
      'End with the question — "Does anyone else experience this? Is it the same for you?" Full stop. Nothing else'
    ],
    provenHooks: [
      '"Does anyone else freeze when they need to reply to a difficult message?"',
      '"When you can\'t reply to a message and it ruins your whole day"'
    ],
    killSignals: [
      { pattern: /\b(AI|GPT|artificial intelligence|chatbot|language model)\b/i, message: 'AI mention detected' },
      { pattern: /\b(check out|try this|download|sign up|visit)\b/i, message: 'Promotional language detected' },
      { pattern: /https?:\/\/|www\./i, message: 'URL in post body' },
      { pattern: /calm[\s-]?reply/i, message: 'Product name in post body' },
      { pattern: /\b(I built|I made|I created|free tool|my tool|a tool)\b/i, message: 'Do NOT mention building anything in this sub' },
      { pattern: /\b(autis|ADHD|AuDHD|neurodiverg|masking|stimming|RSD|rejection sensitiv)\w*/i, message: 'No ND language in r/socialanxiety — keep it universal' },
      { pattern: /\b(you should|try this|have you tried)\b/i, message: 'No prescriptive advice — feels like homework to this community' }
    ],
    doList: [
      'Universal anxiety framing — no ND identity',
      'Keep it SHORT — under 250 words',
      'Use "Success" flair if sharing that something helped',
      'Post the feeling, wait, share only when asked'
    ],
    dontList: [
      'DO NOT mention building anything',
      'DO NOT mention a tool in the post body',
      'DO NOT mention autism, ADHD, or neurodivergence',
      'DO NOT use clinical language',
      'DO NOT post a first comment with a link — wait to be asked',
      'DO NOT use prescriptive language ("you should try...")'
    ],
    commentTone: 'Quiet, empathetic, short. Many posts get upvotes but no comments. Don\'t force engagement.',
    structureChecks: {
      requiresQuestion: true,
      requiresFirstPerson: true,
      requiresParamedic: false,
      requiresAuDHD: false
    }
  }
];
```

**Step 2: Verify data loads**

Add `console.log(SUBS.length, 'subs loaded');` temporarily. Open in browser. Console should show `8 subs loaded`.

**Step 3: Commit**

```bash
git add reddit-post-coach.html
git commit -m "feat: add subreddit intelligence data (8 subs)"
```

---

## Task 3: Schedule Dashboard (Screen 1)

**Files:**
- Modify: `reddit-post-coach.html`

**Step 1: Add the schedule rendering function**

This reads SUBS data + localStorage status, renders cards, highlights the next upcoming post.

```javascript
const STORAGE_KEY = 'reddit-coach';

function getState() {
  return JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
}

function saveState(state) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

function getPostStatus(subId) {
  const state = getState();
  return (state.posts && state.posts[subId]?.status) || 'upcoming';
}

function renderSchedule() {
  const now = new Date();
  const container = document.getElementById('schedule');
  let foundNext = false;

  let html = '<div style="max-width:640px;margin:0 auto;">';
  html += '<h2 style="font-size:1.1rem;color:var(--text-light);margin:16px 0 8px;">Your posting schedule</h2>';

  SUBS.forEach((sub, i) => {
    const date = new Date(sub.scheduledDate);
    const status = getPostStatus(sub.id);
    const isNext = !foundNext && status === 'upcoming' && date > now;
    if (isNext) foundNext = true;

    const statusColors = {
      'upcoming': 'var(--text-light)',
      'draft-started': 'var(--warning)',
      'ready': 'var(--success)',
      'posted': 'var(--accent)'
    };
    const statusLabels = {
      'upcoming': 'Upcoming',
      'draft-started': 'Draft Started',
      'ready': 'Ready',
      'posted': 'Posted'
    };

    html += `
      <div class="card" style="cursor:pointer;${isNext ? 'border:2px solid var(--accent);' : ''}"
           onclick="openWriter('${sub.id}')">
        ${isNext ? '<div style="font-size:0.75rem;font-weight:700;color:var(--accent);text-transform:uppercase;margin-bottom:8px;">Write this one next</div>' : ''}
        <div style="display:flex;justify-content:space-between;align-items:center;">
          <div>
            <div style="font-weight:700;font-size:1rem;">${sub.name}</div>
            <div style="font-size:0.85rem;color:var(--text-light);">
              ${date.toLocaleDateString('en-GB', { weekday: 'short', day: 'numeric', month: 'short' })} at ${date.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' })} GMT
            </div>
            <div style="font-size:0.85rem;color:var(--text-light);margin-top:2px;">${sub.angle}</div>
          </div>
          <span style="font-size:0.75rem;font-weight:600;color:${statusColors[status]};background:${statusColors[status]}15;padding:4px 10px;border-radius:20px;">
            ${statusLabels[status]}
          </span>
        </div>
      </div>`;
  });

  html += '</div>';
  container.innerHTML = html;
}
```

**Step 2: Call `renderSchedule()` on page load**

Add to the bottom of the script: `renderSchedule();`

**Step 3: Verify in browser**

Open the file. Should see 8 cards in order. The first upcoming post (r/ADHD, Tue 24 Feb) should have a teal border and "Write this one next" label. Click a card — nothing happens yet (no `openWriter` function).

**Step 4: Commit**

```bash
git add reddit-post-coach.html
git commit -m "feat: schedule dashboard with status cards and next-post highlighting"
```

---

## Task 4: Write & Score Screen (Screen 2)

**Files:**
- Modify: `reddit-post-coach.html`

**Step 1: Add the writer screen rendering**

The writer shows: header with sub info, collapsible cheat sheet, title input, body textarea with live word count, and three action buttons.

```javascript
let currentSub = null;

function openWriter(subId) {
  currentSub = SUBS.find(s => s.id === subId);
  if (!currentSub) return;

  // Load any saved draft from localStorage
  const state = getState();
  const savedTitle = state.posts?.[subId]?.title || '';
  const savedBody = state.posts?.[subId]?.body || '';

  const date = new Date(currentSub.scheduledDate);
  const container = document.getElementById('writer');

  let html = `
    <div style="max-width:640px;margin:0 auto;">
      <button onclick="showScreen('schedule');renderSchedule();"
        style="background:none;border:none;color:var(--accent);cursor:pointer;font-size:0.9rem;padding:8px 0;">
        &larr; Back to schedule
      </button>

      <div class="card">
        <div style="display:flex;justify-content:space-between;align-items:start;margin-bottom:16px;">
          <div>
            <h2 style="font-size:1.2rem;margin-bottom:4px;">${currentSub.name}</h2>
            <div style="font-size:0.85rem;color:var(--text-light);">
              ${date.toLocaleDateString('en-GB', { weekday: 'long', day: 'numeric', month: 'long' })} at ${date.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' })} GMT
            </div>
          </div>
          <div style="text-align:right;">
            <div style="font-size:0.8rem;color:var(--text-light);">Target</div>
            <div style="font-size:0.9rem;font-weight:600;">${currentSub.wordCountMin}–${currentSub.wordCountMax} words</div>
          </div>
        </div>

        ${currentSub.flair ? `<div style="font-size:0.8rem;color:var(--text-light);margin-bottom:4px;">Flair: <strong>${currentSub.flair}</strong></div>` : ''}
        <div style="font-size:0.8rem;color:var(--text-light);margin-bottom:16px;">
          Link strategy: <strong>${currentSub.linkStrategy === 'first-comment' ? 'Post link in first comment' : 'ONLY share link if someone asks'}</strong>
        </div>

        <!-- Cheat Sheet -->
        <details style="margin-bottom:20px;background:var(--bg);border-radius:8px;padding:12px 16px;">
          <summary style="cursor:pointer;font-weight:600;font-size:0.9rem;color:var(--accent);">Cheat Sheet</summary>
          <div style="margin-top:12px;font-size:0.85rem;line-height:1.7;">
            <div style="font-weight:600;margin-bottom:4px;">Key points to hit:</div>
            <ol style="padding-left:20px;margin-bottom:12px;">
              ${currentSub.keyPoints.map(p => `<li style="margin-bottom:4px;">${p}</li>`).join('')}
            </ol>
            <div style="font-weight:600;margin-bottom:4px;">Proven hooks:</div>
            <ul style="padding-left:20px;margin-bottom:12px;">
              ${currentSub.provenHooks.map(h => `<li style="margin-bottom:4px;color:var(--accent);">${h}</li>`).join('')}
            </ul>
            <div style="display:grid;grid-template-columns:1fr 1fr;gap:12px;">
              <div>
                <div style="font-weight:600;color:var(--success);margin-bottom:4px;">DO:</div>
                <ul style="padding-left:16px;">
                  ${currentSub.doList.map(d => `<li style="margin-bottom:4px;">${d}</li>`).join('')}
                </ul>
              </div>
              <div>
                <div style="font-weight:600;color:var(--error);margin-bottom:4px;">DON'T:</div>
                <ul style="padding-left:16px;">
                  ${currentSub.dontList.map(d => `<li style="margin-bottom:4px;">${d}</li>`).join('')}
                </ul>
              </div>
            </div>
            <div style="margin-top:12px;">
              <div style="font-weight:600;margin-bottom:4px;">Comment tone:</div>
              <div style="color:var(--text-light);">${currentSub.commentTone}</div>
            </div>
          </div>
        </details>

        <!-- Title input -->
        <label style="font-size:0.85rem;font-weight:600;margin-bottom:6px;display:block;">Post title</label>
        <input type="text" id="postTitle" value="${savedTitle.replace(/"/g, '&quot;')}"
          placeholder="Lead with a feeling or question, not a product..."
          style="width:100%;padding:10px 14px;border:1px solid var(--border);border-radius:8px;font-size:0.95rem;margin-bottom:16px;">

        <!-- Body textarea -->
        <label style="font-size:0.85rem;font-weight:600;margin-bottom:6px;display:block;">Post body</label>
        <textarea id="postBody" rows="14" placeholder="Write your rough notes here..."
          oninput="updateWordCount()"
          style="width:100%;padding:12px 14px;border:1px solid var(--border);border-radius:8px;font-size:0.95rem;resize:vertical;font-family:inherit;line-height:1.6;"
        >${savedBody}</textarea>
        <div id="wordCount" style="font-size:0.8rem;color:var(--text-light);margin-top:4px;text-align:right;"></div>

        <!-- Buttons -->
        <div style="display:flex;gap:10px;margin-top:16px;flex-wrap:wrap;">
          <button onclick="scoreDraft()" style="flex:1;min-width:140px;padding:12px;background:var(--accent);color:white;border:none;border-radius:8px;font-size:0.95rem;font-weight:600;cursor:pointer;">
            Score My Draft
          </button>
          <button onclick="doItForMe()" id="btnGenerate" style="flex:1;min-width:140px;padding:12px;background:var(--text);color:white;border:none;border-radius:8px;font-size:0.95rem;font-weight:600;cursor:pointer;">
            Do It For Me
          </button>
          <button onclick="saveToNotion()" style="flex:1;min-width:140px;padding:12px;background:white;color:var(--accent);border:2px solid var(--accent);border-radius:8px;font-size:0.95rem;font-weight:600;cursor:pointer;">
            Save to Notion
          </button>
        </div>

        <!-- Score report appears here -->
        <div id="scoreReport"></div>

        <!-- First comment section -->
        <div style="margin-top:24px;padding-top:16px;border-top:1px solid var(--border);">
          <label style="font-size:0.85rem;font-weight:600;margin-bottom:6px;display:block;">First comment (if applicable)</label>
          <textarea id="firstComment" rows="3" placeholder="${currentSub.linkStrategy === 'only-if-asked' ? 'Only post this if someone asks...' : 'Post this immediately after your post...'}"
            style="width:100%;padding:10px 14px;border:1px solid var(--border);border-radius:8px;font-size:0.9rem;resize:vertical;font-family:inherit;"
          >${state.posts?.[subId]?.firstComment || ''}</textarea>
        </div>
      </div>
    </div>`;

  container.innerHTML = html;
  showScreen('writer');
  updateWordCount();
}

function updateWordCount() {
  const body = document.getElementById('postBody').value.trim();
  const count = body ? body.split(/\s+/).length : 0;
  const el = document.getElementById('wordCount');
  const min = currentSub.wordCountMin;
  const max = currentSub.wordCountMax;

  let color = 'var(--success)';
  if (count < min * 0.7 || count > max * 1.3) color = 'var(--error)';
  else if (count < min || count > max) color = 'var(--warning)';

  el.style.color = color;
  el.textContent = `${count} / ${min}–${max} words`;
}
```

**Step 2: Auto-save drafts to localStorage on input**

Add a debounced auto-save so drafts persist across browser reloads:

```javascript
let saveTimeout;
function autoSaveDraft() {
  clearTimeout(saveTimeout);
  saveTimeout = setTimeout(() => {
    if (!currentSub) return;
    const state = getState();
    if (!state.posts) state.posts = {};
    if (!state.posts[currentSub.id]) state.posts[currentSub.id] = {};
    state.posts[currentSub.id].title = document.getElementById('postTitle').value;
    state.posts[currentSub.id].body = document.getElementById('postBody').value;
    state.posts[currentSub.id].firstComment = document.getElementById('firstComment').value;
    if (document.getElementById('postBody').value.trim()) {
      state.posts[currentSub.id].status = 'draft-started';
    }
    saveState(state);
  }, 1000);
}
```

Add `oninput="autoSaveDraft()"` to the title input, body textarea, and first comment textarea.

**Step 3: Verify in browser**

Click a card on the schedule. Should see the writer screen with sub name, date, word count target, link strategy, collapsible cheat sheet, title input, body textarea, live word count, and three buttons. Type some text — word count should update and change colour. Click "Back to schedule" — should return.

**Step 4: Commit**

```bash
git add reddit-post-coach.html
git commit -m "feat: writer screen with cheat sheet, inputs, word count, auto-save"
```

---

## Task 5: Client-Side Scoring Engine

**Files:**
- Modify: `reddit-post-coach.html`

**Step 1: Add the scoring function**

```javascript
function scoreDraft() {
  const title = document.getElementById('postTitle').value;
  const body = document.getElementById('postBody').value;
  const results = [];
  let totalScore = 0;
  let totalWeight = 0;

  // --- 1. Title Quality (25%) ---
  const titleChecks = [];
  const titlePromoPatterns = [
    { pattern: /\bfree\b/i, fix: 'Remove "free" from title — product language' },
    { pattern: /\b(tool|app|website|built|made|created)\b/i, fix: 'Title leads with the tool — rewrite to lead with a feeling or question' },
    { pattern: /\b(check out|try this|looking for feedback)\b/i, fix: 'Promotional phrasing in title' }
  ];
  let titleScore = 10;
  titlePromoPatterns.forEach(p => {
    if (p.pattern.test(title)) {
      titleChecks.push({ pass: false, message: p.fix });
      titleScore -= 3;
    }
  });
  if (!title.trim()) {
    titleChecks.push({ pass: false, message: 'No title written yet' });
    titleScore = 0;
  } else {
    if (title.includes('?') || /^(does|do|has|have|is|am|are|what|why|how|who|when|anyone)/i.test(title)) {
      titleChecks.push({ pass: true, message: 'Title uses question/hook format' });
    } else {
      titleChecks.push({ pass: false, message: 'Consider starting with a question or "Does anyone else..."' });
      titleScore -= 1;
    }
    if (title.length > 120) {
      titleChecks.push({ pass: false, message: `Title is ${title.length} chars — keep it under 120` });
      titleScore -= 1;
    } else {
      titleChecks.push({ pass: true, message: 'Title length is good' });
    }
  }
  titleScore = Math.max(0, Math.min(10, titleScore));
  results.push({ category: 'Title Quality', weight: 25, score: titleScore, checks: titleChecks });

  // --- 2. Kill Signals (25%) ---
  const killChecks = [];
  let killScore = 10;
  currentSub.killSignals.forEach(ks => {
    const re = new RegExp(ks.pattern.source, ks.pattern.flags);
    // Check both title and body
    const inTitle = re.test(title);
    const inBody = re.test(body);
    if (inTitle || inBody) {
      const where = inTitle && inBody ? 'title and body' : inTitle ? 'title' : 'body';
      killChecks.push({ pass: false, message: `${ks.message} (found in ${where})` });
      killScore -= 3;
    }
  });
  if (killChecks.length === 0) {
    killChecks.push({ pass: true, message: 'No kill signals detected' });
  }
  killScore = Math.max(0, Math.min(10, killScore));
  results.push({ category: 'Kill Signals', weight: 25, score: killScore, checks: killChecks });

  // --- 3. Word Count (15%) ---
  const wordCount = body.trim() ? body.trim().split(/\s+/).length : 0;
  const wcChecks = [];
  let wcScore = 10;
  if (wordCount === 0) {
    wcChecks.push({ pass: false, message: 'No post body written yet' });
    wcScore = 0;
  } else if (wordCount < currentSub.wordCountMin) {
    const diff = currentSub.wordCountMin - wordCount;
    wcChecks.push({ pass: false, message: `${wordCount} words — ${diff} below minimum (${currentSub.wordCountMin}). Add more detail` });
    wcScore = wordCount < currentSub.wordCountMin * 0.5 ? 2 : 5;
  } else if (wordCount > currentSub.wordCountMax) {
    const diff = wordCount - currentSub.wordCountMax;
    wcChecks.push({ pass: false, message: `${wordCount} words — ${diff} over maximum (${currentSub.wordCountMax}). Cut it down` });
    wcScore = wordCount > currentSub.wordCountMax * 1.5 ? 3 : 6;
  } else {
    wcChecks.push({ pass: true, message: `${wordCount} words — within target range (${currentSub.wordCountMin}–${currentSub.wordCountMax})` });
  }
  results.push({ category: 'Word Count', weight: 15, score: wcScore, checks: wcChecks });

  // --- 4. Structure (20%) ---
  const structChecks = [];
  let structScore = 10;
  const sc = currentSub.structureChecks;

  if (sc.requiresQuestion && !body.includes('?')) {
    structChecks.push({ pass: false, message: 'Post doesn\'t contain a question — end with one to invite engagement' });
    structScore -= 3;
  } else if (sc.requiresQuestion) {
    structChecks.push({ pass: true, message: 'Contains a question' });
  }

  if (sc.requiresFirstPerson && !/\bI\b/.test(body)) {
    structChecks.push({ pass: false, message: 'No first-person "I" found — personal stories win in ND subs' });
    structScore -= 3;
  } else if (sc.requiresFirstPerson) {
    structChecks.push({ pass: true, message: 'First-person framing' });
  }

  if (sc.requiresParamedic && !/\bparamedic\b/i.test(body)) {
    structChecks.push({ pass: false, message: 'Paramedic context missing — this sub\'s blueprint requires it' });
    structScore -= 2;
  } else if (sc.requiresParamedic) {
    structChecks.push({ pass: true, message: 'Paramedic context included' });
  }

  if (sc.requiresAuDHD && !/\b(AuDHD|autis\w+ (with|and) ADHD|ADHD (and|with) autis\w+)\b/i.test(body)) {
    structChecks.push({ pass: false, message: 'AuDHD identity missing — this sub\'s blueprint requires it' });
    structScore -= 2;
  } else if (sc.requiresAuDHD) {
    structChecks.push({ pass: true, message: 'AuDHD identity included' });
  }

  structScore = Math.max(0, Math.min(10, structScore));
  results.push({ category: 'Structure', weight: 20, score: structScore, checks: structChecks });

  // --- 5. Link Discipline (15%) ---
  const linkChecks = [];
  let linkScore = 10;
  if (/https?:\/\/|www\./i.test(body)) {
    linkChecks.push({ pass: false, message: 'URL detected in post body — move to first comment' });
    linkScore -= 5;
  }
  if (/calm[\s-]?reply/i.test(body)) {
    linkChecks.push({ pass: false, message: 'Product name in post body — remove it' });
    linkScore -= 5;
  }
  if (linkChecks.length === 0) {
    linkChecks.push({ pass: true, message: 'No links or product names in body' });
  }
  linkScore = Math.max(0, Math.min(10, linkScore));
  results.push({ category: 'Link Discipline', weight: 15, score: linkScore, checks: linkChecks });

  // --- Calculate weighted total ---
  results.forEach(r => {
    totalScore += r.score * r.weight;
    totalWeight += r.weight;
  });
  const finalScore = Math.round(totalScore / totalWeight * 10) / 10;

  renderScoreReport(finalScore, results);
}
```

**Step 2: Add the score report renderer**

```javascript
function renderScoreReport(score, results) {
  let color, label;
  if (score >= 8) { color = 'var(--success)'; label = 'Ready to post'; }
  else if (score >= 5) { color = 'var(--warning)'; label = 'Getting there'; }
  else { color = 'var(--error)'; label = 'Needs work'; }

  let html = `
    <div style="margin-top:24px;padding-top:20px;border-top:1px solid var(--border);">
      <div style="display:flex;align-items:center;gap:16px;margin-bottom:16px;">
        <div style="font-size:2rem;font-weight:800;color:${color};">${score}/10</div>
        <div style="font-size:1rem;font-weight:600;color:${color};">${label}</div>
      </div>`;

  results.forEach(r => {
    html += `
      <div style="margin-bottom:16px;">
        <div style="font-size:0.85rem;font-weight:600;margin-bottom:6px;">
          ${r.category} <span style="color:var(--text-light);font-weight:400;">(${r.weight}%)</span>
          <span style="float:right;color:${r.score >= 8 ? 'var(--success)' : r.score >= 5 ? 'var(--warning)' : 'var(--error)'};">${r.score}/10</span>
        </div>`;
    r.checks.forEach(c => {
      html += `
        <div style="font-size:0.85rem;padding:4px 0 4px 20px;position:relative;">
          <span style="position:absolute;left:0;">${c.pass ? '&#10003;' : '&#10007;'}</span>
          <span style="color:${c.pass ? 'var(--success)' : 'var(--error)'};">${c.message}</span>
        </div>`;
    });
    html += '</div>';
  });

  html += '</div>';
  document.getElementById('scoreReport').innerHTML = html;
}
```

**Step 3: Verify in browser**

Open writer for r/socialanxiety. Type a title with "I built a free tool" and body with "I have ADHD and made this website https://calm-reply.com". Hit "Score My Draft". Should see a low score with multiple red items: URL in body, product name, "I built" in title, ND language, etc.

Then clear it and type a clean post. Score should improve.

**Step 4: Commit**

```bash
git add reddit-post-coach.html
git commit -m "feat: client-side scoring engine with 5 weighted categories"
```

---

## Task 6: "Do It For Me" — Frontend Integration

**Files:**
- Modify: `reddit-post-coach.html`

**Step 1: Add the generation function**

```javascript
const N8N_COACH_URL = 'https://aswales.app.n8n.cloud/webhook/calmreply-reddit-coach';

function getMemory() {
  const state = getState();
  if (!state.memory) return [];
  return state.memory;
}

function addToMemory(subId, body) {
  const state = getState();
  if (!state.memory) state.memory = [];
  // Don't duplicate
  const existing = state.memory.findIndex(m => m.subreddit === subId);
  if (existing >= 0) state.memory[existing].body = body;
  else state.memory.push({ subreddit: subId, body });
  saveState(state);
}

async function doItForMe() {
  if (!currentSub) return;
  const btn = document.getElementById('btnGenerate');
  const origText = btn.textContent;
  btn.textContent = 'Generating...';
  btn.disabled = true;

  const userNotes = document.getElementById('postBody').value;
  const title = document.getElementById('postTitle').value;
  const previousPosts = getMemory();

  try {
    const res = await fetch(N8N_COACH_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        subreddit: currentSub.id,
        userNotes,
        title,
        previousPosts
      })
    });

    if (!res.ok) throw new Error(`n8n returned ${res.status}`);
    const data = await res.json();

    if (data.title) document.getElementById('postTitle').value = data.title;
    if (data.body) {
      document.getElementById('postBody').value = data.body;
      updateWordCount();
    }
    if (data.firstComment) {
      document.getElementById('firstComment').value = data.firstComment;
    }

    // Show generation notes if provided
    if (data.notes) {
      const report = document.getElementById('scoreReport');
      report.innerHTML = `
        <div style="margin-top:16px;padding:12px 16px;background:var(--bg);border-radius:8px;font-size:0.85rem;">
          <strong>Generation notes:</strong> ${data.notes}
        </div>`;
    }

    autoSaveDraft();
  } catch (err) {
    alert('Generation failed: ' + err.message);
  } finally {
    btn.textContent = origText;
    btn.disabled = false;
  }
}
```

**Step 2: Verify in browser**

The "Do It For Me" button should show "Generating..." when clicked. It will fail with a network error until Task 8 (n8n workflow) is done — that's expected. Verify no JS errors and that the button re-enables after the error.

**Step 3: Commit**

```bash
git add reddit-post-coach.html
git commit -m "feat: 'do it for me' AI generation with memory system"
```

---

## Task 7: Save to Notion — Frontend Integration

**Files:**
- Modify: `reddit-post-coach.html`

**Step 1: Add the save function and status updater**

```javascript
const N8N_SAVE_URL = 'https://aswales.app.n8n.cloud/webhook/calmreply-reddit-save';

async function saveToNotion() {
  if (!currentSub) return;
  const body = document.getElementById('postBody').value.trim();
  const title = document.getElementById('postTitle').value.trim();
  if (!body) { alert('Write something first!'); return; }

  const btn = event.target;
  const origText = btn.textContent;
  btn.textContent = 'Saving...';
  btn.disabled = true;

  try {
    const res = await fetch(N8N_SAVE_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        subreddit: currentSub.name,
        scheduledDate: currentSub.scheduledDate,
        title,
        body,
        firstComment: document.getElementById('firstComment').value || null,
        score: 0, // will be filled if scored
        linkStrategy: currentSub.linkStrategy
      })
    });

    if (!res.ok) throw new Error(`n8n returned ${res.status}`);
    const data = await res.json();

    // Update status to ready
    const state = getState();
    if (!state.posts) state.posts = {};
    if (!state.posts[currentSub.id]) state.posts[currentSub.id] = {};
    state.posts[currentSub.id].status = 'ready';
    saveState(state);

    // Add to memory for future AI generations
    addToMemory(currentSub.id, body);

    btn.textContent = 'Saved!';
    btn.style.background = 'var(--success)';
    btn.style.color = 'white';
    btn.style.borderColor = 'var(--success)';

    if (data.notionUrl) {
      const link = document.createElement('a');
      link.href = data.notionUrl;
      link.target = '_blank';
      link.textContent = 'Open in Notion';
      link.style.cssText = 'display:block;text-align:center;margin-top:8px;color:var(--accent);font-size:0.85rem;';
      btn.parentNode.appendChild(link);
    }
  } catch (err) {
    alert('Save failed: ' + err.message);
    btn.textContent = origText;
    btn.disabled = false;
  }
}
```

**Step 2: Pass the current score to saveToNotion**

Add a module-level variable to store the last score. In `renderScoreReport`, save it: `lastScore = score;`. In `saveToNotion`, use `score: lastScore || 0` in the payload. Add `let lastScore = 0;` near the top of the script.

**Step 3: Verify in browser**

Click "Save to Notion" with an empty textarea — should show "Write something first!" alert. Type text and click — should show "Saving..." then fail with network error until n8n is wired. Verify button re-enables on error.

**Step 4: Commit**

```bash
git add reddit-post-coach.html
git commit -m "feat: save to Notion integration with memory system and status tracking"
```

---

## Task 8: n8n Workflow — Reddit Coach (AI Generation)

**Files:**
- This is built in the n8n UI, not in code. Instructions below.

**Step 1: Create a new workflow in n8n**

1. Go to https://aswales.app.n8n.cloud
2. Create new workflow: "Reddit Post Coach"
3. Add a **Webhook** node:
   - Method: POST
   - Path: `calmreply-reddit-coach`
   - Response Mode: "Respond to Webhook"

**Step 2: Add a Switch node**

- Connect Webhook → Switch
- Switch on: `{{ $json.body.subreddit }}`
- 8 outputs, one per sub ID: `ADHD`, `AutisticAdults`, `autism`, `AutisticWithADHD`, `aspergers`, `adhdwomen`, `neurodivergent`, `socialanxiety`

**Step 3: Add 8x OpenAI Chat nodes (GPT-4o)**

One per Switch output. Each node:
- Model: `gpt-4o`
- System prompt: See below (unique per sub)
- User message:

```
Here are my rough notes for a Reddit post in {{ $json.body.subreddit }}:

{{ $json.body.userNotes }}

{% if $json.body.title %}
I'm considering this title: {{ $json.body.title }}
{% endif %}

{% if $json.body.previousPosts && $json.body.previousPosts.length > 0 %}
Here are my previous Reddit posts for tone matching:
{% for post in $json.body.previousPosts %}
---
Subreddit: {{ post.subreddit }}
{{ post.body }}
---
{% endfor %}
{% endif %}
```

**Step 4: System prompt template (customise per sub)**

Each OpenAI node gets a system prompt built from the intelligence doc. Here is the template — replace the bracketed sections with sub-specific data:

```
You are helping write a Reddit post for r/[SUBREDDIT]. You must match the author's natural voice — messy, personal, British English, imperfect. NOT polished marketing copy.

RULES FOR THIS SUBREDDIT:
- Word count: [MIN]-[MAX] words. HARD LIMIT.
- Link strategy: [first-comment / only-if-asked]
[SUB-SPECIFIC DO LIST]
[SUB-SPECIFIC DON'T LIST]

KILL SIGNALS (your output MUST NOT contain any of these):
- No AI/GPT/chatbot mentions
- No promotional language (free, tool, check out, try this, visit, sign up, download)
- No URLs in the post body
- No mention of "CalmReply" or "calm-reply"
[SUB-SPECIFIC KILL SIGNALS]

STRUCTURE:
[SUB-SPECIFIC KEY POINTS — numbered beats to hit]

TITLE:
[SUB-SPECIFIC PROVEN HOOKS]
The title must lead with a feeling or question, never with "I built" or "free tool".

STYLE:
If previous posts are provided, match their tone, phrasing, sentence structure, and personality exactly. The author is a paramedic from Wales with AuDHD. They write casually, use British spelling, and are naturally funny in a self-deprecating way. They don't polish. They ramble authentically.

If no previous posts are provided, write in a casual, raw, first-person British English style.

OUTPUT FORMAT (respond in valid JSON):
{
  "title": "suggested post title",
  "body": "full post body text",
  "firstComment": "suggested first comment text, or null if link strategy is only-if-asked",
  "notes": "1-2 sentences explaining your choices"
}
```

**Step 5: Add a Set node (Format Response)**

Connect all 8 OpenAI nodes → single Set node.
- Parse the OpenAI JSON response
- Output: `{{ JSON.parse($json.message.content) }}`

**Step 6: Add Respond to Webhook node**

Connect Set → Respond to Webhook.
- Response body: `{{ $json }}`

**Step 7: Test the webhook**

Activate the workflow. Use curl or the HTML tool to POST:
```bash
curl -X POST https://aswales.app.n8n.cloud/webhook/calmreply-reddit-coach \
  -H "Content-Type: application/json" \
  -d '{"subreddit":"ADHD","userNotes":"paramedic, ghosting people, built a button to apologise","title":"","previousPosts":[]}'
```

Expected: JSON response with title, body, firstComment, notes.

**Step 8: Commit n8n workflow export**

Export the workflow from n8n as JSON. Save to `n8n/RedditPostCoach.json`.

```bash
git add n8n/RedditPostCoach.json
git commit -m "feat: n8n Reddit Post Coach workflow (GPT-4o, 8 sub prompts)"
```

---

## Task 9: n8n Workflow — Save to Notion

**Files:**
- This extends the "Reddit Post Coach" workflow in n8n.

**Step 1: Add a second Webhook node**

In the same "Reddit Post Coach" workflow:
- Add a new Webhook node
- Method: POST
- Path: `calmreply-reddit-save`
- Response Mode: "Respond to Webhook"

**Step 2: Create the Notion database**

Before wiring n8n, create the database in Notion:
1. Open Reddit Launch HQ page in Notion
2. Create a new inline database: "Reddit Posts"
3. Add properties:
   - Post Title (Title) — default
   - Subreddit (Select) — options: r/ADHD, r/autism, r/AutisticAdults, r/AutisticWithADHD, r/aspergers, r/adhdwomen, r/neurodivergent, r/socialanxiety
   - Status (Select) — options: Draft, Ready, Posted
   - Scheduled (Date)
   - Post Body (Rich Text)
   - First Comment (Rich Text)
   - Link Strategy (Select) — options: first-comment, only-if-asked
   - Score (Number)

**Step 3: Add Notion node in n8n**

Connect second Webhook → Notion: Create Database Item
- Credential: Connect Notion account (if not already connected)
- Database: Select "Reddit Posts"
- Map fields:
  - Post Title ← `{{ $json.body.title }}`
  - Subreddit ← `{{ $json.body.subreddit }}`
  - Status ← `Ready`
  - Scheduled ← `{{ $json.body.scheduledDate }}`
  - Post Body ← `{{ $json.body.body }}`
  - First Comment ← `{{ $json.body.firstComment }}`
  - Link Strategy ← `{{ $json.body.linkStrategy }}`
  - Score ← `{{ $json.body.score }}`

**Step 4: Add Set node to extract Notion URL**

Connect Notion → Set node:
- Set field `notionUrl` = `https://notion.so/{{ $json.id.replace(/-/g, '') }}`

**Step 5: Add Respond to Webhook**

Connect Set → Respond to Webhook.
- Response body: `{{ $json }}`

**Step 6: Test the webhook**

```bash
curl -X POST https://aswales.app.n8n.cloud/webhook/calmreply-reddit-save \
  -H "Content-Type: application/json" \
  -d '{"subreddit":"r/ADHD","scheduledDate":"2026-02-24T11:30:00Z","title":"Test post","body":"Test body","firstComment":"Test comment","score":8,"linkStrategy":"first-comment"}'
```

Expected: JSON with `notionUrl`. Check Notion — new entry should appear in Reddit Posts database.

**Step 7: Update n8n export and commit**

```bash
git add n8n/RedditPostCoach.json
git commit -m "feat: n8n Notion save webhook for Reddit Posts database"
```

---

## Task 10: End-to-End Test

**Files:**
- No changes — verification only.

**Step 1: Full workflow test**

1. Open `reddit-post-coach.html` in browser
2. Verify schedule shows 8 cards with correct dates and times
3. Click r/AutisticWithADHD card (safest sub, your community)
4. Expand cheat sheet — verify key points, hooks, DO/DON'T all render
5. Type a rough title and some bullet points in body
6. Click "Score My Draft" — verify score report appears with pass/fail items
7. Click "Do It For Me" — verify it calls n8n, returns a draft, populates the textarea
8. Edit the draft, re-score — verify score updates
9. Click "Save to Notion" — verify it saves and shows Notion link
10. Click "Back to schedule" — verify the card now shows "Ready" status
11. Reload the page — verify the draft persists (localStorage) and status shows "Ready"
12. Open a different sub — verify the cheat sheet changes, word count targets change

**Step 2: Memory test**

1. Save a post for r/AutisticWithADHD
2. Open r/aspergers and click "Do It For Me"
3. Check the n8n execution log — the `previousPosts` array should include the r/AutisticWithADHD post

**Step 3: Commit final state**

```bash
git add reddit-post-coach.html n8n/RedditPostCoach.json
git commit -m "feat: Reddit Post Coach complete — schedule, score, generate, save to Notion"
```

---

## Summary

| Task | What | Commit message |
|------|------|----------------|
| 1 | HTML skeleton + CSS + screen switching | `feat: reddit post coach — HTML skeleton` |
| 2 | Subreddit intelligence data (8 subs) | `feat: add subreddit intelligence data` |
| 3 | Schedule Dashboard (Screen 1) | `feat: schedule dashboard with status cards` |
| 4 | Write & Score UI (Screen 2) | `feat: writer screen with cheat sheet, inputs, word count` |
| 5 | Client-side scoring engine | `feat: client-side scoring engine` |
| 6 | "Do It For Me" frontend | `feat: AI generation with memory system` |
| 7 | "Save to Notion" frontend | `feat: save to Notion integration` |
| 8 | n8n workflow — AI generation | `feat: n8n Reddit Post Coach workflow` |
| 9 | n8n workflow — Notion save | `feat: n8n Notion save webhook` |
| 10 | End-to-end test | `feat: Reddit Post Coach complete` |
