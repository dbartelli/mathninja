# Skip Easy Facts — Design

**Date:** 2026-07-29
**Status:** Approved, ready for planning

## Problem

In Single Number Power Mode, selecting one number X generates problems pairing X
against the full 0–12 range. Several of those partners produce facts the student
has already mastered — multiplying by 0 or 1 is trivial, and 2, 10, and 11 have
shortcut tricks. These easy problems dilute practice time that should go to the
facts still being learned.

The student needs a way to exclude them without cluttering the settings screen or
creating confusion with the number selection grid above.

## Scope

Applies to **Single Number Power Mode only** (`settings.numbers.length === 1`).

Multi-number mode builds its cross-product from exactly the numbers the student
selected, so there is nothing extra to exclude there. Multi-number problem
generation is unchanged.

## Data Model

Two independent booleans on `settings`:

- `skipEasy01` — excludes partners **0 and 1**
- `skipEasy2Etc` — excludes partners **2, 10, and 11**

Both default to `false`.

At deck-build time the excluded set is derived from the two flags:

```
excluded = (skipEasy01 ? [0, 1] : []) + (skipEasy2Etc ? [2, 10, 11] : [])
```

Two flags rather than one because the groups represent different kinds of "easy":
0 and 1 are identity/annihilator facts, while 2, 10, and 11 are shortcut-trick
facts. A student may outgrow one group before the other.

### Persistence

Both flags join the existing `mn_lastSettings` localStorage blob alongside
`operation`, `numbers`, `sessionType`, and `sessionValue`. Load applies the same
defensive type-checking pattern already used for the other fields — a missing or
non-boolean value falls back to `false`.

State persists while the controls are hidden. Switching from `8` to `3,4,5` and
back to `8` restores the student's previous choices.

## Core Invariant

**Exclusions apply only to the partner operand, never to the number selected in
the grid.**

If the student selects `0` as their single number and also enables the `0, 1`
skip group, they still get `0 × 3`, `0 × 7`, and so on. The grid selection always
wins. This is what keeps the feature from contradicting the control above it.

## Problem Generation

### Multiply (single-number mode)

Current behavior pairs X against every `n` in 0–12, in both orderings
(`n × X` and `X × n`, the latter skipped when `n === X`).

New behavior: skip any `n` present in the excluded set. X itself is never
filtered.

### Divide (single-number mode)

Current behavior makes X the divisor and generates quotients `n` from 1–12,
excluding `n === X`.

New behavior: also skip any `n` in the excluded set. The quotient plays the same
role as the multiply partner — `2X ÷ X = 2` is the same trivial fact as
`X × 2` — so one shared exclusion set keeps the mental model simple rather than
having multiply and divide behave differently. Quotient 0 never occurs, so
enabling the `0, 1` group removes only quotient 1.

The existing guard disabling Enter Dojo for "single 0 + divide/mix" is unchanged.

### Mix

Both filtered generators are combined, as today.

### Deck-Size Safety

Worst case with both groups enabled:

- **Multiply:** partners reduce to `{3,4,5,6,7,8,9,12}` (8 values), yielding 15
  problems (8 in `n × X` order, 7 in `X × n` order).
- **Divide:** quotients reduce to at most `{3,4,5,6,7,9,12}` — 12 values minus
  the 4 excluded values in range (1, 2, 10, 11) minus X itself — for a minimum of
  7 problems.

Neither can produce an empty deck, so no guard rail, warning, or "not enough
problems" state is required. The existing reshuffle-on-exhaustion in
`loadNextProblem` handles small decks.

## UI

### Placement

Inside the existing "🎯 Numbers to Train" fieldset, directly below the number
grid and beneath the existing Single Number Power Mode hint line.

Rendered **only when exactly one number is selected**. In multi-number mode the
controls are absent from the DOM entirely, which is what keeps the settings
screen from getting cluttered — they are invisible in the common case.

### Layout

```
🎯 NUMBERS TO TRAIN
┌───┬───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │
├───┼───┼───┼───┼───┼───┼───┤
│ 7 │[8]│ 9 │10 │11 │12 │All│
└───┴───┴───┴───┴───┴───┴───┘

⚡ Single Number Power Mode — crushing the 8 facts!

  SKIP THE EASY ONES
  ┌──────────────────┐ ┌────────────────────────┐
  │  8 × 0, 1        │ │  8 × 2, 10, 11         │
  └──────────────────┘ └────────────────────────┘
```

### Wording

Chip labels show the **literal problems being skipped**, interpolating the
currently selected number: `8 × 0, 1` and `8 × 2, 10, 11`.

This is the design decision that resolves the confusion risk. Rather than
describing the second operand abstractly ("second number", "partner"), the
student reads an actual problem and sees their selected number in the first
position. There is no ambiguity about which operand the toggle affects.

Labels re-render whenever the selected number changes.

### Chip States

Small toggles in the existing `.toggle-btn` visual language.

- **Inactive** (default): dark background, muted text — these problems are
  included.
- **Active**: red fill with the label struck through — these problems are
  skipped.

The strikethrough plus the `SKIP THE EASY ONES` header together make "on" read
unmistakably as "excluded", resolving the tension with the red-means-selected
convention used elsewhere on the screen.

Chips follow the existing 44px minimum touch target and play the standard tap
sound like other settings controls.

## Testing

Manual verification in the browser:

1. Select a single number, enable each group independently and together, confirm
   the generated problems omit exactly the expected partners in multiply,
   divide, and mix.
2. Confirm the invariant: select `0` with the `0, 1` group enabled and verify
   `0 × n` problems still appear.
3. Confirm chips are hidden in multi-number mode and that multi-number problem
   generation is byte-identical to current behavior.
4. Confirm chip labels update when switching selected number.
5. Confirm state survives a page reload and a round trip through multi-number
   mode.
6. Confirm the "single 0 + divide" disabled state still behaves as before.
