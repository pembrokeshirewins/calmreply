# MVP Realignment Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace 7 business-focused situation types with 1 auto-detecting personal-first prompt, add Protective tone, make coaching note universal.

**Architecture:** Single HTML frontend sends `{ userMessage, tone }` to n8n webhook. n8n routes directly to 1 OpenAI node (no Switch), formats response, returns. Frontend splits reply from coaching note on `\n---\n` for all responses (not just Avoiding).

**Tech Stack:** Single HTML file (inline CSS/JS), n8n Cloud (workflow `lMlLswwqDFGKfhVJ`), GPT-4o-mini, Vercel from `/public`

---

### Task 1: Write the unified prompt

**Files:**
- Create: `docs/prompts/calmreply-unified-prompt.md`

**Step 1: Create the prompt file**

Write the full unified system prompt to `docs/prompts/calmreply-unified-prompt.md`. The prompt must contain ALL of the following sections in this order:

```markdown
# CalmReply — Unified System Prompt (v6.0)

## For n8n OpenAI node: paste into the "Instructions" field (System message)

---

ROLE AND CONTEXT:
You are a communication assistant embedded in a tool called CalmReply. Your user experiences significant anxiety around difficult conversations — they may be neurodivergent, they may have been avoiding this message for days, and they are coming to you in a moment of freeze. Your job is to get them unstuck with a response they can send immediately.

You help with ALL difficult conversations — family, relationships, friendships, work, social situations. Personal messages are just as important as professional ones. A guilt-tripping text from a parent. A hard conversation with a partner. An honest reply to a friend. A message from a bullying colleague. Whatever they're stuck on.

CRITICAL — WHO IS WHO:
The user has pasted a message that SOMEONE ELSE sent TO THEM, or they have described a situation where they need to send a message. The user is the RECIPIENT or INITIATOR. You are drafting a reply FROM the user. Never confuse these roles. The user is not the person who wrote the pasted message.

LANGUAGE:
Always write in British English. Use British spelling throughout: apologise (not apologize), recognise (not recognize), organise (not organize), colour (not color), behaviour (not behavior), centre (not center), defence (not defense), etc.

NEVER use placeholder text like [Name], [Your Name], [Recipient's Name], [date], [specific date], [Company], etc. If you don't know a name, omit it. If you don't know a date, use a relative timeframe like "by the end of this week" or "within the next few days." The user should be able to copy and send your response without editing anything.

AUTO-DETECTION:
Before writing, silently analyse the pasted message or description for:
- Who sent it (parent, partner, friend, colleague, manager, ex, stranger, in-law, sibling)
- The relationship dynamic (loving, manipulative, professional, passive-aggressive, controlling, supportive, toxic, transactional)
- The communication channel (text, email, DM, work chat, Zoom, phone, letter, in-person follow-up)
- The formality level and register
- Emotional temperature (angry, guilt-tripping, cold, desperate, passive-aggressive, threatening, confused, hurt)
- Whether there is a power imbalance (parent/child, boss/employee, etc.)
- Whether the message contains manipulation tactics (guilt-tripping, gaslighting, emotional blackmail, DARVO, love-bombing, silent treatment)
- The sender's name (use it in the reply if present)

If you must make assumptions about context not provided, state your single most important assumption in one short line at the top of your response, italicised. Example: *Assuming this is someone you want to stay in contact with.*

TONE — THE USER SELECTS ONE:

If the user's message ends with [TONE: softer]:
- Warm, bridges the gap, seeks understanding
- Maximum 200 words
- Prioritises maintaining the relationship
- Acknowledges the other person's perspective

If the user's message ends with [TONE: firmer]:
- Direct, boundaried, no padding
- Maximum 100 words (hard limit)
- States position clearly without aggression
- No over-explaining or justifying

If the user's message ends with [TONE: protective]:
- Firm but compassionate — holds ground without escalating
- Maximum 150 words
- Includes one clear boundary statement
- Does NOT engage with manipulation, guilt-tripping, or emotional blackmail — responds only to the legitimate content
- Never cruel, never submissive
- Does not over-explain or justify the boundary

CORE RULES:
- Sound like a real human, not a corporate PR statement or therapist. No "I understand your frustration" as an opener. No "Please don't hesitate to reach out." No "I hope this finds you well."
- Match the formality AND register of the incoming message. If they texted casually, don't reply with a formal letter. If they wrote "Hey," reply with "Hey" or "Hi" — not "Dear."
- Never grovel or beg for forgiveness. One acknowledgment is enough.
- Never accept blame prematurely. Do NOT admit fault, confess to failures, or say things like "a lack of prioritisation on my part" unless the user explicitly says they were at fault. You don't know the full story. Acknowledge the impact without conceding the cause.
- Never be passive-aggressive, sarcastic, or escalatory
- End with a clear next step when appropriate (not a vague "let's discuss")
- Never offer discounts, refunds, compensation, or gifts unless the user's context explicitly mentions wanting this
- Never insert placeholder text in square brackets
- Be concise — the word limits per tone are real limits

DELAY SHIELD:
If the context suggests the user has been avoiding this reply (they mention delay, the message is clearly old, or they say they've been putting it off):
- Include a brief opener that acknowledges the delay in 1 short sentence max
- Never grovel, never stack apologies
- Create forward momentum immediately
- Use ONE of these intent patterns (adapt to context, never copy verbatim):
  - Acknowledge-and-pivot: "Thanks for your patience — here's where I'm at."
  - Normalise-the-gap: "I wanted to give this proper thought before replying."
  - Skip-it-entirely: If the delay is ambiguous or short, just reply naturally.
- HARD RULE: The opener must be 15 words or fewer. The rest of the reply is the substance.

SHORT REPLY MODE:
If the user's message contains [SHORT REPLY MODE]:
- Maximum 40 words. Hard cap 60 if absolutely unavoidable.
- One paragraph only. No line breaks.
- Must still contain: acknowledgment + clear action or ask.
- No filler. No sign-offs unless the input had one.
- If Delay Shield is also active, the delay acknowledgment is one clause within the single sentence, not a separate opener.

USER CONTEXT:
If the user's message contains [USER CONTEXT: ...]:
- Use this as background guidance for your reply
- Never quote the context back to the user or reference it in the reply
- It tells you what the user wants to achieve or what constraints they have

OUTPUT FORMAT — ALWAYS:
Write your reply first, then add a coaching note after a separator. Always use this exact format:

[Your drafted reply here]
---
[Coaching note here]

The coaching note (2-4 sentences) must:
- Name the communication pattern in the original message (guilt-tripping, boundary testing, stonewalling, emotional dumping, passive aggression, love-bombing, DARVO, deflection, catastrophising, silent treatment, etc.)
- Briefly explain why this message might trigger freeze or avoidance
- Validate without diagnosing. No therapy language ("attachment style", "trauma response"). No "you should get help." No moral judgement of either party.
- Be specific to THIS message, not generic advice

Example coaching note: "This message shifts responsibility for their feelings onto you — that's guilt-tripping. It's designed to make 'no' feel cruel, which is why you froze. Your reply holds the boundary without taking the bait."

For professional/transactional messages where there's no emotional pattern to name, the coaching note should briefly explain the communication dynamic instead. Example: "This is a standard escalation — they're frustrated by the wait, not by you personally. Your reply acknowledges the delay without over-apologising."
```

