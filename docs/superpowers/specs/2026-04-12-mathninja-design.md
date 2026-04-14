# MathNinja — Design Spec
**Date:** 2026-04-12  
**Status:** Approved for implementation

---

## Overview

MathNinja is a single HTML file web app for 2nd and 3rd graders to practice multiplication and division facts (0–12). It runs in Safari on iPad and is hosted on GitHub Pages so any kid can access it via a URL. No graphics — text, color, and emojis only. Dark and bold visual theme. Positive reinforcement with humor throughout.

Primary user: elementary school kids (ages 7–9) operating the app independently on their own iPad.  
Secondary user: parents/teachers who configure the session before handing it to a kid.

---

## Technology

- **Delivery:** Single self-contained `.html` file. All CSS, JS, and audio inline — zero external dependencies, zero build step.
- **Hosting:** GitHub Pages (free static hosting). Shareable via URL. Works as an iPad home screen shortcut ("Add to Home Screen" in Safari).
- **Audio:** Web Audio API — tones and sounds generated programmatically, no audio files required.
- **Persistence:** `localStorage` for per-device history (best rank, best streak, total sessions). Each kid uses their own device so no multi-user collision.
- **No backend, no accounts, no data leaves the device.**
- All `localStorage` calls wrapped in try/catch — fails silently if unavailable (e.g., Safari private mode). App is fully functional without persistence; history features simply don't appear.
- AudioContext initialized on the "Enter Dojo" tap to comply with browser autoplay policy (iOS requires a user gesture before audio can play).

---

## App Structure — 3 Screens

All three screens live in one HTML file. JavaScript shows/hides sections. No page navigation.

```
Settings Screen → Practice Screen → Results Screen → (back to Settings)
```

A small ⚙️ gear icon on the Practice screen returns to Settings at any time (abandons current session).

---

## Screen 1: Settings / Start

### Layout
- **Header (branding):** 🥷 emoji, "MATHNINJA" in large bold text, tagline below: *"⚔️ Welcome to Diego's Math Dojo. Select your training. ⚔️"*
- Thin horizontal rule separates branding from settings
- Settings sections below, then "ENTER DOJO" button at the bottom
- Sound toggle (🔊 / 🔇) in small text below the button

### Settings Controls

**Operation Mode** — 3-button toggle (one active at a time):
- ✖️ Multiply
- ➗ Divide
- 🔀 Mix (both interleaved randomly)

**Numbers to Train** — grid of 13 tap-toggle buttons (0–12):
- Displayed in a 7-column grid (0–6 top row, 7–12 bottom row)
- Inactive: dark background, dim text
- Active/selected: red background, white text
- Minimum 2 numbers must be selected. The last remaining selected button cannot be deselected; the second-to-last can only be deselected if there are currently ≥ 3 selected.
- "Enter Dojo" button is disabled (grayed) if fewer than 2 numbers are selected
- Settings are saved to localStorage so they persist for the next session

**Session Length** — 2-button toggle:
- 📋 Problems: stepper showing a number (default 20), with − and + tap buttons to adjust (range: 5–50, step 5)
- ⏱ Time: stepper showing minutes (default 3), with − and + tap buttons (range: 1–10 minutes, step 1)

**Enter Dojo button** — full-width, prominent red gradient button, bold text: `⚔️ ENTER DOJO ⚔️`
- Disabled (grayed) if no numbers are selected

---

## Screen 2: Practice

### Problem Generation

- Problems are generated from the selected numbers and operation mode.
- **Multiplication:** both factors drawn from the selected number set. Example: if {2,3,4,6} selected, problems include 2×3, 6×4, 4×4, etc.
- **Division:** dividend and divisor generated so the answer is always a whole number. Divisor is drawn from the selected set; dividend = divisor × another number from the set. **0 is silently excluded from the divisor pool** (dividing by 0 is undefined) but 0 can appear as a dividend (e.g., 0÷3=0 is valid).
- **Mix mode:** multiplication and division problems interleaved randomly.
- Problems are shuffled so the same problem doesn't repeat until the full set has been exhausted (deck-of-cards model).
- **Session length:** if Problems mode, session ends after N attempts. If Time mode, the timer expiring does NOT cut off the current problem — the kid finishes submitting their current answer (or sees the reveal), then the results screen appears. The session may run a few seconds over the chosen time; that is intentional.

### Layout

**Top bar (left → right):**
- ⚙️ gear icon (tapping shows a "Leave session?" confirmation prompt before abandoning — no accidental exit)
- 🔥 streak counter badge (e.g., "🔥 5 streak") — hidden until streak ≥ 2
- Progress indicator: "8 / 20" or countdown timer "2:14" depending on session mode

