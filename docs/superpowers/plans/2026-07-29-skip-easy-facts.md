# Skip Easy Facts Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Let a student in Single Number Power Mode exclude easy partner numbers (0, 1 as one group; 2, 10, 11 as another) from generated problems.

**Architecture:** All logic lives in `index.html` (single-file app). Two boolean flags on `settings` drive a shared `excludedPartners()` helper that both problem builders consult in their single-number paths. Two toggle chips render under the number grid, visible only when exactly one number is selected, labeled with the literal problems they skip.

**Tech Stack:** Vanilla JS, single HTML file, no build step, no test framework. Verification is browser-console inspection of `buildDeck()` output plus manual UI testing.

## Global Constraints

- Exclusions apply **only** to Single Number Power Mode (`settings.numbers.length === 1`). Multi-number problem generation must remain byte-identical to current behavior.
- Exclusions apply **only** to the partner operand, never to the number selected in the grid. Selecting `0` with the `0, 1` group enabled must still produce `0 × 3`, `0 × 7`, etc.
- Both flags default to `false` — existing users see no change until they opt in.
- Chips use a 44px minimum touch target, matching every other control on the settings screen.
- Chip labels interpolate the currently selected number: `8 × 0, 1` and `8 × 2, 10, 11`.
- Existing "single 0 + divide/mix" disabled state must continue to work unchanged.

---

## File Map

| File | What changes |
|---|---|
| `index.html:98` | Add `.skip-easy`, `.skip-easy-label`, `.skip-chips`, `.skip-chip` CSS after `.grid-hint` |
| `index.html:369` | Add skip-chips markup after the `grid-hint` paragraph |
| `index.html:537-543` | `settings` object — add `skipEasy01`, `skipEasy2Etc` |
| `index.html:546-571` | `loadSettings` — read both flags defensively |
| `index.html:574-584` | `saveSettings` — persist both flags |
| `index.html:599-604` | Tap-sound id list — add `skip-easy` |
| `index.html:629-649` | `renderNumberGrid` — call `renderSkipChips()` |
| `index.html:740-744` | New `renderSkipChips()` + chip click wiring |
| `index.html:1093-1117` | `buildMultiplyProblems` — filter partners in single-number path |
| `index.html:1119-1144` | `buildDivideProblems` — filter quotients in single-number path |

Line numbers are from the current `main` (`f4d0d35`). Each task uses find-and-replace on exact text, so drift is tolerable.

---

## Task 1: Settings state and persistence

Adds the two flags and makes them survive a reload. No visible behavior yet.

**Files:**
- Modify: `index.html:537-543` (settings object)
- Modify: `index.html:546-571` (`loadSettings`)
- Modify: `index.html:574-584` (`saveSettings`)

**Interfaces:**
- Produces: `settings.skipEasy01` (boolean, default `false`), `settings.skipEasy2Etc` (boolean, default `false`). Tasks 2 and 3 read and write these.

- [ ] **Step 1: Add the two flags to the settings object**

Find:
```js
    let settings = {
      operation: 'multiply',
      numbers: [],
      sessionType: 'problems',
      sessionValue: 20,
      muted: false,
    };
```

Replace with:
```js
    let settings = {
      operation: 'multiply',
      numbers: [],
      sessionType: 'problems',
      sessionValue: 20,
      skipEasy01: false,
      skipEasy2Etc: false,
      muted: false,
    };
```

- [ ] **Step 2: Read the flags in `loadSettings`**

Find:
```js
          if (Number.isInteger(saved.sessionValue)) {
            if (settings.sessionType === 'problems') {
              settings.sessionValue = Math.min(50, Math.max(5, Math.round(saved.sessionValue / 5) * 5));
            } else {
              settings.sessionValue = Math.min(10, Math.max(1, saved.sessionValue));
            }
          }
```

Replace with:
```js
          if (Number.isInteger(saved.sessionValue)) {
            if (settings.sessionType === 'problems') {
              settings.sessionValue = Math.min(50, Math.max(5, Math.round(saved.sessionValue / 5) * 5));
            } else {
              settings.sessionValue = Math.min(10, Math.max(1, saved.sessionValue));
            }
          }
          if (typeof saved.skipEasy01 === 'boolean') {
            settings.skipEasy01 = saved.skipEasy01;
          }
          if (typeof saved.skipEasy2Etc === 'boolean') {
            settings.skipEasy2Etc = saved.skipEasy2Etc;
          }
```

