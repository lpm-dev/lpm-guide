# LPM Deploy Setup

Configure deployment platforms to install and/or publish LPM packages. Covers token setup, `.npmrc` configuration, caching, auto-publish workflows, and troubleshooting common build failures.

## Two Use Cases

| Role | Goal | Key Config |
|------|------|------------|
| **Consumer** | Install `@lpm.dev/` packages during build | `.npmrc` + `LPM_TOKEN` env var |
| **Author** | Auto-publish to LPM on push/tag | `LPM_TOKEN` + publish workflow |

## Workflow

1. **Detect the situation** — What platform, consumer or author, what's failing
2. **Generate .npmrc** — Registry configuration for `@lpm.dev` scope
3. **Configure the platform** — Token setup, environment variables
4. **Set up caching** (optional) — Speed up CI builds
5. **Set up auto-publish** (optional) — Publish on tag/release
6. **Verify** — Test the configuration

## Step 1: Detect the Situation

Check the current project:

- `package.json` — Does it depend on `@lpm.dev/` packages? (consumer) Does it have `name: "@lpm.dev/..."?` (author)
- `.npmrc` — Already configured for LPM?
- `.github/workflows/` — Existing CI config
- `.gitlab-ci.yml` — GitLab CI
- `vercel.json` — Vercel config
- `netlify.toml` — Netlify config
- `package-lock.json` / `pnpm-lock.yaml` / `yarn.lock` / `bun.lockb` — Package manager

Ask the user if not clear from context:
- Which platform are you deploying to?
- Are you installing LPM packages (consumer) or publishing (author) or both?
- Are you seeing a specific error?

## Step 2: Generate .npmrc

The `.npmrc` file tells npm/pnpm/yarn where to find packages. LPM supports two modes:

### Proxy Mode (Default)

LPM acts as your default registry — all packages (LPM + npm) are fetched through LPM for faster, edge-cached installs.

```ini
# LPM Registry (proxy mode) — all packages route through LPM
# Token provided via LPM_TOKEN environment variable
# This file is safe to commit. See: https://lpm.dev/docs/ci-setup
registry=https://lpm.dev/api/registry
//lpm.dev/api/registry/:_authToken=${LPM_TOKEN}
```

### Scoped Mode

Only `@lpm.dev` packages route through LPM. Use when you have another default registry (e.g., Artifactory).

```ini
# LPM Registry (scoped mode) — only @lpm.dev packages route through LPM
@lpm.dev:registry=https://lpm.dev/api/registry
//lpm.dev/api/registry/:_authToken=${LPM_TOKEN}
```

Use `lpm setup --scoped` or `lpm npmrc --scoped` to generate scoped mode config.

### Package Manager Variations

**npm / pnpm** — Standard `.npmrc` above works for both.

**yarn (v1)** — Same `.npmrc` format works.

**yarn (v2+/berry)** — Uses `.yarnrc.yml` instead:

```yaml
# Proxy mode
npmRegistryServer: "https://lpm.dev/api/registry"
npmAuthToken: "${LPM_TOKEN}"
```

Or for scoped mode:

```yaml
npmScopes:
  lpm.dev:
    npmRegistryServer: "https://lpm.dev/api/registry"
    npmAuthToken: "${LPM_TOKEN}"
```

**bun** — Uses `bunfig.toml`:

```toml
# Scoped mode (bun uses scoped config)
[install.scopes]
"@lpm.dev" = { token = "$LPM_TOKEN", url = "https://lpm.dev/api/registry" }
```

### .npmrc Placement

| Scenario | Location | Notes |
|----------|----------|-------|
| Single project | Project root `.npmrc` | Committed to git (token is env var, not hardcoded) |
| Monorepo | Root `.npmrc` | All workspaces inherit it |
| Global (not recommended) | `~/.npmrc` | Only for local dev, not CI |

### Git Safety

The `.npmrc` file is safe to commit because it references `${LPM_TOKEN}` — the actual token comes from the environment.

