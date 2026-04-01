# LPM Package Scaffolding

Create a new LPM package from scratch with the right file structure, configuration, exports, types, tests, and documentation — optimized for a high quality score on first publish.

Supports JavaScript, Swift, and XCFramework ecosystems.

## Workflow

1. **Ask what they're building** — Understand the package type and purpose
2. **Detect project context** — Check for existing framework, monorepo, toolchain config
3. **Generate the package** — Create all files with sensible defaults
4. **Verify quality** — Run quality checks to confirm the scaffold scores well

## Step 1: Gather Requirements

Ask the user (skip questions they've already answered):

| Question | Why |
|----------|-----|
| Package name? | Determines the `@lpm.dev/owner.package-name` identifier. Run `lpm check-name owner.package-name` to verify availability before proceeding. |
| What does it do? (1 sentence) | Used for `description` in package metadata and README |
| Package type? | Determines file structure and templates |
| Language/ecosystem? | JavaScript/TypeScript, Swift, or XCFramework |
| Distribution mode? | Affects README content and whether to include `lpm.config.json` |

### Package Types

| Type | Ecosystem | Structure | Install command | Example |
|------|-----------|-----------|-----------------|---------|
| **Utility library** (`package`) | JS | `src/` with functions, single entry point | `lpm install` | `@lpm.dev/acme.utils` |
| **React component library** (`package`) | JS | `src/components/` with components, Storybook-ready | `lpm install` | `@lpm.dev/acme.ui-kit` |
| **CLI tool** (`package`) | JS | `bin/` + `lib/`, Commander/citty setup | `lpm install` | `@lpm.dev/acme.cli` |
| **Swift library** (`package`) | Swift | `Sources/`, `Tests/`, `Package.swift` | `lpm install` | `@lpm.dev/acme.swift-utils` |
| **XCFramework** (`xcframework`) | XCF | Pre-compiled `.xcframework` binary | `lpm install` | `@lpm.dev/acme.analytics` |
| **Source package** (`source`) | JS | Components with `lpm.config.json` for `lpm add` | `lpm add` | `@lpm.dev/acme.blocks` |
| **MCP server** (`mcp-server`) | JS | MCP server with tools/resources, `lpm.config.json` with `type: "mcp-server"` | `lpm add` | `@lpm.dev/acme.stripe-mcp` |
| **VS Code extension** (`vscode-extension`) | Any | Extension with `contributes`, `lpm.config.json` with `type: "vscode-extension"` | `lpm add` | `@lpm.dev/acme.monokai-pro` |
| **Cursor rules** (`cursor-rules`) | Any | AI agent config files, `lpm.config.json` with `type: "cursor-rules"` | `lpm add` | `@lpm.dev/acme.react-rules` |
| **GitHub Action** (`github-action`) | Any | Workflow/action files, `lpm.config.json` with `type: "github-action"` | `lpm add` | `@lpm.dev/acme.deploy-aws` |
| **Other** (`other`) | Any | Anything else, `lpm.config.json` with `type: "other"` | `lpm add` | `@lpm.dev/acme.custom-tool` |

The value in parentheses is the `type` field in `lpm.config.json`. Types other than `package`/`xcframework` require a `lpm.config.json` — see [Config Spec](../references/config-spec.md) for the full specification.

## Step 2: Detect Context

Before generating files, check the current directory for:

**JavaScript:**
- `tsconfig.json` / `jsconfig.json` — TypeScript preference
- `package.json` (parent) — Monorepo detection (workspaces field)
- `.nvmrc` / `.node-version` — Node version preference
- Existing test config — `vitest.config.*`, `jest.config.*`
- Package manager — `pnpm-lock.yaml`, `bun.lockb`, `yarn.lock`, `package-lock.json`

**Swift:**
- `.swift-version` — Swift version preference
- `Package.swift` (parent) — Monorepo detection
- Existing platform targets — infer `.iOS`, `.macOS`, etc. from sibling packages

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
  2. Add Agent Skills for AI coding assistants (see "lpm skills validate")
  3. Run "lpm publish --check" to verify quality score
  4. Run "lpm publish" when ready
```

## Swift Library Scaffold

### File Structure

```
PackageName/
├── package.json
├── Package.swift
├── README.md
├── CHANGELOG.md
├── LICENSE
├── Sources/
│   └── PackageName/
│       └── PackageName.swift    # Main entry point
└── Tests/
    └── PackageNameTests/
        └── PackageNameTests.swift
```

### package.json Template

```json
{
  "name": "@lpm.dev/owner.package-name",
  "version": "0.1.0",
  "description": "{user's description}",
  "author": "{owner}",
  "license": "MIT",
  "files": [
    "Sources/",
    "Tests/",
    "Package.swift",
    "README.md",
    "LICENSE",
    "CHANGELOG.md"
  ],
  "keywords": []
}
```

The `files` field is critical — without it, Swift build artifacts (`.build/`, often 100MB+) may be included in the published tarball. This field whitelists only what consumers need.

### Package.swift Template

```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "PackageName",
    platforms: [
        .iOS(.v16),
        .macOS(.v13),
        .watchOS(.v9),
        .tvOS(.v16),
    ],
    products: [
        .library(
            name: "PackageName",
            targets: ["PackageName"]
        ),
    ],
    dependencies: [],
    targets: [
        .target(
            name: "PackageName",
            dependencies: []
        ),
        .testTarget(
            name: "PackageNameTests",
            dependencies: ["PackageName"]
        ),
    ]
)
```

### Swift Source Template (`Sources/PackageName/PackageName.swift`)

```swift
/// {description from user}
public struct PackageName {
    /// Creates a new instance.
    public init() {}

