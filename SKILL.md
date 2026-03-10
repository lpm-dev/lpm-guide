---
name: lpm-guide
description: Comprehensive guide for LPM packages — create, migrate, improve quality, configure source packages, choose distribution mode, set pricing, deploy to CI/CD, review packages, and create Agent Skills. Covers JavaScript, Swift, and XCFramework ecosystems. Triggers on "create package", "new package", "scaffold package", "init package", "lpm init", "migrate to lpm", "npm to lpm", "dual publish", "convert to lpm", "improve package", "check quality", "quality score", "package health", "quality audit", "prepare for publish", "lpm publish --check", "lpm config", "make configurable", "source package config", "lpm.config.json", "add presets", "configurable package", "distribution mode", "pool vs marketplace", "monetize package", "revenue model", "how to sell", "make money from package", "go public", "distribution strategy", "pricing strategy", "set price", "pricing tiers", "how much to charge", "license type", "per seat pricing", "marketplace pricing", "deploy setup", "vercel lpm", "netlify lpm", "ci setup", "build fails", "401 during install", "LPM_TOKEN", "npmrc setup", "ci cd lpm", "auto publish", "github actions lpm", "review package", "is this package safe", "should I use", "package review", "evaluate package", "compare packages", "check package", "swift package", "swift library", "ios package", "macos package", "spm package", "swift sdk", "Package.swift", "xcframework", "xcf package", "binary framework", "ios library", "apple platform package", "agent skills", "create skills", "add skills", "lpm skills", "skills for package", ".lpm/skills", "ai coding assistant skills".
---

# LPM Guide

Your guide for the entire LPM package lifecycle. Identify what the user needs from the table below, then **read the linked section file** for detailed instructions before proceeding.

## Context

LPM (lpm.dev) is a package registry. Packages use the format `@lpm.dev/owner.package-name`. Distribution modes:

- **Private** — Internal use only (default, reversible)
- **Pool** — Available via $12/month Pool subscription, revenue shared by usage (permanent)
- **Marketplace** — Listed for sale with per-seat or per-org licensing (permanent)

## Workflows

| User Intent | Guide | When to read |
|-------------|-------|--------------|
| Create a new package from scratch | [Scaffold](./sections/scaffold.md) | "create package", "new package", "scaffold", "init" |
| Bring an existing npm package to LPM | [Migrate](./sections/migrate.md) | "migrate to lpm", "npm to lpm", "dual publish" |
| Check or improve package quality score | [Improve](./sections/improve.md) | "improve quality", "quality score", "prepare for publish" |
| Make a source package configurable | [Source Config](./sections/source-config.md) | "lpm config", "make configurable", "lpm.config.json" |
| Choose Private, Pool, or Marketplace | [Distribution](./sections/distribution.md) | "distribution mode", "pool vs marketplace", "monetize" |
| Design Marketplace pricing and licensing | [Pricing](./sections/pricing.md) | "pricing strategy", "how much to charge", "license type" |
| Set up CI/CD for installing or publishing | [Deploy](./sections/deploy.md) | "deploy setup", "CI setup", "build fails 401", "LPM_TOKEN" |
| Evaluate a package before installing | [Review](./sections/review.md) | "review package", "is this safe", "compare packages" |
| Create Agent Skills for AI assistants | [Skills](./sections/skills.md) | "agent skills", "create skills", "lpm skills", ".lpm/skills" |

## References

These are detailed specifications referenced by the section files:

| Reference | Used by |
|-----------|---------|
| [Quality Checks](./references/quality-checks.md) | Improve, Review |
| [Config Spec](./references/config-spec.md) | Source Config |

## How to Use This Skill

1. Match the user's request to a workflow in the table above
2. **Read the linked section file** using the Read tool — it contains the full instructions
3. Follow the workflow steps in the section file
4. If the user's request spans multiple workflows (e.g., "create a package and set up CI"), read each relevant section in sequence
