# ESPHome voice-assistant protocol gotchas from building a standalone, non-HA client

*Originally posted as a comment on [FutureProofHomes/Satellite1-ESPHome#540](https://github.com/FutureProofHomes/Satellite1-ESPHome/issues/540#issuecomment-5937969925), 2026-10-01.*

We hit a related class of problem building our own standalone (non-HA) client against this same API — a different specific bug than what's reported here, but the same general area (voice-assistant session state not recovering cleanly from a subscriber-side disconnect), so sharing in case it's useful corroborating context.

**Our case**: a second `subscribe_voice_assistant` call — even a genuine reconnect from the SAME logical client after a crash/restart, if the device still thinks the old connection is alive — gets refused *silently*. The device logs `Multiple API Clients attempting to connect to Voice Assistant`, naming the current holder and the new claimant, but the refused client is never told anything over the API itself — no error response, nothing. Wake detection just doesn't work on the new connection, with no obvious symptom pointing at "someone else still holds the session." We work around it by identifying our client at the API level (`client_info=...` on `APIClient()`), watching the device's own log stream for that specific ERROR line naming our client as the refused party, and retrying the subscribe every ~10s until the old session actually clears (eventually a dead TCP connection gets noticed and released).

Could be the same underlying root cause as what's reported here (session cleanup not happening properly on an abrupt disconnect), or a genuinely different one — I don't have visibility into the device's internal state machine, just the two symptoms (ours: silent refusal + a log-only error, recoverable by retrying; the reported one: permanent deafness with no error logged at all, needing a reboot). Flagging it as a data point that this general area — session survival across an abrupt subscriber disconnect — seems to be a real rough edge for anyone building a non-HA client, not an isolated one-off.

A few other protocol-level things that cost us real debugging time building a similar standalone pipeline, in case they're useful to anyone else in this position:
- After `STT_VAD_END`, the device sends nothing further and just waits — there's no "end of audio" signal; the *server* has to decide when the turn is over and send `RUN_END` itself, or the light stays stuck.
- Voice-assistant events (wake, run-end, errors) are only honored on the connection that's actually subscribed — consistent with the single-session behavior above.
- `TTS_START` needs real `text`, or the device logs a warning ("No text in TTS_START event").
- The announcement/speaker path only plays 16kHz stereo 16-bit WAV — nothing else.
- Running too many simultaneous API connections (we saw it happen around 4) can make the device drop one.

---

*I work with an AI coding assistant (Claude, Anthropic) on this project — it helped research and verify these findings and drafted this comment; I own the hardware, ran the real tests, and made the actual calls on what's reported here.*
