# LPM Package Quality Improvement

Analyze the current package directory and provide a quality score with actionable improvements. This runs the same checks as `lpm publish --check` plus deeper source code analysis.

## Workflow

1. **Detect ecosystem** — Determine if the package is JavaScript, Swift, or XCFramework
2. **Read package files** to gather context
3. **Run quality checks** (deterministic, same as CLI)
4. **Deep analysis** of source code beyond the CLI checks
5. **Generate report** with score, findings, and actionable fixes
6. **Offer to fix** issues that can be automated

## Step 1: Gather Context

**First, detect the ecosystem:**
- `Package.swift` present → **Swift** ecosystem (23 checks)
- `.xcframework` bundle present → **XCFramework** ecosystem (19 checks, no Testing category)
- `package.json` present → **JavaScript** ecosystem (26 checks)
- If none found, ask the user what kind of package this is

**Read these files from the current directory (skip missing files silently):**

For JavaScript packages:
- `package.json` (required — abort if missing)
- `README.md` or `readme.md`
- `CHANGELOG.md` or `changelog.md`
- `LICENSE` or `LICENSE.md`
- `lpm.config.json` (source package config)
- `.github/workflows/*.yml` (CI config)

For Swift/XCFramework packages:
- `Package.swift` (Swift packages)
- `README.md` or `readme.md`
- `CHANGELOG.md` or `changelog.md`
- `LICENSE` or `LICENSE.md`
- `Sources/` directory listing (Swift packages)
- `Tests/` directory listing (Swift packages)

Also scan the file tree:
- List all files to check for test files, `.d.ts` files, CI configs
- Count source files by ecosystem (`.js`/`.ts`/`.jsx`/`.tsx` for JS; `.swift` for Swift)
- Read the main entry point file

## Step 2: Run Quality Checks

Run each check and record pass/fail. See [references/quality-checks.md](../references/quality-checks.md) for the full check list per ecosystem.

**JavaScript scoring:**
- Documentation: 25 points (6 checks)
- Code Quality: 34 points (9 checks)
- Testing: 11 points (2 checks)
- Package Health: 30 points (9 checks)
- Total: 100 points

**Swift scoring:**
- Documentation: 25 points (6 checks)
- Code Quality: 34 points (6 checks)
- Testing: 11 points (2 checks)
- Package Health: 30 points (9 checks)
- Total: 100 points

**XCFramework scoring:**
- Documentation: 25 points (6 checks)
- Code Quality: 45 points (4 checks)
- Testing: None (pre-compiled binary)
- Package Health: 30 points (9 checks)
- Total: 100 points

**Note:** Several checks are server-only and can only be approximated locally. The CLI shows an estimated score. The server computes the final score after merging server-only checks.

**Tiers:**
- 90+: Excellent
- 70-89: Good
- 50-69: Fair
- 0-49: Needs Work

## Step 3: Deep Analysis

Go beyond the CLI checks by reading actual source code:

### README Completeness

- Read the main entry point and follow re-exports to build a full export map
- For each export, capture: name, kind (function/class/component/constant), parameters, return type, JSDoc description
- Compare exports against README content — flag every undocumented export by name
- Check if code examples in README actually match the current API (function names, argument order, return types)
- Verify install command uses the correct LPM package name format (`@lpm.dev/owner.package-name` or `lpm install`/`lpm add`)
- Check for stale examples referencing removed or renamed exports
- Flag README sections that reference functions no longer in the public API

### Changelog Quality

