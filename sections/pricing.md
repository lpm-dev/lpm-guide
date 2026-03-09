# LPM Marketplace Pricing Strategy

Help Marketplace package authors design pricing that maximizes revenue while being fair to buyers. Covers license types, tier structure, price points, commercial licenses, and free trial strategy.

## LPM Marketplace Model

Authors sell packages through the LPM Marketplace. Key facts:

- **Platform fee**: 10% (Pro/Org plans) or 15% (Free plan)
- **Payments**: Stripe Connect (authors receive direct payouts)
- **License types**: Individual (per-seat) or Organization (flat price)
- **Commercial licenses**: Optional add-on that allows buyers to bundle/redistribute
- **No free tier on Marketplace** — For free distribution, use Pool instead

## Workflow

1. **Understand the package** — What it does, who uses it, competitive landscape
2. **Choose license type** — Individual vs. Organization
3. **Design tier structure** — How many tiers, what differentiates them
4. **Set price points** — Based on value, competition, and audience
5. **Configure commercial licensing** — If applicable
6. **Summarize the pricing table** — Ready to implement in LPM dashboard

## Step 1: Understand the Package

Read from the current directory:
- `package.json` — description, features
- `README.md` — what it does, who it's for

Ask the user (skip if already clear):

| Question | Why |
|----------|-----|
| Who is the primary buyer? (Individual dev, team lead, CTO) | Determines price sensitivity |
| What problem does it solve? | Determines value-based pricing |
| Are there competitors? What do they charge? | Anchors pricing |
| How will it be used? (Dev tool, production component, internal tool) | Determines license type |
| How many people typically use this on a team? | Determines per-seat vs per-org |

## Step 2: Choose License Type

### Individual (Per-Seat)

```
Each person who uses the package needs their own license.
Org admins can purchase bulk seats and assign them to members.
```

**Best for:**
- Developer tools used by individual contributors
- IDE plugins, CLI tools, personal utilities
- Packages where usage scales with team size

**Pricing psychology:** Lower price per seat, revenue scales with team size.

### Organization (Flat Price)

```
One purchase covers the entire organization. All members get access.
```

**Best for:**
- Shared infrastructure (design systems, component libraries)
- Packages where everyone in the org benefits equally
- High-value packages where per-seat feels nickel-and-diming

**Pricing psychology:** Higher single price, simpler purchasing decision.

### Decision Framework

| Signal | Individual | Organization |
|--------|-----------|-------------|
| Used by specific roles only | Yes | — |
| Used by entire engineering team | — | Yes |
| Price sensitivity is high | Yes (lower per-seat) | — |
| Buyer is a company with budget | — | Yes |
| Usage varies by team size | Yes | — |
| Value is same regardless of team size | — | Yes |

## Step 3: Design Tier Structure

### How Many Tiers?

| Count | When | Example |
|-------|------|---------|
| **1 tier** | Simple package, one use case | Utility library, single component |
| **2 tiers** | Standard + premium features | Standard: core features, Pro: advanced |
| **3 tiers** | Good/better/best strategy | Starter, Pro, Enterprise |

More than 3 tiers creates decision paralysis. Default to 2 unless there's a clear reason for 3.

### What Differentiates Tiers?

| Differentiator | Example |
|---------------|---------|
| **Feature set** | Basic components vs. full library |
| **Usage limits** | Up to 3 projects vs. unlimited |
| **Support level** | Community vs. priority support |
| **Commercial rights** | Internal use vs. redistribution allowed |
| **Updates** | 6 months vs. lifetime |

**Tier naming suggestions:**
- Standard / Pro
- Starter / Professional / Enterprise
- Personal / Team / Business
- Community / Commercial

## Step 4: Set Price Points

### Value-Based Pricing

Ask: "How much time/money does this save the buyer per month?"

| Time Saved | Suggested Price Range |
|------------|----------------------|
| 1-2 hours/month | $5-15/seat/month |
| 5-10 hours/month | $15-40/seat/month |
| 20+ hours/month | $40-100/seat/month |
| Replaces a $X/mo SaaS tool | 30-50% of that tool's price |

### Competitive Anchoring

If `lpm marketplace compare --json` is available, use it to fetch real competitor pricing data. Run `lpm marketplace compare <category-or-package> --json` to get actual price ranges, quality scores, and download counts for comparable packages in the same category.

If competitors exist:
- **No competitors**: Price on value (you have pricing power)
- **Cheaper competitors exist**: Justify premium with quality/features or match price
- **Expensive competitors exist**: Undercut by 30-50% (your distribution advantage)

### Price Psychology

- End prices in 9 ($19, $29, $49) for consumer-facing
- Use round numbers ($20, $50, $100) for enterprise/B2B
- The middle tier should be the one you want most people to buy
- Make the top tier 2-3x the middle tier (anchoring effect)

### Common Price Points for Developer Packages

| Package Type | Per-Seat/Month | Per-Org/Month |
|-------------|---------------|---------------|
| Small utility | $5-10 | $20-50 |
| Component library | $10-25 | $50-150 |
| Full framework/toolkit | $25-50 | $100-300 |
| Enterprise tool | $50-100 | $200-500 |

These are starting points — adjust based on Steps 1-3 findings.

## Step 5: Commercial Licensing

Commercial licenses allow buyers to bundle your package in their own products and redistribute it.

### When to Offer

- Your package is a building block others might ship in their products
- Component libraries that agencies use in client projects
- SDKs or utilities that other package authors might depend on

### Pricing Commercial Licenses

Commercial licenses typically cost 2-5x the standard license because they enable the buyer to generate revenue from your work.

| Standard Price | Commercial Price |
|---------------|-----------------|
| $10/seat | $25-50/seat |
| $50/org | $150-250/org |

### When NOT to Offer

- End-user tools (CLI tools, dev utilities) — no redistribution use case
- Packages with copyleft dependencies (GPL) — can't offer commercial terms anyway

## Step 6: Present Pricing Table

Format the recommendation as:

```
## Pricing Recommendation: {package-name}

### License Type: {Individual/Organization}
{1-2 sentence rationale}

### Pricing Table

| | Standard | Pro |
|--|----------|-----|
| **Price** | ${X}/seat/mo | ${Y}/seat/mo |
| **Features** | {list} | {list} |
| **Support** | Community | Priority |
| **Commercial rights** | No | Yes |

### Rationale
- {Why this license type}
- {Why these price points}
- {How this compares to alternatives}

### Revenue Projections
- At {N} customers: ${X}/month (after {fee}% platform fee)
- Break-even: {N} customers
- Target: {N} customers for ${X}/month

### Implementation
1. Go to LPM Dashboard → Package Settings → Pricing
2. Create "{tier name}" plan: ${price}/{period}
3. {Repeat for each tier}
4. {Enable commercial license if applicable}
```

## Important Notes

- Platform fee is 10% on Pro/Org plans, 15% on Free plan — factor this into pricing
- Recommend the user upgrades to Pro ($7/mo) if they're on Free to reduce fees from 15% to 10%
- Don't overcomplicate pricing — 1-2 tiers is fine for most packages
- Prices can be changed later (unlike distribution mode which is permanent)
- If the user is unsure, suggest starting with one tier and adding a premium tier after getting initial customers
- For Pool packages, pricing doesn't apply — revenue is automatic from usage. Redirect to [Distribution](./distribution.md) if they're choosing between Pool and Marketplace
