# Data model (structure only)

> **No data is published in this repository** — only the *shape* of the tables, so you can build an
> equivalent rule store. Logical names below use a neutral placeholder prefix; use your own publisher
> prefix in your environment.

## Table 1 — Compliance rules *(the active read source)*

The extractor reads this table in COUNT and PAGE modes.

| Field (logical) | Type | Purpose |
| --- | --- | --- |
| `complianceRuleId` | Unique id | Primary key / stable row id |
| `generalArea` | Text | High‑level area (used as a sort key) |
| `guidelineCode` | Text | Rule code, e.g. `AAA.01.L2` (used as a sort key) |
| `guidelineText` | Text (long) | The rule statement to check |
| `ruleIdentifier` | Text | Stable business identifier |
| `rulePack` | Text | Rule‑pack name (filter key) |
| `specificIssue` | Text | Short issue label |
| `referenceWording` | Text (long) | Optional remediation hint (analyst input only) |
| `statecode` / `statuscode` | Choice | Active/inactive state (only active rules are counted/paged) |

**Stable sort** for paging: `generalArea` ASC, then `guidelineCode` ASC, then `complianceRuleId` ASC — so
the same rules always fall in the same page across runs.

## Table 2 — Audit records *(part of the model; not written by the documented Excel flow)*

| Field (logical) | Type | Purpose |
| --- | --- | --- |
| `auditRecordId` | Unique id | Primary key |
| `campaign` / `auditName` | Text | Run identifier |
| `rulePack` | Text | Pack analyzed |
| `country` | Text | Target country |
| `conformantCount` / `nonConformantCount` / `notFoundCount` | Number | Roll‑up counters |
| `status` | Choice | Run status |
| `startDate` / `endDate` | DateTime | Run timing |
| `userEmail` | Text | Who launched the run |

> In the documented full‑AI design, results are accumulated in **Excel**, not written back to Dataverse.
> This table is included for completeness; a production build might use it for progress tracking.

## Table 3 — Per‑rule results *(part of the model; not written by the documented Excel flow)*

| Field (logical) | Type | Purpose |
| --- | --- | --- |
| `testRuleId` | Unique id | Primary key |
| `→ auditRecord` | Lookup | Parent run |
| `→ complianceRule` | Lookup | The rule evaluated |
| `conformanceStatus` | Choice | Conformant / Non‑conformant / Not found / To verify |
| `signingConformanceStatus` | Choice | Secondary conformance dimension |
| `confidenceScore` | Number/Choice | Model confidence |
| `conformanceExplanation` | Text (long) | Reasoning |
| `sourceText` | Text (long) | Contract/country evidence excerpt |
| `suggestedWording` | Text (long) | Remediation suggestion |
| `testDate` | DateTime | When evaluated |

## Reminder

The canonical status vocabulary and the 19‑column Excel mapping live in
[`../docs/04-data-contracts.md`](../docs/04-data-contracts.md).