**Progress bar:**
- **Problems mode:** fills left to right as problems are completed. Gradient red→gold.
- **Time mode:** starts full and depletes right-to-left as time runs out. Turns red in the final 30 seconds.

**Problem display (center of screen):**
- Operation label in small caps above (e.g., "MULTIPLICATION")
- Problem in very large bold text: `7 × 8` (64px+)
- `= ?` on the next line in red
- Answer input box below: large, centered, accepts integers only, auto-focused on each new problem
- "type your answer" hint text below input

**Answer input behavior:**
- Accepts digits only (non-digit keypresses ignored, including minus signs)
- Maximum 3 characters (largest possible answer is 144)
- Enter key or tapping a submit button submits the answer
- Input is locked/disabled during the feedback window (1.2s correct / 1.8s reveal) to prevent accidental early advancement
- On iPad: numeric keyboard shown automatically (`inputmode="numeric"`)

### Answer Feedback States

**Correct (1st try only):**
- Input box border flashes green
- Problem line updates: `7 × 8 = 56 ✓` in green
- Feedback bubble appears below with a message from the correct-answer pool (see below)
- Streak counter increments; streak milestone messages at 3, 5, 10, and every 5 after
- Sound: cheerful ascending tone
- Auto-advances to next problem after ~1.2 seconds

**Correct (2nd try):**
- Same green feedback and sound as above
- Streak does NOT increment (only 1st-try correct answers build the streak)
- Auto-advances after ~1.2 seconds

**Wrong (1st try):**
- Input box border flashes red and clears
- Feedback bubble: `Try Again! 💪` with "1 try left" subtext in red
- Sound: gentle descending tone
- Stays on same problem

**Wrong (2nd try — reveal):**
- Problem line updates: `7 × 8 = 56 💡` in gold/yellow
- Feedback bubble: `The answer is 56!` + `"You'll get it next time 🥷"` in gold
- Sound: soft low tone (different from 1st wrong)
- Streak resets to 0
- Auto-advances after ~1.8 seconds (slightly longer so they can read the answer)

**Streak milestones (on correct answer that hits milestone):**
- 3-in-a-row: feedback message notes the streak
- 5-in-a-row: larger celebration message + sound flourish
- 10-in-a-row: biggest in-session celebration

### Correct Answer Message Pool

Messages are selected randomly without repeating until the full pool is exhausted. Categories:

**Solo correct (no streak context):**
- "Correct! ⚡"
- "Nailed it! 🥷"
- "Your brain is a weapon."
- "That problem never stood a chance."
- "Math agrees with you."
- "Boom. 💥"
- "Easy for a ninja."
- "Was that even hard? Didn't think so."
- "The answer feared you."
- "Clean. Sharp. Correct."
- "Calculator? You don't need one."
- "Facts only. 🔥"
- "Zero hesitation. Full marks."
- "Did you even blink?"
- "Smooth."

**Streak messages (shown when streak ≥ 3):**
- "🔥 3 in a row! You're on fire!"
- "🔥 4 in a row! The numbers are scared."
- "🔥 5 in a row! Full ninja mode activated."
- "🔥 5 in a row! Someone call the math police."
- "🔥 6 in a row! Unstoppable."
- "🔥 7 in a row! Is this even fair?"
- "🔥 8 in a row! You might actually be a robot. A cool robot."
- "🔥 9 in a row! The dojo bows before you."
- "🔥 10 in a row! 🏆 LEGENDARY. TEN. IN. A. ROW."
- "🔥 10 in a row! Mathematicians are writing papers about you."
- Every 5 after 10: "🔥 [N] in a row! Stop showing off. (Actually don't.)"

**Funny/clever:**
- "7 times 8 fears you."
- "Multiplication has given up trying to trick you."
- "Your brain just did a backflip."
- "That fact didn't even have time to be nervous."
- "If math were a video game, you'd be at the final boss."
- "Even the hard ones are easy for you. Suspicious."
- "Sir/Ma'am, this is a math app, not a demolition site."
- "You just divided that like it owed you money."
- "That one was a trap. You walked right through it."
- "12 × 12 is sweating right now."

---

## Screen 3: Results

### Layout

**Ninja Rank Badge (top center):**
- Large 🥷 emoji
- Colored rank badge based on **1st-try correct percentage only** (see tiers below)
- Subtitle: "Ranked on first-try score: 75%" — always shown so the basis is transparent
- If new personal best rank (strictly higher than localStorage `mn_bestRankIndex`): additional line "Your highest rank yet! 🎉"
- First-ever session: no rank comparison, just the badge and score subtitle

**Session Message:**
- One message drawn from a performance-tiered pool (funny, encouraging, scaled to score)
- Displayed in a styled quote box

