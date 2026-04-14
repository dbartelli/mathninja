# MathNinja Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single self-contained `index.html` math facts practice app for 2nd/3rd graders, hosted on GitHub Pages, covering multiplication and division of numbers 0–12 with gamification and audio feedback.

**Architecture:** One HTML file — all CSS, JavaScript, and audio synthesis inline. Three screens (Settings, Practice, Results) are shown/hidden with CSS. No build tools, no dependencies, no external requests.

**Tech Stack:** Vanilla HTML/CSS/JavaScript, Web Audio API, localStorage.

---

## File Structure

```
index.html          ← entire app (HTML + inline CSS + inline JS)
```

That's it. All tasks modify this one file.

---

## Global State Shape

Every task references these two objects. `settings` is defined in Task 3; `session` is defined in Task 7.

```javascript
// Persisted to/from localStorage
let settings = {
  operation: 'multiply',   // 'multiply' | 'divide' | 'mix'
  numbers: [2, 3, 4, 6],
  sessionType: 'problems', // 'problems' | 'time'
  sessionValue: 20,
  muted: false,
};

// Live session state — reset on each Enter Dojo
let session = {
  deck: [],           // shuffled array of problem objects
  deckIndex: 0,       // current position in deck
  tries: 0,           // tries on current problem (0 before first submission)
  streak: 0,
  bestStreak: 0,
  firstTryCorrect: 0,
  secondTryCorrect: 0,
  revealed: 0,
  revealedFacts: [],  // e.g. ['7×8', '56÷8'] for results screen
  attempted: 0,       // total problems attempted this session
  timeRemaining: 0,   // seconds, for time mode
  timerInterval: null,
  timerExpired: false,
  inputLocked: false,
  msgPool: [],        // shuffled message indices, refilled when exhausted
};
```

**Problem object shape:**
```javascript
{
  type: 'multiply',   // 'multiply' | 'divide'
  a: 7,              // first displayed operand
  b: 8,              // second displayed operand
  answer: 56,
  display: '7 × 8',  // shown in problem area
  key: '7×8',        // used in revealedFacts
}
```

---

## Task 1: Repo, HTML skeleton, and GitHub Pages

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the GitHub repository**

  Go to github.com → New repository → name it `mathninja` → Public → Create.
  Then clone locally:
  ```bash
  cd /Users/davebartelli/Projects/mathninja
  git init
  git remote add origin https://github.com/<your-username>/mathninja.git
  ```

- [ ] **Step 2: Create `index.html` with dark theme skeleton and three empty screens**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="MathNinja">
    <title>MathNinja</title>
    <style>
      :root {
        --bg:    #1a1a2e;
        --bg2:   #16213e;
        --bg3:   #2a2a4e;
        --red:   #e94560;
        --teal:  #00b4d8;
        --gold:  #ffd166;
        --green: #4ade80;
        --text:  #e0e0e0;
        --muted: #555555;
        --font:  system-ui, -apple-system, sans-serif;
      }
      *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
      html, body {
        height: 100%;
        background: var(--bg);
        color: var(--text);
        font-family: var(--font);
        -webkit-tap-highlight-color: transparent;
      }
      .screen { display: none; min-height: 100vh; }
      .screen.active { display: block; }
      .container {
        max-width: 600px;
        margin: 0 auto;
        padding: 24px 20px 40px;
      }
    </style>
  </head>
  <body>

    <div id="screen-settings" class="screen active">
      <div class="container">
        <p style="color:var(--teal)">Settings screen — coming soon</p>
      </div>
    </div>

    <div id="screen-practice" class="screen">
      <div class="container">
        <p style="color:var(--teal)">Practice screen — coming soon</p>
      </div>
    </div>

    <div id="screen-results" class="screen">
      <div class="container">
        <p style="color:var(--teal)">Results screen — coming soon</p>
      </div>
    </div>

    <script>
      function showScreen(id) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById(id).classList.add('active');
      }
    </script>

  </body>
  </html>
  ```

- [ ] **Step 3: Verify skeleton in browser**

  Open `index.html` in Safari. You should see a dark page with the text "Settings screen — coming soon" in teal. No errors in console.

- [ ] **Step 4: Push and enable GitHub Pages**

  ```bash
  git add index.html
  git commit -m "feat: project scaffold with dark theme skeleton"
  git branch -M main
  git push -u origin main
  ```

  Then: GitHub repo → Settings → Pages → Branch: `main` / `/ (root)` → Save.
  URL will be `https://<username>.github.io/mathninja` — takes ~60 seconds to go live.

- [ ] **Step 5: Verify GitHub Pages URL loads in Safari on iPad**

  Open the URL on iPad Safari. Should show the dark skeleton. Bookmark it or use "Add to Home Screen."

---

## Task 2: Settings screen — HTML and CSS

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the settings screen placeholder with full HTML**

  Replace the contents of `<div id="screen-settings">` with:

  ```html
  <div id="screen-settings" class="screen active">
    <div class="container">

      <!-- Branding -->
      <div class="brand">
        <div class="brand-ninja">🥷</div>
        <h1 class="brand-title">MATHNINJA</h1>
        <p class="brand-tagline">⚔️ Welcome to Diego's Math Dojo. Select your training. ⚔️</p>
      </div>
      <hr class="divider">

      <!-- Operation mode -->
      <div class="setting-section">
        <div class="setting-label">⚡ Operation</div>
        <div class="toggle-group" id="op-toggle">
          <button class="toggle-btn active" data-op="multiply">✖️ Multiply</button>
          <button class="toggle-btn" data-op="divide">➗ Divide</button>
          <button class="toggle-btn" data-op="mix">🔀 Mix</button>
        </div>
      </div>

      <!-- Numbers to train -->
      <div class="setting-section">
        <div class="setting-label">🎯 Numbers to Train</div>
        <div class="number-grid" id="number-grid"></div>
        <p class="grid-hint">Select at least 2 numbers</p>
      </div>

      <!-- Session length -->
      <div class="setting-section">
        <div class="setting-label">⏱ Session Length</div>
        <div class="session-row">
          <div class="toggle-group" id="st-toggle">
            <button class="toggle-btn active" data-st="problems">📋 Problems</button>
            <button class="toggle-btn" data-st="time">⏱ Time</button>
          </div>
          <div class="stepper">
            <button class="stepper-btn" id="stepper-minus">−</button>
            <span class="stepper-val" id="session-display">20</span>
            <button class="stepper-btn" id="stepper-plus">+</button>
          </div>
        </div>
      </div>

      <!-- Enter Dojo -->
      <button class="dojo-btn" id="dojo-btn">⚔️ ENTER DOJO ⚔️</button>
      <div class="sound-row" id="sound-toggle-row">
        <span id="sound-icon">🔊</span>
        <span id="sound-label">Sound On</span>
      </div>

    </div>
  </div>
  ```

