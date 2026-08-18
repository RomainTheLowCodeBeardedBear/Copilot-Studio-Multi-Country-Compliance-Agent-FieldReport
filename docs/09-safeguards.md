# 09 · Safeguards

> A distilled "never do this" list, learned the hard way. Most are direct consequences of
> [chapter 06](06-harness-limits-field-notes.md). Keep them visible while you build.

## Reliability & evidence

- **Never** claim a technical root cause without a trace or a reproducing test.
- **Always** separate what is *verified*, *observed*, a *hypothesis*, or *unresolved*.
- **Never** turn a transient tool/read failure into a permanent belief ("the document is unreadable").
- **Never** describe a component as active merely because it exists in the solution package.

## Output integrity

- **Never** let the model reconstruct the final envelope after validation — emit the validated output
  verbatim.
- **Never** allow native citation markers (`[1]`, `[2]`, `[source]`) inside analyst JSON.
- **Never** silently rewrite or "correct" the source document URL.
- **Never** create an output file from a truncated/preview payload.

## Analytical discipline (country pass)

- **Never** perform country research for a rule that only describes an attribute of an **absent**
  mechanism.
- **Never** treat negative wording alone as a reason to trigger country research.
- **Never** default to "To verify" just because an official source was not found.
- **Never** accept a source as relevant merely because it is official — the cited excerpt must directly
  address the obligation under review.

## Human‑facing output

- **Never** expose technical notes, extra sheets, columns, or headers in the human‑facing workbook — add
  result **rows** only.
- **Never** regenerate the whole report by rereading all rules and results after the loop.

## Scale & control

- **Never** hand the loop, pagination, or termination to an agent — the **flow** owns them.
- **Never** change the batch size or insertion design without evidence that the batch size is the cause of
  whatever you are trying to fix.
- Before replaying a failed batch, **inspect the result tables** for rows already inserted from that batch
  (idempotency is not guaranteed).
