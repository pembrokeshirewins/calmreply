# CalmReply vNext Wave 1 — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Restructure the CalmReply frontend with progressive disclosure, mode toggle, collapsible context field, Delay Shield, and Short Reply mode. Ship fast, measure behaviour.

**Architecture:** All changes in a single HTML file (index.html). No new backend endpoints. Short Reply is a client-side prompt modifier appended to the userMessage. Delay Shield is a prompt text enhancement applied to the existing Avoiding n8n node. Mode toggle is UI-only in Wave 1 (Interpret Mode shows placeholder).

**Tech Stack:** HTML + CSS + JS (inline, no frameworks). localStorage for state persistence. n8n prompt edit for Delay Shield.

---

### Task 1: Restructure HTML — Progressive Disclosure Layout

**Files:**
- Modify: `index.html:523-591` (the `.card` section)

**Step 1: Replace the card HTML**

Replace everything inside `<div class="card">` (lines 523-591) with the new progressive disclosure structure:

```html
<div class="card">

  <!-- Message Input -->
  <div class="field">
    <label for="message" id="messageLabel">Paste what you received, or describe the situation</label>
    <textarea id="message" rows="6" placeholder="Paste the message you've been avoiding, or describe who you need to reach out to"></textarea>
  </div>

  <!-- Collapsible Context Field -->
  <div class="context-toggle" id="contextToggle">
    <button type="button" id="contextBtn" class="link-btn">+ Add a note (optional)</button>
  </div>
  <div class="context-field" id="contextField" style="display:none;">
    <label for="contextInput" id="contextLabel">Add a note (optional)</label>
    <textarea id="contextInput" rows="3" placeholder="What you want to say, your goal, any constraints..."></textarea>
  </div>

  <!-- Mode Toggle -->
  <div class="mode-toggle" id="modeToggle">
    <button type="button" class="mode-btn active" data-mode="reply" id="modeReply">Reply Mode</button>
    <button type="button" class="mode-btn" data-mode="interpret" id="modeInterpret">Interpret Mode</button>
  </div>

  <!-- Submit -->
  <button class="btn-primary" id="submitBtn" type="button">Help me respond</button>

  <!-- Options Expander -->
  <div class="options-toggle" id="optionsToggle">
    <button type="button" id="optionsBtn" class="link-btn">Options</button>
  </div>
  <div class="options-panel" id="optionsPanel" style="display:none;">
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
    <div class="field">
      <label>Tone</label>
      <div class="tone-options">
        <label class="tone-pill">
          <input type="radio" name="tone" value="softer" checked />
          <span>Softer</span>
        </label>
        <label class="tone-pill">
          <input type="radio" name="tone" value="firmer" />
          <span>Firmer</span>
        </label>
      </div>
    </div>
    <div class="field">
      <label class="checkbox-label">
        <input type="checkbox" id="shortReply" />
        <span>Short reply only</span>
      </label>
    </div>
  </div>

  <!-- Error -->
  <div class="error-msg" id="errorMsg">Something went wrong. Try again in a moment.</div>

  <!-- Interpret Mode Placeholder -->
  <div class="interpret-placeholder" id="interpretPlaceholder" style="display:none;">
    <div class="output-section visible">
      <div class="output-label">Interpret Mode</div>
      <div class="output-text">Coming soon. For now, use Reply Mode to get a response you can send.</div>
    </div>
  </div>

  <!-- Output -->
  <div class="output-section" id="outputSection">
    <div class="output-label">Your suggested response</div>
    <div class="output-text" id="outputText"></div>
    <div class="coaching-note" id="coachingNote"></div>
    <button class="btn-copy" id="copyBtn" type="button">
      <svg width="16" height="16" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><rect x="5.5" y="5.5" width="8" height="8" rx="1.5"/><path d="M10.5 5.5V3.5a1.5 1.5 0 0 0-1.5-1.5H3.5A1.5 1.5 0 0 0 2 3.5V9a1.5 1.5 0 0 0 1.5 1.5h2"/></svg>
      <span id="copyLabel">Copy to clipboard</span>
    </button>
    <p class="response-disclaimer">Always review and personalise before sending.</p>

    <!-- Feedback -->
    <div class="feedback-section" id="feedbackSection">
      <div class="feedback-prompt">Was this response helpful?</div>
      <div class="feedback-buttons">
        <button class="feedback-btn" data-rating="helpful" type="button">&#128077; Helpful</button>
        <button class="feedback-btn" data-rating="not_helpful" type="button">&#128078; Not quite</button>
      </div>
      <div class="feedback-comment" id="feedbackComment">
        <input type="text" id="feedbackInput" placeholder="What could be better?" maxlength="200" />
        <button type="button" id="feedbackSend">Send</button>
      </div>
      <div class="feedback-thanks" id="feedbackThanks">Thanks for the feedback.</div>
    </div>
  </div>
</div>
```