- [ ] **Step 2: Add settings screen CSS inside the `<style>` block**

  ```css
  /* ── Branding ── */
  .brand { text-align: center; padding: 12px 0 20px; }
  .brand-ninja { font-size: 52px; line-height: 1; margin-bottom: 6px; }
  .brand-title {
    font-size: 32px; font-weight: 900; letter-spacing: 4px;
    color: var(--gold); text-transform: uppercase;
  }
  .brand-tagline {
    font-size: 13px; font-weight: 600; color: var(--red);
    letter-spacing: 0.5px; margin-top: 8px;
  }
  .divider { border: none; border-top: 1px solid var(--bg3); margin: 0 0 24px; }

  /* ── Setting sections ── */
  .setting-section { margin-bottom: 24px; }
  .setting-label {
    font-size: 11px; font-weight: 700; letter-spacing: 2px;
    text-transform: uppercase; color: var(--teal); margin-bottom: 10px;
  }

  /* ── Toggle buttons ── */
  .toggle-group { display: flex; gap: 8px; }
  .toggle-btn {
    flex: 1; background: var(--bg2); color: var(--muted);
    border: 2px solid var(--bg3); border-radius: 10px;
    padding: 12px 8px; font-size: 14px; font-weight: 800;
    font-family: var(--font); cursor: pointer;
    min-height: 44px; transition: all 0.15s;
  }
  .toggle-btn.active {
    background: var(--red); color: white; border-color: var(--red);
  }

  /* ── Number grid ── */
  .number-grid {
    display: grid; grid-template-columns: repeat(7, 1fr); gap: 8px;
  }
  .num-btn {
    background: var(--bg2); color: var(--muted);
    border: 2px solid var(--bg3); border-radius: 8px;
    padding: 10px 0; font-size: 17px; font-weight: 800;
    font-family: var(--font); cursor: pointer; text-align: center;
    min-height: 44px; transition: all 0.15s;
  }
  .num-btn.selected { background: var(--red); color: white; border-color: var(--red); }
  .grid-hint { font-size: 11px; color: var(--muted); text-align: center; margin-top: 8px; }

  /* ── Session stepper ── */
  .session-row { display: flex; gap: 12px; align-items: center; }
  .stepper { display: flex; align-items: center; gap: 10px; }
  .stepper-btn {
    background: var(--bg2); color: var(--text); border: 2px solid var(--bg3);
    border-radius: 8px; width: 44px; height: 44px; font-size: 22px;
    font-weight: 700; font-family: var(--font); cursor: pointer; line-height: 1;
  }
  .stepper-val { font-size: 22px; font-weight: 900; min-width: 48px; text-align: center; }

  /* ── Enter Dojo button ── */
  .dojo-btn {
    width: 100%; padding: 18px; font-size: 20px; font-weight: 900;
    font-family: var(--font); letter-spacing: 2px; color: white;
    background: linear-gradient(135deg, var(--red), #c23152);
    border: none; border-radius: 14px; cursor: pointer; margin-top: 8px;
    box-shadow: 0 4px 20px rgba(233,69,96,0.4);
    min-height: 56px; transition: opacity 0.15s;
  }
  .dojo-btn:disabled { opacity: 0.35; cursor: not-allowed; box-shadow: none; }

  /* ── Sound toggle ── */
  .sound-row {
    text-align: center; margin-top: 14px; color: var(--muted);
    font-size: 12px; cursor: pointer; user-select: none;
  }
  ```

- [ ] **Step 3: Verify layout in browser**

  Open `index.html`. You should see: 🥷 branding, three sections with placeholder buttons/grid area, disabled-looking Enter Dojo button, mute row. No JS wired yet — nothing interactive. No console errors.

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: settings screen HTML and CSS"
  ```

---

## Task 3: Settings screen — JavaScript interactivity

**Files:**
- Modify: `index.html` (add to `<script>` block)

- [ ] **Step 1: Add the settings state and initialization**

  Inside `<script>`, after the `showScreen` function:

  ```javascript
  // ── Settings state ──────────────────────────────────────────
  let settings = {
    operation: 'multiply',
    numbers: [2, 3, 4, 6],
    sessionType: 'problems',
    sessionValue: 20,
    muted: false,
  };

  // ── Boot ─────────────────────────────────────────────────────
  document.addEventListener('DOMContentLoaded', () => {
    loadSettings();
    renderNumberGrid();
    syncSettingsUI();
  });
  ```

- [ ] **Step 2: Add number grid rendering and toggle logic**

  ```javascript
  function renderNumberGrid() {
    const grid = document.getElementById('number-grid');
    grid.innerHTML = '';
    for (let n = 0; n <= 12; n++) {
      const btn = document.createElement('button');
      btn.className = 'num-btn' + (settings.numbers.includes(n) ? ' selected' : '');
      btn.textContent = n;
      btn.dataset.num = n;
      btn.addEventListener('click', () => toggleNumber(n));
      grid.appendChild(btn);
    }
    updateDojoBtn();
  }

  function toggleNumber(n) {
    if (settings.numbers.includes(n)) {
      if (settings.numbers.length <= 2) return; // enforce minimum
      settings.numbers = settings.numbers.filter(x => x !== n);
    } else {
      settings.numbers = [...settings.numbers, n].sort((a, b) => a - b);
    }
    renderNumberGrid();
  }

  function updateDojoBtn() {
    document.getElementById('dojo-btn').disabled = settings.numbers.length < 2;
  }
  ```

- [ ] **Step 3: Add operation mode and session type toggle logic**

  ```javascript
  function syncSettingsUI() {
    // Operation toggle
    document.querySelectorAll('#op-toggle .toggle-btn').forEach(btn => {
      btn.classList.toggle('active', btn.dataset.op === settings.operation);
    });
    // Session type toggle
    document.querySelectorAll('#st-toggle .toggle-btn').forEach(btn => {
      btn.classList.toggle('active', btn.dataset.st === settings.sessionType);
    });
    // Session value display
    const unit = settings.sessionType === 'time' ? ' min' : '';
    document.getElementById('session-display').textContent = settings.sessionValue + unit;
    // Sound
    document.getElementById('sound-icon').textContent = settings.muted ? '🔇' : '🔊';
    document.getElementById('sound-label').textContent = settings.muted ? 'Sound Off' : 'Sound On';
  }

  // Wire operation buttons
  document.querySelectorAll('#op-toggle .toggle-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      settings.operation = btn.dataset.op;
      syncSettingsUI();
    });
  });

  // Wire session type buttons
  document.querySelectorAll('#st-toggle .toggle-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      settings.sessionType = btn.dataset.st;
      settings.sessionValue = btn.dataset.st === 'problems' ? 20 : 3;
      syncSettingsUI();
    });
  });

  // Wire stepper
  document.getElementById('stepper-minus').addEventListener('click', () => adjustSession(-1));
  document.getElementById('stepper-plus').addEventListener('click', () => adjustSession(1));

  function adjustSession(dir) {
    if (settings.sessionType === 'problems') {
      settings.sessionValue = Math.min(50, Math.max(5, settings.sessionValue + dir * 5));
    } else {
      settings.sessionValue = Math.min(10, Math.max(1, settings.sessionValue + dir));
    }
    syncSettingsUI();
  }

  // Wire sound toggle
  document.getElementById('sound-toggle-row').addEventListener('click', () => {
    settings.muted = !settings.muted;
    syncSettingsUI();
  });
  ```

- [ ] **Step 4: Verify interactivity**

  Open `index.html`. Confirm:
  - Tapping operation buttons highlights the active one in red
  - Tapping number buttons toggles red highlight; can't deselect when only 2 remain
  - Enter Dojo is grayed when < 2 numbers; enabled when ≥ 2
  - − and + buttons change the number shown; Problems steps by 5 (5–50), Time by 1 (1–10)
  - Sound toggle switches 🔊/🔇

- [ ] **Step 5: Commit**

  ```bash
  git add index.html
  git commit -m "feat: settings screen interactivity"
  ```

---

## Task 4: localStorage settings persistence

**Files:**
- Modify: `index.html` (add to `<script>` block)

- [ ] **Step 1: Add save/load functions**

  ```javascript
  const LS_SETTINGS = 'mn_lastSettings';
  const LS_MUTED    = 'mn_muted';

  function saveSettings() {
    try {
      localStorage.setItem(LS_SETTINGS, JSON.stringify({
        operation:    settings.operation,
        numbers:      settings.numbers,
        sessionType:  settings.sessionType,
        sessionValue: settings.sessionValue,
      }));
      localStorage.setItem(LS_MUTED, settings.muted ? '1' : '0');
    } catch(e) { /* localStorage unavailable — silently skip */ }
  }

  function loadSettings() {
    try {
      const raw = localStorage.getItem(LS_SETTINGS);
      if (raw) {
        const s = JSON.parse(raw);
        if (s.operation)    settings.operation    = s.operation;
        if (Array.isArray(s.numbers) && s.numbers.length >= 2)
                            settings.numbers      = s.numbers;
        if (s.sessionType)  settings.sessionType  = s.sessionType;
        if (s.sessionValue) settings.sessionValue = s.sessionValue;
      }
      settings.muted = localStorage.getItem(LS_MUTED) === '1';
    } catch(e) { /* ignore */ }
  }
  ```

- [ ] **Step 2: Note — dojo-btn is wired in Task 7**

  `saveSettings()` is called inside `enterDojo()` which is defined and wired to the button in Task 7. No event listener is added here — do not add one yet.

- [ ] **Step 3: Verify persistence**

  Open `index.html`, change operation to "Mix", deselect 4 and 6. Refresh the page. Settings should restore to Mix mode with {2,3} selected. Check DevTools → Application → localStorage to see `mn_lastSettings`.

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: localStorage settings persistence"
  ```

