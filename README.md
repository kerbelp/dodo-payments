# Dodo Payments

An independent agent skill for implementing and operating Dodo Payments billing:
checkout, subscriptions, signed webhooks, refunds, prepaid credit, automatic
top-ups, and financial recovery.

It focuses on the parts that extend beyond creating a checkout URL: binding a
payment to its owner, handling duplicate and delayed events, reconciling lost
responses, honoring paid access and refund terms, and keeping automatic charges
within explicit consent and limits.

## Use it

```sh
git clone https://github.com/kerbelp/dodo-payments.git
```

The repository root is the skill folder. Place the complete folder in the skills
directory supported by your agent, or ask the agent to read and follow
[SKILL.md](SKILL.md) directly. Installation and discovery depend on your agent.

The skill bundles instructions and references. It does not include an SDK,
payment server, API credentials, or account access. Your agent supplies its own
development tools and uses your project's existing stack.

Example requests:

> Use $dodo-payments to add one-time checkout for our existing product. Follow
> the repository's architecture, bind payment to the signed-in owner, and qualify
> fulfillment and refunds in Test Mode. Keep Live payments disabled.

> Use $dodo-payments to investigate why a refund succeeded but access remains
> active. Inspect our handler and sanitized event evidence. Prepare a focused fix
> and verification plan; do not issue another refund.

> Use $dodo-payments to design optional automatic replenishment for our prepaid
> API. Separate spending permission from the charging mandate, model count and
> cash limits, and explain recovery after a charge response is lost.

## What it covers

| Area | Focus |
| --- | --- |
| Checkout | Server-owned intents, trusted offers, owner isolation, uncertain outcomes |
| Webhooks | Raw-body signatures, durable claims, delivery and effect deduplication |
| Subscriptions | Paid service periods, mixed carts, renewals, cancellation |
| Refunds | Partial/remainder coverage, tax and currency, revocation, lost responses |
| Prepaid credit | Consent, gross cash versus credit, automatic top-up limits |
| Recovery | Financial fences, closed reconstruction, reconciliation before reopening |
| Rollouts | Test/Live separation, matching deployment evidence, safe purchase pauses |

The workflow adapts to the request. A small diagnosis stays focused; a complete
billing integration needs evidence for its promised purchase, delivery, renewal,
and refund behavior. It distinguishes simulated tests, real sandbox results, and
live operation rather than treating them as interchangeable.

## Files

- [SKILL.md](SKILL.md): workflow and routing to the relevant reference.
- [references/lifecycle.md](references/lifecycle.md): checkout, subscriptions,
  signed events, refunds, and integration traps.
- [references/prepaid-and-recovery.md](references/prepaid-and-recovery.md):
  credit, mandates, limits, uncertain charges, and financial restoration.
- [agents/openai.yaml](agents/openai.yaml): optional agent interface metadata.
- [LICENSE](LICENSE): MIT license.

## Provider contracts and contributions

Use [Dodo's official documentation](https://docs.dodopayments.com/llms.txt),
[SDK](https://github.com/dodopayments/dodopayments-typescript), and
[current pricing](https://dodopayments.com/pricing) for provider contracts. The
skill's design patterns do not establish endpoint guarantees, account approval,
fees, or production readiness. Verify current schemas and installed versions.

Contributions should explain the failure case, distinguish provider guarantees
from defensive application policy, and use public sources or fictional examples.
Keep credentials, account identifiers, customer data, support correspondence,
private runbooks, and raw operational evidence outside this public repository.

Maintained by [Pavel Kerbel](https://github.com/kerbelp). This is an independent
community skill, not affiliated with or endorsed by Dodo Payments. Product names
and linked third-party materials belong to their respective owners.

## License

[MIT](LICENSE).