**Key changes from current:**
- Textarea moved to top (was below situation dropdown)
- Situation dropdown + tone pills moved inside Options panel
- Mode toggle added between context and submit
- Context field added (collapsible)
- Options expander added below submit
- Short Reply checkbox added inside Options
- Interpret placeholder added (hidden by default)

**Step 2: Verify HTML renders**

Open `index.html` in browser. Confirm:
- Textarea visible at top
- "+ Add a note" link visible below
- Reply/Interpret toggle visible
- "Help me respond" button visible
- "Options" link visible below button
- Situation/tone/checkbox hidden until Options clicked

**Step 3: Commit**

```bash
git add index.html
git commit -m "refactor: restructure card layout for progressive disclosure"
```

---

### Task 2: Add CSS for New Components

**Files:**
- Modify: `index.html` (CSS section, lines 7-512)

**Step 1: Add CSS rules**

Add after the `.coaching-note.visible` rule (line 503) and before `.response-disclaimer`:

```css
/* ── Mode Toggle ── */
.mode-toggle {
  display: flex;
  gap: 0;
  margin-top: 20px;
  border-radius: 8px;
  overflow: hidden;
  border: 1.5px solid var(--border);
}

.mode-btn {
  flex: 1;
  padding: 10px 16px;
  font-family: inherit;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-light);
  background: var(--card);
  border: none;
  cursor: pointer;
  transition: all 0.2s;
  -webkit-tap-highlight-color: transparent;
}

.mode-btn.active {
  background: var(--accent);
  color: #fff;
}

.mode-btn:not(.active):hover {
  background: #f0f4f7;
}

/* ── Context Field ── */
.context-toggle {
  margin-top: 10px;
}

.link-btn {
  font-family: inherit;
  font-size: 0.85rem;
  color: var(--text-light);
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px 0;
  transition: color 0.2s;
}

.link-btn:hover {
  color: var(--accent);
}

.context-field {
  margin-top: 8px;
}

.context-field textarea {
  min-height: 80px;
  font-size: 0.9rem;
}

.context-field label {
  font-size: 0.8rem;
  font-weight: 500;
  color: var(--text-light);
}

/* ── Options Panel ── */
.options-toggle {
  margin-top: 12px;
  text-align: center;
}

.options-panel {
  margin-top: 12px;
  padding: 16px;
  background: #f7fafb;
  border-radius: 8px;
  border: 1px solid var(--border);
}

.options-panel .field + .field {
  margin-top: 14px;
}

/* ── Short Reply Checkbox ── */
.checkbox-label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 500;
  color: var(--text);
  cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
  width: 18px;
  height: 18px;
  accent-color: var(--accent);
  cursor: pointer;
}

/* ── Interpret Placeholder ── */
.interpret-placeholder {
  margin-top: 24px;
}

.interpret-placeholder .output-text {
  color: var(--text-light);
  font-style: italic;
}
```

**Step 2: Verify styling**

Open in browser at 375px width. Confirm:
- Mode toggle looks like a segmented control (two equal-width buttons)
- Options panel has subtle background when expanded
- Checkbox aligns nicely
- No horizontal overflow

**Step 3: Commit**

```bash
git add index.html
git commit -m "style: add CSS for mode toggle, options panel, context field"
```

---

