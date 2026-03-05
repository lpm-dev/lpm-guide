# LPM Package Scaffolding

Create a new LPM package from scratch with the right file structure, package.json configuration, exports, types, tests, and documentation — optimized for a high quality score on first publish.

## Workflow

1. **Ask what they're building** — Understand the package type and purpose
2. **Detect project context** — Check for existing framework, monorepo, TypeScript config
3. **Generate the package** — Create all files with sensible defaults
4. **Verify quality** — Run quality checks to confirm the scaffold scores well

## Step 1: Gather Requirements

Ask the user (skip questions they've already answered):

| Question | Why |
|----------|-----|
| Package name? | Determines the `@lpm.dev/owner.package-name` identifier. Run `lpm check-name owner.package-name` to verify availability before proceeding. |
| What does it do? (1 sentence) | Used for `description` in package.json and README |
| Package type? | Determines file structure and templates |
| TypeScript or JavaScript? | Determines tooling and file extensions |
| Distribution mode? | Affects README content and whether to include `lpm.config.json` |

### Package Types

| Type | Structure | Install command | Example |
|------|-----------|-----------------|---------|
| **Utility library** (`package`) | `src/` with functions, single entry point | `lpm install` | `@lpm.dev/acme.utils` |
| **React component library** (`package`) | `src/components/` with components, Storybook-ready | `lpm install` | `@lpm.dev/acme.ui-kit` |
| **CLI tool** (`package`) | `bin/` + `lib/`, Commander/citty setup | `lpm install` | `@lpm.dev/acme.cli` |
| **Source package** (`source`) | Components with `lpm.config.json` for `lpm add` | `lpm add` | `@lpm.dev/acme.blocks` |
| **MCP server** (`mcp-server`) | MCP server with tools/resources, `lpm.config.json` with `type: "mcp-server"` | `lpm add` | `@lpm.dev/acme.stripe-mcp` |
| **VS Code extension** (`vscode-extension`) | Extension with `contributes`, `lpm.config.json` with `type: "vscode-extension"` | `lpm add` | `@lpm.dev/acme.monokai-pro` |
| **Cursor rules** (`cursor-rules`) | AI agent config files, `lpm.config.json` with `type: "cursor-rules"` | `lpm add` | `@lpm.dev/acme.react-rules` |
| **GitHub Action** (`github-action`) | Workflow/action files, `lpm.config.json` with `type: "github-action"` | `lpm add` | `@lpm.dev/acme.deploy-aws` |
| **Other** (`other`) | Anything else, `lpm.config.json` with `type: "other"` | `lpm add` | `@lpm.dev/acme.custom-tool` |

The value in parentheses is the `type` field in `lpm.config.json`. Types other than `package` require a `lpm.config.json` — see [Config Spec](../references/config-spec.md) for the full specification.

## Step 2: Detect Context

Before generating files, check the current directory for:

- `tsconfig.json` / `jsconfig.json` — TypeScript preference
- `package.json` (parent) — Monorepo detection (workspaces field)
- `.nvmrc` / `.node-version` — Node version preference
- Existing test config — `vitest.config.*`, `jest.config.*`
- Package manager — `pnpm-lock.yaml`, `bun.lockb`, `yarn.lock`, `package-lock.json`

Use detected context to match existing project conventions.

## Step 3: Generate Files

### All Package Types

```
package-name/
├── package.json
├── README.md
├── CHANGELOG.md
├── LICENSE
├── src/
│   └── index.{js,ts}         # Main entry point
├── src/__tests__/
│   └── index.test.{js,ts}    # Test for entry point
└── vitest.config.{js,ts}
```

### package.json Template

```json
{
  "name": "@lpm.dev/owner.package-name",
  "version": "0.1.0",
  "description": "{user's description}",
  "author": "{from git config user.name and user.email}",
  "license": "MIT",
  "type": "module",
  "main": "./src/index.js",
  "exports": {
    ".": "./src/index.js"
  },
  "types": "./src/index.d.ts",
  "sideEffects": false,
  "files": ["src", "README.md", "CHANGELOG.md", "LICENSE"],
  "engines": { "node": ">=18" },
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest"
  },
  "keywords": [],
  "repository": {
    "type": "git",
    "url": "{from git remote or ask}"
  },
  "homepage": "https://lpm.dev/owner.package-name",
  "devDependencies": {
    "vitest": "^4.0.0"
  }
}
```

Adjustments by package type:

| Type | Additions |
|------|-----------|
| **React component** | Add `react` + `react-dom` as `peerDependencies`, add `@testing-library/react` to devDeps |
| **CLI tool** | Add `"bin"` field, add `commander` or `citty` to dependencies, create `bin/cli.js` |
| **Source package** | Generate `lpm.config.json` (see [Source Config](./source-config.md)) |
| **TypeScript** | Add `typescript` to devDeps, add `"build": "tsc"` script, create `tsconfig.json` with `declaration: true`, update `exports`/`types` to point to `./dist/` |

### README Template

```markdown
# package-name

{description}

## Installation

\`\`\`bash
lpm install @lpm.dev/owner.package-name
\`\`\`

## Usage

\`\`\`javascript
import { mainExport } from "@lpm.dev/owner.package-name"
\`\`\`

## API Reference

### mainExport()

{placeholder — fill in as you build}

## License

MIT
```

For source packages, use `lpm add` instead of `lpm install` in the install section.

### CHANGELOG Template

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [0.1.0] - {today's date}

### Added
- Initial release
```

### Test Template

```javascript
import { describe, it, expect } from "vitest"
import { mainExport } from "../index.js"

describe("mainExport", () => {
  it("should work", () => {
    // TODO: Add your first test
    expect(mainExport).toBeDefined()
  })
})
```

### Type-Specific Files

**React component library:**
```
src/
├── index.{js,ts}              # Re-exports all components
├── components/
│   └── Button/
│       ├── index.{js,ts}
│       ├── Button.{jsx,tsx}
│       └── Button.test.{jsx,tsx}
```

**CLI tool:**
```
bin/
└── cli.js                     # #!/usr/bin/env node, imports from src/
src/
├── index.js                   # Programmatic API
├── commands/
│   └── hello.js               # Starter command
```

**Source package:**
```
src/
├── components/
│   └── Example/
│       ├── Example.jsx
│       └── Example.style.jsx  # If styling option selected
├── lpm.config.json
```

### TypeScript Project Setup

When the user chooses TypeScript, generate a `tsconfig.json` with `declaration: true` so `.d.ts` files are emitted automatically:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "declaration": true,
    "declarationDir": "./dist",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

And adjust the package.json fields to point to the `dist/` output:

```json
{
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "default": "./dist/index.js"
    }
  },
  "files": ["dist", "README.md", "CHANGELOG.md", "LICENSE"],
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "prepublishOnly": "npm run build"
  }
}
```

`declaration: true` generates `.d.ts` files automatically — consumers get full IntelliSense with no extra effort, and it earns `has-types` (8 pts) and `intellisense-coverage` (4 pts) on the quality score.

### JavaScript with JSDoc

When the user chooses plain JavaScript, encourage adding JSDoc annotations to exported functions for IntelliSense:

```javascript
/**
 * Parse a configuration file and return validated options.
 * @param {string} filePath - Path to the config file
 * @param {object} [defaults] - Default values to merge
 * @returns {Promise<object>} Parsed and validated configuration
 */