The `typeof ... === 'boolean'` guard matches the defensive style already used for the other fields — a missing or corrupt value leaves the `false` default in place.

- [ ] **Step 3: Persist the flags in `saveSettings`**

Find:
```js
        localStorage.setItem(LS_SETTINGS, JSON.stringify({
          operation:    settings.operation,
          numbers:      settings.numbers,
          sessionType:  settings.sessionType,
          sessionValue: settings.sessionValue,
        }));
```

Replace with:
```js
        localStorage.setItem(LS_SETTINGS, JSON.stringify({
          operation:    settings.operation,
          numbers:      settings.numbers,
          sessionType:  settings.sessionType,
          sessionValue: settings.sessionValue,
          skipEasy01:   settings.skipEasy01,
          skipEasy2Etc: settings.skipEasy2Etc,
        }));
```

- [ ] **Step 4: Verify in the browser console**

Open `index.html`, then in the console run:

```js
settings.skipEasy01 = true;
saveSettings();
JSON.parse(localStorage.getItem('mn_lastSettings'));
```

Expected: the returned object contains `skipEasy01: true` and `skipEasy2Etc: false`.

Then reload the page and run:

```js
settings.skipEasy01;   // expected: true
settings.skipEasy2Etc; // expected: false
```

Finally reset for the next task:

```js
settings.skipEasy01 = false; saveSettings();
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add skipEasy settings flags with persistence"
```

---

## Task 2: Exclusion helper and problem builder filtering

Makes the flags actually change which problems are generated.

**Files:**
- Modify: `index.html:1093-1117` (`buildMultiplyProblems`)
- Modify: `index.html:1119-1144` (`buildDivideProblems`)

**Interfaces:**
- Consumes: `settings.skipEasy01`, `settings.skipEasy2Etc` from Task 1.
- Produces: `excludedPartners()` returning `number[]` — the partner values to skip. Task 3 does not use it; it exists only for the builders.

- [ ] **Step 1: Add the `excludedPartners` helper**

Insert immediately above `function buildMultiplyProblems(numbers) {`, under the existing `// ── Problem generation ───` comment banner.

Find:
```js
    // ── Problem generation ───────────────────────────────────────
    function buildMultiplyProblems(numbers) {
```

Replace with:
```js
    // ── Problem generation ───────────────────────────────────────
    // Partner values the student has opted to skip in single-number mode.
    // Never filters the selected number itself — only its partner.
    function excludedPartners() {
      const out = [];
      if (settings.skipEasy01)   out.push(0, 1);
      if (settings.skipEasy2Etc) out.push(2, 10, 11);
      return out;
    }

    function buildMultiplyProblems(numbers) {
```

- [ ] **Step 2: Filter partners in `buildMultiplyProblems`**

Find:
```js
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
```

Replace with:
```js
      if (numbers.length === 1) {
        const X = numbers[0];
        const skip = excludedPartners();
        const problems = [];
        for (let n = 0; n <= 12; n++) {
          if (skip.includes(n)) continue;
          problems.push({ type:'multiply', a:n, b:X, answer:n*X,
            display:`${n} × ${X}`, key:`${n}×${X}` });
          if (n !== X) {
            problems.push({ type:'multiply', a:X, b:n, answer:X*n,
              display:`${X} × ${n}`, key:`${X}×${n}` });
          }
        }
        return problems;
      }
```

`n` is the partner and `X` is the student's selection, so filtering on `n` alone upholds the invariant — `X` is never filtered out.

- [ ] **Step 3: Filter quotients in `buildDivideProblems`**

Find:
```js
      if (numbers.length === 1) {
        const X = numbers[0];
        if (X === 0) return []; // cannot divide by zero
        const problems = [];
        for (let n = 1; n <= 12; n++) {
          if (n === X) continue; // answer must not equal divisor
          const dividend = n * X;
```