---

## Task 5: Problem generation engine

**Files:**
- Modify: `index.html` (add to `<script>` block)

- [ ] **Step 1: Add the shuffle utility and problem generators**

  ```javascript
  // ── Utilities ────────────────────────────────────────────────
  function shuffle(arr) {
    const a = [...arr];
    for (let i = a.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [a[i], a[j]] = [a[j], a[i]];
    }
    return a;
  }

  // ── Problem generation ───────────────────────────────────────
  function buildMultiplyProblems(numbers) {
    const problems = [];
    for (const a of numbers) {
      for (const b of numbers) {
        problems.push({ type:'multiply', a, b, answer: a*b,
          display:`${a} × ${b}`, key:`${a}×${b}` });
      }
    }
    return problems;
  }

  function buildDivideProblems(numbers) {
    const problems = [];
    const divisors = numbers.filter(n => n !== 0); // never divide by 0
    for (const d of divisors) {
      for (const f of numbers) {
        const dividend = d * f;
        problems.push({ type:'divide', a:dividend, b:d, answer:f,
          display:`${dividend} ÷ ${d}`, key:`${dividend}÷${d}` });
      }
    }
    return problems;
  }

  function buildDeck(operation, numbers) {
    let problems = [];
    if (operation === 'multiply') problems = buildMultiplyProblems(numbers);
    else if (operation === 'divide') problems = buildDivideProblems(numbers);
    else { // mix
      problems = [
        ...buildMultiplyProblems(numbers),
        ...buildDivideProblems(numbers),
      ];
    }
    return shuffle(problems);
  }
  ```

- [ ] **Step 2: Verify problem generation in browser console**

  Open `index.html`, open DevTools console. Run:
  ```javascript
  buildDeck('multiply', [2, 3]).slice(0, 5).map(p => p.display)
  // Expected: 5 problems like ["2 × 2","3 × 2","2 × 3","3 × 3","2 × 2"] (shuffled)

  buildDeck('divide', [2, 3]).slice(0, 5).map(p => p.display)
  // Expected: problems like ["4 ÷ 2", "9 ÷ 3", "6 ÷ 2", ...] — no division by zero

  buildDeck('divide', [0, 3]).map(p => p.display)
  // Expected: only problems with divisor 3 (0÷3, 9÷3) — 0 is NOT a divisor
  ```

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: problem generation engine with deck-of-cards shuffle"
  ```

---

## Task 6: Practice screen — layout and CSS

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the practice screen placeholder with full HTML**

  Replace `<div id="screen-practice">` with:

  ```html
  <div id="screen-practice" class="screen">
    <div class="container">

      <!-- Top bar -->
      <div class="top-bar">
        <button class="icon-btn" id="exit-btn">⚙️</button>
        <div class="streak-badge" id="streak-badge">
          🔥 <span id="streak-count">0</span>
        </div>
        <div class="progress-label" id="progress-label">0 / 20</div>
      </div>

      <!-- Progress bar -->
      <div class="prog-track">
        <div class="prog-fill" id="prog-fill"></div>
      </div>

      <!-- Problem area -->
      <div class="problem-area">
        <div class="op-label" id="op-label">MULTIPLICATION</div>
        <div class="problem-text" id="problem-text">7 × 8</div>
        <div class="problem-eq" id="problem-eq">= ?</div>

        <input type="text" id="answer-input" class="answer-input"
          inputmode="numeric" autocomplete="off" autocorrect="off"
          spellcheck="false" maxlength="3" placeholder="?">
        <div class="input-hint" id="input-hint">type your answer</div>

        <div class="feedback-bubble" id="feedback-bubble">
          <div class="fb-main" id="fb-main"></div>
          <div class="fb-sub"  id="fb-sub"></div>
        </div>
      </div>

    </div>

    <!-- Exit confirmation overlay -->
    <div class="overlay" id="exit-overlay">
      <div class="overlay-box">
        <p class="overlay-title">Leave session?</p>
        <p class="overlay-sub">Your progress will be lost.</p>
        <div class="overlay-btns">
          <button class="overlay-cancel" id="overlay-cancel">Keep Training</button>
          <button class="overlay-leave"  id="overlay-leave">Leave</button>
        </div>
      </div>
    </div>
  </div>
  ```

- [ ] **Step 2: Add practice screen CSS**

  ```css
  /* ── Top bar ── */
  .top-bar {
    display: flex; align-items: center; justify-content: space-between;
    margin-bottom: 10px;
  }
  .icon-btn {
    background: none; border: none; font-size: 22px; cursor: pointer;
    padding: 8px; min-width: 44px; min-height: 44px;
  }
  .streak-badge {
    background: var(--bg3); border-radius: 20px; padding: 4px 14px;
    font-weight: 800; font-size: 14px; color: var(--gold);
    transition: opacity 0.2s;
  }
  .streak-badge.hidden { opacity: 0; pointer-events: none; }
  .progress-label { font-size: 13px; font-weight: 700; color: var(--muted); }

  /* ── Progress bar ── */
  .prog-track {
    background: var(--bg2); border-radius: 4px; height: 6px;
    margin-bottom: 36px; overflow: hidden;
  }
  .prog-fill {
    height: 100%; border-radius: 4px; width: 0%;
    background: linear-gradient(90deg, var(--red), var(--gold));
    transition: width 0.3s ease, background 0.5s;
  }
  .prog-fill.urgent { background: var(--red); }

  /* ── Problem area ── */
  .problem-area {
    display: flex; flex-direction: column; align-items: center;
    padding-top: 16px;
  }
  .op-label {
    font-size: 11px; font-weight: 700; letter-spacing: 3px;
    color: var(--muted); text-transform: uppercase; margin-bottom: 12px;
  }
  .problem-text {
    font-size: clamp(52px, 12vw, 72px); font-weight: 900;
    letter-spacing: 4px; line-height: 1; color: var(--text);
  }
  .problem-eq {
    font-size: clamp(52px, 12vw, 72px); font-weight: 900;
    letter-spacing: 4px; line-height: 1; color: var(--red);
    margin-bottom: 32px;
  }
  .problem-eq.correct { color: var(--green); }
  .problem-eq.reveal  { color: var(--gold);  }

  /* ── Answer input ── */
  .answer-input {
    background: var(--bg2); border: 3px solid var(--teal);
    border-radius: 14px; width: 160px; padding: 14px;
    font-size: 36px; font-weight: 900; font-family: var(--font);
    color: var(--teal); text-align: center;
    outline: none; transition: border-color 0.2s;
    -webkit-appearance: none;
  }
  .answer-input.shake { animation: shake 0.3s; }
  .answer-input:focus { border-color: var(--teal); }
  @keyframes shake {
    0%,100%{transform:translateX(0)}
    20%{transform:translateX(-6px)}
    40%{transform:translateX(6px)}
    60%{transform:translateX(-6px)}
    80%{transform:translateX(3px)}
  }
  .input-hint { font-size: 11px; color: var(--muted); margin-top: 8px; }

  /* ── Feedback bubble ── */
  .feedback-bubble {
    margin-top: 24px; border-radius: 14px; padding: 14px 20px;
    text-align: center; min-width: 240px; opacity: 0;
    transition: opacity 0.2s;
  }
  .feedback-bubble.visible { opacity: 1; }
  .feedback-bubble.correct { background: rgba(74,222,128,0.15); border: 2px solid var(--green); }
  .feedback-bubble.wrong   { background: rgba(233,69,96,0.15);  border: 2px solid var(--red);   }
  .feedback-bubble.reveal  { background: rgba(255,209,102,0.15);border: 2px solid var(--gold);  }
  .fb-main { font-size: 18px; font-weight: 900; }
  .fb-sub  { font-size: 12px; margin-top: 4px; opacity: 0.8; }
  .feedback-bubble.correct .fb-main { color: var(--green); }
  .feedback-bubble.wrong   .fb-main { color: var(--red);   }
  .feedback-bubble.reveal  .fb-main { color: var(--gold);  }
  .feedback-bubble.correct .fb-sub  { color: var(--green); }
  .feedback-bubble.wrong   .fb-sub  { color: var(--red);   }
  .feedback-bubble.reveal  .fb-sub  { color: var(--gold);  }

  /* ── Exit overlay ── */
  .overlay {
    display: none; position: fixed; inset: 0;
    background: rgba(0,0,0,0.7); z-index: 100;
    align-items: center; justify-content: center;
  }
  .overlay.visible { display: flex; }
  .overlay-box {
    background: var(--bg2); border: 2px solid var(--bg3);
    border-radius: 16px; padding: 28px 24px; text-align: center;
    max-width: 300px; width: 90%;
  }
  .overlay-title { font-size: 20px; font-weight: 900; margin-bottom: 8px; }
  .overlay-sub   { font-size: 13px; color: var(--muted); margin-bottom: 20px; }
  .overlay-btns  { display: flex; gap: 10px; }
  .overlay-cancel {
    flex: 1; padding: 12px; border-radius: 10px; font-weight: 800;
    font-size: 14px; font-family: var(--font); cursor: pointer;
    background: var(--bg3); color: var(--text); border: 2px solid var(--bg3);
    min-height: 44px;
  }
  .overlay-leave {
    flex: 1; padding: 12px; border-radius: 10px; font-weight: 800;
    font-size: 14px; font-family: var(--font); cursor: pointer;
    background: var(--red); color: white; border: none;
    min-height: 44px;
  }
  ```

- [ ] **Step 3: Verify layout in browser**

  Temporarily call `showScreen('screen-practice')` in console. You should see: top bar with ⚙️/streak/counter, progress bar, giant problem text, answer input box, hidden feedback bubble. No interactivity yet.

  Restore settings screen: `showScreen('screen-settings')`.

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: practice screen layout and CSS"
  ```

