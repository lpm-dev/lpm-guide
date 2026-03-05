# lpm-guide

Comprehensive agent skill for the entire LPM package lifecycle. One skill, eight workflows — the LLM reads only the section it needs.

## Installation

```bash
npx skills add https://github.com/lpm-dev/lpm-guide
```

## What It Does

Triggers automatically on phrases related to any part of the LPM package lifecycle:

| Workflow | Triggers | What It Does |
|----------|----------|-------------|
| **Scaffold** | "create package", "new package", "lpm init" | Create a new LPM package from scratch |
| **Migrate** | "migrate to lpm", "npm to lpm", "dual publish" | Move an npm package to LPM |
| **Improve** | "improve quality", "quality score", "prepare for publish" | Run 27 quality checks + deep analysis, offer fixes |
| **Source Config** | "lpm config", "make configurable", "lpm.config.json" | Generate configurable source package config |
| **Distribution** | "distribution mode", "pool vs marketplace", "monetize" | Choose Private, Pool, or Marketplace |
| **Pricing** | "pricing strategy", "how much to charge", "license type" | Design Marketplace pricing tiers |
| **Deploy** | "deploy setup", "CI setup", "build fails 401", "LPM_TOKEN" | Configure CI/CD for install or publish |
| **Review** | "review package", "is this safe", "compare packages" | Evaluate a package before installing |

## How It Works

The entry point (`SKILL.md`) is a lightweight routing table (~3KB). When triggered, the LLM:

1. Matches the user's intent to a workflow
2. Reads only the relevant section file (~5-12KB)
3. Follows the detailed instructions in that section

This keeps context usage low (~8-15KB per invocation) while covering all 8 workflows.

## Structure

```
lpm-guide/
├── SKILL.md                    # Entry point with routing table
├── sections/
│   ├── scaffold.md             # Create new packages
│   ├── migrate.md              # Migrate from npm
│   ├── improve.md              # Quality scoring & fixes
│   ├── source-config.md        # lpm.config.json generation
│   ├── distribution.md         # Distribution mode selection
│   ├── pricing.md              # Marketplace pricing design
│   ├── deploy.md               # CI/CD setup
│   └── review.md               # Package evaluation
├── references/
│   ├── quality-checks.md       # All 27 quality checks
│   └── config-spec.md          # Full lpm.config.json spec
├── README.md
└── LICENSE
```

## Related

- [LPM CLI](https://github.com/lpm-dev/cli) — The `lpm` command-line tool
- [lpm.dev](https://lpm.dev) — LPM package registry

## License

MIT
