# LPM Package Quality Improvement

Score a JS or Swift package against the LPM quality model, explain where it lost points, and offer fixes. Check IDs, points, rules and tiers live in [quality-checks.md](../references/quality-checks.md). Don't restate them from memory.

## Workflow

1. **Ecosystem:** `Package.swift` means Swift. Otherwise `package.json` means JS. If neither exists, ask.
2. **Local score:** run `lpm publish --check --json` in the package directory. It needs no auth, makes no registry requests, runs no lifecycle scripts and uploads nothing. Read `quality`:
   - `checks[]` has `id`, `points`, `maxPoints`, `status` (`passed`, `partial`, `failed`, `pending` or `after_publish`), `detail` and `tip`.
   - `archivePoints` / `archiveMaxPoints`, `registryMaxPoints`, and `preflightScore`. `preflightScore` is a % of archive points, not the /100 score.
   - `tarball_files` lists every file that would ship, with its size.
   - If the CLI isn't available, evaluate by hand against quality-checks.md.
3. **Published package:** `lpm quality owner.package-name --json` gives the Registry report, including the 5 Registry checks. Public reports are anonymous. Private ones need publisher access (`lpm login` or `LPM_TOKEN`). `status: pending` means a Registry check is still running.
4. **Deep analysis** (below). Then **report**, then **offer fixes**, and apply only the fixes the user approves.

## Deep Analysis

Go past the check output by reading the actual source.

- **README against exports:** follow the entry point and its re-exports to build the export map. Flag exports the README doesn't document and examples that use renamed or removed APIs. Examples that import names the package doesn't export lose `readme-example` points. Install-block rules are under `readme-install` in quality-checks.md.
- **Doc coverage (`api-docs`):** count exports with a doc comment of ≥20 characters. For JS that means the `.d.ts` files, or the ESM entry when no `.d.ts` ships. For Swift it means `///` on public/open declarations. Name the undocumented ones.
- **Changelog:** the current `package.json` version needs its own entry. Flag mismatches and placeholder text.
- **Entry points:** every path in `main`/`module`/`types`/`typings`/`bin`/`exports` must exist. A `types` path pointing at a `.d.ts` that never ships is a common failure.
- **Dependencies:**
  - Build, test and lint tools or `@types/*` listed in `dependencies` cost `lean-install` points. Move them to `devDependencies`.
  - Suggest built-ins for trivial dependencies, such as `fetch`, `crypto.randomUUID()`, or `fs.rm`/`fs.mkdir` with `{ recursive: true }`.
  - Flag deprecated dependencies, and dependencies that are copyleft (GPL/AGPL/SSPL), `UNLICENSED` or have no license.
- **Archive contents:** look through `tarball_files` for fixtures, docs builds, editor files and anything `clean-archive` rejects. Fix it with a `files` whitelist or `.npmignore`.
- **License against distribution:** LPM enforces access and purchases. It doesn't judge whether license terms are compatible, so this is advice:
  - **Marketplace with a permissive license:** see [Distribution → License](./distribution.md#license). For proprietary terms, start from [commercial-license-template.md](../references/commercial-license-template.md); its How to Use covers the `license` field.
  - **Commercial License plans** (buyer bundling rights): see [Pricing](./pricing.md).
  - **Free/Pool:** any license works, so pick terms that match how users will redistribute.

## Report

```
## Quality: {score}/100 ({tier})
   (unpublished: "Archive {archivePoints}/{archiveMaxPoints} ({preflightScore}%); {registryMaxPoints} pts decided by the Registry after publish")

| Category | Points | Max |      ← maxes from quality-checks.md for this ecosystem
Checks: ✓ / ◐ / ✗  id: detail
Findings: grouped, with file:line
Quick wins: top 3–5 by points gained for the effort
Projected: {now} → {after fixes}
```

## Fixes

| Lost points on | Fix |
|----------------|-----|
| `description` / `keywords` / `repository` | A description taken from the README or source. Keywords taken from the name and purpose. `repository` from `git remote get-url origin`. |
| `engines` | A valid `engines.node` range matching the lowest Node version the code supports. |
| `entry-points` / `exports-map` / `esm` / `side-effects` / `types` | See **JS types and exports** below. |
| `readme` / `readme-install` / `readme-example` | Generate only the missing sections from real exports. See **README** below. |
| `api-docs` | Add doc comments of ≥20 characters to undocumented exports. |
| `changelog` | Generate it from git history. See **Changelog** below. |
| `license` / `license-file` | Add a LICENSE file and an SPDX id, or `SEE LICENSE IN LICENSE` for proprietary terms. |
| `no-install-scripts` | Ship prebuilt files. Remove `preinstall`/`install`/`postinstall` and any root `binding.gyp`. |
| `clean-archive` | Use a `files` whitelist or `.npmignore`, and remove large build or fixture output. |
| `lean-install` / `lean-deps` | Remove dependencies or move them to `devDependencies`. |
| `trusted-publishing` / `signed-provenance` / `repository-verified` | Publish from CI with a Trusted Publisher and `--provenance`. See [Deploy](./deploy.md). |
| `verified-domain` | The owner adds a DNS TXT record in their LPM.dev settings. |
| `agent-skills` | Follow the [Skills workflow](./skills.md). |
| Swift `platforms` / `swift-6` / `versioned-deps` / `no-unsafe-flags` / `has-tests` / `public-api` | Add `platforms:` with minimum versions. Use `// swift-tools-version: 6.0` without `.v5` modes. Use `from:`/`exact:` requirements. Remove `unsafeFlags`. Add an XCTest or Swift Testing target. Mark the consumer API `public`. |

Tests (for JS), `homepage` and CI files earn no points on their own. Offer them as plain improvements, never as score fixes.

### README

Read the actual exports and never guess signatures. The README needs enough prose (threshold in quality-checks.md) and these code blocks:

- An install block naming the exact package, using a command `readme-install` accepts.
- A ```` ```js ````/```` ```ts ```` (or ```` ```swift ````) example that imports only names the package really exports.

If a README already exists, offer each missing section separately and don't overwrite what's there.

### Changelog

Build it from `git log` and the semver tags. Never invent entries. Use Keep a Changelog sections, and map conventional-commit prefixes: `feat` goes to Added, `fix` to Fixed, and everything else to Changed. Skip `chore`, `ci` and `build`. The current version must have an entry. In an existing changelog, only prepend what's missing.

### JS types and exports

`types` wants declarations for every exports entry under both `import` and `require`. `api-docs` wants doc comments. Rank the options:

1. **TypeScript** with `declaration: true` generates the `.d.ts` files. Point each `types` condition at them.
2. **Hand-written `.d.ts`** next to each entry (`.d.mts` for `.mjs`, `.d.cts` for `.cjs`), with doc comments. No build step.
3. **JSDoc in the ESM entry only.** This earns `api-docs` when no `.d.ts` ships, but not `types`.

```json
"type": "module",
"sideEffects": false,
"exports": { ".": { "types": "./dist/index.d.ts", "default": "./dist/index.js" } }
```

Keep `types` first and `default` last in each condition object, and make sure every target exists. Re-run `lpm publish --check` to confirm.
