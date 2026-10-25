# Lifecycle and integration

These are integration design recommendations and cases to qualify in the target
application. Confirm provider-specific fields and guarantees in current official
documentation. Concrete examples below are hypothetical, not account records.

## Provisioning and isolation

Declare products, prices, intervals, currency, tax semantics, and brand explicitly.
Prefer a dry-run-first provisioner that matches stable identities and rejects
duplicate or ambiguous matches. Read back exact remote IDs, brand, price, mode,
enabled state, and webhook filters. Do not delete unrelated products or overwrite
a shared credential tuple to launch a separate surface.

Prepare runtime bindings and callback handling before making checkout payable.
A sale flag should block new purchases while preserving existing callback,
entitlement, download, refund, and reconciliation paths. Confirm the saved
webhook URL, filters, and state before enabling delivery; a successful probe to a
new URL does not prove the provider targets that URL.

Use the webhook-management schema for the specific operation. Endpoint headers
may be managed separately from ordinary endpoint fields, and returned values
may be redacted. An empty successful mutation response can be valid: do not
blindly repeat a mutation because a JSON parser expected a body. After a lost
response, read back authoritative state before deciding whether to retry.

Keep secrets out of argv, committed plans, and logs. Credential-bearing requests
must not follow untrusted redirects; check the target runtime's redirect support
and reject redirects explicitly. Bound request duration and response size.
Verify runtime-specific bindings and the running deployment after secret changes;
an accepted configuration write does not prove an existing deployment uses it.

## Checkout identity and unknown outcomes

Persist the purchase intent before `POST /checkouts`, binding the authenticated
owner, business/brand, environment, offer, quantity, currency, and accepted terms.
Store the provider session identity durably when received. Correlation metadata
can help locate candidates, but cannot replace this ownership boundary.

Before treating a POST as retryable, establish its documented idempotency scope,
retention, lookup, and failure behavior. If these are unverified, a timeout leaves
the attempt uncertain. Search signed events and authoritative provider records
using supported filters and bounded, complete pagination for the chosen scope.
Retrieve candidates and verify their immutable identity. An empty page or scan
is not proof that an attempt never existed or can no longer charge.

Record whether the checkout URL was received and exposed. An ordinary hosted
checkout the customer never received differs from `confirm: true`, saved-method,
and off-session flows; do not transfer a safety argument between them. Session
expiry can prevent a fresh payment start while a started payment remains
unresolved. Reconcile that payment's terminal outcome before releasing its fence.