### Task 3: Add JavaScript — Progressive Disclosure Logic

**Files:**
- Modify: `index.html` (JS section, lines 612-861)

**Step 1: Replace the JS IIFE**

Replace the entire `<script>` block (lines 612-861) with updated JS. Key additions:

```javascript
(function () {
  // ── Config ──
  var WEBHOOK_URL = 'https://aswales.app.n8n.cloud/webhook/calmreply';
  var TRACK_URL = 'https://aswales.app.n8n.cloud/webhook/calmreply-track';
  var EMAIL_WEBHOOK_URL = 'https://aswales.app.n8n.cloud/webhook/calmreply-email';

  // ── Elements ──
  var situationEl = document.getElementById('situation');
  var messageEl = document.getElementById('message');
  var messageLabel = document.getElementById('messageLabel');
  var submitBtn = document.getElementById('submitBtn');
  var outputSection = document.getElementById('outputSection');
  var outputText = document.getElementById('outputText');
  var copyBtn = document.getElementById('copyBtn');
  var copyLabel = document.getElementById('copyLabel');
  var errorMsg = document.getElementById('errorMsg');
  var emailForm = document.getElementById('emailForm');
  var emailInput = document.getElementById('emailInput');
  var emailConfirm = document.getElementById('emailConfirm');
  var feedbackSection = document.getElementById('feedbackSection');
  var feedbackComment = document.getElementById('feedbackComment');
  var feedbackInput = document.getElementById('feedbackInput');
  var feedbackSend = document.getElementById('feedbackSend');
  var feedbackThanks = document.getElementById('feedbackThanks');
  var feedbackBtns = document.querySelectorAll('.feedback-btn');
  var coachingNote = document.getElementById('coachingNote');

  // New elements
  var contextBtn = document.getElementById('contextBtn');
  var contextField = document.getElementById('contextField');
  var contextInput = document.getElementById('contextInput');
  var contextLabel = document.getElementById('contextLabel');
  var contextToggle = document.getElementById('contextToggle');
  var modeReply = document.getElementById('modeReply');
  var modeInterpret = document.getElementById('modeInterpret');
  var optionsBtn = document.getElementById('optionsBtn');
  var optionsPanel = document.getElementById('optionsPanel');
  var shortReply = document.getElementById('shortReply');
  var interpretPlaceholder = document.getElementById('interpretPlaceholder');

  // ── State ──
  var currentMode = 'reply'; // 'reply' or 'interpret'
  var currentRequestId = null;

  // ── Placeholders by mode ──
  var placeholders = {
    reply: {
      avoiding: 'Paste the message you\'ve been avoiding, or describe who you need to reach out to',
      default: 'Paste the message here, or describe what you need to say...'
    },
    interpret: {
      default: 'Paste the message you want help understanding'
    }
  };

  // ── Helpers ──
  function getSelectedTone() {
    var checked = document.querySelector('input[name="tone"]:checked');
    return checked ? checked.value : 'softer';
  }

  function getDeviceType() {
    return window.innerWidth <= 768 ? 'mobile' : 'desktop';
  }

  function generateId() {
    return Date.now().toString(36) + Math.random().toString(36).substr(2, 5);
  }

  function track(data) {
    try {
      fetch(TRACK_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      }).catch(function () {});
    } catch (e) {}
  }

  // ── Local event counters (no backend, sanity-check only) ──
  function bump(key) {
    var k = 'calmreply_count_' + key;
    try { localStorage.setItem(k, (parseInt(localStorage.getItem(k) || '0', 10) + 1).toString()); } catch (e) {}
  }

  // ── localStorage helpers ──
  function lsGet(key) {
    try { return localStorage.getItem('calmreply_' + key); } catch (e) { return null; }
  }
  function lsSet(key, val) {
    try { localStorage.setItem('calmreply_' + key, val); } catch (e) {}
  }

  // ── Update placeholder based on mode + situation ──
  function updatePlaceholder() {
    if (currentMode === 'interpret') {
      messageEl.placeholder = placeholders.interpret.default;
    } else {
      messageEl.placeholder = situationEl.value === 'avoiding'
        ? placeholders.reply.avoiding
        : placeholders.reply.default;
    }
  }

  // ── Update labels and CTA based on mode ──
  function updateModeUI() {
    if (currentMode === 'reply') {
      modeReply.classList.add('active');
      modeInterpret.classList.remove('active');
      submitBtn.textContent = 'Help me respond';
      messageLabel.textContent = 'Paste what you received, or describe the situation';
      contextLabel.textContent = 'Add a note (optional)';
    } else {
      modeInterpret.classList.add('active');
      modeReply.classList.remove('active');
      submitBtn.textContent = 'Help me understand this';
      messageLabel.textContent = 'Paste the message you want help understanding';
      contextLabel.textContent = 'Anything you want me to consider when interpreting this? (optional)';
    }
    updatePlaceholder();
  }

  // ── Context field toggle ──
  function showContext() {
    contextField.style.display = 'block';
    contextToggle.style.display = 'none';
    lsSet('context_expanded', 'true');
  }

  function hideContext() {
    // Never hide if user has typed in it before
    if (lsGet('context_used') === 'true') return;
    contextField.style.display = 'none';
    contextToggle.style.display = 'block';
    lsSet('context_expanded', 'false');
  }

  // ── Options panel toggle ──
  function toggleOptions() {
    var visible = optionsPanel.style.display !== 'none';
    if (!visible) bump('options_opened');
    optionsPanel.style.display = visible ? 'none' : 'block';
    optionsBtn.textContent = visible ? 'Options' : 'Hide options';
    lsSet('options_expanded', visible ? 'false' : 'true');
  }

  // ── Restore localStorage state ──
  if (lsGet('context_expanded') === 'true' || lsGet('context_used') === 'true') {
    showContext();
  }
  if (lsGet('options_expanded') === 'true') {
    optionsPanel.style.display = 'block';
    optionsBtn.textContent = 'Hide options';
  }

  // ── Event Listeners — UI controls ──
  contextBtn.addEventListener('click', showContext);

  modeReply.addEventListener('click', function () {
    currentMode = 'reply';
    updateModeUI();
  });

  modeInterpret.addEventListener('click', function () {
    currentMode = 'interpret';
    updateModeUI();
  });

  optionsBtn.addEventListener('click', toggleOptions);

  situationEl.addEventListener('change', updatePlaceholder);

  // ── Submit Handler ──
  submitBtn.addEventListener('click', function () {
    var message = messageEl.value.trim();
    if (!message) {
      messageEl.focus();
      return;
    }

    bump('generate_clicked');

    // Interpret Mode — show placeholder for now (Wave 2)
    if (currentMode === 'interpret') {
      outputSection.classList.remove('visible');
      interpretPlaceholder.style.display = 'block';
      interpretPlaceholder.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
      return;
    }

    // Mark context as used if they typed something
    var contextText = contextInput ? contextInput.value.trim() : '';
    if (contextText) {
      lsSet('context_used', 'true');
    }

    // Reset state
    errorMsg.classList.remove('visible');
    outputSection.classList.remove('visible');
    interpretPlaceholder.style.display = 'none';
    coachingNote.classList.remove('visible');
    feedbackSection.classList.remove('visible');
    feedbackComment.classList.remove('visible');
    feedbackThanks.classList.remove('visible');
    feedbackBtns.forEach(function (btn) { btn.classList.remove('selected'); });
    submitBtn.disabled = true;
    submitBtn.textContent = 'Finding the right words\u2026';

    currentRequestId = generateId();
    var situationType = situationEl.value;
    var tone = getSelectedTone();

    // Build the user message with optional context and short reply modifier
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

    var controller = new AbortController();
    var timeout = setTimeout(function () { controller.abort(); }, 15000);

    fetch(WEBHOOK_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
      signal: controller.signal
    })
      .then(function (res) {
        clearTimeout(timeout);
        if (!res.ok) throw new Error('Server error');
        return res.json();
      })
      .then(function (data) {
        var result = Array.isArray(data) ? data[0] : data;
        if (!result || !result.response) throw new Error('Empty response');

        var fullResponse = result.response;
        coachingNote.classList.remove('visible');
        coachingNote.textContent = '';

        // Split coaching note for "avoiding" type
        if (situationType === 'avoiding' && fullResponse.indexOf('\n---\n') !== -1) {
          var parts = fullResponse.split('\n---\n');
          outputText.textContent = parts[0].trim();
          coachingNote.textContent = parts.slice(1).join('\n---\n').trim();
          coachingNote.classList.add('visible');
        } else {
          outputText.textContent = fullResponse;
        }

        outputSection.classList.add('visible');
        feedbackSection.classList.add('visible');
        outputSection.scrollIntoView({ behavior: 'smooth', block: 'nearest' });

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
      })
      .catch(function () {
        errorMsg.classList.add('visible');
        track({
          event: 'error',
          requestId: currentRequestId,
          timestamp: new Date().toISOString(),
          situationType: situationType,
          tone: tone,
          device: getDeviceType()
        });
      })
      .finally(function () {
        submitBtn.disabled = false;
        submitBtn.textContent = 'Help me respond';
      });
  });

  // ── Feedback Handlers (unchanged logic) ──
  feedbackBtns.forEach(function (btn) {
    btn.addEventListener('click', function () {
      var rating = btn.getAttribute('data-rating');
      feedbackBtns.forEach(function (b) { b.classList.remove('selected'); });
      btn.classList.add('selected');

      if (rating === 'not_helpful') {
        feedbackComment.classList.add('visible');
        feedbackInput.focus();
      } else {
        feedbackComment.classList.remove('visible');
        track({
          event: 'feedback',
          requestId: currentRequestId,
          timestamp: new Date().toISOString(),
          rating: rating,
          comment: ''
        });
        setTimeout(function () {
          feedbackSection.innerHTML = '';
          feedbackThanks.classList.add('visible');
          feedbackSection.appendChild(feedbackThanks);
        }, 300);
      }
    });
  });

  feedbackSend.addEventListener('click', function () {
    var comment = feedbackInput.value.trim();
    track({
      event: 'feedback',
      requestId: currentRequestId,
      timestamp: new Date().toISOString(),
      rating: 'not_helpful',
      comment: comment
    });
    feedbackComment.classList.remove('visible');
    feedbackBtns.forEach(function (b) { b.style.display = 'none'; });
    feedbackThanks.classList.add('visible');
  });

  // ── Copy Handler ──
  copyBtn.addEventListener('click', function () {
    var text = outputText.textContent;
    if (!text) return;
    bump('copy_clicked');

    navigator.clipboard.writeText(text).then(function () {
      copyBtn.classList.add('copied');
      copyLabel.textContent = 'Copied \u2713';
      track({
        event: 'copy',
        requestId: currentRequestId,
        timestamp: new Date().toISOString()
      });
      setTimeout(function () {
        copyBtn.classList.remove('copied');
        copyLabel.textContent = 'Copy to clipboard';
      }, 2000);
    });
  });

  // ── Email Capture (unchanged) ──
  emailForm.addEventListener('submit', function (e) {
    e.preventDefault();
    var email = emailInput.value.trim();
    if (!email) return;

    fetch(EMAIL_WEBHOOK_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: email })
    })
      .then(function () {
        emailForm.style.display = 'none';
        emailConfirm.classList.add('visible');
        track({ event: 'email_signup', timestamp: new Date().toISOString() });
      })
      .catch(function () {
        emailForm.style.display = 'none';
        emailConfirm.classList.add('visible');
      });
  });

  // ── Initialise ──
  updateModeUI();
  track({
    event: 'pageview',
    timestamp: new Date().toISOString(),
    device: getDeviceType(),
    referrer: document.referrer || 'direct'
  });
})();
```

