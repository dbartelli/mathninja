# Single Number Power Mode — Design Spec

**Date:** 2026-04-14
**Status:** Approved

## Overview

Allow a student to select exactly one number on the settings screen and enter the Dojo to practice all facts involving that number. When one number X is selected, every question pairs X against the full 0–12 range (rather than requiring at least 2 numbers to cross-multiply/divide). Two or more numbers selected continues to behave exactly as today.

---

## Problem Generation (Approach A — detect inside builders)

### `buildMultiplyProblems` — single-number path

When `numbers.length === 1` (X = the selected number), generate both orderings of X paired with each n in 0–12:
- For each n in [0, 1, ..., 12]: emit `n × X` and `X × n`
- When `n === X`, emit only once (`X × X`) to avoid a duplicate

Result for X=5: 25 problems — `0×5`, `5×0`, `1×5`, `5×1`, ..., `5×5` (once), ..., `12×5`, `5×12`.

### `buildDivideProblems` — single-number path

When `numbers.length === 1` and X ≠ 0, generate `(n × X) ÷ X = n` for n in [1..12], excluding n = X (so the answer never equals the divisor X).

Result for X=5: 11 problems — answers 1,2,3,4,6,7,8,9,10,11,12 (answer 5 excluded, 0 excluded per division rule).

When X = 0: no valid divide problems (zero cannot be a divisor). See Button Guards below.

### Mix mode

Combines the single-number multiply and divide problem sets using the same rules above. If X = 0, mix mode produces multiply-only problems (divide returns empty).

---

## UI — Hint Text

The existing `<p class="grid-hint">Select at least 2 numbers</p>` (line 369) becomes a dynamic element. Its text content is updated whenever the number selection changes (inside `renderNumberGrid`):

| Selection state       | Message displayed                                      |
|-----------------------|--------------------------------------------------------|
| 0 numbers selected    | *(empty — no message)*                                 |
| 1 number selected (N) | `⚡ Single Number Power Mode — crushing the [N] facts!` |
| 2+ numbers selected   | *(empty — no message)*                                 |

`[N]` is replaced with the actual selected number at render time.

---

## Button Enable / Guards

**Enable condition (settings screen):**
- General: `settings.numbers.length >= 1` (changed from `>= 2`)
- Exception: if `settings.numbers.length === 1` AND the selected number is `0` AND `settings.operation === 'divide'` → button remains disabled (no valid divide problems for X=0)

**`enterDojo` function (line 1130):**
- Guard updated to `if (settings.numbers.length < 1) return;`

**`dojo-btn` click handler (line 736):**
- Guard updated to `if (settings.numbers.length < 1) return;`

---

## Changed Locations Summary

| Location | Change |
|---|---|
| Line 369 — `<p class="grid-hint">` | Static text removed; text set dynamically |
| Line 668 — button `disabled` logic | `< 2` → `< 1`, plus X=0/divide exception |
| Line 736 — click handler guard | `< 2` → `< 1` |
| Line 1081 — `buildMultiplyProblems` | Add single-number path at top of function |
| Line 1092 — `buildDivideProblems` | Add single-number path at top of function |
| Line 1130 — `enterDojo` guard | `< 2` → `< 1` |
| `renderNumberGrid` (line ~635) | Add hint text update logic after rendering buttons |

---

## Out of Scope

- No changes to results screen, streak logic, or session length behavior.
- No changes to the "all selected" toggle.
- No changes to how 2+ number selection works.
