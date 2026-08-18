# 01 · The scenario

> Neutral, client‑free framing of the business problem the agent was built to solve.

## The business problem

An organization must check a long **services/consulting contract** (typically a tender response, tens of
pages with appendices) against a **compliance rule pack** — on the order of **~200 rules**. Each rule is a
short, checkable statement ("a bid guarantee is required", "liability is capped at X", "the contract
references the applicable procurement law", …).

The check is performed **twice per rule**:

1. **Internal comparison** — is the contract compliant with the organization's own internal guideline for
   this rule?
2. **Country comparison** — considering the **law and business context of a target country**, does the
   local legal framework change the conclusion? This pass only does research when it is actually relevant
   (see [chapter 04](04-data-contracts.md) and [06](06-harness-limits-field-notes.md)).

The output is a **human‑readable workbook**: one row per rule per comparison, with status, evidence,
analysis, confidence, and suggested remediation wording.

## Why it is a good stress test for the platform

- **Volume**: ~200 rules × 2 comparisons = ~400 reasoned verdicts per run, each requiring the model to
  read a long document and produce a structured result.
- **Long‑running**: a full run is measured in **hours**, not seconds — exactly where reliability and drift
  problems surface.
- **Structured hand‑off**: results flow Agent → JSON → Agent → Python → storage. Any language or format
  drift becomes a contract failure downstream.
- **Judgement + retrieval**: the country pass mixes *contractual reasoning* with *legal‑source retrieval*,
  stressing both the model's judgement and its web/knowledge behavior.

## Design intent: deliberately "full‑AI"

The proof of concept intentionally used **AI for almost every step** — reading the rule store, extracting
document text, judging each rule, and even writing rows into the report. This was a **choice to probe the
limits**, not a recommendation. The production‑oriented counterpart is described in
[chapter 07](07-where-to-remove-ai-in-production.md).

## Running example (fictional)

Throughout this report the example uses a **fictional** rule pack and target country. No real client,
tender, country, tenant, or dataset is referenced. Numbers such as "~200 rules" are indicative of the
documented run and are kept intentionally round.
