# Single Number Power Mode Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Allow a student to select exactly one number and practice all facts involving that number (0–12 range) in multiply, divide, or mix mode.

**Architecture:** All logic lives in `index.html` (single-file app). Problem builders detect `numbers.length === 1` and generate against the full 0–12 range instead of cross-multiplying the selected set. The `updateDojoBtn` function is the single source of truth for both button state and hint text — it already runs on every selection change.

**Tech Stack:** Vanilla JS, single HTML file, no build step, no test framework — all verification is manual browser testing.

---

## File Map

| File | What changes |
|---|---|
| `index.html:98` | `.grid-hint` CSS — no change needed, already styled correctly |
| `index.html:369` | Add `id="grid-hint"` to hint paragraph, remove static text |
| `index.html:667-668` | `updateDojoBtn` — new enable logic + hint text update |
| `index.html:694-700` | Operation toggle listener — add `updateDojoBtn()` call |
| `index.html:735-737` | `dojo-btn` click handler guard — `< 2` → `< 1` |
| `index.html:1081-1090` | `buildMultiplyProblems` — add single-number path |
| `index.html:1092-1103` | `buildDivideProblems` — add single-number path |
| `index.html:1129-1130` | `enterDojo` guard — `< 2` → `< 1` |

---

## Task 1: Update problem builders for single-number mode

**Files:**
- Modify: `index.html:1081-1103`

- [ ] **Step 1: Replace `buildMultiplyProblems`**

Find and replace the entire function (lines 1081–1090):

```js
function buildMultiplyProblems(numbers) {
  // Single-number mode: pair X against full 0–12 range, both orderings
  if (numbers.length === 1) {
    const X = numbers[0];
    const problems = [];
    for (let n = 0; n <= 12; n++) {
      problems.push({ type:'multiply', a:n, b:X, answer:n*X,
        display:`${n} × ${X}`, key:`${n}×${X}` });
      if (n !== X) {
        problems.push({ type:'multiply', a:X, b:n, answer:X*n,
          display:`${X} × ${n}`, key:`${X}×${n}` });
      }
    }
    return problems;
  }
  // Multi-number mode: original cross-product
  const problems = [];
  for (const a of numbers) {
    for (const b of numbers) {
      problems.push({ type:'multiply', a, b, answer: a*b,
        display:`${a} × ${b}`, key:`${a}×${b}` });
    }
  }
  return problems;
}
```

- [ ] **Step 2: Replace `buildDivideProblems`**

Find and replace the entire function (lines 1092–1103):

```js
function buildDivideProblems(numbers) {
  // Single-number mode: X is always the divisor, answers span 1–12 excluding X
  if (numbers.length === 1) {
    const X = numbers[0];
    if (X === 0) return []; // cannot divide by zero
    const problems = [];
    for (let n = 1; n <= 12; n++) {
      if (n === X) continue; // answer must not equal divisor
      const dividend = n * X;
      problems.push({ type:'divide', a:dividend, b:X, answer:n,
        display:`${dividend} ÷ ${X}`, key:`${dividend}÷${X}` });
    }
    return problems;
  }
  // Multi-number mode: original cross-product
  const problems = [];
  const nonZero = numbers.filter(n => n !== 0);
  for (const answer of nonZero) {
    for (const divisor of nonZero) {
      const dividend = answer * divisor;
      problems.push({ type:'divide', a:dividend, b:divisor, answer,
        display:`${dividend} ÷ ${divisor}`, key:`${dividend}÷${divisor}` });
    }
  }
  return problems;
}
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: single-number problem generation in multiply and divide builders"
```

---

## Task 2: Add ID to hint paragraph

**Files:**
- Modify: `index.html:369`

- [ ] **Step 1: Add `id` attribute and clear static text**

Find:
```html
        <p class="grid-hint">Select at least 2 numbers</p>
```

Replace with:
```html
        <p class="grid-hint" id="grid-hint"></p>
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: give hint paragraph an id for dynamic text control"
```

---

## Task 3: Update `updateDojoBtn` — button enable logic and hint text

**Files:**
- Modify: `index.html:667-669`

- [ ] **Step 1: Replace `updateDojoBtn`**

Find:
```js
    function updateDojoBtn() {
      document.getElementById('dojo-btn').disabled = settings.numbers.length < 2;
    }
```

