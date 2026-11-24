# Quality Checks

**The CLI gives the live list:** `lpm publish --check --json` covers a local package and `lpm quality owner.package-name --json` covers a published one. Prefer their output to this table. The table mirrors `crates/lpm-security/src/quality/spec.rs`, and the JSON `rulesVersion` tells you which rules produced a report.

## Model

- Each ecosystem totals **100 points** across 4 categories. JS has 24 checks and Swift has 23.

  | | Docs | API | Provenance | Health |
  |--|--|--|--|--|
  | JS | 32 | 26 | 12 | 30 |
  | Swift | 32 | 30 | 12 | 26 |

- **Archive checks** run in LPM CLI code. LPM.dev Registry runs the same code on the uploaded tarball, so the two agree when the CLI versions match. A quality report sent with a publish earns nothing.
- **Registry checks** (marked **R** below) need Registry data. There are 5. `lpm publish --check` shows them as `after_publish`.
- `preflightScore` in `--check --json` is archive points ÷ archive max points, rounded down. It is the number `--min-score` compares, not the /100 score.
- **Tiers:** `excellent` is 90 or more, `good` is 70–89, `fair` is 50–69, and `needs-work` is under 50. A low score never fails a command unless you pass `--min-score`.
- A report from before the 4 categories existed has `rulesVersion: null` and shows the old categories (Code, Testing) until the Registry scores it again.

## Checks shared by JS and Swift

| id | Cat | Pts | Earns points when |
|----|-----|-----|-------------------|
| `readme` | Docs | 6 | A UTF-8 root README (README.md preferred; README, .txt, .markdown also count) has ≥300 characters of prose. Code blocks, images, link URLs and HTML don't count. |
| `readme-install` | Docs | 3 | A code block installs the exact package name. **JS:** `lpm install`/`lpm i`, npm/pnpm/bun `install`/`i`/`add`, or `yarn add`. `lpm add` counts only for source packages, meaning a root `lpm.config.json` or the legacy `package.json` `lpm.source`. **Swift:** `.package(id: "lpmdev.owner_name", ...)` or `lpm install @lpm.dev/owner.name`. |
| `readme-example` | Docs | 5 | **JS:** a ```` ```js ```` or ```` ```ts ```` block imports the package (2), and imports only names it actually exports (5). **Swift:** a ```` ```swift ```` block imports a library module (2) and uses its public API (5). |
| `api-docs` | Docs | 10 | Doc comments of ≥20 characters sit on exports. **JS:** in the `.d.ts` files, or in the ESM entry when no `.d.ts` ships. **Swift:** `///` on public/open declarations outside tests. Coverage of 80% earns 10, 50% earns 6, 20% earns 3. |
| `changelog` | Docs | 3 | CHANGELOG, CHANGES or HISTORY at the root has an entry that names this version. |
| `agent-skills` | Docs | 5 | **R:** at least one Agent Skill is approved by the security scan. See [Skills](../sections/skills.md). |
| `trusted-publishing` | Prov | 5 | **R:** published from CI through OIDC trusted publishing. See [Deploy](../sections/deploy.md). |
| `signed-provenance` | Prov | 2 | **R:** published with a Sigstore provenance bundle (`lpm publish --provenance`). |
| `repository-verified` | Prov | 2 | **R:** trusted-published from the repo that `package.json` `repository` names. |
| `verified-domain` | Prov | 3 | **R:** the owner (the org, for an org package) verified a domain with a DNS TXT record in LPM.dev settings. |
| `license` | Health | 5 | `license` is a current SPDX expression, a `LicenseRef-*`, or `SEE LICENSE IN <file>` with that file shipped. `UNLICENSED` earns 0, so proprietary terms should use `SEE LICENSE IN`. |
| `license-file` | Health | 2 | LICENSE, LICENCE or COPYING is at the root. |
| `description` | Health | 2 | `package.json` `description` is 20–280 characters and isn't just the package name. |
| `keywords` | Health | 1 | `keywords` has at least one entry. |
| `repository` | Health | 3 | `package.json` `repository` gives the source repository's URL. |
| `clean-archive` | Health | 6 | **JS:** no `.env` files (example, sample and template files are fine), `.git/`, `coverage/`, OS/editor files, nested `.tgz`, or unbundled `node_modules`, and under 10 MB unpacked. **Swift:** no `.build/`, `.swiftpm/`, `xcuserdata`, `DerivedData` or `.git/`. |

## JS only

| id | Cat | Pts | Earns points when |
|----|-----|-----|-------------------|
| `entry-points` | API | 5 | Every path in `main`, `module`, `types`, `typings`, `bin` and `exports` names a file in the package. |
| `types` | API | 10 | Every exports entry has declarations under both `import` and `require`: either a `types` condition, or a `.d.ts` next to each `.js` (`.d.mts` for `.mjs`, `.d.cts` for `.cjs`). Some entries typed earns 5. |
| `esm` | API | 4 | The `import` condition (or `module`) points at a real ESM file: `.mjs`, or `.js` with `"type": "module"`. |
| `exports-map` | API | 4 | An `exports` map exists, all its targets exist, and each condition object lists `types` first and `default` last. |
| `side-effects` | API | 2 | `"sideEffects": false`, or a list of the files that have side effects. |
| `engines` | API | 1 | `engines.node` is a valid range. |
| `lean-install` | Health | 6 | Counts dependencies, optional dependencies and required peers: 0 earns 6, ≤2 earns 5, ≤5 earns 4, ≤10 earns 3, ≤20 earns 2, ≤40 earns 1. |
| `no-install-scripts` | Health | 5 | No `preinstall`, `install` or `postinstall` script, and no root `binding.gyp`. |

## Swift only

| id | Cat | Pts | Earns points when |
|----|-----|-----|-------------------|
| `public-api` | API | 4 | At least one `public` or `open` declaration outside tests. |
| `platforms` | API | 5 | `Package.swift` declares platforms with minimum versions. |
| `swift-6` | API | 6 | `swift-tools-version` 5.9+ earns 3. Swift 6 language mode on every target (tools 6.0, no `.v5` mode) earns 6. |
| `versioned-deps` | API | 5 | Every dependency uses `from:`, `exact:` or a range, never a branch, revision or local path. |
| `no-unsafe-flags` | API | 4 | No `unsafeFlags` in target settings. |
| `has-tests` | API | 6 | A test target contains an `XCTestCase` subclass or a `@Test` function. |
| `lean-deps` | Health | 7 | 0 dependencies earns 7, 1 earns 6, 2 earns 5, ≤4 earns 4, ≤6 earns 3, ≤10 earns 2, ≤20 earns 1. |

Nothing else is scored, including tests for JS, a `homepage`, vulnerability counts, release cadence or author profiles.