**Key JS changes:**
- Mode toggle: switches `currentMode`, updates UI labels/placeholders/CTA
- Context field: collapsible with localStorage persistence + auto-promote
- Options panel: collapsible with localStorage persistence
- Short Reply: checkbox value appended as `[SHORT REPLY MODE: ...]` instruction in userMessage (client-side, no backend change)
- Context: appended as `[USER CONTEXT: ...]` in userMessage
- Interpret Mode: shows placeholder message, no API call (Wave 2)
- Tracking: new fields (mode, shortReply, contextProvided)
- **Local event counters** (localStorage only, no backend) for sanity-checking behaviour changes

**Local event counters:**

Add a lightweight `bump()` helper and call it at each interaction point:

```javascript
// ── Local event counters (no backend, sanity-check only) ──
function bump(key) {
  var k = 'calmreply_count_' + key;
  try { localStorage.setItem(k, (parseInt(localStorage.getItem(k) || '0', 10) + 1).toString()); } catch (e) {}
}
```

Call sites:
- `bump('generate_clicked')` — inside the submit handler, before the fetch call
- `bump('copy_clicked')` — inside the copy handler, before `navigator.clipboard.writeText`
- `bump('options_opened')` — inside `toggleOptions()`, only on the open transition (when `visible` was false)