---

## Task 7: Session start, input handling, and answer checking

**Files:**
- Modify: `index.html` (add to `<script>` block)

- [ ] **Step 1: Add audio stubs, session state, and enterDojo()**

  Add these stubs first — they will be replaced by real implementations in Task 11. They prevent ReferenceErrors during testing of Tasks 7–10:

  ```javascript
  // ── Audio stubs (replaced in Task 11) ────────────────────────
  let audioCtx = null;
  function initAudio() {}
  function playCorrect()   {}
  function playWrong1()    {}
  function playReveal()    {}
  function playMilestone() {}
  function playComplete()  {}
  function playTap()       {}
  ```

  Then add the session state and enterDojo():

  ```javascript
  // ── Session state ─────────────────────────────────────────────
  let session = {
    deck: [], deckIndex: 0, tries: 0,
    streak: 0, bestStreak: 0,
    firstTryCorrect: 0, secondTryCorrect: 0, revealed: 0,
    revealedFacts: [], attempted: 0,
    timeRemaining: 0, timerInterval: null, timerExpired: false,
    inputLocked: false,
    msgPool: [], msgIndex: 0,
  };

  function enterDojo() {
    if (settings.numbers.length < 2) return;
    saveSettings();
    initAudio(); // Task 12 — safe to call early, just initializes AudioContext

    // Reset session
    session.deck = buildDeck(settings.operation, settings.numbers);
    session.deckIndex = 0;
    session.tries = 0; session.streak = 0; session.bestStreak = 0;
    session.firstTryCorrect = 0; session.secondTryCorrect = 0;
    session.revealed = 0; session.revealedFacts = [];
    session.attempted = 0; session.inputLocked = false;
    session.timerExpired = false;
    if (session.timerInterval) clearInterval(session.timerInterval);
    session.timerInterval = null;
    session.msgPool = []; session.msgIndex = 0;

    if (settings.sessionType === 'time') {
      session.timeRemaining = settings.sessionValue * 60;
    }

    showScreen('screen-practice');
    loadNextProblem();
    if (settings.sessionType === 'time') startTimer();
  }
  ```

  Update the dojo-btn click handler from Task 4 to call `enterDojo()`:
  ```javascript
  document.getElementById('dojo-btn').addEventListener('click', () => {
    if (settings.numbers.length < 2) return;
    enterDojo();
  });
  ```

- [ ] **Step 2: Add loadNextProblem()**

  ```javascript
  function loadNextProblem() {
    // Refill deck if exhausted
    if (session.deckIndex >= session.deck.length) {
      session.deck = buildDeck(settings.operation, settings.numbers);
      session.deckIndex = 0;
    }
    const p = session.deck[session.deckIndex];
    session.tries = 0;

    const opNames = { multiply:'MULTIPLICATION', divide:'DIVISION', mix:'MIXED' };
    document.getElementById('op-label').textContent = opNames[p.type] || 'MULTIPLICATION';
    document.getElementById('problem-text').textContent = p.display;

    const eq = document.getElementById('problem-eq');
    eq.textContent = '= ?';
    eq.className = 'problem-eq';

    hideFeedback();
    const input = document.getElementById('answer-input');
    input.value = '';
    input.disabled = false;
    session.inputLocked = false;
    document.getElementById('input-hint').style.visibility = 'visible';
    updateProgressUI();

    // Focus input — slight delay prevents iOS keyboard flicker on first load
    setTimeout(() => input.focus(), 50);
  }

  function currentProblem() {
    return session.deck[session.deckIndex];
  }
  ```

- [ ] **Step 3: Add input filtering and submission**

  ```javascript
  const answerInput = document.getElementById('answer-input');

  answerInput.addEventListener('input', () => {
    // Strip non-digits
    answerInput.value = answerInput.value.replace(/\D/g, '').slice(0, 3);
  });

  answerInput.addEventListener('keydown', e => {
    if (e.key === 'Enter' && !session.inputLocked) submitAnswer();
  });

  // Also submit on blur (iOS "Done" button)
  answerInput.addEventListener('blur', () => {
    if (!session.inputLocked && answerInput.value.length > 0) submitAnswer();
  });
  ```

