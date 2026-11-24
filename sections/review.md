# LPM Package Review

Help a consumer evaluate an `@lpm.dev/owner.package-name` package before adding it. The LPM-specific inputs are below. Use normal dependency-review judgment for API design, fit and alternatives.

## Access by Distribution Mode

- **Private:** only the owner or org members can install it.
- **Free:** anyone can install it without an account.
- **Pool:** included with a Pool subscription.
- **Marketplace:** a license purchase is required. See [Pricing](./pricing.md) for license types.

## Gather Data

1. **Quality report:** `lpm quality owner.package-name --json`.
   - Fields: `score`, `tier`, `status` (`complete`, `pending` or `unavailable`), and `checks[]` with category, points and status.
   - Public reports are anonymous. Private ones need publisher access.
   - `rulesVersion: null` with categories like Code/Testing means the report is from the earlier rules. Say so.
2. **Metadata:** `lpm info @lpm.dev/owner.package-name --json`. It gives versions and publish times, dependencies (including peer and optional), integrity and ecosystem. Distribution and download totals appear when the registry supplies them. It has no README or maintainer data.
3. **Code, without installing:** `lpm download @lpm.dev/owner.package-name --output <dir>`.
   - It extracts one archive and runs no scripts and installs no dependencies.
   - Read `package.json` (scripts, `exports`, dependencies), the README, the CHANGELOG, the LICENSE, the entry points and the `.d.ts` files.
   - If the package is already installed, read `node_modules/@lpm.dev/owner.package-name/` instead.
4. **If it's installed in the project:** run `lpm audit` for vulnerabilities, source capabilities, suspicious source patterns and Registry quality findings. Run `lpm audit --secrets` for the separate scan for hardcoded credentials.

## Assess

| Dimension | LPM signals |
|-----------|-------------|
| Quality | Score/100 and tier. Category points for Docs / API / Provenance / Health. Maxes are in [quality-checks.md](../references/quality-checks.md). |
| Authenticity | Provenance checks: `trusted-publishing`, `signed-provenance`, `repository-verified` and `verified-domain`. LPM has no "author verified" badge. |
| Security | `lpm audit` findings, plus a read of the source for install scripts, `eval`/`new Function`, `child_process`, network or fs access the package shouldn't need, and obfuscated code. A static read is best-effort. |
| Maintenance | Version history and publish times from `lpm info`, and a changelog entry for the current version. |
| Footprint | Dependency count (`lean-install` / `lean-deps`), ESM, `sideEffects`, archive size. |
| License and access | The license is compatible with the user's project. For Marketplace, which license type they need. |

## Present

```
## Review: @lpm.dev/owner.package-name
Recommendation: Install | Install with caution | Avoid. {1–2 sentence reason}
| Dimension | Rating | Evidence |   (rows from the table above; Quality as {score}/100 ({tier}))
Strengths / Concerns (with severity and file paths)
```

For a comparison, put the same rows side by side and end with a verdict tied to the user's needs.

| Recommendation | When |
|----------------|------|
| Install | `good`/`excellent` tier, no security red flags, recent releases |
| Install with caution | `fair` tier, or minor concerns |
| Avoid | `needs-work` tier, red flags such as harmful install scripts or obfuscation, or abandoned |

Base the verdict on evidence, not a bare yes/no. Don't compare Pool and Marketplace packages on price alone. A package that isn't on LPM has no quality report, so review its source only.