Counts accessible via browser console: `localStorage.getItem('calmreply_count_generate_clicked')` etc.

**Step 2: Full browser verification**

Open `index.html` at 375px width. Test each:
1. Default view: only textarea, context link, mode toggle, CTA visible
2. Click "+ Add a note" → context textarea appears, link hides
3. Reload → context stays expanded (localStorage)
4. Click "Options" → situation/tone/checkbox appear
5. Click "Hide options" → they collapse
6. Click "Interpret Mode" → CTA changes to "Help me understand this", placeholder changes
7. Click "Reply Mode" → CTA changes back
8. Type message, click "Help me respond" → existing flow works
9. Check "Short reply only", generate → verify shorter output from API
10. Click "Help me understand this" in Interpret Mode → shows "Coming soon" placeholder
11. Open console, check `localStorage.getItem('calmreply_count_generate_clicked')` → should match number of times you clicked Generate
12. Check `calmreply_count_copy_clicked` and `calmreply_count_options_opened` similarly

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: progressive disclosure JS — mode toggle, context field, short reply, options panel"
```

---

### Task 4: Update Avoiding Prompt — Delay Shield

**Files:**
- Modify: `docs/prompts/calmreply-prompts.md` (NODE 7: AVOIDING section)
- Action: Paste updated prompt into n8n OpenAI - Avoiding node

**Step 1: Add Delay Shield block to Avoiding prompt**

In the NODE 7: AVOIDING Instructions field, add after the existing tone modifier section:

```
DELAY SHIELD:
When the user has been avoiding a reply, include a brief opener that:
- Acknowledges the delay in 1 short sentence max (≤15 words)
- Never grovels, never stacks apologies
- Creates forward momentum immediately