- [ ] **Step 4: Add submitAnswer() with all three feedback states**

  ```javascript
  function submitAnswer() {
    const input = document.getElementById('answer-input');
    const raw = input.value.trim();
    if (!raw) return;
    const guess = parseInt(raw, 10);
    const p = currentProblem();
    session.tries++;

    if (guess === p.answer) {
      // ── Correct ──
      if (session.tries === 1) {
        session.firstTryCorrect++;
        session.streak++;
        if (session.streak > session.bestStreak) session.bestStreak = session.streak;
      } else {
        session.secondTryCorrect++;
        // streak does NOT increment on 2nd-try correct
      }
      session.attempted++;
      session.deckIndex++;

      const eq = document.getElementById('problem-eq');
      eq.textContent = `= ${p.answer} ✓`;
      eq.className = 'problem-eq correct';

      showFeedback('correct', getFeedbackMessage(), getStreakSub());
      playCorrect();
      lockAndAdvance(1200);

    } else if (session.tries < 2) {
      // ── Wrong first try ──
      input.value = '';
      triggerShake();
      showFeedback('wrong', 'Try Again! 💪', '1 try left');
      playWrong1();

    } else {
      // ── Wrong second try — reveal ──
      session.revealed++;
      session.streak = 0;
      session.attempted++;
      session.revealedFacts.push(p.key);
      session.deckIndex++;

      const eq = document.getElementById('problem-eq');
      eq.textContent = `= ${p.answer} 💡`;
      eq.className = 'problem-eq reveal';

      showFeedback('reveal', `The answer is ${p.answer}!`, "You'll get it next time 🥷");
      playReveal();
      lockAndAdvance(1800);
    }

    updateProgressUI();
    updateStreakUI();
  }

  function lockAndAdvance(delay) {
    session.inputLocked = true;
    document.getElementById('answer-input').disabled = true;
    document.getElementById('input-hint').style.visibility = 'hidden';
    setTimeout(() => {
      if (isSessionOver()) { endSession(); return; }
      loadNextProblem();
    }, delay);
  }

  function isSessionOver() {
    if (session.timerExpired) return true;
    if (settings.sessionType === 'problems') {
      return session.attempted >= settings.sessionValue;
    }
    return false; // time mode ends via timer
  }
  ```

- [ ] **Step 5: Add UI helper functions**

  ```javascript
  function showFeedback(type, main, sub) {
    const bubble = document.getElementById('feedback-bubble');
    bubble.className = `feedback-bubble ${type} visible`;
    document.getElementById('fb-main').textContent = main;
    document.getElementById('fb-sub').textContent = sub || '';
  }

  function hideFeedback() {
    document.getElementById('feedback-bubble').className = 'feedback-bubble';
  }

  function triggerShake() {
    const input = document.getElementById('answer-input');
    input.classList.remove('shake');
    void input.offsetWidth; // force reflow
    input.classList.add('shake');
  }

  function updateStreakUI() {
    const badge = document.getElementById('streak-badge');
    document.getElementById('streak-count').textContent = session.streak;
    badge.classList.toggle('hidden', session.streak < 2);
  }

  function updateProgressUI() {
    if (settings.sessionType === 'problems') {
      document.getElementById('progress-label').textContent =
        `${session.attempted} / ${settings.sessionValue}`;
      const pct = (session.attempted / settings.sessionValue) * 100;
      document.getElementById('prog-fill').style.width = pct + '%';
    }
    // Time mode progress updated by timer
  }
  ```

- [ ] **Step 6: Wire the exit overlay buttons**

  ```javascript
  document.getElementById('exit-btn').addEventListener('click', () => {
    document.getElementById('exit-overlay').classList.add('visible');
  });
  document.getElementById('overlay-cancel').addEventListener('click', () => {
    document.getElementById('exit-overlay').classList.remove('visible');
    document.getElementById('answer-input').focus();
  });
  document.getElementById('overlay-leave').addEventListener('click', () => {
    document.getElementById('exit-overlay').classList.remove('visible');
    if (session.timerInterval) clearInterval(session.timerInterval);
    showScreen('screen-settings');
  });
  ```

- [ ] **Step 7: Verify the practice loop end-to-end**

  Open `index.html`. Set numbers {2, 3}, 5 problems, tap Enter Dojo.
  - First answer: type correct value, press Enter → green feedback, auto-advances in ~1.2s
  - Wrong first try: type wrong, press Enter → red "Try Again!", input clears, same problem stays
  - Wrong twice: two wrong answers → gold reveal, auto-advances in ~1.8s
  - After 5 problems: `endSession()` is called (not yet implemented — console error is expected)
  - ⚙️ button shows overlay; "Leave" returns to settings

- [ ] **Step 8: Commit**

  ```bash
  git add index.html
  git commit -m "feat: session start, answer checking, and feedback states"
  ```

---

## Task 8: Message pools and streak system

**Files:**
- Modify: `index.html` (add to `<script>` block)

- [ ] **Step 1: Add message pool data**

  ```javascript
  const MESSAGES_SOLO = [
    'Your brain is a weapon.',
    'That problem never stood a chance.',
    'Math agrees with you.',
    'Boom. 💥',
    'Easy for a ninja.',
    'Was that even hard? Didn\'t think so.',
    'The answer feared you.',
    'Clean. Sharp. Correct.',
    'Calculator? You don\'t need one.',
    'Facts only. 🔥',
    'Zero hesitation. Full marks.',
    'Did you even blink?',
    'Smooth.',
    'Your brain just did a backflip.',
    'That fact didn\'t even have time to be nervous.',
    'If math were a video game, you\'d be at the final boss.',
    'Even the hard ones are easy for you. Suspicious.',
    'That one was a trap. You walked right through it.',
    '12 × 12 is sweating right now.',
  ];

  const MESSAGES_STREAK = {
    3:  '🔥 3 in a row! You\'re on fire!',
    4:  '🔥 4 in a row! The numbers are scared.',
    5:  '🔥 5 in a row! Full ninja mode activated.',
    6:  '🔥 6 in a row! Unstoppable.',
    7:  '🔥 7 in a row! Is this even fair?',
    8:  '🔥 8 in a row! You might actually be a robot. A cool robot.',
    9:  '🔥 9 in a row! The dojo bows before you.',
    10: '🔥 10 in a row! 🏆 LEGENDARY. TEN. IN. A. ROW.',
  };

  function getFeedbackMessage() {
    // Streak messages take priority at milestones
    if (session.streak >= 3) {
      if (MESSAGES_STREAK[session.streak]) return MESSAGES_STREAK[session.streak];
      if (session.streak > 10 && session.streak % 5 === 0)
        return `🔥 ${session.streak} in a row! Stop showing off. (Actually don't.)`;
    }
    // Otherwise draw from shuffled pool
    if (session.msgPool.length === 0) {
      session.msgPool = shuffle([...Array(MESSAGES_SOLO.length).keys()]);
    }
    return MESSAGES_SOLO[session.msgPool.pop()];
  }

  function getStreakSub() {
    if (session.streak >= 5) return `${session.streak} in a row! 🔥`;
    return '';
  }
  ```

- [ ] **Step 2: Add milestone audio trigger in submitAnswer()**

  In the correct-answer branch of `submitAnswer()`, after `session.streak` is incremented, add:
  ```javascript
  if (session.streak === 5 || session.streak === 10 ||
      (session.streak > 10 && session.streak % 5 === 0)) {
    playMilestone(); // overrides playCorrect — called after
  }
  ```

  Update the correct branch to use this pattern:
  ```javascript
  // (replace playCorrect() in the correct branch with:)
  if (session.tries === 1 &&
      (session.streak === 5 || session.streak === 10 ||
       (session.streak > 10 && session.streak % 5 === 0))) {
    playMilestone();
  } else {
    playCorrect();
  }
  ```

- [ ] **Step 3: Verify messages in browser**

  Play a session. Verify:
  - Each correct answer shows a different message from the solo pool
  - At streak 3+, the streak message overrides the solo message
  - At streak 10, the "LEGENDARY" message appears
  - After going through many correct answers, messages cycle without repeating until the pool is exhausted then reshuffled

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: message pools and streak milestone messages"
  ```

