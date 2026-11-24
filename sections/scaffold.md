# LPM Package Scaffolding

Create a new LPM package (JavaScript/TypeScript or Swift) that passes the quality checks on its first publish.

## Workflow

1. **Gather requirements.** Skip any question the user already answered:
   - name
   - a one-sentence description
   - package type
   - ecosystem (default to JavaScript unless they say Swift or the project uses it)
   - planned distribution mode
2. **Detect context.** Match what the project already has:
   - JS: `tsconfig.json`/`jsconfig.json`, a workspace root, the lockfile or package manager, `.nvmrc`/`.node-version`, the test runner config.
   - Swift: a parent or sibling `Package.swift`, `.swift-version`, the platforms sibling packages use.
3. **Generate the files** using the rules below.
4. **Verify** with `lpm publish --check`.

## Names

- lpm.dev packages are named `@lpm.dev/owner.package-name`. The owner is the user's LPM username or organization. Each part starts with a lowercase letter, followed by lowercase letters, digits or hyphens.
- There is no `lpm check-name` command. Names live inside the owner's namespace. `lpm publish --dry-run` checks permissions, account limits and version availability.

## Package Types

| Type | Ecosystem | Consumer installs with | Notes |
|------|-----------|------------------------|-------|
| Library (utilities, React components, CLI tools) | JS | `lpm install` | A regular dependency package |
| Swift library | Swift | `lpm install` | Writes `.package(id:)` into the consumer's `Package.swift` |
| MCP server | JS | `lpm install`, or the tool's own setup | A regular package. Document the MCP client setup in the README |
| Source package (components, editor rules, workflow templates, any copied files) | Any | `lpm add` | Add `lpm.config.json`. See [Source Config](./source-config.md) |

`lpm.config.json` accepts an optional top-level `type`, which must be `"package"` or `"source"`:
- It sets the Registry's classification only and doesn't change how `lpm add` copies files.
- Any other label, such as `"mcp-server"`, is rejected.
- See [Config Spec](../references/config-spec.md).

## JavaScript Package

Start with `lpm init`, which writes `package.json` (and `AGENTS.md`, unless you pass `--no-agents`):

```bash
lpm init --lpm -y --owner <owner> --name <package-name>
```

What it creates:
- **Fields:** `name`, `version: "1.0.0"`, `main` and `types` pointing into `dist/`, `type: "module"`, `license: "MIT"`, `files: ["dist"]` and `packageManager`.
- **Not created:** source files, a build setup or `dist/`.
- **Non-interactive shells:** use `-y`.

Then adjust `package.json`:

- **Entry paths must exist.** Every path in `main`, `module`, `types`, `bin` and `exports` must name a file that ships in the package. Plain JS without a build:
  - Point `main`/`exports` at `./src/index.js`.
  - Set `files` to include `src`.
  - Remove `types` unless you ship a `.d.ts`.
- **TypeScript:**
  - Add `typescript` and a `build` script (`tsc`).
  - Use a `tsconfig.json` with `declaration: true` that emits to `dist/`.
  - Give each `exports` entry a `types` condition.
  - Build before `lpm publish --check`, because `--check` runs no lifecycle scripts.
- **Add the metadata the quality checks read:** `description`, `keywords`, `repository`, `exports`, `sideEffects`, `engines`, and a `license` matching a `LICENSE` file. List `CHANGELOG.md` in `files`: README and license files always ship, but a changelog ships only if `files` includes it.
- **React components:** put `react` and `react-dom` in `peerDependencies`.
- **CLI tools:** add a `bin` entry.
- **Keep `dependencies` minimal** and avoid `preinstall`/`install`/`postinstall` scripts. Both are scored.

Create these files:
- `src/index.{js,ts}` and a starter test.
- `README.md`, with an install code block naming the exact package and a usage example that imports it.
- `CHANGELOG.md`, with an entry for the current version.
- `LICENSE`.

For the README, CHANGELOG and doc-comment guidance, see [Improve](./improve.md). For a source package, also run the [Source Config](./source-config.md) workflow.

## Swift Package

Publishing needs both `Package.swift` and a `package.json` that identifies the release:

```json
{
  "name": "@lpm.dev/owner.package-name",
  "version": "0.1.0",
  "description": "{one sentence}",
  "license": "MIT",
  "keywords": ["{keyword}"],
  "repository": { "type": "git", "url": "{git remote}" },
  "files": ["Package.swift", "Sources", "Tests", "CHANGELOG.md"]
}
```

Package layout:
- `Package.swift`
- `Sources/<Name>/`
- `Tests/<NameTests>/`
- `README.md`, `CHANGELOG.md`, `LICENSE`

Add any resources or alternate manifests the package needs to `files`.

In `Package.swift`:
- Use `// swift-tools-version: 6.0` with no `.v5` language mode.
- Declare `platforms` with minimum versions.
- Expose a `.library` product.
- Add a test target using XCTest or Swift Testing.
- Give every dependency a version requirement, not a branch, revision or local path.
- Don't use `unsafeFlags`.

The Swift scaffold's README should show consumers both install methods:

```bash
lpm install @lpm.dev/owner.package-name
```

```swift
.package(id: "lpmdev.owner_package-name", from: "0.1.0")
// target dependency:
.product(name: "<ProductName>", package: "lpmdev.owner_package-name")
```

For the identity mapping, consumer setup and publishing commands, see [Swift Registry](./swift-registry.md#publishing).

## Verify

1. Run `lpm publish --check`:
   - It runs locally, with no scripts or network.
   - It lists every quality check, with tips for checks that lost points.
   - Checks that need Registry data show as checked after publish.
2. Fix failures with [Improve](./improve.md). The scoring model is in [Quality Checks](../references/quality-checks.md).
3. Optionally add Agent Skills in `lpm-skills/` (see [Skills](./skills.md)) and run `lpm skills validate`.
4. Report the result `--check` prints. Don't promise a score in advance.

New packages start as Private. For Free, Pool or Marketplace, see [Distribution](./distribution.md).
