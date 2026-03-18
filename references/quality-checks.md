# LPM Quality Checks Reference

LPM supports three ecosystems with different check sets. The total is always 100 points, but the checks and categories differ.

| Ecosystem | Checks | Detected by |
|-----------|--------|-------------|
| **JavaScript** | 29 checks | `package.json` present |
| **Swift** | 25 checks | `Package.swift` present |
| **XCFramework** | 21 checks | `.xcframework` bundle present |

---

## JavaScript Ecosystem (29 checks)

### Documentation (22 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-readme` | Has README | 8 | README.md exists and has >100 characters |
| `readme-install` | README has install section | 3 | README contains "## Install", "## Getting Started", "## Setup", "npm install", "lpm add", or "lpm install" (case-insensitive) |
| `readme-usage` | README has usage examples | 3 | README has "## Usage" or "## Example" section, OR has >= 2 code blocks (``` pairs) |
| `readme-api` | README has API docs | 2 | README has "## API", "## Reference", "## Props", "## Parameters", or "## Options" section (case-insensitive) |
| `has-changelog` | Has CHANGELOG | 3 | Any file with "changelog" in its name (case-insensitive) |
| `has-license` | Has LICENSE file | 3 | File with "license"/"licence" in name exists, OR package.json has `license` field |

### Code Quality (29 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-types` | Has TypeScript types | 8 | package.json has `types` or `typings` field, OR any `.d.ts`/`.d.mts` file exists |
| `intellisense-coverage` | IntelliSense coverage | 4 | Full points if `.d.ts` files exist. **Server-augmented:** server re-scores if JSDoc `@param`/`@returns` annotations found in source (partial credit possible) |
| `esm-exports` | ESM exports | 3 | package.json has `"type": "module"`, `"module"` field, or `"exports"` field |
| `tree-shakable` | Tree-shakable | 3 | package.json has `"sideEffects": false`, OR has both `"type": "module"` and `"exports"` field |
| `no-eval` | No eval/Function() patterns | 3 | **Server-only** - CLI assumes pass. Server strips comments then scans source for `eval(` and `new Function(` |
| `has-engines` | Has "engines" field | 1 | package.json has `engines.node` |
| `has-exports-map` | Has "exports" map | 3 | package.json has `exports` field |
| `small-deps` | Small dependency footprint | 3 | Based on `dependencies` count: 0 = 3pts, 1-3 = 2pts, 4-7 = 1pt, 8+ = 0pts |
| `source-maps` | Has source maps | 1 | Any `.js.map` or `.mjs.map` file in the package |

### Testing (11 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-test-files` | Has test files | 7 | **CLI:** Scans local project directory (not tarball) for `.test.`, `.spec.`, `__tests__/`, `test/`, or `tests/` patterns. Falls back to tarball files for backward compat. **Server:** Checks `devDependencies` for known test frameworks (vitest, jest, mocha, ava, tap, cypress, playwright, @testing-library/*, etc.). |
| `has-test-script` | Has test script | 4 | package.json has `scripts.test` that is not the default `echo "Error: no test" && exit 1` |

### Package Health (38 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-description` | Has description | 3 | package.json `description` exists and has >10 characters |
| `has-keywords` | Has keywords | 1 | package.json `keywords` is a non-empty array |
| `has-repository` | Has repository URL | 2 | package.json has `repository` (string or object with `url`) |
| `has-homepage` | Has homepage | 1 | package.json has `homepage` field |
| `reasonable-size` | Reasonable bundle size | 3 | Based on unpacked size: <100KB = 3pts, <500KB = 2pts, <1MB = 1pt, 1MB+ = 0pts |
| `no-vulnerabilities` | No known vulnerabilities | 5 | **Server-only** - CLI assumes pass. Server checks vulnerability database |
| `no-lifecycle-scripts` | No lifecycle scripts | 2 | Passes if package.json has no `preinstall`, `install`, `postinstall`, `preuninstall`, `uninstall`, or `postuninstall` scripts. Lifecycle scripts are a common supply chain attack vector. |
| `maintenance-health` | Maintenance health | 4 | **Server-only** - CLI assumes pass. Last publish within 90 days |
| `semver-consistency` | SemVer consistency | 4 | CLI checks valid semver format. **Server-augmented:** server also checks version history for wild jumps (>1 major skip) |
| `author-verified` | Author verified | 3 | **Server-only** - CLI assumes pass. Author has linked GitHub or LinkedIn in profile settings |
| `has-skills` | Has Agent Skills | 7 | **Server-only** - CLI assumes fail. Package has 1+ approved Agent Skills published in `.lpm/skills/` |
| `skills-comprehensive` | Comprehensive Agent Skills | 3 | **Server-only** - CLI assumes fail. Package has 3+ approved Agent Skills |

---

## Swift Ecosystem (25 checks)

Swift packages use `Package.swift` instead of `package.json`. Documentation and health checks are the same; code quality checks are Swift-specific.

### Documentation (22 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-readme` | Has README | 8 | README.md exists and has >100 characters |
| `readme-install` | README has install section | 3 | README contains "## Install", "## Getting Started", "## Setup", "## Requirements", "swift package manager", "Package.swift", ".package(", "lpm add", or "lpm install" (case-insensitive) |
| `readme-usage` | README has usage examples | 3 | README has "## Usage" or "## Example" section, OR has >= 2 code blocks |
| `readme-api` | README has API docs | 2 | README has "## API", "## Reference", "## Props", "## Parameters", or "## Options" section |
| `has-changelog` | Has CHANGELOG | 3 | Any file with "changelog" in its name (case-insensitive) |
| `has-license` | Has LICENSE file | 3 | LICENSE file exists |

### Code Quality (31 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-platforms` | Has platform declarations | 6 | Package.swift declares at least one platform (`.iOS(.v16)`, `.macOS(.v13)`, etc.) |
| `recent-tools-version` | Uses recent swift-tools-version | 5 | Package.swift uses `swift-tools-version: 5.9` or higher |
| `multi-platform` | Supports multiple platforms | 4 | 3+ platforms: 4pts, 2 platforms: 3pts, 1 platform: 2pts |
| `has-public-api` | Has public API surface | 5 | **Server-augmented** - CLI passes if Swift source files exist; server scans for `public` declarations |
| `has-doc-comments` | Has DocC documentation | 7 | **Server-only** - CLI assumes pass. Server scans for `///` doc comments in source |
| `small-deps` | Small dependency footprint | 4 | Based on Package.swift dependencies: 0 = 4pts, 1-2 = 3pts, 3-5 = 2pts, 6-10 = 1pt, 11+ = 0pts |

### Testing (11 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-test-files` | Has test targets | 7 | testTarget in Package.swift, OR files in `Tests/` directory |
| `has-test-script` | Has test configuration | 4 | Test targets defined in Package.swift |

### Package Health (36 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `has-description` | Has description | 3 | `description` in package metadata (>10 characters) |
| `has-keywords` | Has keywords | 1 | `keywords` is a non-empty array in package metadata |
| `has-repository` | Has repository URL | 2 | `repository` field in package metadata |
| `has-homepage` | Has homepage | 1 | `homepage` field in package metadata |
| `reasonable-size` | Reasonable bundle size | 3 | Based on unpacked size: <100KB = 3pts, <500KB = 2pts, <1MB = 1pt, 1MB+ = 0pts |
| `no-vulnerabilities` | No known vulnerabilities | 5 | **Server-only** - CLI assumes pass |
| `maintenance-health` | Maintenance health | 4 | **Server-only** - CLI assumes pass |
| `semver-consistency` | SemVer consistency | 4 | CLI checks valid semver format; server checks version history |
| `author-verified` | Author verified | 3 | **Server-only** - CLI assumes pass |
| `has-skills` | Has Agent Skills | 7 | **Server-only** - CLI assumes fail. 1+ approved Agent Skills |
| `skills-comprehensive` | Comprehensive Agent Skills | 3 | **Server-only** - CLI assumes fail. 3+ approved Agent Skills |

---

## XCFramework Ecosystem (21 checks)

XCFrameworks are pre-compiled binary bundles (`.xcframework`). There is **no Testing category** - the binary is already compiled. Code Quality checks validate the binary's slice coverage and architecture support.

### Documentation (22 points)

Same 6 checks as Swift.

### Code Quality (40 points)

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `xcf-valid-plist` | Valid Info.plist | 10 | Info.plist exists and defines at least one platform slice |
| `xcf-multi-slice` | Supports multiple platform slices | 15 | 4+ platforms: 15pts, 3: 11pts, 2: 9pts, 1: 4pts, 0: 0pts |
| `xcf-size` | Reasonable framework size (binary) | 5 | <=10MB: 5pts, <=50MB: 4pts, <=100MB: 3pts, <=200MB: 1pt, >200MB: 0pts |
| `xcf-architectures` | Supports arm64 architecture | 10 | arm64 present in any slice's architecture list |

### Testing

None - XCFrameworks are pre-compiled binaries. Testing is done in the source project that produces the framework.

### Package Health (38 points)

Same checks as Swift with these adjustments:

| ID | Label | Points | Pass Condition |
|----|-------|--------|----------------|
| `reasonable-size` | Reasonable framework size | 3 | <=10MB: 3pts, <=50MB: 2pts, <=100MB: 1pt, >100MB: 0pts |
| `maintenance-health` | Maintenance health | 5 | **Server-only** - CLI assumes pass (XCF keeps original value) |
| `author-verified` | Author verified | 4 | **Server-only** - CLI assumes pass (XCF keeps original value) |
| `has-skills` | Has Agent Skills | 7 | **Server-only** - 1+ approved Agent Skills |
| `skills-comprehensive` | Comprehensive Agent Skills | 3 | **Server-only** - 3+ approved Agent Skills |
| *(all others)* | Same as Swift | - | Same conditions as Swift health checks |

---

## Server as Source of Truth

The server independently verifies **ALL** quality checks using data it has: tarball contents, package.json, README, extracted source text, and database records. CLI-reported check results are used only for structure (IDs, categories, labels, maxPoints) — results are always overridden by server verification.

This means:
- Authors cannot inflate scores by manipulating CLI output
- The CLI computes an **estimated score** that may differ from the final server score
- When server data is unavailable (e.g., old CLI versions), the CLI value is preserved as a fallback

### Server verification by data source

| Data Source | Checks Verified |
|-------------|----------------|
| **README content** | `has-readme`, `readme-install`, `readme-usage`, `readme-api` |
| **Tarball file listing** | `has-changelog`, `source-maps`, `has-types` (`.d.ts`), `has-license` (LICENSE file) |
| **package.json fields** | `has-description`, `has-keywords`, `has-repository`, `has-homepage`, `has-test-script`, `has-engines`, `has-exports-map`, `esm-exports`, `tree-shakable`, `small-deps`, `has-test-files` (via devDependencies), `no-lifecycle-scripts` |
| **Extracted source text** | `no-eval` (JS), `intellisense-coverage` (JSDoc fallback), `has-public-api` (Swift), `has-doc-comments` (Swift) |
| **Tarball size** | `reasonable-size` |
| **Database/APIs** | `no-vulnerabilities`, `maintenance-health`, `semver-consistency`, `author-verified`, `has-skills`, `skills-comprehensive` |

### Server-only checks (no CLI equivalent)

These checks can only be computed server-side. The CLI sends a provisional value:

| Check | Ecosystem | CLI Provisional | Server Behavior |
|-------|-----------|-----------------|-----------------|
| `no-eval` | JS | Assumes pass (3pts) | Scans source tarball for `eval(`/`new Function(` |
| `intellisense-coverage` | JS | Passes if `.d.ts` exists, else 0 | Checks types field + `.d.ts` files + JSDoc annotations |
| `has-public-api` | Swift | Passes if Swift source files present | Scans for `public` declarations |
| `has-doc-comments` | Swift | Assumes pass (7pts) | Scans for `///` doc comments |
| `no-vulnerabilities` | All | Assumes pass (5pts) | Checks vulnerability database |
| `maintenance-health` | JS/Swift | Assumes pass (4pts) | Checks last publish date in database |
| `maintenance-health` | XCF | Assumes pass (5pts) | XCF keeps original 5-point value |
| `semver-consistency` | All | Checks valid semver format | Also checks version history for wild jumps |
| `author-verified` | JS/Swift | Assumes pass (3pts) | Checks profile for linked GitHub/LinkedIn |
| `author-verified` | XCF | Assumes pass (4pts) | XCF keeps original 4-point value |
| `has-skills` | All | Assumes fail (0pts) | Checks if 1+ approved skills exist for version |
| `skills-comprehensive` | All | Assumes fail (0pts) | Checks if 3+ approved skills exist for version |

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