---

## Task 9: Time mode and session end

**Files:**
- Modify: `index.html` (add to `<script>` block)

- [ ] **Step 1: Add the timer**

  ```javascript
  function startTimer() {
    updateTimerUI();
    session.timerInterval = setInterval(() => {
      session.timeRemaining--;
      updateTimerUI();
      if (session.timeRemaining <= 0) {
        clearInterval(session.timerInterval);
        session.timerInterval = null;
        session.timerExpired = true;
        // Don't end session immediately — let current problem finish
        // isSessionOver() will catch it after the next lockAndAdvance
      }
    }, 1000);
  }

  function updateTimerUI() {
    const total = settings.sessionValue * 60;
    const t = session.timeRemaining;
    const mins = Math.floor(t / 60);
    const secs = String(t % 60).padStart(2, '0');
    document.getElementById('progress-label').textContent = `${mins}:${secs}`;
    const pct = (t / total) * 100;
    const fill = document.getElementById('prog-fill');
    fill.style.width = pct + '%';
    fill.classList.toggle('urgent', t <= 30);
  }
  ```

- [ ] **Step 2: Add results screen HTML**

  Replace `<div id="screen-results">` with:

  ```html
  <div id="screen-results" class="screen">
    <div class="container">

      <div class="results-ninja">🥷</div>
      <div class="rank-badge" id="rank-badge">Shadow Ninja</div>
      <div class="rank-score" id="rank-score">Ranked on first-try score: 90%</div>
      <div class="rank-best"  id="rank-best">Your highest rank yet! 🎉</div>

      <div class="session-msg" id="session-msg">
        "The numbers barely had a chance."
      </div>

      <div class="stats-row">
        <div class="stat-box">
          <div class="stat-num" id="stat-first">15</div>
          <div class="stat-lbl">⚡ First try!</div>
        </div>
        <div class="stat-box">
          <div class="stat-num" id="stat-second">3</div>
          <div class="stat-lbl">💪 Second try</div>
        </div>
        <div class="stat-box">
          <div class="stat-num" id="stat-helped">2</div>
          <div class="stat-lbl">💡 Needed help</div>
        </div>
      </div>

      <div class="best-streak-line" id="best-streak-line">
        Best streak: <strong id="result-streak">7</strong> 🔥
      </div>

      <div class="extra-training" id="extra-training">
        <div class="extra-label">⚔️ These need more dojo time</div>
        <div class="extra-facts" id="extra-facts"></div>
      </div>

      <div class="results-btns">
        <button class="dojo-btn" id="train-again-btn">⚔️ TRAIN AGAIN</button>
        <button class="settings-btn" id="results-settings-btn">⚙️</button>
      </div>

    </div>
  </div>
  ```

- [ ] **Step 3: Add results screen CSS**

  ```css
  /* ── Results screen ── */
  .results-ninja { font-size: 60px; text-align: center; margin-bottom: 10px; }
  .rank-badge {
    text-align: center; font-size: 20px; font-weight: 900; letter-spacing: 2px;
    text-transform: uppercase; border-radius: 12px;
    padding: 10px 24px; display: inline-block;
    margin: 0 auto 6px; display: block; width: fit-content; margin: 0 auto 6px;
  }
  .rank-score { text-align: center; font-size: 13px; color: var(--muted); margin-bottom: 4px; }
  .rank-best  { text-align: center; font-size: 14px; color: var(--gold); font-weight: 700;
                margin-bottom: 20px; }
  .rank-best.hidden { display: none; }

  .session-msg {
    background: var(--bg2); border-left: 4px solid var(--gold);
    border-radius: 10px; padding: 14px 18px; margin-bottom: 20px;
    font-size: 15px; font-style: italic; color: var(--text); line-height: 1.5;
  }

  .stats-row {
    display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 10px; margin-bottom: 14px;
  }
  .stat-box {
    background: var(--bg2); border-radius: 10px; padding: 14px 8px; text-align: center;
  }
  .stat-num { font-size: 28px; font-weight: 900; color: var(--text); }
  .stat-lbl { font-size: 11px; color: var(--muted); font-weight: 700; margin-top: 4px; }

  .best-streak-line {
    text-align: center; color: var(--muted); font-size: 13px; margin-bottom: 20px;
  }

  .extra-training {
    background: var(--bg2); border-radius: 12px; padding: 14px 16px; margin-bottom: 24px;
  }
  .extra-training.hidden { display: none; }
  .extra-label { font-size: 11px; font-weight: 700; color: var(--red);
                 letter-spacing: 1px; margin-bottom: 10px; }
  .extra-facts { display: flex; gap: 8px; flex-wrap: wrap; }
  .fact-pill {
    background: var(--bg3); border-radius: 8px; padding: 6px 14px;
    font-size: 15px; font-weight: 800; color: var(--text);
  }

  .results-btns { display: flex; gap: 10px; }
  .settings-btn {
    background: var(--bg2); color: var(--muted); border: 2px solid var(--bg3);
    border-radius: 12px; padding: 14px 18px; font-size: 18px; cursor: pointer;
    min-height: 56px; min-width: 56px;
  }
  ```