export async function parseConfig(filePath, defaults = {}) {
```

Adding `@param` and `@returns` to exported functions gives editors (VS Code, Cursor) IntelliSense and earns the `intellisense-coverage` quality check (+4 points). No build step needed.

## Step 4: Post-Scaffold

After generating files:

1. Run `npm install` (or detected package manager) to install devDependencies
2. Run `npm test` to verify the test template passes
3. Initialize git repo if not already in one (`git init`)
4. Print a summary of what was created and next steps:

```
Created @lpm.dev/owner.package-name

  src/index.js          — Main entry point (start coding here)
  src/__tests__/        — Test directory
  package.json          — Configured for LPM with quality fields
  README.md             — Template with install + usage sections
  CHANGELOG.md          — Keep a Changelog format
  LICENSE               — MIT

Next steps:
  1. Write your package code in src/
  2. Run "lpm publish --check" to verify quality score
  3. Run "lpm publish" when ready
```

## Important Notes

- Always use `@lpm.dev/owner.package-name` format for the package name
- Default to JavaScript unless the user specifies TypeScript or the project already uses it
- Match the existing package manager if detected (pnpm, bun, yarn, npm)
- Don't install unnecessary dependencies — keep `dependencies` minimal for a high quality score
- The scaffold should score 70+ on quality checks out of the box (Good tier)
- If the user says "source package", follow up with the [Source Config](./source-config.md) workflow for `lpm.config.json` generation
