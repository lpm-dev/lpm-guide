# lpm.config.json Specification

This reference is for source packages consumed by `lpm add`. The live schema is published at `https://cli.lpm.dev/schemas/lpm.config.json`; when in doubt, prefer that schema or run:

```bash
lpm schema lpm.config.json
```

## Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `$schema` | `string` | No | Recommended editor hint: `https://cli.lpm.dev/schemas/lpm.config.json` |
| `ecosystem` | `"js"` or `"swift"` | No | Defaults to `"js"`. Drives default install-dir detection. |
| `importAlias` | `string` | No | Author's import alias prefix, e.g. `"@/components"`. No trailing slash requirement. |
| `configSchema` | `object` | No | Interactive config options for `lpm add`. |
| `defaultConfig` | `object` | No | Package-level defaults, useful for `lpm add --yes`. |
| `files` | `array` | No | Ordered file-copy rules with optional conditions and destinations. |
| `dependencies` | `object` | No | Conditional dependencies injected into the consumer project. |

There is no top-level `type` field and no `mcpConfig` field in the current schema.

## Minimal Example

```json
{
  "$schema": "https://cli.lpm.dev/schemas/lpm.config.json",
  "ecosystem": "js",
  "importAlias": "@/components",
  "configSchema": {
    "component": {
      "type": "select",
      "multiSelect": true,
      "label": "Components",
      "options": ["dialog", "button"],
      "default": "dialog",
      "required": true
    },
    "styling": {
      "type": "select",
      "label": "Styling framework",
      "options": ["panda", "tailwind"],
      "default": "panda",
      "required": true
    },
    "withTests": {
      "type": "boolean",
      "label": "Include tests?",
      "default": false
    }
  },
  "defaultConfig": {
    "component": "dialog",
    "styling": "panda",
    "withTests": false
  },
  "files": [
    {
      "src": "src/dialog/**",
      "dest": "dialog/",
      "include": "when",
      "condition": { "component": "dialog" }
    },
    {
      "src": "src/button/**",
      "dest": "button/",
      "include": "when",
      "condition": { "component": "button" }
    },
    {
      "src": "tests/**",
      "dest": "__tests__/",
      "include": "when",
      "condition": { "withTests": true }
    }
  ],
  "dependencies": {
    "styling": {
      "panda": ["@pandacss/dev"],
      "tailwind": ["tailwindcss", "autoprefixer"]
    },
    "withTests": {
      "true": ["vitest"]
    }
  }
}
```

## configSchema

Each key becomes an interactive prompt unless the user pre-answers it in the package spec query string.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"string"`, `"boolean"`, or `"select"` | Defaults to `"string"`. |
| `label` | `string` | Prompt label. Defaults to the field name. |
| `default` | `string`, `boolean`, or `number` | Initial value. Native JSON values are accepted. |
| `required` | `boolean` | With `--yes`, fill from the default instead of skipping. Also prevents accidental include-all behavior for conflicting choices. |
| `multiSelect` | `boolean` | Only for `select`; selected values become comma-joined. |
| `options` | `array` | Only for `select`; plain strings or `{ "value": "...", "label": "..." }` objects. |

## files

When `files` is omitted, `lpm add` copies every file in the tarball. When present, rules are evaluated in order.

| Property | Type | Description |
|----------|------|-------------|
| `src` | `string` | Required. Source path relative to the tarball root. Exact paths plus trailing `/**` and `/*` globs are supported. |
| `dest` | `string` | Destination relative to the resolved install directory. A trailing `/` treats the value as a directory and preserves the source filename. |
| `include` | `"always"`, `"when"`, or `"never"` | Defaults to `"always"`. |
| `condition` | `object` | Required for `include: "when"`. Keys match resolved config values. Booleans and numbers are accepted. |

When a condition key was not supplied by the consumer, matching rules are included by default. Use `required: true` plus defaults for choices where "include all" would create conflicts.

## dependencies

Dependencies are conditional by config key and value:

```json
{
  "dependencies": {
    "icons": {
      "lucide": ["lucide-react"],
      "heroicons": ["@heroicons/react"],
      "lpm": ["@lpm.dev/acme.icons@^1"]
    }
  }
}
```

Entries can be bare names or `name@range` specs. Bare names and dist-tags are resolved before mutating `package.json`; explicit ranges, exact versions, and wildcards are preserved. Names from npm, `.npmrc` private registries, and `@lpm.dev/*` packages all flow through the same registry-agnostic resolver.

Declaring `dependencies` opts out of the legacy fallback from the package's own `package.json` `dependencies` and `peerDependencies`, even if no branch matches.

## Import Alias

`importAlias` tells `lpm add` what prefix the author used internally. The CLI rewrites matching imports to the consumer's detected `tsconfig.json` paths or explicit `--alias` value.

```json
{ "importAlias": "@/components" }
```

Without `importAlias`, no alias-prefix rewrite is promised. Relative imports still work when the copied file layout preserves them.

## Consumer Usage

```bash
lpm add @lpm.dev/acme.kit
lpm add "@lpm.dev/acme.kit?component=dialog&styling=panda"
lpm add "@lpm.dev/acme.kit?component=dialog,button&styling=panda"
lpm add @lpm.dev/acme.kit --yes
```

`lpm add` can also consume npm-published source packages:

```bash
lpm add lpm-source-package
```

## File Placement

Place `lpm.config.json` at the tarball root:

```text
my-package/
├── package.json
├── lpm.config.json
├── src/
└── README.md
```
