# Engineering case study

[← Back to BillEase](../README.md)

## Design problem

Multiple phones can record related business events while disconnected. The server must establish one authoritative financial history without losing the original intention behind a saved operation. A receipt, delivery and return cannot safely be treated as arbitrary edits to an invoice row.

## Data flow and invariants

1. A user records work in the mobile UI. The local record and upload operation are persisted together where applicable.
2. The outbox retains a stable operation identifier, original payload and dependencies. Repeating an uncertain action must reuse that intention.
3. The API authenticates the session/device and checks membership and business role.
4. Services perform the business decision and associated financial writes transactionally. The saved operation result supports idempotent retries.
5. Incremental pull synchronization applies role-appropriate record changes and cursor progress atomically.

### Catalogue versus historical truth

The current catalogue is mutable. An issued bill's snapshot is historical evidence. Updating a phone's catalogue price changes the current item/unit rows, while existing bills retain their original values. Treating both as the same data would make old documents depend on today's settings.

### Amounts and quantities

Money uses integer paise, quantities use thousandths where required, and unit factors use exact decimal representations. Return/cancellation valuation refers back to the original bill. Cumulative reversals must not exceed original eligible quantities or allocated tax.

### Ledger behavior

Bills, confirmed receipts, repayments, advance applications and corrections contribute explainable ledger effects. A return reduces debt first. Excess is either entirely refunded or entirely held as advance; the product deliberately excludes splitting that remainder.

### Access boundaries

- Administrator: business/account/membership/access setup.
- Owner: business financial data, decisions, settings and reconciliation.
- Staff: permitted operational actions and restricted synchronized projections.

API enforcement is required even if the UI hides unavailable controls. Device/session revocation affects online access and queued notification recipients. Offline access uses a bounded authorization lease.

## A concrete debugging example

During development, an owner bill stayed local with a misleading review status. The business had no initialized document-number series, so the server rejected issuance with a setup error. The client had classified every HTTP 409 as a review conflict and then stopped retrying that saved operation.

The fix distinguished explicit financial-conflict outcomes from setup/validation failures, displayed the saved server reason, and allowed scoped owner retry of the persisted original bill. It also corrected owner-entered digital receipts so they confirm immediately while staff digital claims still require owner verification. Invoice-series initialization remains explicit and cannot reset issued numbering.

This illustrates why HTTP status alone is insufficient to decide whether a saved financial operation needs review, correction or retry.

## Validation evidence

For that 1 November 2026 change, the isolated backend runner passed 62 tests across 35 files. Thirteen focused Flutter tests covered retry integrity, role/scope checks and review behavior. Targeted Dart analysis passed and the development APK was built and its signature checked. Historical results describe the tested change, not all future revisions.

## Tradeoffs and limits

| Choice | Benefit | Cost / limit |
| --- | --- | --- |
| Local cache plus outbox | Work survives interrupted connectivity | Requires dependency ordering, conflict handling and device access rules |
| Explicit financial conflicts | Preserves an understandable history | Some ambiguous work requires an owner decision |
| Frozen issuance snapshots | Stable historical PDFs | Snapshot/version management adds storage and validation work |
| Integer and exact decimal representations | Predictable financial allocation | Parsing, rounding and display must be handled deliberately |
| Free development hosting | Low-cost development access | Sleeping services can delay requests and in-process notification jobs |

Physical-phone acceptance, production credentials/signing, independent hosted backups/restore rehearsals and actual push delivery remain rollout requirements. Email recovery is inactive until provider/sender verification. Unsupported tax cases require explicit future scope rather than an implied compliance claim.

## Reading this repository

The public repository is a case study, not a runnable distribution. Screenshots use the existing fictional Flutter UI preview. The full implementation, environment configuration and operational records remain outside this public repository.
