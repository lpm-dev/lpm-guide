# LPM Package Review

Help consumers evaluate an LPM package before adding it as a dependency. Covers quality score, security, maintenance health, API design, bundle impact, and alternatives — so the user can make an informed decision.

## Context

LPM packages use the format `@lpm.dev/owner.package-name`. They can be:
- **Pool** — Included in the $12/mo Pool subscription
- **Marketplace** — Purchased individually with per-seat or per-org licensing
- **Private** — Not available to external users

## Workflow

1. **Identify the package** — Get the package name from the user
2. **Gather package data** — Read installed source or fetch info via CLI
3. **Analyze multiple dimensions** — Quality, security, maintenance, API, bundle size
4. **Present a review** — Structured assessment with a clear recommendation
5. **Compare alternatives** — If the user is deciding between packages

## Step 1: Identify the Package

The user may provide:
- A full LPM name: `@lpm.dev/owner.package-name`
- A short reference: `owner.package-name`
- A package they're already looking at in their `package.json`

If the package is already installed (in `node_modules/@lpm.dev/`), read it directly. Otherwise, use `lpm info @lpm.dev/owner.package-name` to fetch metadata.

## Step 2: Gather Package Data

### From Installed Package (node_modules)

Read from `node_modules/@lpm.dev/owner.package-name/`:
- `package.json` — All metadata
- `README.md` — Documentation quality
- Source files — Code quality, patterns
- `CHANGELOG.md` — Update frequency
- `LICENSE` — License terms
- `.d.ts` files — Type quality

### From CLI Info

Run `lpm info @lpm.dev/owner.package-name --json` to get:
- Version history and dates
- Download counts
- Quality score
- Description and keywords
- Dependencies

### From README/Source Analysis

If the user has access to the source (installed or browsing on lpm.dev):
- Read the main entry point and key exports
- Assess code patterns and conventions
- Check error handling approach
- Evaluate TypeScript support quality

## Step 3: Multi-Dimensional Analysis

### Quality Score

Report the package's quality score from the 27-check system:

| Category | Score | Assessment |
|----------|-------|------------|
| Documentation | /25 | {good/fair/poor} |
| Code Quality | /30 | {good/fair/poor} |
| Testing | /15 | {good/fair/poor} |
| Package Health | /30 | {good/fair/poor} |
| **Total** | **/100** | **{tier}** |

If `lpm quality` is available, use `lpm quality owner.package-name --json` to fetch the full server-side quality report with all 27 checks resolved (including server-only checks like no-vulnerabilities, maintenance-health, author-verified). If `lpm quality` is not available, fall back to local analysis or `lpm info`.

### Security Assessment

Check for red flags:

| Check | What to Look For |
|-------|-----------------|
| **eval/Function()** | Scan source for `eval(`, `new Function(`, `Function(` |
| **Network calls** | Scan for `fetch(`, `http.request`, `XMLHttpRequest` in non-networking packages |
| **File system access** | Scan for `fs.write`, `fs.unlink`, `fs.rm` in packages that shouldn't need it |
| **Child process** | Scan for `exec(`, `spawn(`, `execSync(` — potential command injection |
| **Obfuscated code** | Minified source without source maps in a package that should ship readable code |
| **Install scripts** | Check for `preinstall`, `postinstall`, `prepare` scripts — common malware vector |
| **License** | Verify license is compatible with the user's project |

Severity levels:
- **Critical**: eval, install scripts executing arbitrary code
- **Warning**: Unexpected network/fs access, obfuscated code
- **Info**: Minor concerns, stylistic issues

### Maintenance Health

| Signal | Good | Concerning |
|--------|------|------------|
| **Last publish** | Within 90 days | Over 6 months ago |
| **Version count** | 3+ versions | Only 1 version |
| **Changelog** | Exists and up to date | Missing or stale |
| **Open issues** | Responded to | Ignored for months |
| **Author verified** | Linked GitHub/LinkedIn | No verification |

### API Design Review

Assess the public API:

- **Consistency**: Do functions follow a consistent naming pattern?
- **Documentation**: Are parameters and return types documented (JSDoc or TypeScript)?
- **Error handling**: Does the API throw meaningful errors or silently fail?
- **Defaults**: Are sensible defaults provided for optional parameters?
- **Breaking changes**: Does the version history show frequent breaking changes?

### Bundle Impact

Estimate the impact of adding this dependency:

| Metric | Value | Assessment |
|--------|-------|------------|
| Package size | {size} | {small/medium/large} |
| Dependencies | {count} | {few/moderate/many} |
| Tree-shakable | {yes/no} | {good if yes} |
| ESM support | {yes/no} | {good if yes} |
| Side effects | {declared/undeclared} | {good if declared false} |

Size thresholds:
- **Small**: <50KB — no concerns
- **Medium**: 50-500KB — acceptable for feature-rich packages
- **Large**: >500KB — consider if the value justifies the size

### Dependency Chain

List all transitive dependencies and flag:
- Deep chains (>3 levels)
- Known problematic packages
- Duplicate dependencies that might conflict with existing ones in the user's project
- License conflicts in the dependency tree

## Step 4: Present the Review

Format as a structured assessment:

```
## Package Review: @lpm.dev/owner.package-name

### Summary
{1-2 sentence verdict}

### Recommendation: {Install / Install with Caution / Avoid}

### Scores
| Dimension | Rating | Notes |
|-----------|--------|-------|
| Quality | {score}/100 ({tier}) | {brief} |
| Security | {Safe/Caution/Risk} | {brief} |
| Maintenance | {Active/Moderate/Stale} | {brief} |
| API Design | {Good/Fair/Poor} | {brief} |
| Bundle Impact | {Light/Moderate/Heavy} | {brief} |

### Strengths
- {strength 1}
- {strength 2}

### Concerns
- {concern 1 with severity}
- {concern 2 with severity}

### Details
{Expanded findings from each dimension, with specific file paths and evidence}
```

## Step 5: Compare Alternatives

If the user is deciding between packages, present a side-by-side:

```
## Comparison: package-a vs package-b

| | @lpm.dev/owner.a | @lpm.dev/owner.b |
|--|---|---|
| Quality Score | {x}/100 | {y}/100 |
| Bundle Size | {size} | {size} |
| Dependencies | {n} | {n} |
| Last Updated | {date} | {date} |
| TypeScript | {yes/no} | {yes/no} |
| Tree-shakable | {yes/no} | {yes/no} |
| License | {license} | {license} |
| Price | {Pool/$X} | {Pool/$Y} |

### Verdict
{Which to choose and why, based on the user's specific needs}
```

## Recommendation Criteria

| Recommendation | When |
|---------------|------|
| **Install** | Quality 70+, no security concerns, actively maintained, clean API |
| **Install with Caution** | Quality 50-69, minor concerns, or some stale signals — but the package is still useful |
| **Avoid** | Quality <50, security red flags, abandoned (>1 year no updates), critical issues found |

## Important Notes

- Be objective — present evidence, not opinions
- Security scanning is best-effort from static analysis, not a substitute for a proper security audit
- If the package isn't installed locally, some checks (source analysis, security scan) are limited to what `lpm info` provides
- Don't compare Pool and Marketplace packages on price alone — they serve different purposes
- If the user asks "should I use X?", always provide the review framework rather than a bare yes/no
- For npm packages (not LPM), this skill can still analyze the installed source in `node_modules/`, but quality scoring and LPM-specific features won't apply
