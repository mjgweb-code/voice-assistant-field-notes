# The Voice PE / ESPHome microWakeWord "sensitivity" selector is a real runtime API, not a cosmetic label

*Originally posted as a comment on [esphome/home-assistant-voice-pe#471](https://github.com/esphome/home-assistant-voice-pe/issues/471#issuecomment-5937132033), 2026-10-01.*

One thing worth confirming here since a few people above are describing the same sensitivity/false-accept tradeoff: the "Wake word sensitivity" selector isn't just a cosmetic three-way toggle — it's a real runtime API. It calls `set_probability_cutoff()` directly on the loaded wake model (see `home-assistant-voice.yaml`'s "Wake word sensitivity" select entity and its `on_value` lambda), and the three levels are real, documented cutoff values: Slightly sensitive = 0.97, Moderately sensitive = 0.92, Very sensitive = 0.83. I confirmed this both by reading the firmware source and live on a real device (read the current value, changed it, confirmed the change took effect, reverted it) — no reflash needed either way.

I ran some actual numbers on this, specifically for the "loud background audio causes false accepts" case a few of you are describing (@lweberru's TV, @jdranchman's calls):

- **False-accept rate under real loud/noisy audio**: replayed ~24 minutes of recorded loud-room audio (TV/music-level interference) through the wake model at each cutoff — **0 false accepts** at 0.97 or 0.92, rising to **about 2.5/hour** once pushed down to 0.83 ("Very sensitive") or lower. (Small-corpus caveat: ~24 minutes only loosely bounds the true rate, it's not a tight guarantee — but directionally, going more sensitive does cost you here, as you'd expect.)
- **The other side of the tradeoff** — does going MORE sensitive actually help you get heard over loud audio — turned out weaker than I expected: a cutoff sweep recovered at most 3 of 9 near-miss failures even at the most permissive setting tested, and the shipped Moderate level (0.92) recovered none at all. A final, properly powered paired live test (20 pairs, Slightly vs. Very Sensitive) found only a small, inconsistent effect overall (+5 points), and the direction flipped by interference type — +20 points better for vocal interference, but **10 points worse for instrumental/music**.

So: Slightly/Moderately sensitive genuinely don't false-accept on loud audio in my testing, Very sensitive measurably does (though not wildly), and going more sensitive isn't a reliable fix for the opposite problem either — it's a real tradeoff with no clean answer, not something I found a trick around. If you want the fuller investigation into this general failure mode (distant talker + loud, uncontrolled nearby audio), I wrote one up here: [esphome/voice-kit-xmos-firmware#10](https://github.com/esphome/voice-kit-xmos-firmware/issues/10) — several other DSP-level approaches were tried and all closed as negative results for this hardware.

---

*I work with an AI coding assistant (Claude, Anthropic) on this project — it helped research and verify this finding and drafted this comment; I own the hardware, ran the real tests, and made the actual calls on what's reported here.*