- [ ] **Step 4: Add rank data and endSession()**

  ```javascript
  const RANKS = [
    { name:'White Belt',      color:'#00b4d8', bg:'rgba(0,180,216,0.2)',   border:'#00b4d8' },
    { name:'Yellow Belt',     color:'#ffd166', bg:'rgba(255,209,102,0.2)', border:'#ffd166' },
    { name:'Green Belt',      color:'#4ade80', bg:'rgba(74,222,128,0.2)',  border:'#4ade80' },
    { name:'Black Belt',      color:'#e0e0e0', bg:'rgba(224,224,224,0.1)', border:'#888'    },
    { name:'Shadow Ninja',    color:'#ffd166', bg:'rgba(255,209,102,0.2)', border:'#ffd166' },
    { name:'Grand Master 🏆', color:'#1a1a2e', bg:'linear-gradient(135deg,#ffd166,#ff9f1c)', border:'transparent' },
  ];

  const RESULTS_MSGS = [
    // White Belt (index 0)
    ["Even the greatest ninjas had to train. A lot. Like, a whole lot. Go again.",
     "The numbers won this round. They won't next time.",
     "White Belt today, Grand Master eventually. That's the path."],
    // Yellow Belt (1)
    ["Halfway there. Every ninja starts somewhere.",
     "The hard facts fought back. Train them again.",
     "Keep going. The streak is coming."],
    // Green Belt (2)
    ["Good progress. Green Belts keep showing up — that's how it works.",
     "More right than wrong. That's the direction.",
     "The math is getting easier. You're getting harder."],
    // Black Belt (3)
    ["Strong session. Black Belts know which facts to train next.",
     "Solid work. The dojo is pleased.",
     "8 out of 10 ain't bad. The other 2 are running."],
    // Shadow Ninja (4)
    ["Almost flawless. Almost. The dojo respects your effort.",
     "The numbers barely had a chance. Nearly perfect.",
     "One or two slipped through. Shadow Ninjas hunt those down next session."],
    // Grand Master (5)
    ["Perfect. Not a single fact escaped you.",
     "100%. The dojo has nothing left to teach you... for now.",
     "Grand Master status achieved. The numbers kneel."],
  ];

  function calcRankIndex(firstTry, total) {
    const pct = total === 0 ? 0 : (firstTry / total) * 100;
    if (pct === 100) return 5;
    if (pct >= 90)  return 4;
    if (pct >= 80)  return 3;
    if (pct >= 65)  return 2;
    if (pct >= 50)  return 1;
    return 0;
  }

  function endSession() {
    if (session.timerInterval) { clearInterval(session.timerInterval); session.timerInterval = null; }
    playComplete();

    const total     = session.attempted;
    const rankIdx   = calcRankIndex(session.firstTryCorrect, total);
    const rank      = RANKS[rankIdx];
    const pct       = total === 0 ? 0 : Math.round((session.firstTryCorrect / total) * 100);
    const msgs      = RESULTS_MSGS[rankIdx];
    const msg       = msgs[Math.floor(Math.random() * msgs.length)];

    // Rank badge
    const badge = document.getElementById('rank-badge');
    badge.textContent = rank.name;
    badge.style.cssText = `background:${rank.bg};color:${rank.color};
      border:2px solid ${rank.border};`;

    document.getElementById('rank-score').textContent =
      `Ranked on first-try score: ${pct}%`;

    // Stats
    document.getElementById('stat-first').textContent  = session.firstTryCorrect;
    document.getElementById('stat-second').textContent = session.secondTryCorrect;
    document.getElementById('stat-helped').textContent = session.revealed;
    document.getElementById('result-streak').textContent = session.bestStreak;
    document.getElementById('session-msg').textContent = `"${msg}"`;

    // Revealed facts
    const extra = document.getElementById('extra-training');
    if (session.revealedFacts.length > 0) {
      extra.classList.remove('hidden');
      document.getElementById('extra-facts').innerHTML =
        session.revealedFacts.map(f => `<span class="fact-pill">${f}</span>`).join('');
    } else {
      extra.classList.add('hidden');
    }

    // Personal best — handled in Task 11
    document.getElementById('rank-best').classList.add('hidden');

    showScreen('screen-results');
  }

  // Wire results buttons
  document.getElementById('train-again-btn').addEventListener('click', () => {
    enterDojo();
  });
  document.getElementById('results-settings-btn').addEventListener('click', () => {
    showScreen('screen-settings');
  });
  ```

- [ ] **Step 5: Verify full session flow**

  Play a complete session with 5 problems:
  - Answer all 5 → results screen appears with rank badge, stats, and message
  - If any reveals happened, "These need more dojo time" section shows those facts
  - "Train Again" starts a new session immediately
  - ⚙️ returns to settings
  - Play a time-mode session (1 minute) → timer counts down, bar depletes, turns red at 30s, ends after current problem when timer hits 0

- [ ] **Step 6: Commit**

  ```bash
  git add index.html
  git commit -m "feat: time mode timer, results screen, and endSession flow"
  ```

---

## Task 10: localStorage history — best rank and best streak

**Files:**
- Modify: `index.html` (add to `<script>` block)

- [ ] **Step 1: Add history constants and save/load**

  ```javascript
  const LS_BEST_RANK    = 'mn_bestRankIndex';
  const LS_BEST_STREAK  = 'mn_bestStreak';
  const LS_TOTAL        = 'mn_totalSessions';

  function loadHistory() {
    try {
      return {
        bestRankIndex: parseInt(localStorage.getItem(LS_BEST_RANK)  ?? '-1', 10),
        bestStreak:    parseInt(localStorage.getItem(LS_BEST_STREAK) ?? '0',  10),
        totalSessions: parseInt(localStorage.getItem(LS_TOTAL)       ?? '0',  10),
      };
    } catch(e) { return { bestRankIndex: -1, bestStreak: 0, totalSessions: 0 }; }
  }

  function saveHistory(rankIndex, streak, total) {
    try {
      localStorage.setItem(LS_BEST_RANK,   rankIndex);
      localStorage.setItem(LS_BEST_STREAK, streak);
      localStorage.setItem(LS_TOTAL,       total);
    } catch(e) { /* ignore */ }
  }
  ```

- [ ] **Step 2: Update endSession() to read/write history**

  Inside `endSession()`, just before `showScreen('screen-results')`, replace the "Personal best" comment with:

  ```javascript
  const history     = loadHistory();
  const isNewBest   = rankIdx > history.bestRankIndex;
  const newBestStreak  = Math.max(session.bestStreak, history.bestStreak);
  const newTotal    = history.totalSessions + 1;

  saveHistory(
    Math.max(rankIdx, history.bestRankIndex),
    newBestStreak,
    newTotal
  );

  const rankBest = document.getElementById('rank-best');
  if (isNewBest && history.bestRankIndex >= 0) {
    rankBest.textContent = 'Your highest rank yet! 🎉';
    rankBest.classList.remove('hidden');
  } else if (history.bestRankIndex < 0) {
    // First-ever session — show welcome message instead of comparison
    rankBest.textContent = 'First session complete! 🥷';
    rankBest.classList.remove('hidden');
  } else {
    rankBest.classList.add('hidden');
  }
  ```

- [ ] **Step 3: Verify history tracking**

  Play two sessions, getting a low rank on the first and a higher rank on the second. The second session's results screen should show "Your highest rank yet! 🎉". A third session at the same rank should NOT show it. Check DevTools → localStorage for `mn_bestRankIndex`, `mn_bestStreak`, `mn_totalSessions`.

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: localStorage history — best rank, best streak, session count"
  ```

---

## Task 11: Audio system

**Files:**
- Modify: `index.html` (add to `<script>` block)

- [ ] **Step 1: Replace the audio stubs from Task 7 with the real audio engine**

  Find the audio stubs block added in Task 7 and replace it entirely with:

  ```javascript
  // ── Audio ─────────────────────────────────────────────────────
  let audioCtx = null;

  function initAudio() {
    if (!audioCtx) {
      try {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      } catch(e) { audioCtx = null; }
    }
  }

  function playTone(freq, dur, type = 'sine', vol = 0.28, delay = 0) {
    if (settings.muted || !audioCtx) return;
    try {
      const osc  = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.type = type;
      osc.frequency.setValueAtTime(freq, audioCtx.currentTime + delay);
      gain.gain.setValueAtTime(vol, audioCtx.currentTime + delay);
      gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + delay + dur);
      osc.start(audioCtx.currentTime + delay);
      osc.stop(audioCtx.currentTime  + delay + dur + 0.05);
    } catch(e) { /* ignore if audio context suspended */ }
  }

  function playCorrect() {
    playTone(523, 0.10, 'sine', 0.25, 0.00); // C5
    playTone(659, 0.12, 'sine', 0.25, 0.10); // E5
    playTone(784, 0.18, 'sine', 0.25, 0.20); // G5
  }

  function playWrong1() {
    playTone(330, 0.12, 'sine', 0.20, 0.00); // E4
    playTone(294, 0.18, 'sine', 0.20, 0.13); // D4
  }

  function playReveal() {
    playTone(262, 0.35, 'sine', 0.18); // C4 — low and soft
  }

  function playMilestone() {
    // ascending 4-note fanfare
    [523, 659, 784, 1047].forEach((f, i) =>
      playTone(f, 0.14 + i * 0.03, 'sine', 0.28, i * 0.11)
    );
  }

  function playComplete() {
    // celebratory 5-note sequence
    [523, 659, 784, 659, 1047].forEach((f, i) =>
      playTone(f, 0.16 + i * 0.02, 'sine', 0.30, i * 0.13)
    );
  }

  function playTap() {
    playTone(800, 0.04, 'sine', 0.08);
  }
  ```

- [ ] **Step 2: Add tap sounds to buttons**

  After the DOMContentLoaded listener, add tap sounds to the main interactive buttons:

  ```javascript
  // Add tap sounds to key buttons
  ['op-toggle', 'st-toggle', 'number-grid', 'stepper-minus', 'stepper-plus',
   'sound-toggle-row', 'exit-btn', 'overlay-cancel', 'overlay-leave',
   'train-again-btn', 'results-settings-btn'].forEach(id => {
    const el = document.getElementById(id);
    if (el) el.addEventListener('click', playTap);
  });
  // dojo-btn has its own sound (playCorrect on enter)
  ```

- [ ] **Step 3: Verify all sounds**

  Play a full session with sound on:
  - Correct 1st try → ascending 3-note chime
  - Wrong 1st try → descending 2-note tone
  - Reveal → single low soft tone
  - 5-in-a-row → 4-note fanfare instead of normal chime
  - Session complete → 5-note celebratory sequence
  - Button taps → subtle click
  - Mute toggle → all sounds stop immediately

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: Web Audio API sound system with mute support"
  ```

