# LPM Distribution Advisor

Help package authors choose the right distribution mode for their LPM package. This is the highest-stakes decision an author makes — choosing Free, Pool, or Marketplace is **permanent and irreversible**.

## Distribution Modes

| Mode | Access | Revenue | Reversible? |
|------|--------|---------|-------------|
| **Private** | Only you / your org | None | Yes — can switch to Free, Pool, or Marketplace |
| **Free** | Anyone, without an account | None | **No — permanently locked** |
| **Pool** | All Pool subscribers ($12/mo) | Usage-based share of subscriber pool | **No — permanently locked** |
| **Marketplace** | Buyers who purchase a license | Per-seat or per-org pricing you set | **No — permanently locked** |

## Workflow

1. **Understand the package** — Read package.json, check existing distribution, assess the package
2. **Ask strategic questions** — Goals, audience, maintenance commitment
3. **Analyze fit** — Score each mode against the author's situation
4. **Present recommendation** — Clear comparison with projected outcomes
5. **Warn about permanence** — Ensure the author understands the lock-in

## Step 1: Understand the Package

Read from the current directory:
- `package.json` — name, description, dependencies, version history
- `README.md` — understand what it does, target audience
- Check dependency count and types — utility vs. application-level

Classify the package:
| Type | Examples | Typical Fit |
|------|----------|-------------|
| **Small utility** | String helpers, date formatters, validators | Pool |
| **UI component library** | Design systems, component kits | Marketplace or Pool |
| **Framework/tool** | Build tools, testing frameworks | Marketplace |
| **Niche utility** | Industry-specific logic, specialized algorithms | Marketplace |
| **Open alternative** | Free competitor to paid tools | Pool |

## Step 2: Ask Strategic Questions

Ask these (skip any already answered):

### Revenue Goals
- Are you looking for passive income or active sales?
- Do you want predictable revenue or usage-based?
- What's your monthly revenue target?

### Audience
- How large is the potential audience? (Hundreds? Thousands? Tens of thousands?)
- Is this a general-purpose tool or niche/specialized?
- Would users pay individually or do organizations buy?

### Maintenance
- How actively will you maintain this? (Weekly updates? Quarterly? Set-and-forget?)
- Do you plan to add features over time or is it "done"?

### Dependencies
- Does your package depend on other LPM packages?
- If so, what distribution mode are they? (Private deps block Free, Pool, and Marketplace. A Free release can depend only on Free LPM packages)

## Step 3: Analyze Fit

### Pool — Best When

- Package has **broad appeal** (general-purpose utilities, common components)
- Author wants **passive income** without sales/marketing effort
- Package benefits from **network effects** (more installs = more revenue)
- Author is comfortable with **usage-based revenue** (no guaranteed minimums)
- Package will be **actively maintained** (maintenance-health check affects visibility)

**Pool Revenue Model:**
```
$12/subscriber/month
├── Platform fee (20%): $2.40
└── Author share (80%): $9.60 ← split among the Pool packages that subscriber installed that month, by weight
```

Revenue depends on:
- How many subscribers install your package each month
- Its weight among each of those subscribers' installs (a package counts once per subscriber a month)
- Depth multiplier (direct installs earn 100%, depth-1 deps earn 70%, depth-2 earn 49%)
- Anti-gaming: a dependency with the same owner as the package that pulls it in gets 30% weight, and a dependency earns only once at least 2 different subscribers installed it that month
- A subscriber who installs no Pool package that month has their share split equally across all Pool packages

**Pool is better when:**
- Your package will be installed by many projects (high volume)
- You'd rather earn automatically than manage sales
- You want your package discovered through the Pool catalog

### Marketplace — Best When

- Package solves a **specific, valuable problem** (worth paying for directly)
- Target audience is **organizations** (they have budgets and buy licenses)
- Package is **complex enough** to justify a price tag
- Author wants **predictable per-sale revenue** (not usage-dependent)
- Author is willing to do some **sales/marketing** effort
- Package has **few competitors** or clear differentiation

**Marketplace Revenue Model:**
```
You set the price per seat or per organization
├── Platform fee (10% Pro/Org, 15% Hobby), on the price before tax
└── Author receives the rest via Stripe Connect
```

License types:
- **Individual** ($X/seat) — Each user needs their own license
- **Organization** ($X flat) — One purchase covers the entire org

**Marketplace is better when:**
- You can charge $10+/month per seat and find buyers
- Your package saves organizations measurable time/money
- You want to control pricing, discounts, and commercial terms

### Free — Best When