**Step 2: Commit**

```bash
git add docs/prompts/calmreply-unified-prompt.md
git commit -m "docs: unified prompt v6.0 — personal-first auto-detecting"
```

---

### Task 2: Update the frontend HTML

**Files:**
- Modify: `index.html` (root — the source file)

All changes are in a single file. Make these edits in order:

**Step 1: Update the textarea placeholder**

Find (line 656):
```html
<textarea id="message" rows="6" placeholder="Paste the message you've been avoiding, or describe who you need to reach out to"></textarea>
```
Replace with:
```html
<textarea id="message" rows="6" placeholder="Paste the message you're stuck on, or describe what you need to say"></textarea>
```

**Step 2: Update the context field label and placeholder**

Find (line 664-665):
```html
<label for="contextInput" id="contextLabel">Add a note (optional)</label>
<textarea id="contextInput" rows="3" placeholder="What you want to say, your goal, any constraints..."></textarea>
```
Replace with:
```html
<label for="contextInput" id="contextLabel">Anything else I should know? (optional)</label>
<textarea id="contextInput" rows="3" placeholder="Who sent this, your relationship, what you want to happen..."></textarea>
```

**Step 3: Remove the situation dropdown from the Options panel**

Find (lines 682-693):
```html
      <div class="field">
        <label for="situation">Situation</label>
        <select id="situation">
          <option value="avoiding">I've Been Avoiding This</option>
          <option value="complaint">Complaint / De-escalation</option>
          <option value="ghosted">Ghosted Follow-Up</option>
          <option value="invoice">Invoice Chase</option>
          <option value="scope">Scope Pushback / Saying No</option>
          <option value="awkward">Awkward or Difficult Message</option>
          <option value="general">General Communication Help</option>
        </select>
      </div>
```
Delete this entire block.

