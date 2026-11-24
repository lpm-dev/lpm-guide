---
name: lpm-guide
description: "Guide for the LPM package lifecycle: scaffold or migrate packages, improve quality, write lpm.config.json source configs, choose Private, Free, Pool, or Marketplace distribution, design Marketplace pricing and licenses, set up CI/CD, review packages, configure the Swift registry, and author Agent Skills. Use for triggers like create package, new package, scaffold, lpm init, migrate to lpm, npm to lpm, dual publish, improve quality, quality score, prepare for publish, lpm publish --check, lpm.config.json, make configurable, source package config, distribution mode, free distribution, pool vs marketplace, monetize, marketplace pricing, pricing strategy, how much to charge, license type, commercial license, deploy setup, CI setup, build fails 401, LPM_TOKEN, npmrc setup, trusted publishing, review package, is this safe, compare packages, swift registry, lpm swift-registry, SE-0292, spm setup, Package.swift registry, swift package resolve, agent skills, create skills, update skills, lpm skills, lpm-skills."
---

# LPM Guide

Your guide for the entire LPM package lifecycle. Identify what the user needs from the table below, then **read the linked section file** for detailed instructions before proceeding.

## Context

LPM is a registry-agnostic CLI and lpm.dev is the hosted registry for `@lpm.dev/*` packages. lpm.dev packages use the format `@lpm.dev/owner.package-name`. Distribution modes:

- **Private** — Internal use only (default, reversible)
- **Free** — Anyone can install it without an account. Needs a paid Pro or Organization plan (permanent)
- **Pool** — Available via $12/month Pool subscription, revenue shared by usage (permanent)
- **Marketplace** — Listed for sale with per-seat or per-org licensing (permanent)

Read current source or docs before asserting CLI flags, config fields, or publish behavior. Treat `lpm.config.json` as a source-package config for `lpm add`, not as a package-type manifest.

## Workflows

| User Intent | Guide |
|-------------|-------|
| Create a new package from scratch | [Scaffold](./sections/scaffold.md) |
| Bring an existing npm package to LPM, or dual publish | [Migrate](./sections/migrate.md) |
| Check or improve the quality score, prepare for publish | [Improve](./sections/improve.md) |
| Make a source package configurable (`lpm.config.json`) | [Source Config](./sections/source-config.md) |
| Choose Private, Free, Pool, or Marketplace | [Distribution](./sections/distribution.md) |
| Design Marketplace pricing and licensing | [Pricing](./sections/pricing.md) |
| Set up CI/CD, tokens, `.npmrc`, or trusted publishing | [Deploy](./sections/deploy.md) |
| Evaluate or compare packages before installing | [Review](./sections/review.md) |
| Set up SwiftPM to use LPM packages | [Swift Registry](./sections/swift-registry.md) |
| Create or update Agent Skills for a package | [Skills](./sections/skills.md) |

## References

Section files link to these when needed:

- [Quality Checks](./references/quality-checks.md): check IDs, points, and tiers
- [Config Spec](./references/config-spec.md): every `lpm.config.json` field and rule
- [Commercial License Template](./references/commercial-license-template.md): starter LICENSE for Marketplace packages

## How to Use This Skill

1. Match the user's request to a workflow in the table above
2. **Read the linked section file** using the Read tool — it contains the full instructions
3. Follow the workflow steps in the section file
4. If the user's request spans multiple workflows (e.g., "create a package and set up CI"), read each relevant section in sequence
