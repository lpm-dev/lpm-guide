# LPM Package Migration

Migrate an existing npm package to LPM format. Handles renaming, package.json updates, README changes, CI configuration, and optionally sets up dual publishing to both npm and LPM.

## Workflow

1. **Analyze the existing package** — Read package.json, understand current setup
2. **Determine migration type** — Full migration or dual publishing
3. **Transform package.json** — Rename, add LPM fields, preserve existing config
4. **Update documentation** — README install instructions, badges
5. **Configure CI/CD** — `.npmrc` setup, `LPM_TOKEN` environment variable
6. **Verify** — Run quality checks to show the package's LPM score

## Step 1: Analyze Existing Package

Read and assess the current package:

- `package.json` — Current name, fields, scripts, dependencies
- `README.md` — Install instructions that need updating
- `.npmrc` — Existing registry configuration
- `.github/workflows/*.yml` — CI that may need LPM publish steps
- `tsconfig.json` — TypeScript setup
- `.npmignore` / `files` field — What gets published

Check for migration blockers:
- Is the name already taken on LPM? Run `lpm check-name owner.package-name` to verify availability.
- Does it have private dependencies that would block Pool/Marketplace?
- Is it a scoped package (`@scope/name`) that needs renaming?

Check types status:
- Does package.json have a `types` or `typings` field?
- Do any `.d.ts` or `.d.mts` files exist?
- Does `tsconfig.json` exist with `declaration: true`?
- If none of the above: note the opportunity — "This package doesn't have type definitions. Adding types improves IntelliSense for your users and earns +8-12 quality points. See [Improve](./improve.md) for options."

## Step 2: Determine Migration Type

| Type | Description | When to use |
|------|-------------|-------------|
| **Full migration** | Replace npm with LPM entirely | New package or willing to deprecate npm version |
| **Dual publish** | Publish to both npm and LPM | Established package, want to keep npm users |
| **Source migration** | Convert to LPM source package (`lpm add`) | Component libraries, UI kits |

Ask the user which approach they want if not clear from context.

## Step 3: Transform package.json

### Name Conversion

```
npm format:          @scope/package-name  →  @lpm.dev/owner.package-name
npm unscoped:        package-name         →  @lpm.dev/owner.package-name
```

The owner is the user's LPM username or org name. Ask if not obvious.

### Fields to Add/Update

```json
{
  "name": "@lpm.dev/owner.package-name",
  "homepage": "https://lpm.dev/owner.package-name",
  "repository": { "type": "git", "url": "..." },
  "exports": { ".": "./src/index.js" },
  "type": "module",
  "sideEffects": false,
  "engines": { "node": ">=18" },
  "files": ["src", "dist", "README.md", "CHANGELOG.md", "LICENSE"]
}
```

**Preserve existing fields** — Don't remove anything that works. Only add missing fields and update the name.

### For Dual Publishing

Keep the original `name` in package.json. Instead, create an `lpm-package.json` override or use the `lpm publish` CLI which reads `lpm.config.json` for name overrides.

Alternatively, maintain two branches or use a publish script:

```json
{
  "scripts": {
    "publish:npm": "npm publish",
    "publish:lpm": "LPM_NAME=@lpm.dev/owner.package-name lpm publish"
  }
}
```

## Step 4: Update Documentation

### README Changes

Find and update install instructions:

| Before | After |
|--------|-------|
| `npm install package-name` | `lpm install @lpm.dev/owner.package-name` |
| `yarn add package-name` | `lpm install @lpm.dev/owner.package-name` |
| `pnpm add package-name` | `lpm install @lpm.dev/owner.package-name` |
| `import { x } from "package-name"` | `import { x } from "@lpm.dev/owner.package-name"` |

For dual publishing, show both options:

```markdown
## Installation

### From LPM
\`\`\`bash
lpm install @lpm.dev/owner.package-name
\`\`\`

### From npm
\`\`\`bash
npm install package-name
\`\`\`
```

### Import Paths

Update any import examples in the README to use the new package name. Scan for patterns like:
- `from "old-package-name"`
- `require("old-package-name")`
- `from "@old-scope/old-name"`

## Step 5: Configure CI/CD

### .npmrc Setup

Create or update `.npmrc` for LPM registry:

```ini
# LPM Registry
@lpm.dev:registry=https://lpm.dev/api/registry
//lpm.dev/api/registry/:_authToken=${LPM_TOKEN}
```

For dual publishing, append to existing `.npmrc` — don't replace npm registry config.

### GitHub Actions

If `.github/workflows/` exists, offer to add an LPM publish step:

```yaml
- name: Publish to LPM
  run: npx lpm publish
  env:
    LPM_TOKEN: ${{ secrets.LPM_TOKEN }}
```

### Environment Variables

Remind the user to set `LPM_TOKEN` on their deployment platform:
- **GitHub Actions**: Repository Settings → Secrets → `LPM_TOKEN`
- **Vercel**: Project Settings → Environment Variables → `LPM_TOKEN`
- **Netlify**: Site Settings → Environment Variables → `LPM_TOKEN`

Get the token via `lpm token rotate` or from the LPM dashboard.

For full CI/CD setup details, see [Deploy](./deploy.md).

## Step 6: Verify

After migration:

1. Run `lpm publish --check` to see the quality score
2. List any failed checks and offer to fix them (see [Improve](./improve.md))
3. Print a migration summary:

```
Migration complete: package-name → @lpm.dev/owner.package-name

Changes made:
  ✓ package.json — name, homepage, exports, engines, files
  ✓ README.md — updated install instructions
  ✓ .npmrc — added LPM registry configuration

Quality score: {score}/100 ({tier})

Next steps:
  1. Set LPM_TOKEN in your CI environment
  2. Run "lpm publish" to publish your first version
  3. (Optional) Run "improve package" to boost your quality score
```

## Migration Checklist

- [ ] package.json name updated to `@lpm.dev/owner.package-name`
- [ ] Missing quality fields added (exports, engines, files, sideEffects)
- [ ] README install instructions updated
- [ ] README import examples updated
- [ ] .npmrc configured for LPM registry
- [ ] CI/CD updated with LPM publish step (if applicable)
- [ ] `LPM_TOKEN` environment variable documented
- [ ] Types status assessed (has .d.ts, TypeScript, or JSDoc annotations)
- [ ] Quality check passed (`lpm publish --check`)

## Important Notes

- Never delete the existing npm package without explicit user approval
- Preserve all existing package.json fields — only add/update, don't remove
- For dual publishing, make sure both registries are configured in `.npmrc`
- The `@lpm.dev/` scope is registered on npm's registry, so `npm install @lpm.dev/...` works after `lpm setup`
- If the package has consumers, suggest a deprecation notice on npm pointing to the LPM version
- Check that all dependencies are available (LPM packages or npm packages both work)
