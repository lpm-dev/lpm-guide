# LPM Source Package Config Generator

Generate `lpm.config.json` files that make LPM source packages configurable. This file controls how the `lpm add` CLI filters and delivers source files to consumers.

## Context

`lpm add` copies source files from a package tarball into the consumer's project. It works with any registry the resolver can reach — lpm.dev, npmjs.org, or a `.npmrc`-declared private registry. The `lpm.config.json` file lives at the tarball root and lets package authors define prompts, file-copy rules, dependency injection, and import rewriting.

lpm.dev package name format: `@lpm.dev/owner.package-name`

Consumer usage:
```bash
# Interactive — prompts for each option
lpm add @lpm.dev/acme.ui-kit

# URL params — pre-configured
lpm add "@lpm.dev/acme.ui-kit?component=dialog,button&styling=panda"

# Skip prompts, use defaults
lpm add @lpm.dev/acme.ui-kit --yes

# npm package using the same registry-agnostic source-delivery path
lpm add lpm-source-package
```

## Workflow

1. **Analyze the package structure** — read the directory tree to understand files and organization
2. **Identify configurable options** — what choices should consumers make?
3. **Generate lpm.config.json** — include `$schema` and target the current schema in [references/config-spec.md](../references/config-spec.md)
4. **Validate** — check against `https://cli.lpm.dev/schemas/lpm.config.json` or run `lpm schema lpm.config.json`

## Config Schema Field Types

| Type | UI | Value |
|------|------|-------|
| `"string"` | Free-form input | `"components/ui"` |
| `"select"` | Single dropdown | `"panda"` |
| `"select"` + `"multiSelect": true` | Checkbox list | `"dialog,button"` |
| `"boolean"` | Yes/No | `true` / `false` |

Key rules:
- `"required": true` — consumer MUST choose (prevents "include all" for this field)
- `"multiSelect": true` — consumer can pick multiple items
- When a param is NOT provided by the consumer, all matching files are INCLUDED (include-all-by-default)

## Example

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
      "options": [
        { "value": "dialog", "label": "Dialog" },
        { "value": "button", "label": "Button" }
      ]
    },
    "styling": {
      "type": "select",
      "required": true,
      "label": "Styling Framework",
      "options": [
        { "value": "panda", "label": "Panda CSS" },
        { "value": "tailwind", "label": "Tailwind CSS" }
      ],
      "default": "panda"
    }
  },
  "defaultConfig": {
    "styling": "panda"
  },
  "files": [
    {
      "src": "components/dialog/**",
      "dest": "components/ui/dialog/",
      "include": "when",
      "condition": { "component": "dialog" }
    },
    {
      "src": "components/button/**",
      "dest": "components/ui/button/",
      "include": "when",
      "condition": { "component": "button" }
    },
    {
      "src": "lib/tokens.js",
      "dest": "lib/tokens.js",
      "include": "always"
    },
    {
      "src": "styles/panda.config.js",
      "dest": "styles/config.js",
      "include": "when",
      "condition": { "styling": "panda" }
    },
    {
      "src": "styles/tailwind.config.js",
      "dest": "styles/config.js",
      "include": "when",
      "condition": { "styling": "tailwind" }
    }
  ],
  "dependencies": {
    "styling": {
      "panda": ["@pandacss/dev"],
      "tailwind": ["tailwindcss"]
    }
  }
}
```

## Best Practices

1. Use `"required": true` for fields where include-all doesn't make sense (e.g., `styling` where panda and tailwind configs conflict at the same `dest`)
2. Use `"multiSelect": true` for fields where users want multiple items (e.g., components)
3. Always provide `defaultConfig` or field-level defaults so `--yes` works
4. Use `"include": "always"` for shared utilities all components need
5. Use `"include": "never"` for internal/test files
6. Use directory globs (`**`) for component directories
7. `dest` is relative to consumer's install path, not project root
8. Declare `importAlias` if you use alias imports during development, e.g. `"@/components"`. The CLI rewrites matching imports to the consumer's detected or explicit `--alias` value.
9. Declare `dependencies` only when you want to control dependency injection yourself; declaring it opts out of the legacy `package.json` dependency fallback.

## Full Specification

For the complete lpm.config.json schema, all field types, validation rules, and detailed examples, see [references/config-spec.md](../references/config-spec.md).