Replace with:
```js
      if (numbers.length === 1) {
        const X = numbers[0];
        if (X === 0) return []; // cannot divide by zero
        const skip = excludedPartners();
        const problems = [];
        for (let n = 1; n <= 12; n++) {
          if (n === X) continue; // answer must not equal divisor
          if (skip.includes(n)) continue; // quotient is an opted-out easy partner
          const dividend = n * X;
```

- [ ] **Step 4: Verify multiply filtering in the browser console**

Reload `index.html`, then run:

```js
settings.skipEasy01 = true; settings.skipEasy2Etc = true;
buildDeck('multiply', [8]).map(p => p.display).sort().join('  ');
```

Expected: 15 problems, with partners drawn only from `{3,4,5,6,7,8,9,12}`. Specifically it must contain `8 × 3` and `3 × 8`, contain `8 × 8` exactly once, and contain none of `0`, `1`, `2`, `10`, or `11` as a partner.

Confirm the count:

```js
buildDeck('multiply', [8]).length;  // expected: 15
```

- [ ] **Step 5: Verify the core invariant — the selection is never filtered**

```js
settings.skipEasy01 = true; settings.skipEasy2Etc = true;
buildDeck('multiply', [0]).map(p => p.display).sort().join('  ');
```

Expected: problems still exist and every one involves `0` — e.g. `0 × 3`, `3 × 0`, `0 × 12`. The student's selection of `0` survives even though the `0, 1` group is enabled.

```js
buildDeck('multiply', [0]).length;  // expected: 16
```

16 rather than 15 here: because `X` is itself in the excluded set, the loop never reaches `n === X`, so the `n !== X` guard never suppresses a reversed pair — all 8 surviving partners contribute both orderings.

- [ ] **Step 6: Verify divide filtering**

```js
settings.skipEasy01 = true; settings.skipEasy2Etc = true;
buildDeck('divide', [8]).map(p => `${p.display} = ${p.answer}`).sort().join('  ');
```

Expected: 7 problems, all of the form `something ÷ 8`, with answers drawn only from `{3,4,5,6,7,9,12}`. No answer of `1`, `2`, `10`, or `11`, and no answer of `8`.

```js
buildDeck('divide', [8]).length;  // expected: 7
```

- [ ] **Step 7: Verify multi-number mode is untouched**

```js
settings.skipEasy01 = true; settings.skipEasy2Etc = true;
buildDeck('multiply', [2, 10, 11]).length;  // expected: 9 (3 × 3 cross-product)
buildDeck('multiply', [2, 10, 11]).map(p => p.display).sort().join('  ');
```

Expected: all nine combinations of 2, 10, and 11 are present despite both skip groups being enabled — multi-number mode ignores the flags entirely.

Reset before the next task:

```js
settings.skipEasy01 = false; settings.skipEasy2Etc = false; saveSettings();
```

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: exclude opted-out easy partners from single-number problems"
```

---

## Task 3: Skip chips UI

Adds the visible controls: styles, markup, render function, and click wiring.

**Files:**
- Modify: `index.html:98` (CSS, after `.grid-hint`)
- Modify: `index.html:369` (markup, after the `grid-hint` paragraph)
- Modify: `index.html:599-604` (tap-sound id list)
- Modify: `index.html:629-649` (`renderNumberGrid`)
- Modify: `index.html:740-744` (new render function + wiring, after the sound toggle)

**Interfaces:**
- Consumes: `settings.skipEasy01`, `settings.skipEasy2Etc` from Task 1; `saveSettings()` and `renderNumberGrid()` from existing code.
- Produces: `renderSkipChips()` taking no arguments and returning nothing.

- [ ] **Step 1: Add the CSS**

Find:
```css
    .grid-hint { font-size: 11px; color: var(--muted); text-align: center; margin-top: 8px; }