**Step 4: Add the Protective tone pill**

Find (lines 697-704):
```html
          <label class="tone-pill">
            <input type="radio" name="tone" value="softer" checked />
            <span>Softer</span>
          </label>
          <label class="tone-pill">
            <input type="radio" name="tone" value="firmer" />
            <span>Firmer</span>
          </label>
```
Replace with:
```html
          <label class="tone-pill">
            <input type="radio" name="tone" value="softer" checked />
            <span>Softer</span>
          </label>
          <label class="tone-pill">
            <input type="radio" name="tone" value="firmer" />
            <span>Firmer</span>
          </label>
          <label class="tone-pill">
            <input type="radio" name="tone" value="protective" />
            <span>Protective</span>
          </label>
```

**Step 5: Remove situationEl from JS element declarations**

Find (line 780):
```javascript
var situationEl = document.getElementById('situation');
```
Delete this line.

**Step 6: Update the placeholders object — remove situation-specific placeholders**

Find (lines 815-823):
```javascript
      var placeholders = {
        reply: {
          avoiding: 'Paste the message you\'ve been avoiding, or describe who you need to reach out to',
          default: 'Paste the message here, or describe what you need to say...'
        },
        interpret: {
          default: 'Paste the message you want help understanding'
        }
      };
```
Replace with:
```javascript
      var placeholders = {
        reply: {
          default: 'Paste the message you\'re stuck on, or describe what you need to say'
        },
        interpret: {
          default: 'Paste the message you want help understanding'
        }
      };
```

**Step 7: Simplify updatePlaceholder — remove situation reference**

Find (lines 860-868):
```javascript
      function updatePlaceholder() {
        if (currentMode === 'interpret') {
          messageEl.placeholder = placeholders.interpret.default;
        } else {
          messageEl.placeholder = situationEl.value === 'avoiding'
            ? placeholders.reply.avoiding
            : placeholders.reply.default;
        }
      }
```
Replace with:
```javascript
      function updatePlaceholder() {
        if (currentMode === 'interpret') {
          messageEl.placeholder = placeholders.interpret.default;
        } else {
          messageEl.placeholder = placeholders.reply.default;
        }
      }
```

**Step 8: Update context field label in updateModeUI**

Find (line 876):
```javascript
          contextLabel.textContent = 'Add a note (optional)';
```
Replace with:
```javascript
          contextLabel.textContent = 'Anything else I should know? (optional)';
```

**Step 9: Remove situationType from the submit handler payload**

Find (lines 963-978):
```javascript
        currentRequestId = generateId();
        var situationType = situationEl.value;
        var tone = getSelectedTone();

        var userMessage = message;
        if (contextText) {
          userMessage += '\n\n[USER CONTEXT: ' + contextText + ']';
        }
        if (shortReply.checked) {
          userMessage += '\n\n[SHORT REPLY MODE: Write a reply of 40 words or fewer. One paragraph. Must contain an acknowledgment and a clear action or ask. Hard cap 60 words. No filler, no sign-offs unless the input had one.]';
        }

        var payload = {
          situationType: situationType,
          userMessage: userMessage,
          tone: tone
        };
```
Replace with:
```javascript
        currentRequestId = generateId();
        var tone = getSelectedTone();

        var userMessage = message;
        if (contextText) {
          userMessage += '\n\n[USER CONTEXT: ' + contextText + ']';
        }
        if (shortReply.checked) {
          userMessage += '\n\n[SHORT REPLY MODE: Write a reply of 40 words or fewer. One paragraph. Must contain an acknowledgment and a clear action or ask. Hard cap 60 words. No filler, no sign-offs unless the input had one.]';
        }
        userMessage += '\n\n[TONE: ' + tone + ']';

        var payload = {
          userMessage: userMessage,
          tone: tone
        };
```

Note: The tone is now appended as `[TONE: softer/firmer/protective]` to the userMessage so the single prompt can read it. The `tone` field is still sent in the payload for tracking purposes.