- Check if CHANGELOG.md follows [Keep a Changelog](https://keepachangelog.com) format
- Verify version entries match versions in git tags (if git repo)
- Flag if latest version in CHANGELOG doesn't match `package.json` version
- Check for empty sections or placeholder text
- If no changelog exists, analyze git history to determine how much content is available:
  - Count commits since last tag or initial commit
  - Check if conventional commits format is used (`feat:`, `fix:`, `chore:`, etc.)
  - Estimate changelog richness for the auto-fix step

### Test Coverage Gaps

- Read test files and identify which source files have corresponding tests
- Flag source files with no test file (e.g., `src/parser.js` exists but no `src/parser.test.js`)

### Error Handling Audit

- Scan source files for empty catch blocks, swallowed errors
- Flag `catch (e) {}` or `catch (e) { /* ignore */ }` patterns
- Check if error messages include enough context

### Bundle Size

- Flag large files (>50KB) that might be unnecessary
- Check for files that should be in `.npmignore` or excluded via `files` field (test fixtures, docs, config files, etc.)
- Flag binary files or images in the package
- If `files` field is missing, estimate published size vs. actual directory size and report the difference

### Dependency Audit

**Misplaced dependencies:**
- Flag test/build tools in `dependencies` that belong in `devDependencies`:
  - Test: `vitest`, `jest`, `mocha`, `chai`, `sinon`, `@testing-library/*`, `nyc`, `c8`
  - Build: `typescript`, `webpack`, `vite`, `rollup`, `esbuild`, `tsup`, `unbuild`, `@swc/*`
  - Lint: `eslint`, `prettier`, `biome`, `@biomejs/*`, `stylelint`
  - Types: `@types/*` (unless the package re-exports these types)

**Lighter alternatives:**

| Heavy | Lighter Alternative | Size Reduction |
|-------|---------------------|----------------|
| `moment` | `dayjs` or `date-fns` | ~95% smaller |
| `lodash` | `lodash-es` or native methods | ~70% smaller |
| `axios` | native `fetch` | Remove dep entirely |
| `node-fetch` | native `fetch` (Node 18+) | Remove dep entirely |
| `uuid` | `crypto.randomUUID()` (Node 19+) | Remove dep entirely |
| `chalk` | `picocolors` or `yoctocolors` | ~95% smaller |
| `commander` | `citty` or `cleye` | ~80% smaller |
| `inquirer` | `@clack/prompts` | ~70% smaller |
| `glob` | `tinyglobby` or `fast-glob` | ~80% smaller |
| `request` | native `fetch` | Deprecated, remove |
| `form-data` | native `FormData` (Node 18+) | Remove dep entirely |
| `string-width` | `get-east-asian-width` | ~90% smaller |
| `rimraf` | `fs.rm(path, { recursive: true })` | Remove dep entirely |
| `mkdirp` | `fs.mkdir(path, { recursive: true })` | Remove dep entirely |

**License compatibility:**
- Read `license` field from each dependency's `package.json` (in `node_modules/`)
- Flag dependencies with restrictive licenses that may conflict with distribution:
  - `GPL-2.0`, `GPL-3.0`, `AGPL-3.0` — copyleft, requires derivative works to use same license
  - `SSPL-1.0` — restrictive server-side license
  - `BSL-1.1` — Business Source License
  - `UNLICENSED` or missing license — cannot legally redistribute
- Note: This matters especially for LPM packages using `pool` or `marketplace` distribution where the code is redistributed

**Deprecated packages:**
- Flag packages that are officially deprecated (check for `deprecated` field in their `package.json`)

## Step 4: Generate Report

Format the report as a structured summary:

```
## Quality Score: {score}/100 ({tier})

### Category Breakdown
| Category | Score | Max |
|----------|-------|-----|
| Documentation | {n} | 25 |
| Code Quality | {n} | {34 for JS/Swift, 45 for XCF} |
| Testing | {n} | {11 for JS/Swift, N/A for XCF} |
| Package Health | {n} | 30 |

### Checks
{For each check: checkmark or x, label, detail}

### Deep Analysis Findings
{Grouped by category, with specific file paths and line numbers}

### Quick Wins
{Top 3-5 easiest improvements ranked by points gained}

### Estimated Score After Fixes
{Current score} -> {projected score} ({tier change if applicable})
```

## Step 5: Offer Fixes

For each failed check, offer to generate or fix the issue. Only apply fixes the user approves.

### Package.json Fixes

| Failed Check | Automated Fix |
|--------------|---------------|
| No description | Suggest description from README first paragraph or source analysis |
| No keywords | Suggest keywords from package name, description, and source analysis |
| No repository | Add `"repository"` field if `.git/config` has a remote URL |
| No homepage | Add `"homepage"` from repository URL or `https://lpm.dev/owner.package-name` |
| No engines field | Add `"engines": { "node": ">=18" }` |
| No exports map | Suggest `"exports"` field based on package entry points |
| No test script | Add `"test": "vitest"` (or match existing runner) |
| Misplaced deps | Move flagged packages from `dependencies` to `devDependencies` |

### README Generation

When README is missing or incomplete, generate a comprehensive README from source code analysis:

1. **Read all exports** — Follow the entry point, resolve re-exports, capture every public function/class/component
2. **Extract signatures** — Parameter names, types (from JSDoc `@param`/TypeScript), return types, default values
3. **Build sections:**

```markdown
# package-name

{description from package.json}

## Installation

\`\`\`bash
lpm install @lpm.dev/owner.package-name
\`\`\`

## Usage

{For the top 3-5 most important exports, write a realistic usage example with import statement and function call. Infer typical usage from parameter names and types.}

## API Reference

### functionName(param1, param2?)

{JSDoc description or inferred description}

**Parameters:**
| Name | Type | Default | Description |
|------|------|---------|-------------|
| param1 | string | — | {from @param or inferred} |
| param2 | object | {} | {from @param or inferred} |

**Returns:** {return type} — {description}

{Repeat for each export}

## License

{from package.json license field}
```

4. **For existing READMEs** — Only generate missing sections, don't overwrite existing content. Offer each section individually.

### Changelog Generation

When CHANGELOG is missing, generate from git history:

1. **Read git log** — `git log --oneline --decorate` to get commits and tags
2. **Group by version** — Use git tags matching semver patterns (`v1.0.0`, `1.0.0`) as version boundaries
3. **Categorize commits** — Parse conventional commit prefixes or infer from message:

| Prefix | Category |
|--------|----------|
| `feat:`, `feature:` | Added |
| `fix:`, `bugfix:` | Fixed |
| `docs:` | Documentation |
| `refactor:`, `perf:` | Changed |
| `BREAKING CHANGE:`, `!:` | Breaking |
| `chore:`, `ci:`, `build:` | Maintenance (omit from changelog) |
| No prefix | Changed (default) |

4. **Format as Keep a Changelog:**

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [1.2.0] - 2026-02-14

### Added
- New `parseConfig` function for reading lpm.config.json files

### Fixed
- Fixed path traversal vulnerability in tarball extraction

## [1.1.0] - 2026-01-20

### Added
- ...
```

5. **For existing CHANGELOGs** — Only prepend new entries since the latest documented version

### Other Fixes

| Failed Check | Automated Fix |
|--------------|---------------|
| No LICENSE | Generate MIT license with author from package.json |
| No TypeScript types | See "Improving Type Coverage" below for 3 options |
| No test files | Generate test template using vitest (match existing test runner if possible) |
| No CI config | Generate `.github/workflows/ci.yml` template |

### Improving Type Coverage

When `has-types` (8 pts) or `intellisense-coverage` (4 pts) checks fail, offer three approaches ranked by benefit:

**Path 1 — Add TypeScript (Best, +12 points)**
- Add `typescript` to devDependencies
- Create `tsconfig.json` with `declaration: true`:
  ```json
  {
    "compilerOptions": {
      "target": "ES2022",
      "module": "Node16",
      "moduleResolution": "Node16",
      "declaration": true,
      "outDir": "./dist",
      "strict": true,
      "esModuleInterop": true,
      "skipLibCheck": true
    },
    "include": ["src"]
  }
  ```
- Rename `.js` → `.ts` (or start with just the entry point)
- Add `"build": "tsc"` script, update `exports` and `types` to `./dist/`
- Result: Full type safety, auto-generated `.d.ts`, IntelliSense everywhere

**Path 2 — Hand-written `.d.ts` (Good, +8 points)**
- Create a `.d.ts` file next to the entry point
- Document exported functions with parameter and return types:
  ```typescript
  export declare function parseConfig(filePath: string, defaults?: object): Promise<object>;
  export declare const VERSION: string;
  ```
- Add `"types": "./src/index.d.ts"` to package.json
- Result: Consumers get IntelliSense, no build step needed

**Path 3 — Add JSDoc annotations (OK, +4 points)**
- Add `@param` and `@returns` tags to exported functions:
  ```javascript
  /**
   * Parse a configuration file.
   * @param {string} filePath - Path to the config file
   * @param {object} [defaults] - Default values
   * @returns {Promise<object>} Parsed configuration
   */
  export async function parseConfig(filePath, defaults = {}) {
  ```
- No build step, no new files, no TypeScript dependency
- Editors pick up JSDoc for IntelliSense automatically
- Result: Better IntelliSense, earns `intellisense-coverage` points

Better types mean better IntelliSense for everyone who uses your package, and a higher quality score on LPM.

## Important Notes

- Never modify files without user approval
- Show the report first, then ask which fixes to apply
- For README generation, read actual source code to ensure accuracy — never guess at function signatures
- For test generation, match the existing test framework (vitest, jest, etc.)
- For changelog generation, read actual git history — never fabricate commit messages
- The quality score shown here matches what `lpm publish --check` would report
- Server-only and server-augmented checks (no-eval, intellisense-coverage, has-public-api, has-doc-comments, no-vulnerabilities, maintenance-health, semver-consistency, author-verified) are approximated locally — the server re-scores these after publish using the actual tarball and database
- If `lpm quality` is available, use `lpm quality owner.package-name --json` to fetch the real server-side score (100-point scale with all server checks resolved) instead of the estimated local score (which caps at ~79 points due to unresolvable server-only checks)