```

Replace with:
```css
    .grid-hint { font-size: 11px; color: var(--muted); text-align: center; margin-top: 8px; }

    /* ── Skip-easy chips (single-number mode only) ── */
    .skip-easy { display: none; margin-top: 12px; }
    .skip-easy.visible { display: block; }
    .skip-easy-label {
      font-size: 10px; font-weight: 700; letter-spacing: 1.5px;
      text-transform: uppercase; color: var(--muted);
      text-align: center; margin-bottom: 8px;
    }
    .skip-chips { display: flex; gap: 8px; }
    .skip-chip {
      flex: 1; background: var(--bg2); color: var(--muted);
      border: 2px solid var(--bg3); border-radius: 10px;
      padding: 10px 8px; font-size: 13px; font-weight: 800;
      font-family: var(--font); cursor: pointer;
      min-height: 44px; transition: all 0.15s;
    }
    .skip-chip.active {
      background: var(--red); color: white; border-color: var(--red);
      text-decoration: line-through;
    }
```

The `.visible` class rather than a `.hidden` class means the chips are hidden by default, so they never flash on load before `renderSkipChips()` runs.

- [ ] **Step 2: Add the markup**

Find:
```html
        <div class="number-grid" id="number-grid"></div>
        <p class="grid-hint" id="grid-hint"></p>
```

Replace with:
```html
        <div class="number-grid" id="number-grid"></div>
        <p class="grid-hint" id="grid-hint"></p>
        <div class="skip-easy" id="skip-easy">
          <p class="skip-easy-label">Skip the easy ones</p>
          <div class="skip-chips">
            <button type="button" class="skip-chip" id="skip-chip-01" aria-pressed="false"></button>
            <button type="button" class="skip-chip" id="skip-chip-2etc" aria-pressed="false"></button>
          </div>
        </div>
