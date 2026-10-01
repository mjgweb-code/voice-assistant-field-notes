# Voice assistant field notes

Scrubbed field notes and findings from building a self-hosted voice assistant on ESPHome voice
satellites (an Atom Echo and a Home Assistant Voice Preview Edition) and the Gemini API — no Home
Assistant, no cloud assistant platform, a from-scratch server.

This repo holds two kinds of notes:
- **Archive copies** of findings that were also posted as comments/issues on the project they're
  actually about (ESPHome, the Gemini SDK, etc.) — kept here too so they survive even if the
  original thread is ever edited, archived, or closed. Each one links back to where it was
  originally posted.
- **Standalone notes** for findings with no single natural upstream home.

Nothing here is a general tutorial or a finished product write-up — these are narrow, specific
things that cost real debugging time and seemed worth writing down for the next person who hits the
same wall.

## Methodology

I work with an AI coding assistant (Claude, Anthropic) on this project. It helps research and verify
these findings and drafts the writeups; I own the hardware/software, run the real tests, and make
the actual calls on what's reported. Every note here was drafted, reviewed, and approved before
being posted anywhere.

## Notes

- [`notes/`](notes/) — one file per finding, newest first in the index below.

| Date | Note | Originally posted at |
|---|---|---|
| 2026-10-01 | [Wake-word sensitivity is a real runtime control](notes/2026-10-01-wake-word-sensitivity-runtime-control.md) | [esphome/home-assistant-voice-pe#471](https://github.com/esphome/home-assistant-voice-pe/issues/471#issuecomment-5937132033) |
| 2026-10-01 | [Gemini's thinking tokens eat your output budget](notes/2026-10-01-gemini-thinking-tokens-eat-output-budget.md) | [googleapis/python-genai#782](https://github.com/googleapis/python-genai/issues/782#issuecomment-5937252172) |
| 2026-10-01 | [ESPHome voice protocol gotchas](notes/2026-10-01-esphome-voice-protocol-gotchas.md) | [FutureProofHomes/Satellite1-ESPHome#540](https://github.com/FutureProofHomes/Satellite1-ESPHome/issues/540#issuecomment-5937969925) |