**Stats Row (3 boxes):**
- ⚡ "First try!" — count of problems answered correctly on the first attempt
- 💪 "Second try" — count of problems answered correctly on the second attempt
- 💡 "Needed help" — count of problems that required the answer to be revealed

The three stats always sum to the total number of problems in the session. This layout makes it immediately clear why a rank was earned — a kid who sees "First try: 15, Second try: 5, Needed help: 0" understands why they're not Grand Master without any explanation.

🔥 **Best streak** shown separately below the stats row as: "Best streak this session: 7 🔥"

**"Extra Dojo Training" section:**
- Lists only facts that required a reveal (wrong twice) as pill badges
- Label: "⚔️ These need more dojo time" — communicates that the kid is already training on these numbers, they just need more focused repetition on these specific facts
- Hidden if no facts were revealed (score may still be < 100% if some were 2nd-try correct)
- Facts answered wrong once but correct on 2nd try are NOT shown here — those are working, just slower

**Buttons:**
- `⚔️ TRAIN AGAIN` — full-width red button, restarts practice with same settings
- `⚙️` — small gear button, returns to Settings screen

### Ninja Rank Tiers

Rank is calculated from **1st-try correct %** = (First try correct) ÷ (total problems attempted).

| 1st-try % | Rank | Badge Color |
|-----------|------|-------------|
| 100% | Grand Master 🏆 | Gold gradient |
| 90–99% | Shadow Ninja | Gold text |
| 80–89% | Black Belt | White/silver |
| 65–79% | Green Belt | Green |
| 50–64% | Yellow Belt | Yellow |
| 0–49% | White Belt | Blue |

### Results Message Pool (by tier)

**Grand Master (100%):**
- "Perfect. Not a single fact escaped you."
- "100%. The dojo has nothing left to teach you... for now."
- "Grand Master status achieved. The numbers kneel."

**Shadow Ninja (90–99%):**
- "Almost flawless. Almost. The dojo respects your effort."
- "The numbers barely had a chance. Nearly perfect."
- "One or two slipped through. Shadow Ninjas hunt those down next session."

**Black Belt (80–89%):**
- "Strong session. Black Belts know which facts to train next."
- "Solid work. The dojo is pleased."
- "8 out of 10 ain't bad. The other 2 are running."

**Green Belt (65–79%):**
- "Good progress. Green Belts keep showing up — that's how it works."
- "More right than wrong. That's the direction."
- "The math is getting easier. You're getting harder."

**Yellow Belt (50–64%):**
- "Halfway there. Every ninja starts somewhere."
- "The hard facts fought back. Train them again."
- "Keep going. The streak is coming."

**White Belt (0–49%):**
- "Even the greatest ninjas had to train. A lot. Like, a whole lot. Go again."
- "The numbers won this round. They won't next time."
- "White Belt today, Grand Master eventually. That's the path."

---

## localStorage Schema

```json
{
  "mn_bestRankIndex": 0,
  "mn_bestStreak": 12,
  "mn_totalSessions": 7,
  "mn_lastSettings": {
    "operation": "multiply",
    "numbers": [2, 3, 4, 6],
    "sessionType": "problems",
    "sessionValue": 20
  }
}
```

- `mn_bestRankIndex`: index into rank tier array (0=White Belt … 5=Grand Master)
- `mn_bestStreak`: highest streak ever achieved across all sessions
- `mn_totalSessions`: total completed sessions
- `mn_lastSettings`: restored on next app open so settings are pre-populated

---

## Audio Design (Web Audio API)

All sounds synthesized — no files.

| Event | Sound |
|-------|-------|
| Correct answer | Short ascending two-tone chime (pleasant, bright) |
| Wrong answer (1st try) | Single descending soft tone (not harsh) |
| Wrong answer (2nd try / reveal) | Lower soft tone, distinct from 1st wrong |
| Streak milestone (5, 10…) | Quick ascending 3-note fanfare |
| Session complete | 4-note celebratory chime sequence |
| Button tap | Very subtle soft click |

Mute toggle persists in localStorage (`mn_muted: true/false`).

---

## Responsive / iPad Considerations

- Designed primary for iPad viewport (~768–1024px wide)
- Works on desktop browsers (centered, max-width container)
- All tap targets minimum 44×44px (Apple HIG guideline)
- `inputmode="numeric"` on answer input to trigger numeric keyboard on iOS
- `font-size` minimum 16px on inputs to prevent Safari auto-zoom
- Tested orientation: portrait primary, landscape functional

---

## Out of Scope

- User accounts or cloud sync
- Teacher dashboard or class-level reporting
- Hints or step-by-step explanations
- Animations beyond CSS transitions
- Multiple named profiles on one device
