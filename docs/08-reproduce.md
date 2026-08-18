# 08 · Reproduce it

> Enough structure to rebuild the pattern in your own environment, without any client data. You bring your
> own rules and documents.

## What you need

- A Copilot Studio environment with **agent flows**, **skills**, and **MCP tools** available.
- A rule store (a Dataverse table — see [`../data-model/tables.md`](../data-model/tables.md)).
- A document library (SharePoint) with the contract(s) to analyze and an Excel **template** containing two
  tables: `InternalResults` and `CountryResults`, each with the 19 columns from
  [chapter 04](04-data-contracts.md).

## Wiring, in order

1. **Create the five agents** with the responsibilities in [chapter 02](02-architecture.md). Keep each
   agent's tool set minimal and its prompt boundary explicit.
2. **Build the two analyst skills** and attach them to the Internal and Country analysts. The packages
   themselves are not shipped here (client‑derived), but their role, inputs, per‑call chain, and output
   contract are documented in [chapter 04](04-data-contracts.md) and
   [chapter 05](05-deterministic-output-pattern.md).
3. **Build the agent flow** that owns pagination and calls the agents in the sequence of
   [chapter 03](03-execution-walkthrough.md).
4. **Disable Memory** on all agents; keep the flow strictly sequential.
5. **Validate the data contracts** ([chapter 04](04-data-contracts.md)) end to end on a **tiny** pack
   (5–10 rules) before attempting a full run.

## Trigger inputs

The documented flow used six required text inputs:

| Input | Purpose |
| --- | --- |
| Contract URL | SharePoint URL of the document to analyze |
| Pack | Exact rule‑pack name |
| Country | Target country for the country comparison |
| Audit name | Run identifier echoed into analyst output |
| Report‑folder URL | Storage folder (was retained by the trigger but unused in the documented flow) |
| Template URL | SharePoint URL of the Excel template |

> The report‑folder input was **present but unused** in the documented flow — a good reminder to prune
> trigger inputs that no action actually consumes.

## Start small

Do **not** start with a 200‑rule pack. Run 1–2 loop iterations first, confirm both result tables fill
correctly with all 19 columns, then scale up. Watch the [safeguards](09-safeguards.md) while you do.

## Production note

Before running this at real volume, read [chapter 07](07-where-to-remove-ai-in-production.md): several of
these agents should be replaced by native actions in a production build.
