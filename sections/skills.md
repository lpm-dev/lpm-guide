# LPM Agent Skills Creation

Create Agent Skills for an `@lpm.dev/*` package: Markdown guidance that helps AI coding agents use the package correctly.

- **Authored in** `lpm-skills/*.md` at the package root, next to `package.json` (or in `lpm-skills/` of `publishConfig.directory` when set). Commit them.
- **Never author in `.lpm/`.** `.lpm/skills/<owner.package>/` is where `lpm install` / `lpm skills add` put a dependency's approved skills in a consumer project; `.lpm/` is local state and never published. Legacy `.md` files directly in `.lpm/skills/` fail `lpm skills validate` — move them to `lpm-skills/`.
- **Delivery:** `lpm install` (default for direct `@lpm.dev/*` deps; `lpm install`, `lpm add`, and `lpm ci` accept `--skills`/`--no-skills`), `lpm skills add @lpm.dev/owner.pkg`, and the MCP tool `lpm_package_skills`. Package skills never create editor/agent links or change editor config.
- **Quality:** at least 1 approved skill earns the `agent-skills` check (+5). See [Quality Checks](../references/quality-checks.md).

## Hard Rules

1. **Never skip the interview.** Ask targeted questions before generating files. Source code is not a substitute for the maintainer's answers.
2. **STOP at gates.** At each `> STOP`, present output and wait. Don't combine gates or skip ahead.
3. **Never guess API signatures.** Read the source. Wrong examples are worse than no skills.

## Workflow

Check for existing `lpm-skills/*.md` (not `.lpm/skills/<owner.package>/`). If only legacy `.lpm/skills/*.md` exist, move them to `lpm-skills/` and use Update Mode.