The [official TypeScript SDK's retry documentation](https://github.com/dodopayments/dodopayments-typescript#retries)
describes automatic retries and `maxRetries`. Inspect the installed SDK and
disable retries on non-idempotent creation calls when duplicate effects are not
prevented by a verified provider contract. Plain fetch still needs application
and queue retry discipline.

## Signed events and durable effects

Use Dodo's maintained SDK or Standard Webhooks verifier over the unchanged raw
request body. Apply timestamp/replay policy deliberately. A valid signature proves
an authentic delivery; it does not prove that a payment belongs to this owner or
offer. Check provider identity and retrieve authoritative financial state.

Deduplicate delivery identity separately from financial effects such as a payment
grant or refund reversal. A claim row is not a completed receipt. Concurrent
duplicates should not acknowledge work that may have died; finish the durable
effect before marking complete. Release or expire failed claims and keep
completion/logging failures from stranding them.

Keep related database effects atomic. For effects outside that transaction, use
verified idempotency or retained progress and reconciliation that converge after
a crash. Target the exact matched purchase when releasing a reservation. Test
old-callback/new-purchase isolation, including a return page still showing an old
refunded order while the new callback is pending.

An unrelated SKU must not enter a default fulfillment handler. When adding a new
billing surface, route before legacy writes and preserve the old handler for old
SKUs. Test the actual signed route and its database effects.

## Mixed checkouts and subscription access

Do not assume payment GET repeats every item shown in checkout. Determine how
the current API represents one-time components, subscriptions, renewals, and
multi-subscription payments. A renewal may lack a one-time cart. Bind each
component through its native payment/session/subscription identities, customer,
product, quantity, price, currency, and paid service period.

Model a mixed offer by its sold terms. For example, refunding a recurring invoice
may stop future service while leaving a separately purchased download intact.
Refunding an initial bundle may affect both components. Do not invent a partial
allocation from a cart sum when the provider's paid invoice has different shape.

Grant paid access from verified successful invoices and their service periods,
not merely subscription creation or activation. Retain already purchased future
periods when the sold cancellation terms require it. Distinguish cancellation at
period end, immediate termination, and expiration using the current event contract.

A paid invoice rejected for terms, price, identity, or period mismatches still
needs a durable financial observation and review. If policy requires stopping
unusable renewals, prove the provider subscription/customer/product identity
before cancelling; arbitrary metadata cannot authorize cancelling another service.
Persist cancellation progress and verify provider readback.

## Refunds and financial proof

Persist a refund case before dispatch. Match the original payment, currency,
items, refundable principal, tax semantics, and prior refunds. Check units for
the actual currency; do not assume every currency has two decimal places. A
tax-inclusive refund amount is not necessarily the same as unused service credit.

Without a verified idempotency contract, a lost refund POST response leaves an
uncertain case. Inspect the original payment's refunds and supported refund-list
filters, retrieve candidates, and match the retained case identity before another
POST. Provider serialization while a refund is pending does not prevent a second
refund after the first succeeds.

Verify the exact refund and payment through authoritative GETs. Require the
matching refund to have succeeded and to agree on payment, currency, and amount.
Use the refund receipt/line items for tax and principal allocation. Treat lagging
or inconsistent provider state as retryable reconciliation, not a fabricated
success or a reason to dispatch a replacement.

A full reversal requires proof for the original paid total, including tax, in the
original presentment currency. Sum only succeeded refunds for that payment and
currency; exclude pending/failed entries and validate integer amounts. Multiple
partial refunds can reach full coverage. Cross-check aggregate payment status
with individual records rather than trusting one scalar flag.

A hypothetical one-cent shortfall remains unresolved unless provider evidence
establishes how it was settled. Do not add a guessed tolerance, mark it fully
refunded, or manufacture a zero financial balance. Label disposable sandbox
entitlement cleanup as operator cleanup and retain financial history.

Write a durable revocation marker when a verified refund arrives before its
payment callback so reordered success cannot restore revoked access. Bind it to
the original purchase and preserve it across later purchases. Derive access
effects from the invoice that actually granted access; a refunded rejected or
duplicate invoice must not revoke a different valid invoice's entitlement.

Keep presentment currency and amount separate from reporting-currency estimates.
Use the original booked identity and provenance for its matching full reversal;
do not recalculate with a newer exchange rate or edited metadata. Track partial
refunds even if the application only books full entitlement reversals. Disputes
need an explicit policy and supported transitions; they are not automatically
equivalent to a full cash refund.

## Qualification and public sources

Exercise the real handlers and schema for duplicate delivery, refund before
payment, cumulative partial refunds, cleanup failure, concurrent claims, and
legacy routing. A real sandbox pilot should establish provider compatibility,
durable grants, the delivered artifact/access window, and refund/revocation.
Nonzero-tax and cross-currency cases expose assumptions hidden by a simple sale.
Distinguish an actual renewal from a second fresh purchase or a simulated clock.

Start with these official sources and follow the current API navigation:

- [Documentation index](https://docs.dodopayments.com/llms.txt)
- [Checkout sessions](https://docs.dodopayments.com/developer-resources/checkout-session)
- [Subscription integration](https://docs.dodopayments.com/developer-resources/subscription-integration-guide)
- [On-demand subscriptions](https://docs.dodopayments.com/developer-resources/ondemand-subscriptions)
- [Create refund](https://docs.dodopayments.com/api-reference/refunds/post-refunds)
- [TypeScript SDK](https://github.com/dodopayments/dodopayments-typescript)
- [Testing process](https://docs.dodopayments.com/miscellaneous/testing-process)
- [Current pricing](https://dodopayments.com/pricing)

Entry points checked 25 October 2026. Recheck their current contents and the
installed SDK before implementation. This workflow does not certify an account's
eligibility, negotiated rates, deployment, or production readiness.
