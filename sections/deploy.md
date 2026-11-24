# LPM Deploy Setup

Configure CI/CD and deployment platforms to install or publish lpm.dev packages. This section is the canonical home for `.npmrc`, `LPM_TOKEN`, and Trusted Publisher guidance.

## Credentials

| Operation | Credential |
| --- | --- |
| Install public npm or Free `@lpm.dev/*` packages | None |
| Install Private, Pool, or Marketplace packages | Read token as `LPM_TOKEN`, created at https://lpm.dev/dashboard/settings/tokens (or org token settings). Expires after 7, 30, 60, or 90 days |
| Publish from GitHub Actions or GitLab.com | **Trusted Publisher** (OIDC), no stored token. Preferred |
| Publish from other CI (Bitbucket, etc.) | Publish token restricted to the package, as `LPM_TOKEN` |

Use a separate read token per deployment context, not an `lpm login` session or a publish token. `lpm token-rotate` rotates the local session and never prints a token, so it cannot produce a CI secret. Secrets go in: GitHub (Settings → Secrets and variables → Actions), Vercel (Settings → Environment Variables), Netlify (Site configuration → Environment variables), GitLab (Settings → CI/CD → Variables, protected).

## Install with LPM CLI (projects with `lpm.lock`)

LPM CLI reads `LPM_TOKEN` directly and needs no `.npmrc`. Commit `package.json`, `lpm.lock`, and `lpm.json` if present; never commit `.lpm/`.

```yaml
permissions: { contents: read }
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with: { node-version: 22 }
  - run: npm install -g @lpm-registry/cli@<version>   # pin a reviewed version
  - uses: actions/cache@v4
    with:
      path: ~/.lpm/store
      key: lpm-store-${{ runner.os }}-${{ runner.arch }}-<version>-${{ hashFiles('lpm.lock') }}
  - run: lpm ci --strict-integrity
    env: { LPM_TOKEN: "${{ secrets.LPM_TOKEN }}" }
  - run: lpm run build
```

- A cache miss is fine; don't add `--offline` (it fails when store data is missing). Use `lpm fetch` then `lpm ci --offline` only for intentionally pre-warmed jobs.
- Cache `~/.lpm/store`, never `node_modules` or the whole LPM home (it holds login state and env secrets).
- OIDC does not authenticate `lpm ci` downloads; use `LPM_TOKEN`.
- Vercel: install command `npm install -g @lpm-registry/cli && lpm ci`. Netlify: build command `npm install -g @lpm-registry/cli && lpm ci && npm run build`, only when the site leaves dependency install to the build command (its automatic install still needs `.npmrc`).
- Docker: pass the token with a BuildKit secret mount (`RUN --mount=type=secret,id=lpm_token ...`), never `ARG`/`ENV`; add `.npmrc`, `.lpm`, `.env*` to `.dockerignore`.

## Install with npm, pnpm, Yarn 1, or Bun

Commit only the routing line (keep the trailing slash; Bun drops `/api/registry` without it):

```ini
@lpm.dev:registry=https://lpm.dev/api/registry/
```

Add the token in the job, outside the checkout:

```bash
npm config set "//lpm.dev/api/registry/:_authToken" "$LPM_TOKEN"
```

Alternatives:

- `lpm setup ci npmrc` writes `//lpm.dev/api/registry/:_authToken=<literal token>` plus the routing line into the project `.npmrc` (mode 0600) and adds it to `.git/info/exclude`. If Git tracks `.npmrc`, it writes to npm's user config instead (CLI 0.87.0+), and stops if the tracked file already has an LPM token line. `--oidc` exchanges the job's OIDC identity for a 30-minute read token (GitHub Actions with `id-token: write`, or GitLab; not on PR triggers; the CI actor must be linked to an LPM account).
- Platforms whose own install step reads a committed `.npmrc` (Vercel, Netlify) can commit `//lpm.dev/api/registry/:_authToken=${LPM_TOKEN}`. npm, Yarn 1, and Bun expand it; pnpm 10.34.2+/11.5.3+ ignores it, so set `PNPM_CONFIG_NPMRC_AUTH_FILE=.npmrc` (pnpm 11+) or run `npm config set ...` in the install command. `lpm setup` refuses projects that commit this line.
- Yarn 2+: `.yarnrc.yml` with `npmScopes: { lpm.dev: { npmRegistryServer: "https://lpm.dev/api/registry/" } }` and `yarn config set --home 'npmScopes["lpm.dev"].npmAuthToken' "$LPM_TOKEN"`.
- Local machine: `lpm setup local --days <1-90>` writes a new read-only token to the project `.npmrc` and adds `/.npmrc` to `.gitignore` (if Git tracks `.npmrc`, it writes to npm's user config instead).

Never commit a literal token, and never upload or cache `.npmrc` / `~/.npmrc`.

## Publish from CI

**Trusted Publisher (preferred).** Publish the package once manually, then in the package dashboard open **Trusted publishers** and enter the repository (owner/name or GitLab project path), workflow or CI config filename, allowed ref (e.g. `refs/tags/v*`), and environment if used. Only `push`, `release`, `workflow_dispatch`, `schedule`, and `repository_dispatch` triggers (GitLab: `push`, `web`, `schedule`, `api`, `trigger`) can exchange a token.

Changes to the install workflow above:

```yaml
on: { push: { tags: ["v*"] } }
permissions: { id-token: write, contents: read }
# ...install and test steps...
  - run: lpm publish --check
  - run: lpm publish --yes   # add --provenance for a Sigstore provenance bundle
```

- The publish step needs no `LPM_TOKEN`; give a read token only to an install step that needs private packages.
- With `id-token: write` (or GitLab `LPM_OIDC_TOKEN`), `lpm publish` always uses the Trusted Publisher exchange and does not fall back to `LPM_TOKEN`.
- GitLab: `id_tokens: { LPM_OIDC_TOKEN: { aud: https://lpm.dev } }`; `--provenance` also needs `SIGSTORE_ID_TOKEN` with `aud: sigstore`.
- `--yes` authorizes lifecycle scripts and `--ignore-scripts` skips them; non-interactive publishes fail with `confirmation_required` if scripts exist and neither is passed.
- `--min-score N` gates on the share of archive-check points (registry-only checks run after publish).
- Trusted publishing and provenance earn Provenance quality points; see [Quality Checks](../references/quality-checks.md).

**Static token.** CI without OIDC: `LPM_TOKEN` (see [Credentials](#credentials)), then `lpm publish --yes`.

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `401` | `LPM_TOKEN` missing, expired, or not passed to that step | Create a new read token; check the step's `env`; `lpm whoami` locally |
| `404` for `@lpm.dev/*` | Missing routing line, wrong working directory, or no access (Pool subscription, Marketplace license, org membership) | Add the `@lpm.dev:registry` line at the project/workspace root; confirm access |
| Trusted Publisher exchange rejected | PR trigger, or repo ID / workflow / ref / environment mismatch | Fix the trusted publisher config or trigger |
| Installs or publishes rejected from CI only | Organization IP allowlist | Allow the runner's address |