---

## Task 12: Polish — iPad optimizations and final touches

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Prevent double-tap zoom on iOS**

  Add to CSS:
  ```css
  button, input {
    touch-action: manipulation; /* prevents double-tap zoom on iOS */
  }
  ```

- [ ] **Step 2: Prevent iOS scroll bounce causing layout shifts**

  Add to CSS:
  ```css
  body {
    overscroll-behavior: none;
    position: fixed; /* prevents body scroll on iPad */
    width: 100%;
  }
  .screen { overflow-y: auto; -webkit-overflow-scrolling: touch; }
  ```

- [ ] **Step 3: Ensure answer input font-size ≥ 16px (prevents Safari auto-zoom)**

  The `answer-input` font-size is already 36px in Task 6. Verify in CSS that it's not overridden anywhere.

- [ ] **Step 4: Add PWA home screen icon color**

  Inside `<head>`, add:
  ```html
  <meta name="theme-color" content="#1a1a2e">
  <link rel="apple-touch-icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🥷</text></svg>">
  ```

- [ ] **Step 5: Audit all tap targets for 44px minimum**

  Inspect each interactive element in DevTools. Confirm:
  - All `.toggle-btn` have `min-height: 44px` ✓ (set in Task 2)
  - All `.num-btn` have `min-height: 44px` ✓ (set in Task 2)
  - `.stepper-btn` is 44×44px ✓ (set in Task 2)
  - `.dojo-btn` is `min-height: 56px` ✓
  - `.icon-btn` is `min-width/height: 44px` ✓
  - `.overlay-cancel` and `.overlay-leave` are `min-height: 44px` ✓

  If any fall short, add `min-height: 44px` to their CSS rule.

- [ ] **Step 6: Test landscape orientation on iPad**

  Rotate iPad to landscape. Verify the settings screen scrolls without clipping. The problem display should still be readable. Practice screen fits in one viewport without scrolling.

- [ ] **Step 7: Final push**

  ```bash
  git add index.html
  git commit -m "feat: iPad polish — touch handling, PWA meta, tap target audit"
  git push
  ```

- [ ] **Step 8: Full end-to-end test on actual iPad Safari**

  Using the GitHub Pages URL on iPad:
  1. Add to Home Screen — icon shows 🥷 on dark background
  2. Launch from home screen — loads in full-screen mode (no browser chrome)
  3. Settings: tap all 13 number buttons, verify toggle on/off, verify minimum 2 enforced
  4. Play a 10-problem multiplication session — verify sound, streak, feedback messages
  5. Deliberately miss two problems — verify reveal at 1.8s, fact appears in "more dojo time"
  6. Complete session — verify rank badge, stats sum to 10, "First session complete! 🥷"
  7. Play again — verify settings are remembered, "Your highest rank yet!" appears if rank improved
  8. Test 2-minute time mode — verify bar depletes, turns red at 30s, session ends after current problem

---

## Spec Coverage Checklist

| Spec Requirement | Task |
|---|---|
| Single HTML file, no dependencies | Task 1 |
| GitHub Pages hosting | Task 1 |
| Dark & bold theme (navy, red, teal, gold) | Task 2 |
| 🥷 MathNinja branding + Diego's Math Dojo tagline | Task 2 |
| Operation toggle: Multiply / Divide / Mix | Task 3 |
| Number grid 0–12, tap-toggle, min 2 | Task 3 |
| Session length: Problems (5–50) or Time (1–10 min) | Task 3 |
| Settings persist via localStorage | Task 4 |
| Problem generation: multiplication, division, mix | Task 5 |
| Division by 0 excluded from divisor pool | Task 5 |
| Deck-of-cards shuffle (no repeats until exhausted) | Task 5 |
| ⚔️ ENTER DOJO button | Task 7 |
| AudioContext init on Enter Dojo tap | Task 7, 11 |
| Big bold problem display (64px+) | Task 6 |
| Numeric-only input, 3 char max, inputmode | Task 7 |
| Input locked during feedback window | Task 7 |
| Correct 1st try: green feedback + message | Task 7 |
| Wrong 1st try: red, clear, "Try Again!" | Task 7 |
| Correct 2nd try: green, no streak | Task 7 |
| Wrong 2nd try: gold reveal, 1.8s delay | Task 7 |
| Message pool (solo, streak, funny) — no repeat until exhausted | Task 8 |
| Streak counter, hidden until ≥ 2 | Task 7, 8 |
| Streak milestone messages (3, 5, 10…) | Task 8 |
| Streak milestone fanfare sound | Task 8, 11 |
| Progress bar fills in Problems mode | Task 7 |
| Progress bar depletes in Time mode, red at 30s | Task 9 |
| Timer display (countdown) | Task 9 |
| Timer expiry: finish current problem | Task 9 |
| Results: rank badge with 6 tiers | Task 9 |
| Results: rank based on 1st-try % | Task 9 |
| Results: "Ranked on first-try score: X%" | Task 9 |
| Results: 3 stat boxes (⚡💪💡) | Task 9 |
| Results: best streak this session | Task 9 |
| Results: performance-tiered message pool | Task 9 |
| Results: "⚔️ These need more dojo time" (reveals only) | Task 9 |
| Train Again / Settings buttons on results | Task 9 |
| localStorage: best rank, best streak, sessions | Task 10 |
| "Your highest rank yet! 🎉" on new personal best | Task 10 |
| "First session complete!" on first ever session | Task 10 |
| localStorage fails silently (try/catch) | Task 4, 10 |
| All 6 sound events (correct, wrong×2, milestone, complete, tap) | Task 11 |
| Mute toggle, persists in localStorage | Task 3, 4, 11 |
| ⚙️ exit with "Leave session?" confirmation | Task 7 |
| iPad tap targets ≥ 44px | Task 12 |
| inputmode="numeric" on answer input | Task 6 |
| font-size ≥ 16px on input (no auto-zoom) | Task 6 |
| PWA "Add to Home Screen" support | Task 12 |
| Works in landscape | Task 12 |
