# MP1 Prompt Lab — Reflection

**Author:** Robin Singh · September 2026

---

## What Was Built

Four prompt strategies were evaluated on a structured extraction task: pull `role`, `company`, and
`years_experience_required` from ten job posting snippets using `gpt-4o-mini`, then score each
extraction against a golden set using both a deterministic field-match metric and a `gpt-4o`
LLM-as-judge (1–4 rubric).

| Strategy   | Accuracy (mean/3) | Judge score | Total cost ($) | Latency p50 (s) |
|------------|:-----------------:|:-----------:|:--------------:|:---------------:|
| zero\_shot | 0.4               | 3.5         | 0.000357       | 4.358           |
| few\_shot  | 0.2               | 3.8         | 0.000507       | 3.994           |
| structured | 0.2               | 3.9         | 0.000398       | 4.131           |
| cot        | 0.2               | 3.9         | 0.000428       | 4.103           |

---

## Key Findings

### 1. Accuracy metric is unreliable — likely a scoring bug

The deterministic accuracy scores are suspiciously low across the board (0.2–0.4), while the LLM
judge consistently awards 3.5–3.9 out of 4. The root cause: the scoring loop passes the entire
`golden` dictionary (keyed by snippet ID) rather than the per-snippet entry to `score_accuracy`.
As a result, `gold.get('role', '')` always returns `''`, making it nearly impossible to earn points
on role or company fields. `zero_shot` scores slightly higher (0.4) because its key name mismatch
(`years_of_experience` vs `years_experience_required`) causes it to fall back to `''`, which
accidentally ties with the broken gold lookup on null-years snippets.

**Fix:** replace `gold = golden` with `gold = golden[r['snippet']['id']]` in the scoring loop.

### 2. LLM judge scores are the more informative signal

Because the judge receives the raw snippet text alongside the gold answer, it can reason about
which golden entry to compare against even when passed the full dict. This makes judge scores
robust to the scoring bug above — and also to the natural variation in how models phrase answers
(e.g., `"5+"` vs `5`). Judge scores reveal a meaningful gap: structured and CoT prompts (3.9)
outperform zero-shot (3.5) by a margin that a real deployment would care about.

### 3. Prompt complexity shows diminishing returns on cost

- **zero\_shot** is cheapest but weakest on quality.
- **few\_shot** is the most expensive (extra example tokens) yet does not beat structured or CoT on
  the judge score.
- **structured** achieves the same judge score as CoT at a 7% lower cost — the explicit JSON schema
  and expert persona do most of the work; few-shot examples add cost without adding accuracy.

### 4. Edge cases expose prompt brittleness

The golden set was designed with tricky snippets — spelled-out numbers ("three years"), experience
ranges ("3–5 years"), and null-experience roles. All prompts received perfect parse rates (every
response decoded as valid JSON), which is a baseline win. But without targeted few-shot examples
covering these edge types, models frequently normalise ranges to a single value or mistype the
output key — losses the accuracy metric can't catch due to the bug, but the judge penalises.

---

## Recommendations

1. **Fix the scoring bug first.** Re-run with `gold = golden[r['snippet']['id']]`; current accuracy
   numbers are not meaningful.

2. **Ship the structured prompt as the production baseline.** It delivers the highest judge score
   tied with CoT, at lower cost than CoT or few-shot, and produces consistent, schema-conformant
   JSON.

3. **Add edge-case few-shot examples only for known failure modes.** Generic 2-example few-shot
   adds cost without quality lift. If post-fix accuracy reveals failures on range or null-experience
   snippets, add one targeted example per failure type rather than a blanket few-shot rewrite.

4. **Use the LLM judge as the primary eval signal, but make it snippet-aware.** Passing the full
   golden dict to the judge is fragile. Each judge call should receive only the matching gold entry.
   This also makes the rubric scores directly comparable across strategies.

5. **Separate parse rate from quality.** 100% parse rate across all strategies means JSON formatting
   is solved — don't let it inflate confidence in extraction quality. Future eval iterations should
   focus on field-level precision, particularly for the ambiguous snippets (j04, j07, j09).
