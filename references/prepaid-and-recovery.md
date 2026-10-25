# Prepaid credit, automatic top-ups, and recovery

These are application design patterns. Adapt them to the product's accepted
terms and storage model; they do not establish Dodo's account eligibility,
idempotency guarantees, default limits, or the state of any deployment. Use the
[lifecycle reference](lifecycle.md) for checkout, refund, and event handling.

## Separate authority from credit

A purchase grants the service credit actually sold. Permission to spend existing
credit and permission to charge a saved payment method are separate authorities.
An on-demand mandate is not itself a successful purchase or a paid-period service
grant. Keep recurring service subscriptions distinct from replenishment mandates.

Bind enrollment to immutable business, brand, Test/Live mode, customer, source
purchase, product, currency, credit principal, gross charge ceiling, limits, and
consent version. An explicit setup action must match the displayed terms and the
confirmed provider mandate. Refreshing an offer or toggling an initial preference
must not enroll the customer. A later purchase must not silently raise old limits.

Choose the product's trigger, count cap, monetary cap, and accounting period
explicitly. Reserve count and gross cash capacity before dispatch. Successful
and unresolved attempts both consume their intended reservations. Retain those
reservations across period boundaries; a late success must not free the original
capacity or erase the obligation by moving into a newer period.

## Gross cash, tax, fees, and service credit

Retain integer money in the actual currency's minor unit, and record provenance.
The gross cash ceiling, included tax, advertised service principal, provider fee,
and payout proceeds are different values. Keep manual and automatic product
price semantics explicit; do not silently convert an existing tax-exclusive SKU
to tax-inclusive pricing to claim a maximum charge.

A checkout preview describes a quote. Do not treat it as a lock on a later
subscription billing profile, tax split, or charge unless the exact provider
contract establishes that binding. Reconcile each actual payment's total, tax,
currency, and sold terms before crediting. A fixed gross total can contain a
different net principal when the tax treatment changes.

Define the mismatch policy before dispatch: for example, pause further dispatch
and route an already successful incompatible charge to reconciliation/refund.
Absorbing a difference requires an explicit product/accounting decision. Do not
silently grant less credit or increase the customer's charge, and do not replace
a mismatched payment as if no money moved.

Consult [current pricing](https://dodopayments.com/pricing) and the account's
actual settlement records for margin estimates. Check method, currency,
subscription/usage classification, refund/dispute costs, and payout route.
Application usage metering alone does not establish which provider billing
surcharges apply. Preserve the sold credit and refund terms when accounting for
merchant fees. Brand provisioning is distinct from account/product eligibility;
use current policy and relevant account correspondence without generalizing one
merchant's approval to others.

## One-use dispatch and uncertain charges

Persist the intent and reserve capacity before the provider call. Consume a
durable one-use dispatch permission before sending a non-idempotent charge.
Metadata, local API idempotency, and counters cannot prevent a second provider
charge if the application resends the POST. Inspect hidden SDK, job, proxy, and
transport retries.

A timeout after dispatch leaves an unresolved intent even when no payment ID
was received. Keep its reservation and reconcile signed callbacks and provider
GET truth against the original identity. An empty search, a new month, or a
database restore is not permission to issue a replacement charge. Retain
mismatches and unexpected additional successes as financial review cases.

Serialize shared manual/automatic purchase slots where the product needs that
boundary. Finish fallible preflight before taking the slot, or release only a
specific provably undispatched slot. Maintenance must cover the intended tenant
inventory, including pagination; a fixed first page can starve later accounts.
Keep mutable scheduling cursors separate from immutable financial receipts.

## Disablement, refunds, and delayed events

Stop new dispatch locally before requesting provider cancellation. Retain
cancellation progress, retry/reconcile uncertain results, and verify provider
state. A lost cancellation reply does not establish cancellation; an already
dispatched payment can still settle and needs financial reconciliation.

A late event for an old mandate must not overwrite newer consent or revive a
disabled enrollment. Recheck identity after provider I/O and within the serialized
state change. Define how disabling affects in-flight work and explain it in the
product's actual terms.

For unused-credit refunds, suspend relevant dispatch/spending under the product's
policy, drain in-flight usage holds, and calculate unused principal from its
original lot after usage and earlier refunds. An operator financial suspension
must not be undone by an ordinary customer toggle. Verify actual payment/refund,
tax, and currency before the local adjustment.

External refund dispatch, provider observation, local credit adjustment, and
case closure are separate states. An adjustment alone does not prove money was
returned or authorize reopening. Handle partial/remainder refunds and reordered
events independently and idempotently.

## Restore balances without recreating charging authority

Map the actual store boundaries before designing restore. Rolling back an
application database does not roll back provider payments or an external journal.
A missing intent cannot mean that dispatch never happened, a reservation is free,
or a cancelled mandate may be revived. Keep rollback fences outside the store
being restored so the fence does not disappear in the same rollback.

Freeze all relevant writers, including provider dispatch, authority/control
updates, journals, callbacks, and privileged maintenance. Drain or account for
in-flight work. A database lock alone is not a complete financial freeze, and a
consistent snapshot can still omit history lost before capture.

For a frozen recovery baseline, retain a candidate externally, finish the final
identity/freeze checks, and retain a separate confirmation tied to that exact
candidate. An interrupted check must not leave an importable unconfirmed
snapshot. Validate source identity, schema, revision, complete expected row sets,
and artifact hash before mutation.

Reconstruct into a fresh, unserved, closed target. Validate native scalar types
before writes; database coercion can hide a malformed artifact. Keep guards and
rows in the same atomic transaction where supported. After a lost commit reply,
read back the exact complete result before retrying. Row counts alone do not
prove identities or balances, and a conflict is not permission to guess a repair.

Restore only the minimal financial parent/audit records needed by the qualified
schema. Do not import authentication grants or API content to recover balances.
Reconstruction must not grant access, recreate consent, or enable dispatch.

Qualify physical rollback and replay on a disposable target, including interrupted
confirmation, concurrent retries, damaged guards, and missing history. Closed
reconstruction, authority repair, provider/journal/ledger reconciliation, and
reopening are separate outcomes. Keep the target closed until the required
reconciliation and authorized reopening criteria are satisfied.

## Match claims to evidence

Distinguish fixtures, static review, real sandbox purchases and mandates,
real provider callbacks/refunds, a disposable recovery rehearsal, matching
application deployment, and live acceptance. A recovery helper's successful
projection does not prove complete financial history or qualify checkout.

Verify the actual deployed worker/process, database, journal, authority store,
business/brand, mode, catalog, event filters, and public disclosures. Keep existing
callback filters when adding new event types. A tool selection file or accepted
secret write is not proof of running configuration.

If activation verification fails or its outcome is uncertain, inspect actual
state and use the reviewed pause path within existing authorization. Retain
migrated financial data, callbacks, journals, and reconciliation. Do not restore
old financial data as a rollback for a new payment application. Report unavailable
checks and unverified outcomes plainly.
