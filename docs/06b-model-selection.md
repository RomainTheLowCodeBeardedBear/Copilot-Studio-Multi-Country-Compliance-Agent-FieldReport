# 06b · Model selection — GPT‑5.5 vs GPT‑5.6

> A practitioner's comparison, not a benchmark. Framed as **observed**, because the two runs did not use
> identical prompts/skills and the GPT‑5.6 run was intentionally time‑capped. Still, the direction of the
> trade‑off was clear and consistent.

## The trade‑off in one line

**GPT‑5.6 buys materially better analytical quality at roughly 2× the time and higher cost.**
**GPT‑5.5 is faster and cheaper, with weaker legal‑retrieval quality.**

| Dimension | GPT‑5.5 Chat | GPT‑5.6 |
| --- | --- | --- |
| Legal‑text retrieval (country pass) | Adequate, more misses | **Noticeably better** — finds the relevant legal texts more reliably |
| Examples & remediation wording | Serviceable | **Richer, more usable** suggestions |
| Depth of analysis | Good enough | **Deeper, better‑justified** reasoning |
| Explicit "cannot read document" errors | Several isolated batches | **None observed** (but see caveat) |
| Execution time | Baseline | **≈ 2× longer** |
| Cost | Baseline | **Higher** |

## Honest caveats

- **Not a clean A/B.** Prompts and skills evolved between the two runs, so this is a *model‑selection
  observation*, not a controlled model‑only benchmark.
- **The 5.6 run was time‑capped on purpose.** Its lower processed‑rule count is *expected* and must **not**
  be read as a "failure to finish".
- **"No explicit read errors" ≠ "no failures".** The GPT‑5.6 output showed a **different failure class** —
  a few cells with placeholder/abbreviated analysis (e.g. text ending in a literal "…") while still
  reporting high confidence. Different model, different failure signature.
- Conformance **distributions differed** between models; because both prompts/skills and model changed,
  this is useful for investigation, not for causal attribution.

## How to choose (practical guidance)

- **Favor GPT‑5.6** when the deliverable's value is dominated by **analytical/legal quality** — the country
  pass, remediation wording, anything a human will act on directly — and the run is allowed to take hours.
- **Favor GPT‑5.5** when you need **throughput and cost control**, or for the **narrow, mechanical** agents
  (Extractor, InsertXls) where there is no judgement to improve.
- **Mix models per agent.** You do not have to pick one model for the whole pipeline. A pragmatic setup is
  **5.6 for the analysts, 5.5 for the mechanical agents** — spend reasoning budget only where judgement
  actually happens.

## Link to production design

The cost/latency side of this trade‑off is a direct input to
[chapter 07](07-where-to-remove-ai-in-production.md): if a step carries **no judgement**, running *any*
model on it is wasted time and money — replace it with a deterministic Power Platform action instead.
