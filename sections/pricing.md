# LPM Marketplace Pricing

Help a Marketplace author design pricing plans. For choosing the mode, permanence, and dependency rules, see [Distribution](./distribution.md). Pool packages have no pricing.

## Marketplace Facts

- **Platform fee**, on the price before tax: Hobby 15%, Pro and Organization 10% (an organization with lapsed plan billing pays 15%; Enterprise terms are custom). Pro costs $10/month, so suggest upgrading from Hobby when expected sales make the lower fee worth it.
- **Payouts:** Stripe Connect. Buyers see the Stripe account's business name as the seller.
- **Setup:** connect a Stripe payout account, then package dashboard → Settings → Distribution & monetization → select Marketplace (this shows the payout and plan fields) → New plan. The Marketplace choice can be saved only once a plan is active; the latest version then goes to publication review.
- **Versions:** every purchase gives access to every available version. Access isn't limited to a major version.

## Plan Fields

A package can have several active plans, e.g. individual monthly, individual one-time, and organization yearly.

| Field | Values |
|---|---|
| Type | One-time fee or subscription |
| Price (USD) | At least $0.50 |
| Billing interval | Monthly or annually (subscriptions only) |
| License type | Individual (per seat) or Organization (flat price, unlimited members) |
| Commercial license (allows bundling) | Optional. At most one active commercial plan per package |

Plans have no name, feature list, usage limit, or support level. They differ only by the fields above. Put feature or support promises in the README and LICENSE, not in plans.

- **Individual:** a personal buyer gets 1 seat. An organization buys 1–1000 seats, and owners or admins assign them to members.
- **Organization:** only an org owner or admin can buy it. All current and future members get access. It can't be bought for a personal account.
- **One-time:** access with no end date, until a full refund or revocation.
- **Subscription:** access for each paid period. Cancelling takes effect at the end of the period.

## Changing Plans

Plans can't be edited. To change a price or term, create a new plan, then archive the old one. Existing buyers keep their access: one-time buyers indefinitely, subscribers for each paid period. You can't archive the last active plan of a Marketplace package. To replace the commercial plan, archive it first (only one is allowed), and if it's the only plan, add a non-commercial plan before that.

## Commercial License

Buying a commercial plan lets the buyer bundle your package inside **their own Marketplace packages**. LPM checks for it when they publish. It's a registry permission only: it doesn't write your LICENSE or grant any other redistribution right, and the buyer's customers get no access of their own. A package never needs one for another package from the same owner. Offer one when other Marketplace authors may want to ship your code inside their packages; skip it for end-user tools.

## License Terms

LPM provides no template. Draft from the [commercial license template](../references/commercial-license-template.md), which lists the terms the Marketplace Seller Agreement requires, and recommend a lawyer review it.

## Workflow

1. Read `package.json` and `README.md`. Ask only what's unclear: who buys (individual developers or organizations), what competitors charge, and whether other authors would bundle it.
2. **License type:** Individual when value grows with the number of users; Organization when the whole team benefits equally. Offering both is fine.
3. **Billing:** subscription for packages that keep getting updates; one-time for stable tools. Offering both is fine.
4. **Price:** base it on the value delivered and on competitor prices from package pages and public listings. There is no `lpm marketplace compare` command.
5. **Output:** one line per plan, `{one-time|subscription} · {monthly|annual} · {Individual|Organization} · ${price}[ · commercial]`, with a short rationale, a revenue estimate after the platform fee, and the setup step from Marketplace Facts. Then point to the license template if the package has no matching LICENSE.
