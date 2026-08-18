# Copilot Studio — Multi‑Country Compliance Agent — Field Report

> A hands‑on field report on building a **multi‑country document‑compliance agent** in **full‑AI mode**
> on Copilot Studio: the architecture, the platform limits we hit, a GPT‑5.5 vs GPT‑5.6 comparison,
> and an honest map of **where you would remove AI in production**.

This is **not** a "look how easy it is" demo. It is a candid account of what happens when you deliberately
push a conceptually simple task — *"check a long contract against a pack of rules, once internally and once
against a target country's context"* — as far as it will go using AI for almost every step. The result is a
**defensive architecture** born of reliability constraints, and a set of lessons that are hard to find
written down honestly anywhere else.

---

## Snapshot — what this report is anchored to

Everything here reflects the platform **as it was at the time of the run**. Copilot Studio moves fast;
treat this as a dated field observation, not a permanent statement of capability.

| Item | Value |
| --- | --- |
| **Report date** | **August 2026** |
| **Platform** | Copilot Studio (agent‑based orchestration) |
| **Orchestration** | Agent flow (Power Automate) invoking agents via Agent Node / InvokeAgent |
| **Model — documented run** | **GPT‑5.5 Chat** (baseline) |
| **Model — compared** | **GPT‑5.6** (see [model selection](docs/06b-model-selection.md)) |
| **Agent capabilities used** | Skills (Python + Markdown), MCP tools, web search, Memory *(preview, disabled)* |
| **Tools** | Dataverse MCP · SharePoint MCP · Excel Online "Add a row" · SharePoint "Create file" |

---

## TL;DR — the findings

- **Full‑AI at scale is fragile.** Long, multi‑step instructions produced partial execution ("agent
  laziness"): the agent would stop early or skip work. The fix was to **decompose into small batches of
  five rules** and hand each responsibility to a **specialized single‑purpose agent**.
- **The flow must own control.** Pagination, sequencing and loop termination live in the **agent flow**,
  not in the agents. Agents that own their own long loops drift.
- **Let the model judge, let scripts build the output.** Free‑form model JSON dropped required fields,
  mixed languages, and leaked citation markers. Moving envelope assembly + validation into **deterministic
  scripts** removed a whole class of failures.
- **Transient failures must not become beliefs.** The same document could read fine, then fail on one
  batch, then read fine again. A robust design **recovers** from a transient read failure instead of
  concluding "the document is unreadable".
- **Model choice is a real trade‑off.** GPT‑5.6 delivered materially better legal‑text retrieval,
  examples, analysis and remediation — at roughly **2× the execution time and higher cost** than GPT‑5.5.
- **You would not ship this as‑is.** In production, several AI steps (Dataverse reads, Excel writes,
  deterministic transforms) should be replaced by native Power Platform actions. The full‑AI version is a
  **limits‑probing exercise**, not the recommended production design.

---

## Read in this order

| # | Chapter | What you get |
| --- | --- | --- |
| 01 | [The scenario](docs/01-scenario.md) | The business problem, neutrally framed |
| 02 | [Full‑AI architecture](docs/02-architecture.md) | The flow + five specialized agents (diagram) |
| 03 | [Execution walkthrough](docs/03-execution-walkthrough.md) | The loop, step by step (sequence diagram) |
| 04 | [Data contracts](docs/04-data-contracts.md) | Page contract, results envelope, Excel mapping |
| 05 | [Deterministic output pattern](docs/05-deterministic-output-pattern.md) | "Model judges, scripts build" |
| 06 | [Harness limits — field notes](docs/06-harness-limits-field-notes.md) | **The flagship chapter**: constraint → decision |
| 06b | [Model selection: 5.5 vs 5.6](docs/06b-model-selection.md) | Quality vs cost & latency |
| 07 | [Where to remove AI in production](docs/07-where-to-remove-ai-in-production.md) | The de‑AI map |
| 08 | [Reproduce it](docs/08-reproduce.md) | Table structure + wiring the analysts |
| 09 | [Safeguards](docs/09-safeguards.md) | The "never do this" list |

The reusable table structure lives in [`data-model/`](data-model/).

> **On the analyst skills:** the two analyst skill packages built for this engagement are
> **intentionally not shipped** in this repository. They are derived from a real client's rule
> taxonomy, so this report focuses on the engineering and the platform limits rather than
> redistributing client‑derived artifacts. The chapters describe their structure, contracts, and
> per‑call chain in enough detail to rebuild your own.

---

## What this repository is — and is not

- ✅ A **reference use case** and honest field report for a real class of problem.
- ✅ **Client‑neutral**: all data, documents, tenant and organization details have been removed; the
  running example uses a fictional scenario.
- ❌ Not a production blueprint. See [chapter 07](docs/07-where-to-remove-ai-in-production.md).
- ❌ Not affiliated with or endorsed by Microsoft. Product names belong to their owners.

## Related work

This report complements the official community skill gallery published by the Microsoft Power CAT team
(<https://microsoft.github.io/cat-agent-skills/>) and my own methodology repo,
**Copilot‑Studio‑Skill‑Builder**.

## License

Released under the [MIT License](LICENSE).