- Author wants **anyone to install** the package, without an account, seat, or license
- Author has a **paid Pro or Organization plan** (an unpaid trial doesn't qualify)
- Every LPM dependency, direct or indirect, is itself a **Free** package
- Author doesn't need revenue from this package
- Author accepts that a **public Free release can't be unpublished or revoked**: replace a bad one by deprecating it and publishing a fix

**If the paid plan ends:** existing Free releases stay downloadable. After 30 days the package is deprecated automatically, and its hosted files are removed 365 days after the plan ended.

### Private — Keep When

- Package is internal tooling for your own projects
- Not ready for public distribution yet
- Contains proprietary business logic
- Dependencies include private packages (blocking Free, Pool, and Marketplace)

## Step 4: Present Recommendation

Format the recommendation as:

```
## Distribution Recommendation: {mode}

### Why {mode}

{2-3 sentence summary of why this mode fits their situation}

### Comparison

| Factor | Free | Pool | Marketplace | Your Fit |
|--------|------|------|-------------|----------|
| Revenue model | None | Usage-based share | Per-seat/org pricing | {assessment} |
| Effort | Passive | Passive (publish & earn) | Active (marketing, support) | {assessment} |
| Audience fit | Everyone | Broad / developer tools | Niche / enterprise | {assessment} |
| Revenue potential | None | {estimate} | {estimate} | {which is higher} |
| Lock-in risk | Permanent | Permanent | Permanent | Same |

### Revenue Projections

If `lpm pool stats --json` is available, use real Pool revenue data for projections instead of estimates. The command returns the user's actual weighted downloads, share percentage, and estimated earnings for the current billing period.

**Pool estimate:**
- If installed by {N} projects/month across {M} subscribers
- Estimated share: ${X}/month (varies with total Pool activity)

**Marketplace estimate:**
- At ${price}/seat with {N} customers
- Estimated revenue: ${X}/month (minus platform fee)

### Before You Switch

{warnings about permanence, dependency restrictions, and what can't be undone}
```

## Step 5: Permanence Warning

**Always include this warning before any mode change:**

> **This change is permanent.** Once a package is set to Free, Pool, or Marketplace distribution, it cannot be changed back to Private or switched to another mode. The package also cannot be deleted — only archived.
>
> Before switching:
> 1. All dependencies must allow redistribution (no private deps)
> 2. A Free release can depend only on Free LPM packages
> 3. Marketplace deps require a commercial license to bundle
> 4. Pool packages cannot depend on Marketplace packages
> 5. A public Free release can't be unpublished or revoked
> 6. Your package version history will be publicly visible
>
> Are you sure you want to proceed?

## Dependency Restrictions Reference

When switching distribution, all dependencies are checked:

| Dependency Mode | Target: Free | Target: Pool | Target: Marketplace |
|-----------------|--------------|-------------|---------------------|
| Private | Blocked | Blocked | Blocked |
| Free | Allowed | Allowed | Allowed |
| Pool | Blocked | Allowed | Allowed |
| Marketplace | Blocked | Blocked | Peer dependency: allowed, each consumer needs their own access. Verified bundle or devDependency: requires a commercial license (same owner exempt). Unbundled in dependencies: blocked |

Full rules: `https://lpm.dev/docs/packages/dependencies`

If any dependency blocks the transition, list each blocking dependency by name and explain what would need to change.

## Licensing Guidance

After recommending a distribution mode, check the package's `LICENSE` file:

**Free:** Any license works. No license change needed.

**Pool:** Any license works (MIT, ISC, Apache, etc.). The Pool subscription is a platform fee, not a license fee. No license change needed.

**Marketplace:** Open-source licenses (MIT, ISC, BSD, Apache) allow free redistribution, which conflicts with per-seat pricing. Recommend:
1. Switching to a commercial license — offer the LPM Commercial License template from `https://lpm.dev/docs/marketplace/commercial-licenses#license-file-template`
2. Or dual-licensing: keep MIT on GitHub, use commercial license on LPM Marketplace

If the author chooses Marketplace and has an open-source license, flag this before they proceed:

```
Your package uses MIT license. Marketplace per-seat pricing requires a
commercial license — MIT allows buyers to freely redistribute your code.

Options:
1. Switch LICENSE to LPM Commercial License (recommended for Marketplace)
2. Dual-license: MIT on GitHub + commercial on LPM
3. Stay with Pool instead (works with any license)
```

## Important Notes

- Never suggest switching distribution without explaining the permanence
- Always check dependencies before recommending Free, Pool, or Marketplace
- Revenue projections are estimates — actual revenue depends on market conditions
- Don't oversell either option — be honest about tradeoffs
- If the author is unsure, recommend staying Private until they have more data
- Pool anti-gaming measures (same-owner penalty, minimum unique installers) affect revenue — mention these if relevant
- If the author chooses Marketplace, follow up with the [Pricing](./pricing.md) workflow