## Step 3: Platform Configuration

### Vercel

**Where to set the token:**
1. Go to Project Settings → Environment Variables
2. Add `LPM_TOKEN` with value from `lpm token rotate` or LPM dashboard
3. Set for Production, Preview, and Development environments

**Vercel auto-detects `.npmrc`** — No additional configuration needed. The install step uses the `.npmrc` in the project root.

**For monorepos with Vercel:**
- Place `.npmrc` in the monorepo root
- Vercel's root directory setting doesn't affect `.npmrc` lookup — npm walks up to find it

**Troubleshooting:**
- If using pnpm, make sure `installCommand` isn't overriding the default
- Vercel caches `node_modules` automatically — after adding `LPM_TOKEN`, trigger a fresh deploy

### Netlify

**Where to set the token:**
1. Go to Site Settings → Environment Variables
2. Add `LPM_TOKEN`

**For Netlify with npm:**
No extra config — `.npmrc` is picked up automatically.

**For Netlify with pnpm:**
Add to `netlify.toml`:
```toml
[build]
  command = "pnpm install --frozen-lockfile && pnpm build"
```

### GitHub Actions

**Where to set the token:**
1. Go to Repository Settings → Secrets and variables → Actions
2. Add `LPM_TOKEN` as a repository secret

**Consumer workflow (install LPM packages):**

```yaml
name: Build
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci
        env:
          LPM_TOKEN: ${{ secrets.LPM_TOKEN }}

      - run: npm run build
      - run: npm test
```

**Author workflow (auto-publish on tag):**

```yaml
name: Publish
on:
  push:
    tags: ['v*']

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci
        env:
          LPM_TOKEN: ${{ secrets.LPM_TOKEN }}

      - run: npm test

      - run: npx lpm publish
        env:
          LPM_TOKEN: ${{ secrets.LPM_TOKEN }}
```

**Combined workflow (install + publish):**

```yaml
name: CI
on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
        env:
          LPM_TOKEN: ${{ secrets.LPM_TOKEN }}
      - run: npm test

  publish:
    needs: test
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
        env:
          LPM_TOKEN: ${{ secrets.LPM_TOKEN }}
      - run: npx lpm publish
        env:
          LPM_TOKEN: ${{ secrets.LPM_TOKEN }}
```

### GitLab CI

**Where to set the token:**
1. Go to Settings → CI/CD → Variables
2. Add `LPM_TOKEN`, mark as masked and protected

```yaml
stages:
  - test
  - publish

test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test

publish:
  stage: publish
  image: node:20
  only:
    - tags
  script:
    - npm ci
    - npx lpm publish
```

### Other Platforms

**AWS Amplify:**
- Environment Variables in Amplify Console → App Settings → Environment Variables
- Add `LPM_TOKEN`

**Railway:**
- Variables tab in the service settings
- Add `LPM_TOKEN`

**Docker:**
- Pass as build arg: `docker build --build-arg LPM_TOKEN=$LPM_TOKEN .`
- In Dockerfile: `ARG LPM_TOKEN` then `RUN npm ci`
- Never store in the image layer — use multi-stage builds

**Generic CI:**
- Set `LPM_TOKEN` as an environment variable in your CI provider
- Ensure `.npmrc` exists in the project root
- Run `npm ci` (or equivalent) — the token is read from the environment

## Step 4: Caching (Optional)

### GitHub Actions

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'  # or 'pnpm' or 'yarn'
```

This caches the global npm/pnpm/yarn cache directory. For faster installs, also cache `node_modules`:

```yaml
- uses: actions/cache@v4
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### Vercel / Netlify

Both platforms cache `node_modules` automatically. No additional config needed.

### LPM CLI Cache

The LPM CLI caches downloaded packages in `~/.cache/lpm/`. To cache this in CI:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/lpm
    key: lpm-${{ hashFiles('package-lock.json') }}
