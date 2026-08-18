# 06 · Harness limits — field notes

> **The flagship chapter.** Almost every design decision in this project is a scar left by a platform
> limit. Here they are, honestly, each mapped to the workaround it forced. Nothing here is presented as a
> defect verdict — it is a dated field observation of what it took to make a full‑AI pipeline reliable.

## Constraint → decision, at a glance

| Observed limit | What it looked like | Design decision it forced |
| --- | --- | --- |
| **"Agent laziness"** on long tasks | Long multi‑step instructions → the agent stopped early, refused to continue, or skipped actions | **Batch into pages of 5**; the **flow owns the loop**, not the agent |
| **Intermittent read failures** | The *same* document read fine, failed on one batch ("cannot be read"), then read fine again | **Controlled retry + current‑batch fallback**; never conclude the file is permanently unreadable |
| **Output / language drift** | FR/EN mixed inside a machine JSON contract; localized status & confidence values | **Normalize + validate** in scripts instead of trusting natural‑language consistency |
| **Structured‑output fragility** | Missing fields, intermediate object instead of final, `[1]` citation leakage, a mutated URL character | **Deterministic build‑and‑emit** ([chapter 05](05-deterministic-output-pattern.md)) |
| **Prompt sensitivity** | Over‑strict wording pushed the agent to default to "To verify" even when unhelpful | Encode the **decision boundary** explicitly (contract‑first, then targeted research) |
| **Per‑call side effects** | Occasional "ConversationBusy" and cross‑call surprises | **Memory off**, strictly **sequential** flow, one clean call per step |

## 6.1 "Agent laziness" — the origin of the whole architecture

The initial goal was *not* a distributed architecture. It emerged as a **reliability workaround**. Given a
long, multi‑task instruction ("process all ~200 rules and all their actions"), the agent would not
reliably carry the whole workload in one invocation. **Batches of five became the stable unit of work**,
and responsibilities were split across specialized agents. This is the single most important thing to
understand about the design: *the shape is defensive, not aspirational.*

## 6.2 The same document, same task — intermittent read failure

The clearest evidence: in a GPT‑5.5 run, isolated five‑rule batches explicitly reported that the contract
"could not be read", even though the **same** file was read successfully in the batches before and after.
A repeated rule could be `Conformant / High` on one occurrence and `To verify / Low` ("document could not
be read completely") on a later occurrence of the *same* rule against the *same* document.

**Lesson.** A robust design must be **failure‑aware**: a one‑off read incident should trigger **recovery**,
not become a persistent belief that the document is unreadable. The Country package therefore performs one
controlled retry, and if that also fails, emits a complete, honest current‑batch envelope ("To verify",
low confidence, no country research) **without** asking a human to restart the loop.

## 6.3 Language and output‑contract drift

Result files showed English and localized values mixed inside a contract meant to be machine‑readable —
status values, confidence labels, and narrative all drifting between languages across adjacent rows. For a
human answer this is cosmetic; for an **Agent → JSON → Agent → Python → storage** chain it is a contract
break. The response was **normalization rules + validation**, not a hope that the model stays consistent.

## 6.4 Structured‑output fragility

Several distinct failures pushed us toward deterministic assembly: an intermediate verdict JSON returned
instead of the final envelope; required fields missing; native web citation markers contaminating JSON;
and, in one output, a single mutated character inside the SharePoint source URL. In another case a
visually complete five‑result JSON coexisted with a downstream report of only one persisted result — and
the exact responsible layer could **not** be proven from the available evidence. The fix was to stop
asking the model to reconstruct the final envelope and to require a deterministic validation‑and‑assembly
chain instead (see [chapter 05](05-deterministic-output-pattern.md)).

## 6.5 Prompt sensitivity in the country pass

The country analyst was strongly sensitive to instruction wording. When the prompt demanded an overly
strict standard of official legal evidence, the agent tended to fall back to "To verify" even when that
was unhelpful. The stable approach: **first** decide whether the contractual element exists at all; **only
then** perform country research, preferring official sources while allowing clearly‑qualified secondary
sources. The lesson for anyone tuning agents (or teaching them from corrections): preserve the underlying
**decision boundary**, don't memorize a blunt rule like "use more sources" or "avoid To verify".

## 6.6 What remains unresolved

Honesty requires listing what the evidence did **not** establish:

- the exact cause of "ConversationBusy" occurrences in an otherwise sequential flow;
- why two visually valid JSON envelopes sometimes produced different downstream outcomes;
- the cause of within‑batch row reordering in the workbook;
- idempotency behavior after an ambiguous/partial insertion (replaying a batch without checking existing
  rows could create duplicates).

These are marked **unresolved** on purpose. No plausible‑sounding explanation is promoted to a root cause
without a trace or a reproducing test — which is itself a working principle worth keeping.

## Takeaway

None of these limits means "don't use Copilot Studio". They mean: for a **long‑running, structured,
full‑AI** workload, plan for a **defensive architecture** from day one — small units of work, a flow that
owns control, deterministic output, and failure‑aware recovery. And revisit the assumptions regularly,
because the platform changes month to month.
