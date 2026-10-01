# Gemini's hidden "thinking" tokens count against `max_output_tokens`, and `thinkingBudget: 0` genuinely fixes it

*Originally posted as a comment on [googleapis/python-genai#782](https://github.com/googleapis/python-genai/issues/782#issuecomment-5937252172), 2026-10-01.*

We hit the same symptom this thread describes (`MAX_TOKENS` finish reason, truncated/empty-looking output, thinking tokens eating the whole budget) building a voice assistant on `gemini-3.6-flash`, called directly via the REST API rather than this SDK — but since the underlying API behavior should be the same, here's a confirmed-working fix plus real before/after numbers, in case it saves someone the day `@cnpierrepapi` mentioned above.

Our case: short, complete spoken replies with a tight `maxOutputTokens` (200, since a voice reply needs to be brief anyway) and latency actually matters (it's a live conversation, not a batch job).

| Config | Result | Thought tokens | Output tokens | Time | Text |
|---|---|---|---|---|---|
| 200 tokens, default thinking | `MAX_TOKENS` | 191 | 5 | 1.55–1.88s | *"It is twelve twenty-"* (cut off mid-word) |
| **200 tokens, `thinkingBudget: 0`** | **`STOP`** | **0** | 9–49 | **0.59–0.79s** | full, complete answer |
| 200 tokens, `thinkingLevel: "low"` | `MAX_TOKENS` (on the longer prompt) | 55–190 | 6–9 | 0.92–1.65s | still truncated on one of two prompts |
| 200 tokens, `thinkingLevel: "minimal"` | `STOP` | 0 | 10–49 | 0.56–1.24s | full, complete answer |

For us, `thinkingBudget: 0` wasn't "ignored" the way a few people above describe — it genuinely zeroed the thought-token spend and fixed the truncation outright, on this model. One nuance worth flagging for anyone reaching for `thinkingLevel` instead: `"low"` is not the same as off — it still burned meaningful thought tokens and still hit `MAX_TOKENS` on the longer of our two test prompts. Only `thinkingBudget: 0` or `thinkingLevel: "minimal"` fully eliminated it for us.

Practical takeaway for anyone in a similar spot (tight, fixed output budget, don't need thinking for simple requests): if raising `max_output_tokens` to absorb the thinking spend isn't attractive (ours is latency-sensitive — every token of thinking is time a user is waiting mid-conversation), setting `thinkingBudget: 0` cut our time-to-first-token roughly 2.5–3x on top of fixing the truncation. And whatever the fix, always log `finishReason` — a `MAX_TOKENS` truncation can look like a plausible, if short, complete answer if you're not checking for it (point 3 in the original report, which cost us some debugging time too before we started logging it).

---

*I work with an AI coding assistant (Claude, Anthropic) on this project — it helped research and verify this finding and drafted this comment; I own the project, ran the real tests, and made the actual calls on what's reported here.*