**Step 10: Make coaching note universal — remove the avoiding-only check**

Find (lines 1000-1010):
```javascript
            coachingNote.classList.remove('visible');
            coachingNote.textContent = '';

            if (situationType === 'avoiding' && fullResponse.indexOf('\n---\n') !== -1) {
              var parts = fullResponse.split('\n---\n');
              outputText.textContent = parts[0].trim();
              coachingNote.textContent = parts.slice(1).join('\n---\n').trim();
              coachingNote.classList.add('visible');
            } else {
              outputText.textContent = fullResponse;
            }
```
Replace with:
```javascript
            coachingNote.classList.remove('visible');
            coachingNote.textContent = '';

            if (fullResponse.indexOf('\n---\n') !== -1) {
              var parts = fullResponse.split('\n---\n');
              outputText.textContent = parts[0].trim();
              coachingNote.textContent = parts.slice(1).join('\n---\n').trim();
              coachingNote.classList.add('visible');
            } else {
              outputText.textContent = fullResponse;
            }
```

**Step 11: Update tracking to remove situationType references**

Find (lines 1016-1028):
```javascript
            track({
              event: 'generation',
              requestId: currentRequestId,
              timestamp: new Date().toISOString(),
              situationType: situationType,
              tone: tone,
              mode: currentMode,
              shortReply: shortReply.checked,
              contextProvided: contextText.length > 0,
              device: getDeviceType(),
              inputLength: message.length,
              outputWords: result.response.split(/\s+/).length
            });
```
Replace with:
```javascript
            track({
              event: 'generation',
              requestId: currentRequestId,
              timestamp: new Date().toISOString(),
              tone: tone,
              mode: currentMode,
              shortReply: shortReply.checked,
              contextProvided: contextText.length > 0,
              device: getDeviceType(),
              inputLength: message.length,
              outputWords: result.response.split(/\s+/).length
            });
```

Find (lines 1032-1038):
```javascript
            track({
              event: 'error',
              requestId: currentRequestId,
              timestamp: new Date().toISOString(),
              situationType: situationType,
              tone: tone,
              device: getDeviceType()
            });
```
Replace with:
```javascript
            track({
              event: 'error',
              requestId: currentRequestId,
              timestamp: new Date().toISOString(),
              tone: tone,
              device: getDeviceType()
            });
```

**Step 12: Remove the situationEl change listener**

Find (line 927):
```javascript
      situationEl.addEventListener('change', updatePlaceholder);
```
Delete this line.

**Step 13: Commit**

```bash
git add index.html
git commit -m "feat: remove situation dropdown, add Protective tone, universal coaching note"
```

---

### Task 3: Deploy n8n backend changes

**Files:**
- No local files — uses n8n REST API

**Step 1: Write and run the deploy script**

Create a temporary Python script at `/tmp/deploy-unified.py` that:

1. Fetches the current CalmReply workflow (`lMlLswwqDFGKfhVJ`)
2. Removes the Switch node ("Route by Situation Type") and all 7 OpenAI nodes (Complaint, Ghosted, Invoice, Scope, Awkward, General, Avoiding)
3. Adds 1 new OpenAI node with the unified prompt from `docs/prompts/calmreply-unified-prompt.md`
4. Rewires connections: Webhook → new OpenAI node → Format Response → Respond to Webhook
5. Deactivates, deploys, waits 5s, reactivates

Key details for the new OpenAI node:
- Type: `@n8n/n8n-nodes-langchain.openAi`
- TypeVersion: `2.1` (same as existing nodes — uses Responses API)
- Credential: `nks5HcjE3xjAB75m` (OpenAI, same as existing)
- Model: `gpt-4o-mini`
- The "Prompt" field receives: `{{ $('Webhook').item.json.body.userMessage }}`
- The "Instructions" field receives the full unified system prompt text

The Format Response (Set) node already extracts `output[0].content[0].text` into `response` — this stays unchanged.

The Respond to Webhook node returns `allIncomingItems` with status 200 — also unchanged.

**Step 2: Run the deploy script**

```bash
python3 /tmp/deploy-unified.py
```

Expected output: workflow deploys with 4 main nodes (Webhook, OpenAI, Format Response, Respond to Webhook) plus the email/tracking webhooks.

---

### Task 4: Test against Reddit pain scenarios

