# LPM Package Migration

Move an existing npm package to lpm.dev, either fully or by publishing to both registries.

`lpm migrate` is a different thing: it converts a project's npm/pnpm/Yarn/Bun lockfile so the project can use LPM CLI as its package manager. It doesn't move a package to lpm.dev.

## Step 1: Analyze

Read these files:
- `package.json`
- `README.md`
- `.npmrc`
- `.github/workflows/*.yml`
- `tsconfig.json`
- `files`/`.npmignore`

Then check for these things:

- **Dependencies:** dependency rules differ for Free, Pool and Marketplace (see [Distribution](./distribution.md)). Flag any private or Marketplace dependencies early.
- **Types:** look for `types`/`typings`, shipped `.d.ts` files, and `declaration: true`. If there are none, tell the user that type declarations and doc comments are scored, and point them to [Improve](./improve.md).
- **Name:** see [Scaffold → Names](./scaffold.md#names).

## Step 2: Choose the Migration Type

| Type | When |
|------|------|
| **Full migration** | A new package, or the user is willing to deprecate the npm version |
| **Dual publish** | An established package that should keep its npm users |
| **Source migration** | Component libraries or UI kits that consumers copy in with `lpm add`. See [Source Config](./source-config.md) |

If the context doesn't make it clear, ask.

## Step 3: Transform package.json

The name becomes `@scope/package-name` or `package-name` → `@lpm.dev/owner.package-name`. The owner is the user's LPM username or organization. Ask if it isn't obvious.

Only add or update fields. Never remove ones that work. Fill in whatever is missing:
- `description`, `keywords`, `repository` and `license`, plus a `LICENSE` file.
- `exports`, `sideEffects` and `engines`.
- A `files` list that includes `CHANGELOG.md`.

Every path in `main`, `module`, `types`, `bin` and `exports` must name a file that ships.

**Dual publish:** keep the npm `name` in `package.json` and set per-registry names in `lpm.json`:

```json
{
  "publish": {
    "registries": ["npm", "lpm"],
    "npm": { "name": "package-name", "access": "public" },
    "lpm": { "name": "@lpm.dev/owner.package-name" }
  }
}
```

`lpm publish --yes` publishes to the configured targets. Any target flag, such as `--lpm` or `--npm`, replaces the list for that run.

Don't put publish names in `lpm.config.json`. That file only controls `lpm add` for source packages.

## Step 4: Update the README

- **Install commands:** change `npm install`/`yarn add`/`pnpm add package-name` to a code block running `lpm install @lpm.dev/owner.package-name`. For dual publish, show both the LPM and npm commands.
- **Imports:** update every `import`/`require` example (`from "old-name"`, `require("old-name")`, `from "@old-scope/old-name"`) to the new name.

## Step 5: CI/CD

Publishing with LPM CLI needs no `.npmrc`. Projects only need one when npm/pnpm/Yarn must install `@lpm.dev/*` dependencies.

For publishing from CI, see [Deploy](./deploy.md). It covers:
- installing the CLI
- Trusted Publisher (OIDC), which is the recommended setup
- `LPM_TOKEN`
- `.npmrc` setup for consumers

Keep npm publish credentials separate from lpm.dev credentials.

## Step 6: Verify

1. Run `lpm publish --check`. It reports archive points and a preflight percentage, not a final score out of 100. Checks that need Registry data run after publish. See [Quality Checks](../references/quality-checks.md).
2. Offer to fix failing checks with [Improve](./improve.md).
3. Summarize what changed and the next steps, such as the first `lpm publish` and CI setup.

## Important Notes

- Never deprecate or delete the npm package without explicit approval. If it has consumers, suggest an npm deprecation notice that points to the LPM version.
- New lpm.dev packages start as Private. Choosing Free, Pool or Marketplace is a separate, permanent decision; see [Distribution](./distribution.md).
