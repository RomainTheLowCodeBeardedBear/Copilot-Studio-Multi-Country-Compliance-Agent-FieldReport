# 03 · Execution walkthrough

> One full pass of the loop, step by step. The flow drives everything; each agent is invoked, returns, and
> is done.

## Before the loop

1. **Create the workbook** — the flow invokes **ExcelInit**, which copies the SharePoint template and
   returns the **workbook URL** (a single string, not a JSON object). Nothing else.
2. **Count the rules** — **Extractor** runs in **COUNT** mode and returns a single integer: the number of
   active rules for the pack.
3. **Initialise pagination** — the flow stores the count, sets `start = 1`, `end = 5`.

## Inside the loop (one iteration = 5 rules)

```mermaid
sequenceDiagram
    autonumber
    participant F as Agent Flow<br/>(owns pagination)
    participant X as Extractor
    participant AI as Internal Analyst
    participant AC as Country Analyst
    participant XL as InsertXls
    participant WB as Excel Workbook

    F->>X: PAGE(pack, country, from, to)
    X-->>F: stable 5‑rule JSON (fixed sort)
    F->>AI: contract URL + audit name + page JSON
    AI-->>F: Internal envelope (built & validated by scripts)
    F->>XL: Target = Internal + envelope
    XL->>WB: Add a row × 5 → InternalResults
    XL-->>F: success { inserted, target, table }
    F->>AC: contract URL + audit name + same page JSON
    AC-->>F: Country envelope (targeted research if relevant)
    F->>XL: Target = Country + envelope
    XL->>WB: Add a row × 5 → CountryResults
    XL-->>F: success { inserted, target, table }
    F->>F: start += 5 ; end += 5
    Note over F: loop until start ≥ total rule count
```

*(Diagram source: [`assets/sequence-loop.mmd`](../assets/sequence-loop.mmd).)*

## Step notes

- **Stable paging (Extractor · PAGE).** The extractor asks the rule store only for the requested slice —
  never the whole pack — and applies a **fixed multi‑field sort** so the same rules always land in the
  same page across runs. It performs no analysis.
- **Internal analysis.** The analyst reads the **entire** contract and appendices, evaluates **only** the
  five rules in the page, and returns one verdict per rule. Reference/remediation wording is treated as a
  *suggestion source*, never as evidence.
- **Internal insertion before Country.** The Country pass runs **only after** the Internal insertion
  reports success. This strict ordering is what keeps pagination honest — the flow advances only when both
  insertions for the batch have succeeded.
- **Conditional country research (Country analysis).** The Country analyst is **contract‑first**: if a
  rule depends on a contractual element that is absent, it returns "Not found" and does **no** search.
  Country research happens only when local law could materially change the conclusion.
- **Pagination increment.** After the Country insertion succeeds, the flow increments both pagination
  variables by five and re‑enters the loop.

## Loop control

| Variable | Meaning |
| --- | --- |
| total | number of active rules (from Extractor COUNT) |
| start | current page start (initial 1) |
| end | current page end (initial 5) |
| stop condition | `start ≥ total` |
| safety caps | max 60 iterations · loop timeout 5 hours |

## Observed timing (indicative)

One measured loop took ≈ **4 min 7 s** (Extractor ≈ 31 s, Internal analysis ≈ 1 min 17 s, Internal
insertion ≈ 46 s, Country analysis ≈ 46 s, Country insertion ≈ 46 s, pagination ≈ 0.07 s). At that rate a
~40‑iteration run projects to **≈ 3 hours** before variation. The two Excel insertions alone accounted for
roughly **92 seconds per loop** — a direct motivation for
[replacing them with native actions](07-where-to-remove-ai-in-production.md).
