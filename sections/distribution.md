# LPM Distribution Advisor

Help an author choose Private, Free, Pool, or Marketplace. Every package starts Private. Free, Pool, and Marketplace can't be undone, so always give the [permanence warning](#permanence-warning) before a change.

## Modes

| Mode | Who can install | Author revenue | Requirements |
|---|---|---|---|
| **Private** | Owner or org members | None | — |
| **Free** | Anyone, no account | None | Paid Pro or Organization plan (an unpaid trial doesn't qualify) |
| **Pool** | Pool subscribers ($12/month) | Usage-based share | — |
| **Marketplace** | License buyers | Sales minus platform fee | Stripe payout account and ≥1 active plan before saving (see [Pricing](./pricing.md)) |

Free, Pool, and Marketplace packages are public: everyone can see the package page. Choosing a mode sends the latest release to publication review, and the package stays private until a release is approved. Earlier private versions stay hidden (only the publisher can see them).

Rough fit: broad, high-volume utilities → Pool; niche or high-value tools that organizations buy → Marketplace; maximum reach with no revenue → Free. If the author is unsure, recommend staying Private.

## Workflow

1. Read `package.json`, `README.md`, and every `@lpm.dev/*` dependency (all dependency fields, `bundleDependencies`, `lpm.config.json`, `Package.swift`).
2. Ask only what's missing: revenue goal, audience (individuals vs organizations), maintenance commitment.
3. Check the [dependency rules](#dependency-rules). Name each blocking dependency and say what must change.
4. Recommend one mode: `## Recommendation: {mode}`, 2–3 sentences on why, a Free/Pool/Marketplace comparison (revenue model, effort, audience fit, estimate labelled as an estimate), blockers, then the permanence warning.
5. Check the [license](#license). For Marketplace, continue with [Pricing](./pricing.md).

## Free

- Every LPM dependency, direct or indirect, must be an available Free release. This covers dev, optional, and peer declarations, aliases, bundled copies, source config, and Swift manifests.
- A public Free release can't be revoked or unpublished. To replace one, deprecate it and publish a fix.
- If the paid plan ends, existing releases stay downloadable. The package is deprecated automatically after 30 days, and its hosted files are removed 365 days after the plan ended. Renewing before removal clears that schedule. Removed files can't be restored.

## Pool

Each $12 subscription: 20% ($2.40) to LPM, 80% ($9.60) split among the Pool packages *that subscriber* used that month, by weight.

- One observation per subscriber, package, and calendar month (UTC). Reinstalls and other machines don't add more.
- Depth weight: direct 1.00, depth 1 0.70, depth 2 0.49, depth 3 0.34. Clients other than LPM CLI report no graph, so their downloads count as direct.
- A dependency (depth ≥1) earns only if at least 2 different subscribers used it that month. A dependency with the same owner as the package that pulls it in gets 30% of its weight.
- A subscriber with no eligible observations has their share split equally across all Pool packages.
- Real numbers: `lpm pool --json` (current billing period only; e.g. `estimatedEarningsCents`, `packages[].weightedDownloads`). It needs a `lpm login` browser session; a CI or `.npmrc` registry token can't read it. Organization packages appear only for org owners and admins.

## Marketplace

Buyers pay per seat or per organization, one-time or by subscription. Fees, plan options, Commercial Licenses, and required license terms: [Pricing](./pricing.md).

## Dependency rules

Checked on the latest release when the mode is chosen, and again before each release becomes public. Pool and Marketplace check direct LPM dependencies; Free checks direct and indirect ones (see [Free](#free)). npm and other public registries are outside these rules.

| Dependency | → Free | → Pool | → Marketplace |
|---|---|---|---|
| Private, or missing from the registry | Blocked | Blocked | Blocked |
| Free | Allowed | Allowed | Allowed |
| Pool | Blocked | Allowed | Allowed |
| Marketplace | Blocked | Blocked | `peerDependencies`: allowed, each consumer buys it separately. Verified bundle or `devDependencies` from another owner: needs your Commercial License (same owner exempt). Unbundled in `dependencies`, `optionalDependencies`, `lpm.config.json` dependencies, or a Swift manifest: blocked |

Full rules: https://lpm.dev/docs/packages/dependencies

## Permanence warning

Always give this before any mode change:

> **This is permanent.** Once you choose Free, Pool, or Marketplace, the package can't go back to Private, switch to another mode, or be deleted (only archived). This applies as soon as you save the choice, even before any release is approved. You can still publish new versions and deprecate releases. [Free only: a public Free release can't be revoked or unpublished.] Do you want to proceed?

## License

The mode doesn't change the package's license, and LPM doesn't judge license compatibility.

- **Free / Pool:** any license works.
- **Marketplace:** a permissive license (MIT, ISC, BSD, Apache-2.0) lets buyers pass the code on freely, which undercuts paid access. This is advice, not an LPM rule. Options: switch to proprietary terms, drafted from the [commercial license template](../references/commercial-license-template.md); dual-license (the open-source copy stays freely usable); or choose Pool instead.
