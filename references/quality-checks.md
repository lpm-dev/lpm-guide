# LPM Quality Checks Reference

These are the 26 deterministic quality checks that `lpm publish` runs. The skill should replicate these exactly to produce a matching score.

## Documentation (25 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-readme` | Has README | 8 | README.md exists and has >100 characters |
| `readme-install` | README has install section | 3 | README contains "## Install", "## Getting Started", "## Setup", "npm install", "lpm add", or "lpm install" (case-insensitive) |
| `readme-usage` | README has usage examples | 4 | README has "## Usage" or "## Example" section, OR has >= 2 code blocks (``` pairs) |
| `readme-api` | README has API docs | 3 | README has "## API", "## Reference", "## Props", "## Parameters", or "## Options" section (case-insensitive) |
| `has-changelog` | Has CHANGELOG | 4 | Any file with "changelog" in its name (case-insensitive) |
| `has-license` | Has LICENSE file | 3 | File with "license"/"licence" in name exists, OR package.json has `license` field |

## Code Quality (34 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-types` | Has TypeScript types | 8 | package.json has `types` or `typings` field, OR any `.d.ts`/`.d.mts` file exists |
| `intellisense-coverage` | Intellisense coverage | 4 | Full points if `.d.ts` files exist. Server can upgrade if JSDoc `@param`/`@returns` found in source |
| `esm-exports` | ESM exports | 3 | package.json has `"type": "module"`, `"module"` field, or `"exports"` field |
| `tree-shakable` | Tree-shakable | 3 | package.json has `"sideEffects": false`, OR has both `"type": "module"` and `"exports"` field |
| `no-eval` | No eval/Function() patterns | 3 | **Server-only** — CLI assumes pass. Server scans source for `eval(` and `new Function(` |
| `has-engines` | Has "engines" field | 2 | package.json has `engines.node` |
| `has-exports-map` | Has "exports" map | 5 | package.json has `exports` field |
| `small-deps` | Small dependency footprint | 4 | Based on dependency count: 0 deps = 4pts, 1-3 = 3pts, 4-7 = 2pts, 8-15 = 1pt, 16+ = 0pts |
| `source-maps` | Has source maps | 2 | Any `.js.map` or `.mjs.map` file in the package |

## Testing (11 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-test-files` | Has test files | 7 | Any file matching `.test.`, `.spec.`, `__tests__/`, `test/`, or `tests/` |
| `has-test-script` | Has test script | 4 | package.json has `scripts.test` that is not the default `echo "Error: no test" && exit 1` |

## Package Health (30 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-description` | Has description | 3 | package.json `description` exists and has >10 characters |
| `has-keywords` | Has keywords | 2 | package.json `keywords` is a non-empty array |
| `has-repository` | Has repository URL | 2 | package.json has `repository` (string or object with `url`) |
| `has-homepage` | Has homepage | 2 | package.json has `homepage` field |
| `reasonable-size` | Reasonable bundle size | 3 | Based on unpacked size: <100KB = 3pts, <500KB = 2pts, <1MB = 1pt, 1MB+ = 0pts |
| `no-vulnerabilities` | No known vulnerabilities | 5 | **Server-only** — CLI assumes pass. Server checks vulnerability database |
| `maintenance-health` | Maintenance health | 5 | **Server-only** — CLI assumes pass. Last publish within 90 days |
| `semver-consistency` | SemVer consistency | 4 | CLI checks valid semver format. **Server** also checks version history for wild jumps (>1 major skip) |
| `author-verified` | Author verified | 4 | **Server-only** — CLI assumes pass. Author has linked GitHub or LinkedIn in profile settings |

## Server-Only Checks

5 checks (21 points) can only be accurately computed server-side:

| Check | Reason |
|-------|--------|
| `no-eval` | Requires scanning actual source code from tarball |
| `no-vulnerabilities` | Requires vulnerability database access |
| `maintenance-health` | Requires publish history from database |
| `semver-consistency` | Full check requires version history from database |
| `author-verified` | Requires user profile data from database |

The CLI computes an **estimated score** out of 79 possible points (100 minus 21 server-only points). The server merges CLI results with server-only checks for the final score.

## Source Package Info (not scored, informational)

If `lpm.config.json` exists, report:
- Has config: true
- Has defaults: whether `defaultConfig` is set
- Option count: number of keys in `configSchema`
- Uses conditional includes: whether any file rule has `include: "when"` with a condition

## Tier Calculation

| Score Range | Tier |
|-------------|------|
| 90-100 | Excellent |
| 70-89 | Good |
| 50-69 | Fair |
| 0-49 | Needs Work |
