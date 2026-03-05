# LPM Source Package Config Generator

Generate `lpm.config.json` files that make LPM source packages configurable. This file controls how the `lpm add` CLI filters and delivers source files to consumers.

## Context

LPM (lpm.dev) is a source code package registry. Unlike npm, LPM delivers source code directly into the consumer's project. The `lpm.config.json` file (placed alongside `package.json`) lets consumers choose which components to install, which styling framework to use, what colors to apply, etc.

Package name format: `@lpm.dev/owner.package-name`

Consumer usage:
```bash
# Interactive — prompts for each option
lpm add @lpm.dev/acme.ui-kit

# URL params — pre-configured
lpm add "@lpm.dev/acme.ui-kit?component=dialog,button&styling=panda"

# Skip prompts, use defaults
lpm add @lpm.dev/acme.ui-kit --yes
```

## Workflow

1. **Analyze the package structure** — read the directory tree to understand files and organization
2. **Identify configurable options** — what choices should consumers make?
3. **Generate lpm.config.json** — see [references/config-spec.md](../references/config-spec.md) for the full specification
4. **Validate** — check against validation rules in the spec

## Config Schema Field Types

| Type | UI | Value |
|------|------|-------|
| `"select"` | Single dropdown | `"panda"` |
| `"select"` + `"multiSelect": true` | Checkbox list | `"dialog,button"` |
| `"boolean"` | Yes/No | `"true"` / `"false"` |

Key rules:
- `"required": true` — consumer MUST choose (prevents "include all" for this field)
- `"multiSelect": true` — consumer can pick multiple items
- When a param is NOT provided by the consumer, all matching files are INCLUDED (include-all-by-default)

## Example

```json
{
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
3. Always provide `defaultConfig` so `--yes` flag works
4. Use `"include": "always"` for shared utilities all components need
5. Use `"include": "never"` for internal/test files
6. Use directory globs (`**`) for component directories
7. `dest` is relative to consumer's install path, not project root
8. Declare `"importAlias": "@/"` if you use alias imports during development. The CLI will intelligently rewrite internal imports for consumers. Relative imports also work and will be rewritten when the consumer provides their alias.

## Full Specification

For the complete lpm.config.json schema, all field types, validation rules, and detailed examples, see [references/config-spec.md](../references/config-spec.md).