    /// {placeholder — fill in as you build}
    public func hello() -> String {
        "Hello from PackageName"
    }
}
```

### Swift Test Template (`Tests/PackageNameTests/PackageNameTests.swift`)

```swift
import Testing
@testable import PackageName

@Suite("PackageName Tests")
struct PackageNameTests {
    @Test func helloReturnsString() {
        let sut = PackageName()
        #expect(sut.hello().isEmpty == false)
    }
}
```

### Swift README Install Section

```markdown
## Installation

### Swift Package Manager

Add to your `Package.swift`:

\`\`\`swift
dependencies: [
    .package(url: "https://lpm.dev/owner.package-name", from: "0.1.0"),
]
\`\`\`

Or add via LPM:

\`\`\`bash
lpm install @lpm.dev/owner.package-name
\`\`\`
```

### Swift Quality Score Notes

The scaffold above earns these checks out of the box:
- `has-readme` ✓, `readme-install` ✓, `readme-usage` ✓, `readme-api` ✓ — from README template
- `has-changelog` ✓, `has-license` ✓ — from files
- `has-platforms` ✓ — 4 platforms declared
- `recent-tools-version` ✓ — swift-tools-version 5.9
- `multi-platform` ✓ (4pts) — 4+ platforms
- `has-public-api` ✓ — server-augmented (CLI passes if source files present)
- `has-doc-comments` ✓ — server-only provisional pass; add `///` comments to maximize
- `small-deps` ✓ (4pts) — 0 dependencies
- `has-test-files` ✓, `has-test-script` ✓ — from testTarget

Estimated scaffold score: **85+/100 (Good)**

---

## XCFramework Scaffold

XCFrameworks are pre-compiled binaries — there is no source template to generate. Instead, scaffold the metadata files and README:

### File Structure

```
FrameworkName/
├── README.md
├── CHANGELOG.md
├── LICENSE
└── FrameworkName.xcframework/
    └── (your pre-compiled binary slices)
```

### XCFramework Quality Score Notes

The key quality drivers for XCFrameworks:
- `xcf-valid-plist` (10pts) — Info.plist must define platform slices
- `xcf-multi-slice` (15pts) — ship as many platform/arch slices as possible (device + simulator for iOS + macOS = 15pts)
- `xcf-architectures` (10pts) — must include arm64
- `xcf-size` (5pts in code, 3pts in health) — keep binary under 10MB if possible

Build your XCFramework with `xcodebuild -create-xcframework` combining device and simulator archives.

---

## Important Notes

- Always use `@lpm.dev/owner.package-name` format for the package name
- Default to JavaScript unless the user specifies Swift/XCFramework or the project clearly uses it
- Match the existing package manager if detected (pnpm, bun, yarn, npm for JS; swift-tools-version from siblings for Swift)
- Don't add unnecessary dependencies — keep `dependencies` minimal for a high quality score
- The scaffold should score 70+ on quality checks out of the box (Good tier)
- If the user says "source package", follow up with the [Source Config](./source-config.md) workflow for `lpm.config.json` generation