Choose ONE approach based on context:
- Acknowledge-and-pivot: e.g. "Thanks for your patience — here's where I'm at."
- Normalise-the-gap: e.g. "I wanted to give this proper thought before replying."
- Skip-it-entirely: If the delay is ambiguous or short, just reply naturally with no opener.

HARD RULE: The delay opener must be ≤15 words. One sentence. Then get to the substance.

If [SHORT REPLY MODE] is present in the user message: the delay acknowledgment is one clause woven into the single reply sentence, not a separate opener. The entire reply must be ≤40 words (hard cap 60).

If [USER CONTEXT] is present in the user message: use it as guidance for what the user wants to say. Do not repeat it back to them.
```

**Step 2: Update the prompts doc**

Add the Delay Shield block to `docs/prompts/calmreply-prompts.md` in the NODE 7 section.

**Step 3: Apply to n8n**

Paste the full updated Instructions field into the n8n OpenAI - Avoiding node. This can be done via the n8n UI or API.

**Step 4: Add [SHORT REPLY MODE] and [USER CONTEXT] handling to ALL 7 nodes**

Add to the end of every node's Instructions field (all 7 situation types):

```
If [SHORT REPLY MODE] is present in the user message: write a reply of 40 words or fewer. One paragraph only. Must still contain an acknowledgment and a clear action or ask. Hard cap 60 words. No filler, no "I hope this finds you well." No sign-offs unless the input had one.