```

Chip labels are left empty in the markup — `renderSkipChips()` fills them in with the selected number.

- [ ] **Step 3: Add `renderSkipChips`**

Find:
```js
    // Wire sound toggle
    document.getElementById('sound-toggle-row').addEventListener('click', () => {
```

Replace with:
```js
    // ── Skip-easy chips ──────────────────────────────────────────
    // Visible only in single-number mode; labels name the selected number
    // so it is unambiguous that the groups apply to its partner.
    function renderSkipChips() {
      const box = document.getElementById('skip-easy');
      const single = settings.numbers.length === 1;
      box.classList.toggle('visible', single);
      if (!single) return;

      const X = settings.numbers[0];
      const chips = [
        [document.getElementById('skip-chip-01'),   `${X} × 0, 1`,       settings.skipEasy01],
        [document.getElementById('skip-chip-2etc'), `${X} × 2, 10, 11`,  settings.skipEasy2Etc],
      ];
      chips.forEach(([el, label, on]) => {
        el.textContent = label;
        el.classList.toggle('active', on);
        el.setAttribute('aria-pressed', on ? 'true' : 'false');
      });
    }

    document.getElementById('skip-chip-01').addEventListener('click', () => {
      settings.skipEasy01 = !settings.skipEasy01;
      renderSkipChips();
      saveSettings();
    });

    document.getElementById('skip-chip-2etc').addEventListener('click', () => {
      settings.skipEasy2Etc = !settings.skipEasy2Etc;
      renderSkipChips();
      saveSettings();
    });

    // Wire sound toggle
    document.getElementById('sound-toggle-row').addEventListener('click', () => {
```

- [ ] **Step 4: Call `renderSkipChips` from `renderNumberGrid`**

`renderNumberGrid` already runs on boot and on every selection change, so it is the single hook needed to keep chip visibility and labels current.

Find:
```js
      allBtn.setAttribute('aria-pressed', allSelected() ? 'true' : 'false');
      grid.appendChild(allBtn);
      updateDojoBtn();
    }
```

Replace with:
```js
      allBtn.setAttribute('aria-pressed', allSelected() ? 'true' : 'false');
      grid.appendChild(allBtn);
      updateDojoBtn();
      renderSkipChips();
    }
```

- [ ] **Step 5: Add the tap sound**

Clicks bubble from each chip up to the `skip-easy` container, so registering the container is enough for both.

Find:
```js
      ['op-toggle', 'st-toggle', 'number-grid', 'stepper-minus', 'stepper-plus',
       'sound-toggle-row', 'about-btn', 'about-close', 'exit-btn', 'overlay-cancel', 'overlay-leave',
       'train-again-btn', 'results-settings-btn', 'numpad'].forEach(id => {
```

Replace with:
```js
      ['op-toggle', 'st-toggle', 'number-grid', 'skip-easy', 'stepper-minus', 'stepper-plus',
       'sound-toggle-row', 'about-btn', 'about-close', 'exit-btn', 'overlay-cancel', 'overlay-leave',
       'train-again-btn', 'results-settings-btn', 'numpad'].forEach(id => {
```

- [ ] **Step 6: Verify the chips render and toggle**

Reload `index.html` and, using the UI only:

1. With no numbers selected, confirm no chips are visible.
2. Tap `8`. Confirm the `SKIP THE EASY ONES` label appears with two chips reading `8 × 0, 1` and `8 × 2, 10, 11`.
3. Tap the first chip. Confirm it turns red with a line through the text, and a tap sound plays.
4. Tap `5` as well (two numbers now selected). Confirm the whole chip block disappears.
5. Tap `5` again to deselect (back to just `8`). Confirm the chips reappear and the first chip is still active.
6. Deselect `8`, then select `3`. Confirm the labels now read `3 × 0, 1` and `3 × 2, 10, 11`.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add skip-easy chips to single-number mode"
```

---

## Task 4: End-to-end manual verification

Full-app checks that the console tests in Task 2 cannot cover.

**Files:** none — verification only.

- [ ] **Scenario 1 — Multiply with both groups off**
  - Select `8`, both chips inactive, operation Multiply, session length 50 problems
  - Enter dojo → confirm `8 × 0`, `8 × 1`, `8 × 2`, `8 × 10`, `8 × 11` all still appear (behavior unchanged from before this feature)

- [ ] **Scenario 2 — Multiply with the `0, 1` group active**
  - Select `8`, activate only the `8 × 0, 1` chip, 50 problems
  - Enter dojo → confirm no problem has `0` or `1` as the partner
  - Confirm `8 × 2`, `8 × 10`, `8 × 11` still appear

- [ ] **Scenario 3 — Multiply with both groups active**
  - Select `8`, activate both chips, 50 problems
  - Enter dojo → partners are only `3,4,5,6,7,8,9,12`; the 15-problem deck cycles and reshuffles without error

- [ ] **Scenario 4 — Divide with both groups active**
  - Select `8`, both chips active, operation Divide
  - Enter dojo → every problem is `something ÷ 8` with an answer in `{3,4,5,6,7,9,12}`
  - Confirm no `8 ÷ 8`, `16 ÷ 8`, `80 ÷ 8`, or `88 ÷ 8`

- [ ] **Scenario 5 — Mix with both groups active**
  - Select `8`, both chips active, operation Mix
  - Enter dojo → both multiply and divide problems appear, all respecting the exclusions

- [ ] **Scenario 6 — The core invariant**
  - Select `0`, activate both chips, operation Multiply
  - Enter dojo → problems still appear and all involve `0`, e.g. `0 × 3`, `7 × 0`
  - The student's own selection is never filtered out

- [ ] **Scenario 7 — Multi-number mode unaffected**
  - Activate both chips while `8` is selected, then also select `2`, `10`, `11`
  - Chips disappear; enter dojo → the full cross-product of `2, 8, 10, 11` appears including `2 × 10`, `11 × 11`

- [ ] **Scenario 8 — Persistence across reload**
  - Select `8`, activate both chips, reload the page
  - Confirm `8` is still selected and both chips are still active
  - Enter dojo → exclusions still apply

- [ ] **Scenario 9 — State survives a round trip through multi-number mode**
  - With `8` selected and both chips active, select `3` as well, then deselect `3`
  - Confirm both chips return in the active state

- [ ] **Scenario 10 — Existing disabled state still works**
  - Select `0` alone, switch operation to Divide
  - Confirm ENTER DOJO is disabled, and re-enables on switching back to Multiply
  - Confirm the chips remain visible and functional throughout

- [ ] **Scenario 11 — Layout on a narrow screen**
  - Load in a ~375px-wide viewport (iPhone size) with `12` selected so labels are at their longest
  - Confirm both chips fit side by side without text wrapping awkwardly or overflowing
