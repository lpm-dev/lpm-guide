# LPM Agent Skills Creation

Create Agent Skills for your LPM package - structured guidelines that help AI coding assistants use your package correctly. Skills are `.md` files in `.lpm/skills/` that get published with your package and served to AI tools via the MCP server.

Skills earn **+7 quality points** (`has-skills`) and an additional **+3 points** (`skills-comprehensive`) when you have 3 or more skills.

## Hard Rules

These rules override any other reasoning. No exceptions.

1. **Never skip the interview.** You must ask targeted questions before generating any skill files. Source code is NOT a substitute for the maintainer's answers.
2. **STOP at gates.** At each `> STOP` gate, present your output and wait for the maintainer to respond. Do not combine gates or skip ahead.
3. **Never guess API signatures.** Read actual source code. Wrong examples in skills are worse than no skills.

## Workflow

Check if `.lpm/skills/` already exists with skill files.

**No existing skills - [Create Mode](#create-mode):**
1. Analyze the package
2. **STOP** - Present findings and skill plan, get approval
3. Interview the maintainer
4. **STOP** - Confirm final skill plan
5. Generate skill files
6. Validate

**Existing skills found - [Update Mode](#update-mode):**
1. Read existing skills and analyze package changes
2. **STOP** - Present diff and proposed updates
3. Targeted interview (only for new/changed areas)
4. Apply surgical updates
5. Validate

---

# Create Mode

## Step 1: Analyze the Package

### 1a - Read Local Files

| File | What to Extract |
|------|-----------------|
| Entry point and public exports | API surface - every function, class, component, type exported |
| README.md | What's already documented, install instructions, usage examples |
| Test files | Expected usage patterns, edge cases being tested |
| Source code | Complex logic, error handling, assertions, environment branches |
| `lpm.config.json` or `package.json` | Package configuration, repository URL, dependencies |
| CHANGELOG.md or migration guides | Breaking changes between versions |

### 1b - Scan GitHub/GitLab Issues (Best-Effort)

Check `package.json` or `lpm.config.json` for a `repository` URL. If the repository is public and accessible:

1. Search issues for the package's primary APIs and function names
2. Focus on high-signal threads: sort by reactions/comments, look at closed bugs with workarounds
3. Check for labels like `bug`, `question`, `breaking-change`
4. Search Discussions (if the repo uses them) for "how do I..." threads

**What to extract:**
- Recurring bug workarounds - these become wrong/correct code pairs
- Frequently asked questions - content for getting-started or best-practices
- Misunderstandings about defaults - anti-pattern entries
- Issues that changed behavior - migration-boundary mistakes

**If the repo is private or unavailable**, skip this step. Note it so you ask about common problems in the interview instead.

### 1c - Build a Mental Model

From your reading, identify:
- **Core API** - what functions/classes/components are exported
- **Common patterns** - how the package is typically used
- **Gotchas** - things that trip up developers (type coercion, async behavior, required config, initialization order)
- **Integration points** - how this works with frameworks (React, Next.js, Express, SwiftUI, etc.)
- **Failure modes found** - patterns from issues/docs/source that look correct but break (note the source for each)

## Step 2: Present Findings and Skill Plan

> **STOP** - Present the following to the maintainer and wait for their response before proceeding to the interview. Do not proceed past this point in the same message.

Present to the maintainer:

**1. Package summary** - 2-3 sentences on what you found.

**2. Complexity assessment:**

| Package Size | Indicators | Skill Count | Interview Depth |
|---|---|---|---|
| Small utility | 1-5 exports, single concern | 2-3 skills | 3-4 questions |
| Medium library | 6-20 exports, multiple modules | 3-5 skills | 5-7 questions |
| Large framework | 20+ exports, plugins/middleware, multiple integration points | 5-8 skills | 7-10 questions |

**3. Proposed skill plan** - List each skill with name and one-line purpose.

**4. Issues scan results** - Top findings, or note if repo was inaccessible.

The maintainer may approve, add/remove skills, or correct misunderstandings.

## Step 3: Interview the Maintainer

Every question must cite a specific function, pattern, or issue you found. Never ask generic questions. Ask at minimum 3 questions total, at least 1 from AI-Agent-Specific and 1 from Implicit Knowledge.

### AI-Agent-Specific (always ask at least 1)

| Question Template | Why |
|---|---|
| "What would an AI get wrong when using `{function}`?" | Surfaces silent failure modes |
| "What pattern screams 'an AI wrote this - that's wrong'?" | Identifies AI-specific anti-patterns |
| "Any implicit config that reading type signatures alone wouldn't reveal?" | Catches undocumented behavior |

### Implicit Knowledge (always ask at least 1)

| Question Template | Why |
|---|---|
| "What does an experienced user know that isn't documented?" | Most valuable skill content is undocumented knowledge |
| "Patterns that work for prototyping but are dangerous in production?" | Catches prototype-to-production pitfalls |

### Usage Patterns (ask based on findings)

| Question Template | Why |
|---|---|
| "I found `{pattern}` in tests - recommended way or test shortcut?" | Tests sometimes use internal APIs |
| "I see `{function}` accepts `{param}` - any surprising valid values?" | Catches parameter edge cases |
| "README shows `{usage}` - still the current best practice?" | Docs can drift from real usage |

### Integration/Framework (ask if applicable)

| Question Template | Why |
|---|---|
| "For `{framework}` users, what setup step do they always forget?" | Critical initialization steps |
| "Does `{function}` behave differently in SSR vs client-side?" | Environment-specific gotchas |

### Pain Points (ask if issues scan was limited)

| Question Template | Why |
|---|---|
| "What's the most common support question you get?" | Highest-value skill content |
| "Which config options interact in unexpected ways?" | Non-obvious coupling |

## Step 4: Confirm Final Skill Plan

> **STOP** - Present the final skill plan incorporating interview answers. Wait for the maintainer's approval before generating files. Do not proceed past this point in the same message.

Present:
- Final list of skills to generate (name + description + key topics each will cover)
- Any skills added, removed, or changed based on the interview
- For each skill, note the primary sources of content (code analysis, interview answer, issues scan)

The maintainer approves or adjusts. Then proceed to generation.

## Step 5: Generate Skill Files

Create `.lpm/skills/*.md` files. Each skill has YAML frontmatter + markdown body.

### File Format

```markdown
---
name: skill-name
description: One-line description of what this skill covers (10-500 chars)
version: "1.2.0"
globs:
  - "**/*.ts"
  - "**/*.tsx"
---

# Skill Title

Markdown content with usage patterns, anti-patterns, examples, etc.
Minimum 100 characters of content.
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase letters, numbers, hyphens only (e.g., `getting-started`, `react-integration`) |
| `description` | Yes | 10-500 characters explaining when this skill applies |
| `version` | No | Package version this skill was written for. Read from `package.json` `version` field and include automatically. Helps LLMs detect stale guidance (e.g., skill says v1.0.0 but installed package is v2.0.0) |
| `globs` | No | File patterns for when this skill is most relevant (e.g., `["**/*.tsx"]` for React files) |

### Size Limits

| Limit | Value |
|-------|-------|
| Max file size | 15KB per skill |
| Max total size | 100KB across all skills |
| Max skill count | 10 per package version |
| Min content length | 100 characters |
| Description length | 10-500 characters |

### Skill Planning by Complexity

**Small utility (2-3 skills):**
- `getting-started` (always)
- `best-practices` or `anti-patterns` (pick based on interview - which has more content?)
- Optional: one integration skill if applicable

**Medium library (3-5 skills):**
- `getting-started` (always)
- `best-practices` (always)
- `anti-patterns` (if interview revealed pitfalls)
- 1-2 integration skills per relevant framework

**Large framework (5-8 skills):**
- `getting-started` (always)
- `best-practices` (always)
- `anti-patterns` (always - large frameworks always have pitfalls)
- Multiple integration skills (one per framework, not combined)
- Composition skills for common library pairings (e.g., your-lib + React Router)
- If any skill approaches 15KB, split into a focused skill + a separate `{topic}-reference` skill

### Structured Anti-Patterns Format

Each entry in the anti-patterns skill must follow this structure:

```markdown
### [PRIORITY] Short description of what goes wrong

Wrong:

\`\`\`js
// code that looks correct but isn't
\`\`\`

Correct:

\`\`\`js
// code that works
\`\`\`

One sentence explaining the specific mechanism by which the wrong version fails.

Source: doc page, source file, GitHub issue URL, or "maintainer interview"
```

Priority levels:
- **CRITICAL** - Breaks in production, security risk, or data loss
- **HIGH** - Incorrect behavior under common conditions
- **MEDIUM** - Incorrect under specific conditions or edge cases

Every anti-pattern must be:
- **Plausible** - an AI agent would realistically generate this code
- **Silent** - no immediate crash or type error; the failure is behavioral
- **Grounded** - traceable to a specific source (interview answer, issue, test, source code)

### Content Guidelines

**DO:** Use concrete code examples (not pseudocode). Show correct and incorrect patterns side by side. Reference specific function names and parameter types. Ground every anti-pattern in a real source (interview, issue, or code).

**DON'T:** Duplicate the README. Include dangerous shell commands, `process.env` secrets, or prompt injection patterns (all blocked by validation). Include general language/framework knowledge. Invent anti-patterns without evidence.

## Step 6: Validate and Review

After generating the skill files:

1. **Run validation:**
   ```bash
   lpm skills validate
   ```
   This checks file format, size limits, blocked patterns, and batch constraints.

2. **Review quality score impact:**
   - 1+ skills: +7 points (`has-skills`)
   - 3+ skills: +3 additional points (`skills-comprehensive`)
   - Total potential: +10 quality points

3. **Ensure `.lpm` is in the tarball:**

   **JS packages (including source packages):** JS uses `npm pack`, which respects the `"files"` field in `package.json`. If `package.json` has a `"files"` field and `.lpm` is not listed, skills will be silently excluded from the tarball.
   **Do this automatically** - read `package.json`, check if `"files"` exists, and if `.lpm` is not already listed, add it and save the file.
   `lpm skills validate` also handles this automatically when skills are valid.

   **Swift packages:** No action needed. Swift packages use LPM's file collector which automatically includes all files except build artifacts (`.build/`, `DerivedData/`, etc.). The `.lpm/` folder is included by default.

4. **Publish:**
   ```bash
   lpm publish
   ```
   Skills are extracted from the tarball, validated, and approved after a security scan.

---

# Update Mode

When `.lpm/skills/` already exists, use surgical updates. Existing skills contain maintainer-reviewed content and failure modes that would be lost in a full regeneration.

## Step 1: Read Existing Skills and Diff Changes

1. Read all existing `.lpm/skills/*.md` files - understand what's already covered
2. Read the current source code, README, and CHANGELOG
3. Identify what changed since the skills were created:
   - New or removed APIs/exports
   - Changed function signatures or default values
   - New framework support
   - Deprecated features
   - Breaking changes documented in CHANGELOG
4. If the repository is public, scan recent GitHub issues for new confusion patterns

## Step 2: Present Update Plan

> **STOP** - Present the following to the maintainer and wait for their response before proceeding.

Present a diff of proposed changes:
- **Skills to modify** - which skills and what specifically needs updating
- **Skills to leave unchanged** - still accurate, no changes needed
- **New skills to create** - for new features or integrations
- **Skills to remove** - if the maintainer indicates something is obsolete

## Step 3: Targeted Interview

Ask 1-3 questions focused only on new or changed areas:

| Question Template | Why |
|---|---|
| "You changed `{API}` in the latest version. Does the old pattern still work, or should I add it as an anti-pattern?" | Determines if old usage becomes a failure mode |
| "I see a new `{feature}`. What are the common mistakes when first using it?" | Captures failure modes for new functionality |
| "Any new framework-specific gotchas with the updated version?" | Catches integration-level regressions |

> **STOP** - Wait for answers before modifying skills.

## Step 4: Apply Surgical Updates

For each skill that needs updating:
- **New API** - Add to getting-started or create new skill
- **Removed/renamed API** - Update examples, add old pattern as anti-pattern: "Agents trained on older versions may still generate this"
- **Changed defaults/behavior** - Update code examples, add anti-pattern if old behavior was common
- **New framework support** - Add new integration skill
- **New failure modes from issues** - Add to anti-patterns with issue URL as source

**Do not rewrite content that is still accurate.** Preserve existing failure modes, maintainer insights, and reviewed content.

## Step 5: Validate

Run the same validation as Create Mode Step 6.

---

## Example: Creating Skills for a Validation Library

For a package like `@lpm.dev/acme.validate`:

**.lpm/skills/getting-started.md:**
```markdown
---
name: getting-started
description: How to import and use the validate library for form and data validation
version: "1.0.0"
globs:
  - "**/*.ts"
  - "**/*.js"
---

# Getting Started with @lpm.dev/acme.validate

## Import Pattern

Always import specific validators rather than the entire library:

// Good - tree-shakable
import { isEmail, isUrl, createSchema } from "@lpm.dev/acme.validate"

// Avoid - imports everything
import * as validate from "@lpm.dev/acme.validate"

## Basic Usage

Create a schema and validate data:

const userSchema = createSchema({
  email: [isEmail(), required()],
  age: [isNumber(), min(0), max(150)],
})

const result = userSchema.validate(data)
if (!result.valid) {
  console.error(result.errors)
}
```

**.lpm/skills/anti-patterns.md** (showing the structured format):
```markdown
---
name: anti-patterns
description: Common mistakes when using the validate library and how to avoid them
version: "1.0.0"
globs:
  - "**/*.ts"
  - "**/*.js"
---

# Anti-Patterns

### [CRITICAL] Creating schemas inside render loops

Wrong:
// function validateUser(data) { const schema = createSchema({...}); ... }

Correct:
// const userSchema = createSchema({...})  <-- create once, reuse
// function validateUser(data) { return userSchema.validate(data) }

createSchema() compiles validators on each call. Creating inside a loop causes O(n) overhead that degrades silently.

Source: source code - createSchema() internals

### [MEDIUM] Ignoring the structured error object

Wrong:
// if (!result.valid) throw new Error("Invalid")

Correct:
// if (!result.valid) { for (const e of result.errors) console.error(e.field, e.message) }

The result contains per-field error details. Discarding them makes debugging impossible.

Source: README.md - Error Handling section
```

## Important Notes

- The `globs` field helps AI tools show the right skill at the right time (e.g., React skills when editing `.tsx` files)
- After publish, skills are served via `lpm skills install` and through the LPM MCP server
- Skills go through a security scan after publish
- In Update Mode, preserve content that is still accurate - surgical updates, not full rewrites
- See [Improve](./improve.md) for full quality score breakdown