If [USER CONTEXT] is present in the user message: use it as guidance for tone, intent, or constraints. Do not quote it back. Do not reference it explicitly.
```

**Step 5: Curl test — Delay Shield**

```bash
curl -X POST https://aswales.app.n8n.cloud/webhook/calmreply \
  -H "Content-Type: application/json" \
  -d '{"situationType":"avoiding","userMessage":"Hey, are we still on for Thursday? I sent you the details last week.","tone":"softer"}'
```

Verify: response has a brief delay acknowledgment opener (≤15 words), then substance.

**Step 6: Curl test — Short Reply**

```bash
curl -X POST https://aswales.app.n8n.cloud/webhook/calmreply \
  -H "Content-Type: application/json" \
  -d '{"situationType":"avoiding","userMessage":"Hey, are we still on for Thursday?\n\n[SHORT REPLY MODE: Write a reply of 40 words or fewer. One paragraph. Must contain an acknowledgment and a clear action or ask. Hard cap 60 words. No filler, no sign-offs unless the input had one.]","tone":"firmer"}'
```

Verify: response is ≤60 words, one paragraph.

**Step 7: Commit**

```bash
git add docs/prompts/calmreply-prompts.md
git commit -m "feat: add Delay Shield and SHORT REPLY/USER CONTEXT handling to all prompts"
```

---

### Task 5: Copy to Public and Deploy

**Files:**
- Copy: `index.html` → `public/index.html`

**Step 1: Copy**

```bash
cp index.html public/index.html
```

**Step 2: Final verification at 375px**

Open `public/index.html` in browser. Full walkthrough:
1. Page loads with minimal controls (textarea, mode toggle, CTA)
2. Paste a message, click "Help me respond" — works
3. Toggle Short Reply, regenerate — shorter output
4. Expand context field, type a note, regenerate — output respects context
5. Switch to Interpret Mode — shows placeholder
6. Options panel expands/collapses, state persists across reload
7. Context field state persists across reload
8. All 7 situation types still work via Options panel
9. Copy button works
10. Feedback widget works
11. Email capture works
12. No horizontal scroll at 375px

**Step 3: Commit and push**

```bash
git add index.html public/index.html
git commit -m "feat: CalmReply vNext Wave 1 — progressive disclosure, mode toggle, short reply, delay shield"
git push origin main
```

Vercel auto-deploys from `/public` on push.

**Step 4: Verify live**

Visit https://calm-reply.com and repeat the walkthrough from Step 2.

---

### Task 6: Update Project Docs

**Files:**
- Modify: `CLAUDE.md` (update current state)
- Modify: `CONVERSATION-LOG.md` (add session entry)
- Modify: `memory/projects/calmreply.md` (update status)

**Step 1: Update CLAUDE.md**

- Add vNext Wave 1 features to "What's built and working"
- Add Wave 2/3 to roadmap section
- Update frontend specification to reflect new layout

**Step 2: Update CONVERSATION-LOG.md**

Add Session 8 entry documenting Wave 1 implementation.

**Step 3: Commit**

```bash
git add CLAUDE.md CONVERSATION-LOG.md memory/projects/calmreply.md
git commit -m "docs: update project docs for vNext Wave 1"
```

---

## Summary

| Task | What | Commit |
|------|------|--------|
| 1 | Restructure HTML layout | `refactor: restructure card layout for progressive disclosure` |
| 2 | Add CSS for new components | `style: add CSS for mode toggle, options panel, context field` |
| 3 | Add JS for all interactive behaviour | `feat: progressive disclosure JS — mode toggle, context field, short reply, options panel` |
| 4 | Update prompts (Delay Shield + SHORT REPLY + USER CONTEXT) | `feat: add Delay Shield and SHORT REPLY/USER CONTEXT handling to all prompts` |
| 5 | Copy to public, deploy, verify live | `feat: CalmReply vNext Wave 1` |
| 6 | Update project docs | `docs: update project docs for vNext Wave 1` |

**Total: 6 tasks, 6 commits. All frontend except Task 4 (prompt edit).**