**No authored skills → [Create Mode](#create-mode):** analyze → **STOP** (findings + plan) → interview → **STOP** (final plan) → generate → validate.

**Authored skills exist → [Update Mode](#update-mode):** read skills + diff package → **STOP** (update plan) → targeted interview → surgical updates → validate.

---

# Create Mode

## Step 1: Analyze the Package

Read: entry point and public exports (full API surface), README, tests (real usage, edge cases), source (error handling, env branches, init order), `package.json`/`lpm.config.json` (repository URL, deps), CHANGELOG or migration guides.

**Issues scan (best effort):** if the `repository` is public, search issues/discussions for the main APIs; prioritize high-reaction threads and closed bugs with workarounds. Extract recurring workarounds (→ wrong/correct pairs), FAQs, misunderstood defaults, behavior changes. If private or unavailable, note it and ask about common problems in the interview.

Build a model of: core API, common patterns, gotchas, framework integration points, and failure modes (with the source for each).

## Step 2: Present Findings and Skill Plan

> **STOP** — present the following and wait for the maintainer before interviewing.

1. 2–3 sentence package summary.
2. Complexity and plan:

| Size | Indicators | Skills | Questions | Typical skills |
|---|---|---|---|---|
| Small utility | 1–5 exports, one concern | 2–3 | 3–4 | `getting-started`, plus `best-practices` or `anti-patterns`, optional one integration |
| Medium library | 6–20 exports, several modules | 3–5 | 5–7 | `getting-started`, `best-practices`, `anti-patterns` if pitfalls, 1–2 integrations |
| Large framework | 20+ exports, plugins, many integrations | 5–8 | 7–10 | all three core skills, one integration skill per framework, composition skills for common pairings |

3. Proposed skills (name + one-line purpose).
4. Issues-scan results, or that the repo was inaccessible.

## Step 3: Interview the Maintainer

Every question must cite a specific function, pattern, or issue you found. Ask at least 3, including **≥1 AI-specific** and **≥1 implicit-knowledge** question. Templates:

- AI-specific: "What would an AI get wrong when using `{function}`?"
- AI-specific: "What pattern screams 'an AI wrote this — that's wrong'?" / "Any implicit config type signatures don't reveal?"
- Implicit knowledge: "What does an experienced user know that isn't documented?"
- Implicit knowledge: "What works for prototyping but is dangerous in production?"
- Usage: "Tests use `{pattern}` — recommended, or a test shortcut?" / "README shows `{usage}` — still current?"
- Parameters: "`{function}` accepts `{param}` — any surprising valid values?"
- Integration: "What setup step do `{framework}` users always forget?" / "Does `{function}` differ in SSR vs client?"
- Pain points (if the issues scan was limited): "Most common support question?" / "Which options interact unexpectedly?"

## Step 4: Confirm Final Skill Plan

> **STOP** — present the final list (name, description, key topics, content sources: code / interview / issues) and changes from the interview. Wait for approval.

## Step 5: Generate Skill Files

Create `lpm-skills/<name>.md` — direct files only; `.md` files in subdirectories and symlinks are rejected, other files are ignored.

```markdown
---
name: getting-started
description: When and how to import and initialize the client (10-500 chars)
version: "1.2.0"
globs:
  - "src/**/*.ts"
---

# Getting Started

Body: at least 100 characters of concrete guidance.
```

| Field | Required | Rule |
|---|---|---|
| `name` | Yes | Lowercase letters, digits, hyphens; must start and end with a letter or digit; unique in the package |
| `description` | Yes | 10–500 characters; say when the skill applies |
| `version` | No | String (quote it). The package version the guidance was written for; it belongs to the skill, not the release |
| `globs` | No | String array of file patterns where the skill applies |

Limits: ≤10 skills, ≤15 KiB per file, ≤100 KiB total, body ≥100 characters. If a skill approaches 15 KiB, split out a `{topic}-reference` skill.

**Anti-patterns format** (each entry):

````markdown
### [CRITICAL|HIGH|MEDIUM] What goes wrong

Wrong:
```js
// looks correct, isn't
```

Correct:
```js
// works
```

One sentence on the mechanism of failure.

Source: doc page, source file, issue URL, or "maintainer interview"
````

CRITICAL = production break, security, or data loss; HIGH = wrong under common conditions; MEDIUM = edge cases. Each entry must be plausible (an agent would write it), silent (no crash or type error), and grounded in a real source.

**Content:** concrete code, real function names and parameter types, correct vs incorrect side by side. Don't duplicate the README, teach general language/framework knowledge, or invent anti-patterns without evidence.

**Security scan (fails validation, even at warning level; code fences are scanned too):** download-and-run (`curl … | sh`, including `sh` on the next line; `iwr … | iex`; download + `chmod +x` + run), credential exfiltration (`curl`/`wget`/`nc` naming `.ssh`, `.aws`, `.npmrc`, `.env`, or credentials), `eval(`, `child_process`, env access next to KEY/SECRET/TOKEN/PASSWORD/CREDENTIAL (`process.env.API_KEY`), destructive deletes (`rm -rf /`/`~`, `fs.rm*(`, `fs.unlink(`, `rimraf`, `Remove-Item -Recurse -Force`), and prompt-injection phrases ("ignore previous instructions", "forget previous instructions", "you are now", `[INST]`, `<<SYS>>`). Anti-pattern examples that contain these will fail — describe them in prose instead.

## Step 6: Validate and Publish

```bash
lpm skills validate
lpm publish
```

`lpm skills validate` checks layout, frontmatter, sizes, duplicate names, and security patterns with the same validator the Registry uses at upload. Valid files in `lpm-skills/` ship automatically, whatever `package.json > files` lists — don't edit `files`.

After publish: Private packages get their skills immediately. For Free, Pool, and Marketplace packages, skills become available only after the publication review and a separate skill security review; only approved skills (including approved-with-warning) reach consumers.

---

# Update Mode

Existing skills hold maintainer-reviewed content — update surgically, never regenerate.

## Step 1: Read and Diff

Read every `lpm-skills/*.md`, then the current source, README, and CHANGELOG. Identify new/removed exports, changed signatures or defaults, new framework support, deprecations, and breaking changes. If the repo is public, scan recent issues for new confusion.

## Step 2: Present Update Plan

> **STOP** — list skills to modify (and what), leave unchanged, create, and remove (only if the maintainer says obsolete). Wait.

## Step 3: Targeted Interview

Ask 1–3 questions about new or changed areas only, e.g. "You changed `{API}` — does the old pattern still work, or should it become an anti-pattern?", "Common mistakes with the new `{feature}`?"

> **STOP** — wait for answers.

## Step 4: Apply Surgical Updates

- New API → add to `getting-started` or a new skill.
- Removed/renamed API → update examples; add the old usage as an anti-pattern ("agents trained on older versions may still generate this").
- Changed defaults → update examples; add an anti-pattern if the old behavior was common.
- New framework → new integration skill. New issue-reported failure → anti-pattern with the issue URL.

Keep accurate content as is. Bump a skill's `version` only after you have actually reviewed it against the current release. The package page shows "These skills haven't changed since v{X}" when skill content is unchanged across releases, and the publish response warns about it; a `version` bump hides that signal, so don't bump unreviewed skills.

## Step 5: Validate

Same as Create Mode Step 6.
