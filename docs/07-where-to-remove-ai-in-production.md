# 07 · Where to remove AI in production

> The full‑AI design in this report is a **limits‑probing exercise**. A production implementation should
> pull AI out of every step that carries **no judgement**. This chapter is the honest counterpart to the
> rest of the report.

## The principle

> **Use AI only where there is judgement. Everywhere else, use deterministic Power Platform components.**

Reading a rule store, counting rows, copying a file, and writing a spreadsheet row are **mechanical**
operations. Running a language model on them adds latency, cost, and a new failure surface — with no
quality upside.

## The de‑AI map

| Step | Full‑AI version (this report) | Production‑oriented replacement | Why |
| --- | --- | --- | --- |
| Count active rules | Extractor agent · COUNT mode | **Dataverse list/count action** (native) | Pure query; no judgement |
| Page 5 rules | Extractor agent · PAGE mode | **Dataverse list rows** with filter, sort, top/skip | Deterministic paging belongs in the flow |
| Create the workbook | ExcelInit agent | **SharePoint "Copy file" / "Create file"** action | File copy is metadata work |
| Insert result rows | InsertXls agent (Add a row) | **Excel "Add a row" action driven by the flow** (or Office Script batch) | ≈ 92 s/loop of pure I/O; no reasoning |
| Metadata assembly | Skill scripts inside the agent | **Flow compose / Power Fx**, or keep as scripts | Deterministic either way |
| **Internal judgement** | Internal Analyst | **Keep AI** | Genuine reasoning over the contract |
| **Country judgement + research** | Country Analyst | **Keep AI** | Reasoning + selective legal retrieval |

Only the **two analyst agents** carry irreducible judgement. Everything else is a candidate for
deterministic replacement.

## What a production shape could look like

- The **agent flow** keeps ownership of pagination and sequencing (it already does).
- **Dataverse** count/list actions replace the Extractor entirely.
- **SharePoint** actions replace ExcelInit.
- The flow calls the **Internal** and **Country** analysts (still AI) per page.
- **Excel/Office Script** actions — invoked by the flow — replace InsertXls and can insert a whole batch at
  once, reclaiming most of the ≈ 92 s/loop insertion overhead.

## The honest framing

The full‑AI cost overhead measured here is therefore **not** presented as a Copilot Studio defect. It is
the price of the *exercise* — pushing the platform to its limits to learn where it bends. Knowing exactly
which steps to de‑AI is itself one of the deliverables of that exercise.

## Rule of thumb

> If you can describe a step as a query, a copy, or a row write — take the model out of it.
> If a step requires reading, weighing, and deciding — keep the model, and protect its output with the
> [deterministic pattern](05-deterministic-output-pattern.md).
