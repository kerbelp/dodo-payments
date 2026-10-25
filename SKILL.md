---
name: dodo-payments
description: Implement and troubleshoot Dodo Payments checkout, subscriptions, signed webhooks, refunds, prepaid credit, automatic top-ups, and financial recovery. Use for Dodo billing integrations, sandbox qualification, and scoped rollout operations.
---

# Dodo Payments

Work within the target repository's payment, authentication, storage, and release
conventions. Read its instructions and relevant decisions, then reuse existing
integration code. Match the work to the request: diagnosis does not require a
new billing architecture, and a sandbox pilot does not authorize live charging.
This skill supplies a workflow; API access and action permissions come from the
user and the active environment.

## Establish the billing boundary

Identify the environment, business and brand, products, currency, price/tax
semantics, one-time or recurring terms, authenticated owner, promised delivery,
refund policy, and current sale state. Distinguish source configuration, remote
provider state, deployed configuration, and an actual successful customer journey.

Test and Live have separate hosts, keys, resource IDs, webhook secrets, and
financial records. Check the complete tuple before writing secrets or operating
resources; do not infer mode or resource type from an ID prefix. Isolate a new
brand or product surface without replacing unrelated credentials or routing.
Keep keys, customer records, payment details, and raw operational evidence out of
public repositories and logs.

Verify current endpoint schemas and SDK behavior through the [official docs
index](https://docs.dodopayments.com/llms.txt). This skill's design recommendations
do not establish provider guarantees. For fees, eligibility, or brand onboarding,
use current official policy and the target account's terms. An answer for one
account is not approval for every product, brand, or billing model.

Read only the references relevant to the work:

- [Lifecycle and integration](references/lifecycle.md): provisioning, checkout,
  signed events, mixed carts, subscription access, refunds, and runtime traps.
- [Prepaid credit and recovery](references/prepaid-and-recovery.md): consent,
  automatic top-ups, limits, unknown charges, tax/credit mismatches, and restoring
  financial state without recreating charging authority.

## Implement checkout and fulfillment

Create checkout on the authenticated server with trusted product, quantity,
price policy, allowed return URLs, and a persisted owner-bound purchase intent.
Enforce sale gates and cross-origin protections on the actual mutation routes.
Browser return parameters and customer-entered email do not prove ownership or
payment.

Persist the intent before requesting a payable session. Reuse its still-valid,
unconsumed session rather than creating duplicate payable attempts. After a
timeout or crash, reconcile the uncertain attempt before allowing another.
Retain original intent identity for replay and close only its matched reservation;
a delayed callback must not deactivate a newer purchase.

Check endpoint-specific idempotency and lookup contracts before adding retries.
Metadata correlation and a local idempotency key do not establish provider
idempotency. Disable automatic retries for non-idempotent creation calls whose
outcome is uncertain; the TypeScript SDK exposes `maxRetries: 0`. Inspect job,
proxy, and application retries too. Expiry alone is not proof that a payment
already in progress cannot settle.

Verify webhook signatures over the original raw body using the maintained
verifier. Reconcile provider GET truth against the original business, brand,
owner, session, offer, and currency before granting anything. Persist durable
financial observations and deduplicate both delivery and business effects.
Complete a delivery claim after durable work succeeds; retry transient failures
instead of acknowledging incomplete processing. Record inconsistent paid events
for review without granting unrelated entitlements.

Keep server credentials, signature handling, and storage clients out of browser
bundles. Exercise the real exported handlers against the project's schema and
runtime, including its existing billing routes.

## Preserve consent and product promises

A manual purchase, permission to consume purchased credit, and permission to
charge a saved payment method are separate decisions. A signed-in session or a
checked preference does not itself authorize enrollment or off-session charging.
Bind an explicit setup action to the displayed product, amount, limits, and terms,
then verify the corresponding provider mandate.

Keep the customer's cash charge, tax, service credit, processor fees, and payout
proceeds distinct. Never silently reduce advertised credit or raise a charge to
repair a tax or fee mismatch. Use the product's accepted reconciliation/refund
policy. Do not claim a preview locks a later charge or tax split without a
documented, verified contract for that exact flow.

Carry forward existing authorization and release scope. Separate an unanswered
provider question from a confirmed constraint; do not add an approval gate just
because a support question exists. Account correspondence does not expand the
user's authorization to change settings, deploy, or charge.

## Qualify the requested outcome

Choose checks proportional to the changed paths. A checkout integration normally
needs an authenticated sandbox purchase, real signed callback, one durable grant,
owner isolation, duplicate/reordered-event handling, and its advertised refund
behavior. A subscription adds renewal and cancellation/access-period checks; a
top-up adds mandate, limits, disablement, and uncertain-outcome checks. Use the
provider's [official testing guidance](https://docs.dodopayments.com/miscellaneous/testing-process).

Verify the actual deliverable, such as an authorized download and its hash, API
credit, or access window. Test revocation through the delivered application's
handler. A mocked response supplements a real provider pilot; it does not prove
current API compatibility. A signed ignored-event probe proves reachability and
signature configuration, not paid fulfillment.

Follow the repository's review and release requirements. When activation is in
scope, verify the matching deployment, bindings, remote catalog, callbacks, and
customer journey before claiming sales are enabled. Preserve callbacks, paid
access, refunds, and reconciliation when pausing new purchases. Do not sell a
subscription merely because its SKU exists.

Report what changed, the tested environment and revision, actual provider
evidence, remaining gaps, and sale/charge state. Label fixtures, sandbox results,
live results, and operator cleanup accurately; omit secrets and customer identity.