Replace with:
```js
    function updateDojoBtn() {
      const n = settings.numbers.length;
      const singleZeroDivide = n === 1 && settings.numbers[0] === 0
        && (settings.operation === 'divide' || settings.operation === 'mix');
      document.getElementById('dojo-btn').disabled = n < 1 || singleZeroDivide;

      const hint = document.getElementById('grid-hint');
      if (n === 1) {
        const X = settings.numbers[0];
        hint.textContent = `⚡ Single Number Power Mode — crushing the ${X} facts!`;
      } else {
        hint.textContent = '';
      }
    }
```

Note: `singleZeroDivide` disables the button when the only selected number is 0 and the operation is divide or mix (no valid divide problems exist for divisor 0; mix with only 0 selected would still produce multiply problems but that edge case is excluded for clarity).

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: update dojo button enable logic and hint text for single-number mode"
```

---

## Task 4: Wire operation toggle to refresh button state

**Files:**
- Modify: `index.html:694-700`

The operation toggle currently calls `syncSettingsUI()` and `saveSettings()` but not `updateDojoBtn()`. If X=0 is selected alone and the user switches to divide, the button must disable. If they switch back to multiply, it must re-enable.

- [ ] **Step 1: Add `updateDojoBtn()` call to operation toggle handler**

Find:
```js
    // Wire operation buttons
    document.querySelectorAll('#op-toggle .toggle-btn').forEach(btn => {
      btn.addEventListener('click', () => {
        settings.operation = btn.dataset.op;
        syncSettingsUI();
        saveSettings();
      });
    });
```

Replace with:
```js
    // Wire operation buttons
    document.querySelectorAll('#op-toggle .toggle-btn').forEach(btn => {
      btn.addEventListener('click', () => {
        settings.operation = btn.dataset.op;
        syncSettingsUI();
        updateDojoBtn();
        saveSettings();
      });
    });
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: refresh dojo button state when operation changes"
```

---

## Task 5: Update guards in click handler and `enterDojo`

**Files:**
- Modify: `index.html:735-737`
- Modify: `index.html:1129-1130`

- [ ] **Step 1: Update `dojo-btn` click handler guard**

Find:
```js
    document.getElementById('dojo-btn').addEventListener('click', () => {
      if (settings.numbers.length < 2) return;
```

Replace with:
```js
    document.getElementById('dojo-btn').addEventListener('click', () => {
      if (settings.numbers.length < 1) return;
```

- [ ] **Step 2: Update `enterDojo` guard**

Find:
```js
    function enterDojo() {
      if (settings.numbers.length < 2) return;
```

Replace with:
```js
    function enterDojo() {
      if (settings.numbers.length < 1) return;
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: allow single-number selection to enter dojo"
```

---

## Task 6: Manual verification

Open `index.html` in a browser (or the local dev server) and verify each scenario:

- [ ] **Scenario 1 — Zero numbers selected**
  - No numbers highlighted → button is disabled → hint is empty

- [ ] **Scenario 2 — One number selected (e.g. 5), Multiply**
  - Hint reads: `⚡ Single Number Power Mode — crushing the 5 facts!`
  - Button is enabled
  - Enter dojo → confirm problems include both orderings: `2 × 5`, `5 × 2`, `5 × 5` (once), `12 × 5`, `5 × 12`, etc.
  - Confirm `0 × 5` and `5 × 0` appear

- [ ] **Scenario 3 — One number selected (e.g. 5), Divide**
  - Button is enabled
  - Enter dojo → all questions are `something ÷ 5`
  - Confirm `25 ÷ 5` does NOT appear (answer would equal 5)
  - Confirm `0 ÷ 5` does NOT appear (zero excluded from divide)

- [ ] **Scenario 4 — One number selected (e.g. 5), Mix**
  - Button is enabled
  - Enter dojo → mix of `n × 5`, `5 × n`, and `something ÷ 5` problems

- [ ] **Scenario 5 — Number 0 selected alone, Divide**
  - Button is disabled (can't divide by 0)
  - Hint reads: `⚡ Single Number Power Mode — crushing the 0 facts!`

- [ ] **Scenario 6 — Number 0 selected alone, Multiply**
  - Button is enabled
  - Enter dojo → problems include `0 × 0`, `0 × 1`, `1 × 0`, etc.

- [ ] **Scenario 7 — Number 0 selected alone, then switch operation to Divide**
  - Button disables immediately when operation changes to Divide
  - Button re-enables when switching back to Multiply

- [ ] **Scenario 8 — Two or more numbers selected**
  - Hint is empty
  - Button is enabled
  - Problems behave exactly as before (cross-product of selected numbers)
