# MathNinja 🥷

A math facts practice app for 2nd and 3rd graders. Built for Diego's Math Dojo.

**[Play it →](https://dbartelli.github.io/mathninja/)**

## What it does

- Multiplication, division, or mixed practice for numbers 0–12
- Two modes: fixed number of problems (5–50) or timed sessions (1–10 min)
- Two tries per problem before the answer is revealed
- Streak tracking with milestone messages
- Positive reinforcement messages display longer when the message is longer — giving young readers enough time to read it; a ⏭ skip button lets kids advance early if they're ready
- Results screen with rank badge (White Belt → Grand Master) based on first-try accuracy
- Sound effects via Web Audio API with mute toggle
- Settings and personal bests persist across sessions via localStorage

## How to use

1. Open the app (or add to iPhone/iPad home screen for full-screen mode)
2. Pick an operation, select which numbers to train, set session length
3. Tap **ENTER DOJO** and type your answers — press Enter or tap Done to submit
4. After two wrong answers the correct answer is shown; those facts appear in the results for targeted review

## Tech

Single `index.html` file — vanilla HTML, CSS, and JavaScript. No build tools, no dependencies, no external requests. Hosted on GitHub Pages.

## Development

```bash
git clone https://github.com/dbartelli/mathninja.git
cd mathninja
open index.html
```