```

## Step 5: Auto-Publish (Optional)

For package authors who want to publish automatically:

### Tag-Based Publishing

1. Bump version in `package.json`
2. Commit and tag: `git tag v1.2.0`
3. Push with tags: `git push --tags`
4. CI detects the tag and runs `lpm publish`

### Quality Gate

Add a quality check before publishing:

```yaml
- run: npx lpm publish --check --min-score 70
  env:
    LPM_TOKEN: ${{ secrets.LPM_TOKEN }}

- run: npx lpm publish
  env:
    LPM_TOKEN: ${{ secrets.LPM_TOKEN }}
```

This blocks publishing if the quality score drops below 70 (Good tier).

## Step 6: Verify

After configuration, verify the setup:

1. **Check .npmrc exists** and has the LPM registry configured (proxy or scoped mode)
2. **Check LPM_TOKEN is set** on the target platform
3. **Trigger a test build** to confirm `@lpm.dev/` packages install successfully
4. **For authors**: Push a test tag to verify auto-publish works

Print a summary:

```
Deploy setup complete for {platform}

Configuration:
  ✓ .npmrc — @lpm.dev scope configured
  ✓ LPM_TOKEN — Set in {platform} environment variables
  {✓ Auto-publish — Triggers on v* tags}
  {✓ Caching — node_modules and LPM cache configured}

Next steps:
  1. Get your token: lpm token rotate
  2. Add LPM_TOKEN={token} to {platform} → {location}
  3. Push a commit to trigger a build and verify
```

## Troubleshooting

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `401 Unauthorized` during install | `LPM_TOKEN` not set or expired | Set/refresh token in platform env vars |
| `404 Not Found` for `@lpm.dev/...` | `.npmrc` missing or wrong registry URL | Run `lpm setup` or create `.npmrc` manually |
| `ENORESOLVE` / `ERESOLVE` | Package version conflict | Check `package-lock.json` is committed and up to date |
| `ERR_SCOPE_NOT_FOUND` | npm doesn't know about `@lpm.dev` scope | Run `lpm setup` or add `registry=https://lpm.dev/api/registry` to `.npmrc` |
| `ECONNREFUSED` | Registry unreachable from CI | Check if CI has outbound network access, verify registry URL |
| Token works locally but not in CI | Different token or environment | Verify the exact token value in CI matches `lpm whoami` locally |

### Token Management

- Tokens are created with `lpm token rotate` or from the LPM dashboard
- Each environment (local, CI, staging, production) can use the same token or separate tokens
- Tokens don't expire by default but can be revoked from the dashboard
- If a token is compromised, rotate immediately: `lpm token rotate`

### Monorepo Tips

- Place `.npmrc` in the monorepo root — all workspaces inherit it
- Use a single `LPM_TOKEN` for the entire monorepo
- If different workspaces need different tokens (rare), use workspace-level `.npmrc` files
- Turborepo/Nx: No special config needed — they respect `.npmrc` from root

## MCP Server Setup (Optional)

If the user or their team uses AI tools (Claude Code, Cursor, etc.), they can install the LPM MCP server for AI-assisted package management:

```bash
npm install -g @lpm-registry/mcp-server
```

Add to MCP client config (e.g., `.claude/settings.json` or `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "lpm-registry": {
      "command": "npx",
      "args": ["@lpm-registry/mcp-server"],
      "env": {
        "LPM_TOKEN": "lpm_your_token_here"
      }
    }
  }
}
```

This gives the AI access to 6 tools: package info, quality reports, name checks, pool stats, marketplace search, and user info.

## Important Notes

- Never hardcode tokens in `.npmrc`, `Dockerfile`, or CI config files
- Always use environment variables (`${LPM_TOKEN}`) for token injection
- The `.npmrc` file with `${LPM_TOKEN}` is safe to commit to git
- If the user doesn't have a token yet, direct them to run `lpm login` then `lpm token rotate`
- For platforms not listed here, the pattern is always the same: `.npmrc` + `LPM_TOKEN` env var
