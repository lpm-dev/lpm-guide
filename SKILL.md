---
name: lpm-guide
description: "Guide for the LPM package lifecycle: scaffold or migrate packages, improve quality, create lpm.config.json source configs, choose Pool or Marketplace distribution, design pricing, set up CI/CD, review packages, configure Swift registry, and create or update Agent Skills. Use for triggers like create package, lpm init, migrate to lpm, npm to lpm, dual publish, quality score, lpm publish --check, source package config, lpm.config.json, distribution mode, pool vs marketplace, marketplace pricing, deploy setup, LPM_TOKEN, npmrc setup, review package, swift package, Package.swift, lpm swift-registry, SE-0292, xcframework, .lpm/skills, lpm skills, and ai coding assistant skills."
---

# LPM Guide

Your guide for the entire LPM package lifecycle. Identify what the user needs from the table below, then **read the linked section file** for detailed instructions before proceeding.

## Context

LPM is a registry-agnostic CLI and lpm.dev is the hosted registry for `@lpm.dev/*` packages. lpm.dev packages use the format `@lpm.dev/owner.package-name`. Distribution modes:

- **Private** — Internal use only (default, reversible)
- **Pool** — Available via $12/month Pool subscription, revenue shared by usage (permanent)
- **Marketplace** — Listed for sale with per-seat or per-org licensing (permanent)

Read current source or docs before asserting CLI flags, config fields, or publish behavior. Treat `lpm.config.json` as a source-package config for `lpm add`, not as a package-type manifest.

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
| Set up SPM to use LPM packages | [Swift Registry](./sections/swift-registry.md) | "swift registry", "SE-0292", "lpm swift-registry", "spm setup", "Package.swift registry", "swift package resolve" |
| Create or update Agent Skills for AI assistants | [Skills](./sections/skills.md) | "agent skills", "create skills", "update skills", "lpm skills", ".lpm/skills" |

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
