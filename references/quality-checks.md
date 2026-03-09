# LPM Quality Checks Reference

LPM supports three ecosystems with different check sets. The total is always 100 points, but the checks and categories differ.

| Ecosystem | Checks | Detected by |
|-----------|--------|-------------|
| **JavaScript** | 26 checks | `package.json` present |
| **Swift** | 23 checks | `Package.swift` present |
| **XCFramework** | 19 checks | `.xcframework` bundle present |

---

## JavaScript Ecosystem (26 checks)

### Documentation (25 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-readme` | Has README | 8 | README.md exists and has >100 characters |
| `readme-install` | README has install section | 3 | README contains "## Install", "## Getting Started", "## Setup", "npm install", "lpm add", or "lpm install" (case-insensitive) |
| `readme-usage` | README has usage examples | 4 | README has "## Usage" or "## Example" section, OR has >= 2 code blocks (``` pairs) |
| `readme-api` | README has API docs | 3 | README has "## API", "## Reference", "## Props", "## Parameters", or "## Options" section (case-insensitive) |
| `has-changelog` | Has CHANGELOG | 4 | Any file with "changelog" in its name (case-insensitive) |
| `has-license` | Has LICENSE file | 3 | File with "license"/"licence" in name exists, OR package.json has `license` field |

### Code Quality (34 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-types` | Has TypeScript types | 8 | package.json has `types` or `typings` field, OR any `.d.ts`/`.d.mts` file exists |
| `intellisense-coverage` | IntelliSense coverage | 4 | Full points if `.d.ts` files exist. **Server-augmented:** server re-scores if JSDoc `@param`/`@returns` annotations found in source (partial credit possible) |
| `esm-exports` | ESM exports | 3 | package.json has `"type": "module"`, `"module"` field, or `"exports"` field |
| `tree-shakable` | Tree-shakable | 3 | package.json has `"sideEffects": false`, OR has both `"type": "module"` and `"exports"` field |
| `no-eval` | No eval/Function() patterns | 3 | **Server-only** — CLI assumes pass. Server strips comments then scans source for `eval(` and `new Function(` |
| `has-engines` | Has "engines" field | 2 | package.json has `engines.node` |
| `has-exports-map` | Has "exports" map | 5 | package.json has `exports` field |
| `small-deps` | Small dependency footprint | 4 | Based on `dependencies` count: 0 = 4pts, 1-3 = 3pts, 4-7 = 2pts, 8-15 = 1pt, 16+ = 0pts |
| `source-maps` | Has source maps | 2 | Any `.js.map` or `.mjs.map` file in the package |

### Testing (11 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-test-files` | Has test files | 7 | Any file matching `.test.`, `.spec.`, `__tests__/`, `test/`, or `tests/` |
| `has-test-script` | Has test script | 4 | package.json has `scripts.test` that is not the default `echo "Error: no test" && exit 1` |

### Package Health (30 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-description` | Has description | 3 | package.json `description` exists and has >10 characters |
| `has-keywords` | Has keywords | 2 | package.json `keywords` is a non-empty array |
| `has-repository` | Has repository URL | 2 | package.json has `repository` (string or object with `url`) |
| `has-homepage` | Has homepage | 2 | package.json has `homepage` field |
| `reasonable-size` | Reasonable bundle size | 3 | Based on unpacked size: <100KB = 3pts, <500KB = 2pts, <1MB = 1pt, 1MB+ = 0pts |
| `no-vulnerabilities` | No known vulnerabilities | 5 | **Server-only** — CLI assumes pass. Server checks vulnerability database |
| `maintenance-health` | Maintenance health | 5 | **Server-only** — CLI assumes pass. Last publish within 90 days |
| `semver-consistency` | SemVer consistency | 4 | CLI checks valid semver format. **Server-augmented:** server also checks version history for wild jumps (>1 major skip) |
| `author-verified` | Author verified | 4 | **Server-only** — CLI assumes pass. Author has linked GitHub or LinkedIn in profile settings |

---

## Swift Ecosystem (23 checks)

Swift packages use `Package.swift` instead of `package.json`. Documentation and health checks are the same; code quality checks are Swift-specific.

### Documentation (25 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-readme` | Has README | 8 | README.md exists and has >100 characters |
| `readme-install` | README has install section | 3 | README contains "## Install", "## Getting Started", "## Setup", "## Requirements", "swift package manager", "Package.swift", ".package(", "lpm add", or "lpm install" (case-insensitive) |
| `readme-usage` | README has usage examples | 4 | README has "## Usage" or "## Example" section, OR has >= 2 code blocks |
| `readme-api` | README has API docs | 3 | README has "## API", "## Reference", "## Props", "## Parameters", or "## Options" section |
| `has-changelog` | Has CHANGELOG | 4 | Any file with "changelog" in its name (case-insensitive) |
| `has-license` | Has LICENSE file | 3 | LICENSE file exists |

### Code Quality (34 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-platforms` | Has platform declarations | 6 | Package.swift declares at least one platform (`.iOS(.v16)`, `.macOS(.v13)`, etc.) |
| `recent-tools-version` | Uses recent swift-tools-version | 5 | Package.swift uses `swift-tools-version: 5.9` or higher |
| `multi-platform` | Supports multiple platforms | 4 | 3+ platforms: 4pts, 2 platforms: 3pts, 1 platform: 2pts |
| `has-public-api` | Has public API surface | 5 | **Server-augmented** — CLI passes if Swift source files exist; server scans for `public` declarations |
| `has-doc-comments` | Has DocC documentation | 9 | **Server-only** — CLI assumes pass. Server scans for `///` doc comments in source |
| `small-deps` | Small dependency footprint | 5 | Based on Package.swift dependencies: 0 = 5pts, 1-2 = 4pts, 3-5 = 3pts, 6-10 = 1pt, 11+ = 0pts |

### Testing (11 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-test-files` | Has test targets | 7 | testTarget in Package.swift, OR files in `Tests/` directory |
| `has-test-script` | Has test configuration | 4 | Test targets defined in Package.swift |

### Package Health (30 points)

Same checks as JavaScript except:

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-description` | Has description | 3 | `description` in package metadata (>10 characters) |
| `has-keywords` | Has keywords | 2 | `keywords` is a non-empty array in package metadata |
| `has-repository` | Has repository URL | 2 | `repository` field in package metadata |
| `has-homepage` | Has homepage | 2 | `homepage` field in package metadata |
| `reasonable-size` | Reasonable bundle size | 3 | Based on unpacked size: <100KB = 3pts, <500KB = 2pts, <1MB = 1pt, 1MB+ = 0pts |
| `no-vulnerabilities` | No known vulnerabilities | 5 | **Server-only** — CLI assumes pass |
| `maintenance-health` | Maintenance health | 5 | **Server-only** — CLI assumes pass |
| `semver-consistency` | SemVer consistency | 4 | CLI checks valid semver format; server checks version history |
| `author-verified` | Author verified | 4 | **Server-only** — CLI assumes pass |

---

## XCFramework Ecosystem (19 checks)

XCFrameworks are pre-compiled binary bundles (`.xcframework`). There is **no Testing category** — the binary is already compiled. Code Quality checks validate the binary's slice coverage and architecture support.

### Documentation (25 points)

Same 6 checks as Swift.

### Code Quality (45 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `xcf-valid-plist` | Valid Info.plist | 10 | Info.plist exists and defines at least one platform slice |
| `xcf-multi-slice` | Supports multiple platform slices | 18 | 4+ platforms: 18pts, 3: 14pts, 2: 9pts, 1: 4pts, 0: 0pts |
| `xcf-size` | Reasonable framework size (binary) | 5 | ≤10MB: 5pts, ≤50MB: 4pts, ≤100MB: 3pts, ≤200MB: 1pt, >200MB: 0pts |
| `xcf-architectures` | Supports arm64 architecture | 12 | arm64 present in any slice's architecture list |

### Testing

None — XCFrameworks are pre-compiled binaries. Testing is done in the source project that produces the framework.

### Package Health (30 points)

Same checks as Swift, but `reasonable-size` uses MB-scale thresholds appropriate for binary frameworks:

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `reasonable-size` | Reasonable framework size | 3 | ≤10MB: 3pts, ≤50MB: 2pts, ≤100MB: 1pt, >100MB: 0pts |
| *(all others)* | Same as Swift | — | Same conditions as Swift health checks |

---

## Server-Only and Server-Augmented Checks

Some checks require server-side computation that the CLI cannot perform locally:

| Check | Ecosystem | CLI Provisional | Server Behavior |
|-------|-----------|-----------------|-----------------|
| `no-eval` | JS | Assumes pass (3pts) | Scans source tarball for `eval(`/`new Function(` |
| `intellisense-coverage` | JS | Passes if `.d.ts` exists, else 0 | Can award partial credit for JSDoc `@param`/`@returns` |
| `has-public-api` | Swift | Passes if Swift source files present | Scans for `public` declarations |
| `has-doc-comments` | Swift | Assumes pass (9pts) | Scans for `///` doc comments |
| `no-vulnerabilities` | All | Assumes pass (5pts) | Checks vulnerability database |
| `maintenance-health` | All | Assumes pass (5pts) | Checks last publish date in database |
| `semver-consistency` | All | Checks valid semver format | Also checks version history for wild jumps |
| `author-verified` | All | Assumes pass (4pts) | Checks profile for linked GitHub/LinkedIn |

The CLI computes an **estimated score** that may differ from the final server score by the points at stake in server-only checks.

---

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