**No files — manual testing via curl or browser**

Test the live webhook with the 8 highest-pain scenarios from the Reddit data. For each test, send a POST to `https://aswales.app.n8n.cloud/webhook/calmreply` and verify:

1. A reply is generated (not empty, not an error)
2. The `\n---\n` separator is present
3. The coaching note names a specific communication pattern
4. The reply matches the selected tone
5. No placeholders like [Name] appear

**Test messages (use these exact payloads):**

**Test 1 — Guilt-tripping parent (Protective tone):**
```json
{
  "userMessage": "I just don't understand why you can't come for Sunday lunch anymore. Your father and I won't be around forever you know. I suppose you have more important things to do. Don't worry about us, we'll manage on our own like we always do.\n\n[USER CONTEXT: This is my mum. She does this every week. I moved out 6 months ago and she makes me feel guilty every time I can't come.]\n\n[TONE: protective]",
  "tone": "protective"
}
```

**Test 2 — Relationship hard conversation (Softer tone):**
```json
{
  "userMessage": "I feel like we're just going through the motions lately. I can't remember the last time we actually talked about something real. Are you even happy?\n\n[USER CONTEXT: My partner of 3 years. I do love them, I just don't know how to respond without making it worse.]\n\n[TONE: softer]",
  "tone": "softer"
}
```

**Test 3 — Friend honesty dilemma (Softer tone):**
```json
{
  "userMessage": "So what do you honestly think of James? I really want your opinion because you're always straight with me.\n\n[USER CONTEXT: My best friend. I think James is controlling and I've seen red flags but I don't want to lose the friendship.]\n\n[TONE: softer]",
  "tone": "softer"
}
```

**Test 4 — Workplace bullying (Firmer tone):**
```json
{
  "userMessage": "I noticed you didn't speak up in the meeting again. If you can't contribute then I'm not sure why we need you in those calls. Just something to think about.\n\n[USER CONTEXT: My manager. This is the third time. I'm scared to respond but I need to.]\n\n[TONE: firmer]",
  "tone": "firmer"
}
```

**Test 5 — Narcissistic parent (Protective tone):**
```json
{
  "userMessage": "After everything I've done for you, this is how you repay me? I sacrificed my career so you could have a good life and now you can't even return my calls. I don't know what I did to deserve a child like this.\n\n[TONE: protective]",
  "tone": "protective"
}
```

**Test 6 — Mother-in-law boundary (Protective tone):**
```json
{
  "userMessage": "Hi love, just wanted to let you know I've booked us all in for Christmas dinner at ours. I've already told everyone you're coming so don't let me down! Also I gave your number to my friend Margaret's son, he's in IT too and could be a good contact for you.\n\n[USER CONTEXT: Mother-in-law. She does this constantly — makes plans without asking and gives out my personal info. My partner won't stand up to her.]\n\n[TONE: protective]",
  "tone": "protective"
}
```

**Test 7 — Professional complaint (Softer tone — regression test):**
```json
{
  "userMessage": "I'm really disappointed with the quality of work on the last project. The deliverables were late and the final output wasn't what we discussed. I need to know what happened.\n\n[TONE: softer]",
  "tone": "softer"
}
```

**Test 8 — Invoice chase (Firmer tone — regression test):**
```json
{
  "userMessage": "Hi, just wondering when I might expect payment for invoice #247? It was due on the 15th and I haven't received anything yet.\n\n[USER CONTEXT: I sent this to a client 2 weeks ago and they haven't replied. I need to follow up more firmly.]\n\n[TONE: firmer]",
  "tone": "firmer"
}
```

For each test, verify the response contains both a reply and a coaching note separated by `\n---\n`.

---

### Task 5: Copy to public/ and deploy

**Files:**
- Copy: `index.html` → `public/index.html`

**Step 1: Copy source to deploy directory**

```bash
cp index.html public/index.html
```

**Step 2: Commit and push**

```bash
git add public/index.html
git commit -m "deploy: copy realigned MVP to public/"
git push
```

Vercel auto-deploys from `/public` on push to main.

**Step 3: Verify live site**

Open `https://calm-reply.com` and verify:
- No situation dropdown visible
- Three tone pills: Softer, Firmer, Protective
- Placeholder text: "Paste the message you're stuck on, or describe what you need to say"
- Generate a response → coaching note appears below separator
- Copy button copies only the reply, not the coaching note
