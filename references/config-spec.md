# `lpm.config.json` Specification

This file configures a source package that consumers install with `lpm add`. It sits at the root of the tarball. The live schema is at `https://cli.lpm.dev/schemas/lpm.config.json`, and `lpm schema lpm.config.json` prints it.

`lpm add`, `lpm publish` (including `--check`) and the Registry all validate the file strictly. An unknown key at any level is rejected. For a worked example, see [source-config.md](../sections/source-config.md).

## Top-level fields (all optional)

| Field | Type | Notes |
|---|---|---|
| `$schema` | URI string | Editor hint. |
| `type` | `"package"` or `"source"` | How the Registry classifies the package. If omitted, it's inferred. Other values are rejected. Has no effect on `lpm add`. |
| `ecosystem` | `"js"` or `"swift"` | Default `js`. Sets the default install directory. JS picks it from the framework (Next.js app router: `components/` if it exists, else `src/components/`; Next.js pages router, Vite, Remix: `src/components/`). Swift uses `Sources/<target>/`, or `Packages/LPMComponents/Sources/<target>/` for Xcode projects. |
| `importAlias` | string | The author's import prefix. See [Import alias](#import-alias). |
| `configSchema` | object | Prompt fields keyed by config key. |
| `defaultConfig` | object | Values for config keys. Overrides a field's `default`. |
| `files` | array | Ordered copy rules. |
| `dependencies` | object | Dependencies added based on config values. |

There is no `mcpConfig` field.

## `configSchema` field

| Property | Notes |
|---|---|
| `type` | `"string"` (default), `"boolean"` or `"select"`. |
| `label` | Prompt text. Defaults to the key. |
| `default` | String, boolean or number. |
| `required` | The field must end up with a non-empty value. See value resolution below. |
| `multiSelect` | Only meaningful with `select`. Selected values are joined with commas. |
| `options` | Required for `select`, with at least 1 item. Each item is a string or `{ "value", "label" }`. |

## Value resolution

Precedence: a query value (`?key=value`) wins over `defaultConfig.<key>`, which wins over the field's `default`.

**Interactive mode** prompts only for keys the query didn't give.
- Defaults are pre-filled.
- An optional field with no default can be left unset. For a multi-select, picking nothing sets it to empty, which skips its gated files.
- A required field must be answered. For a multi-select, that means picking at least 1 value.

**Non-interactive mode** applies when you pass `--yes` or `--json`, or there is no TTY.
- Defaults are applied to every field. Optional fields with no default stay unset.
- If a required field has no default:
  - a boolean becomes `false`
  - a select with exactly 1 option takes that option
  - anything else makes `lpm add` fail with a message asking for `?key=value`

**Value checks:**
- Every key must be declared in `configSchema`.
- A boolean must be `true` or `false`.
- Each select value must be one of the `options`.
- A select that isn't `multiSelect` takes exactly 1 value.

**Unset keys** pass every `when` condition, so all matching rules are included. They add no dependencies. An empty multi-select (`?component=`) counts as set, so rules conditioned on that key are skipped.

## `files`

**If `files` is omitted**, `lpm add` copies the file or directory named in `package.json#lpm.source`. Without that field, it copies every file except:
- `package.json` and `lpm.config.json`
- `node_modules`, `.git` and `.lpm`
- `test`, `tests` and `__tests__` directories
- the root `lpm-skills/` directory
- LPM project record files

**If `files` is present**, rules are evaluated in order, and each matching rule adds files.

| Property | Notes |
|---|---|
| `src` | Required. A relative path inside the tarball. It can't contain `..` or be absolute. It can be an exact file path (a directory path copies nothing), `dir/**` (recursive), or a `*` wildcard in the last segment only, such as `dir/*` or `dir/*.css`. A `**` in the middle of a path matches nothing. |
| `dest` | Relative to the consumer's install directory. For a glob `src`, `dest` is a directory, and subpaths below the glob's base are kept. For an exact `src`, a trailing `/` means a directory and the filename is kept. Without the `/`, it's the full target path. If `dest` is omitted, the source-relative path is used. |
| `include` | `"always"` (default), `"when"` or `"never"`. `never` only turns off this rule. It doesn't remove files that other rules match. |
| `condition` | Required with `"when"`. Every key must match. Values can be strings, booleans or numbers. For a multi-select, it matches if any selected value is equal. Unset keys pass. |

## `dependencies`

```json
{ "dependencies": { "icons": { "lucide": ["lucide-react"], "lpm": ["@lpm.dev/acme.icons@^1"] }, "withTests": { "true": ["vitest"] } } }
```

- The outer key is a config key and the inner key is a selected value. For booleans, the inner key is `"true"` or `"false"`. Each entry is a bare name or `name@range`.
- Bare names and dist-tags are resolved at add time, before `package.json` changes, and saved like `lpm install <pkg>` would save them. Ranges, exact versions and `*` are kept as written.
- Packages can come from any registry: npm, private registries declared in `.npmrc`, or `@lpm.dev/*`. `workspace:` specs are rejected.
- Declaring `dependencies` turns off the fallback to the package's own `package.json` `dependencies` and `peerDependencies`. This holds even when no branch matches.

## Import alias

- **End the alias with `/` and map it to the tarball root**, e.g. `"@/"` with tsconfig `"@/*": ["./*"]`. `lpm add` removes the prefix and looks up the rest among the delivered source files, from the tarball root, with optional extensions and `index` files: `@/components/button` finds `components/button.tsx`, and a file in `src/` is imported as `@/src/...`.
- **Prefix boundary.** Current LPM CLI counts trailing slashes as one, adds `/` to an alias ending in an ASCII letter or digit, `@`, `.`, `_` or `-` (`"@"` becomes `"@/"`, so `@radix-ui/*` stays untouched), keeps `#`, `~` or `/` as written, and drops one `/` after the prefix (with `"#"`, `#util` and `#/util` both work). Relative imports and `//` URLs are never claimed. An empty value is no alias. Older releases matched the raw string, so `"@/components"` produced `@/ui//button` and `"@"` claimed scoped packages. An alias ending in `/` works on both.
- **Found files are re-pointed through the consumer's alias**, or become relative imports when the consumer has none. The alias comes from `--alias`. Otherwise `lpm add` detects it from `paths` in `tsconfig.json` or `jsconfig.json` and asks the consumer to confirm; under `--yes`, `--json`, or no TTY it applies the detected alias without asking. At the prompt, Enter keeps the detected alias and `-` means relative imports; `--alias ''` also means relative imports. Rewritten imports keep the extension the author wrote (including `./x.js` for `x.ts`); through a consumer alias, `.ts`/`.tsx` are dropped when no other file shares the name.
- **Imports it does not find** get the consumer's alias in place of the prefix, or stay unchanged without one. Keep files that such imports need, such as `@/lib/utils`, in the consumer project.
- **Relative imports between copied files follow the files when they move.** They're re-pointed through the consumer's alias when one is set, and stay relative otherwise.
- Bare package imports are left alone unless they start with the prefix.
- Only JavaScript and TypeScript files are rewritten: not `.d.ts`, Swift, `.vue`, `.svelte` or `.css`.

## Registry publish limits

| Item | Limit |
|---|---|
| Config file size | 128 KiB |
| Each label, option value and `importAlias` | 200 characters |
| Options per field | 64 |
| `src` and `dest` | 512 characters each |
| File rules | 1,000 |
| Dependency entries | 200 characters each, 200 per array |
