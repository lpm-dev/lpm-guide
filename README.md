# lpm-guide

Comprehensive agent skill for the entire LPM package lifecycle. One skill, ten workflows — the LLM reads only the section it needs.

## Installation

```bash
npx skills add lpm-dev/lpm-guide
# or
npx skills add https://github.com/lpm-dev/lpm-guide
```

## What It Does

Triggers automatically on phrases related to any part of the LPM package lifecycle:

| Workflow | Triggers | What It Does |
|----------|----------|-------------|
| **Scaffold** | "create package", "new package", "scaffold", "lpm init" | Create a new LPM package from scratch |
| **Migrate** | "migrate to lpm", "npm to lpm", "dual publish" | Move an npm package to LPM |
| **Improve** | "improve quality", "quality score", "prepare for publish", "lpm publish --check" | Analyze package quality, run checks, offer fixes |
| **Source Config** | "lpm.config.json", "make configurable", "source package config" | Generate configurable source package config |
| **Distribution** | "distribution mode", "free distribution", "pool vs marketplace", "monetize" | Choose Private, Free, Pool, or Marketplace |
| **Pricing** | "marketplace pricing", "pricing strategy", "how much to charge", "license type", "commercial license" | Design Marketplace pricing plans and licenses |
| **Deploy** | "deploy setup", "CI setup", "build fails 401", "LPM_TOKEN", "npmrc setup", "trusted publishing" | Configure CI/CD for install or publish |
| **Review** | "review package", "is this safe", "compare packages" | Evaluate a package before installing |
| **Swift Registry** | "swift registry", "lpm swift-registry", "SE-0292", "spm setup", "Package.swift registry", "swift package resolve" | Set up SPM to use LPM packages via SE-0292 |
| **Skills** | "agent skills", "create skills", "update skills", "lpm skills", "lpm-skills" | Create or update Agent Skills for AI coding assistants |

## How It Works

The entry point (`SKILL.md`) is a lightweight routing table (~4KB). When triggered, the LLM:

1. Matches the user's intent to a workflow
2. Reads only the relevant section file (~4-10KB), plus a reference file only when the task needs one
3. Follows the detailed instructions in that section

This keeps context usage low: ~7-13KB per invocation, or ~14-23KB when reference files are also loaded, while covering all 10 workflows.

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
│   ├── review.md               # Package evaluation
│   ├── swift-registry.md       # Swift Package Registry (SE-0292)
│   └── skills.md               # Agent Skills creation & updates
├── references/
│   ├── quality-checks.md       # Ecosystem-specific quality checks
│   ├── config-spec.md          # Full lpm.config.json spec
│   └── commercial-license-template.md  # Starter commercial license for Marketplace packages
├── README.md
└── LICENSE
```

## Related

- [LPM CLI](https://github.com/lpm-dev/rust-client) — The `lpm` command-line tool
- [lpm.dev](https://lpm.dev) — LPM package registry

## License

MIT
