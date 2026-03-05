# lpm.config.json Full Specification

## Table of Contents

- [Top-Level Fields](#top-level-fields)
- [configSchema](#configschema)
- [defaultConfig](#defaultconfig)
- [files](#files)
- [Include All by Default](#include-all-by-default)
- [dependencies](#dependencies)
- [Import Alias](#import-alias-smart-import-rewriting)
- [Validation Rules](#validation-rules)

---

## Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | `string` | No | Package type — determines install behavior. One of: `"source"`, `"mcp-server"`, `"vscode-extension"`, `"cursor-rules"`, `"github-action"`, `"other"`. Defaults to `"source"` if `lpm.config.json` exists. |
| `configSchema` | `object` | No | Defines interactive config options (prompts shown to consumer) |
| `defaultConfig` | `object` | No | Default values for config options |
| `files` | `array` | No | File rules with conditional includes and path mapping |
| `dependencies` | `object` | No | Conditional dependencies based on config choices |
| `importAlias` | `string` | No | Author's import alias prefix (e.g., `"@/"`, `"~/"`). Must end with `/`. Enables smart import rewriting during `lpm add`. |

---

## type

The `type` field declares what kind of developer tool this package is. It affects:
- **Install behavior** — the CLI routes to a type-specific handler (e.g., MCP servers configure editors, VS Code extensions install to the extensions directory)
- **Browse filtering** — users can filter by type on the package directory page
- **AI analysis** — the analysis prompt is tailored to the package type
- **Quality checks** — type-irrelevant checks are skipped (e.g., tree-shakable doesn't apply to MCP servers)

### Type-Specific Config

**MCP Server** (`"type": "mcp-server"`):
```json
{
  "type": "mcp-server",
  "mcpConfig": {
    "command": "node",
    "args": ["server.js"],
    "env": {
      "API_KEY": { "prompt": "Enter your API key", "required": true }
    }
  }
}
```
The `mcpConfig` specifies how to run the MCP server. The `env` field supports prompt definitions — the CLI asks the user for values during `lpm add` and writes them into the editor's MCP config.

**Cursor Rules** (`"type": "cursor-rules"`):
```json
{
  "type": "cursor-rules",
  "files": [
    { "src": "rules/**", "dest": ".cursor/rules/" }
  ]
}
```

**GitHub Action** (`"type": "github-action"`):
```json
{
  "type": "github-action",
  "files": [
    { "src": "action.yml", "dest": ".github/actions/deploy-aws/" },
    { "src": "workflow.yml", "dest": ".github/workflows/" }
  ]
}
```

**Other types** (`"vscode-extension"`, `"other"`) use the standard `files` array to define where content goes. The `type` field sets default install targets when `dest` is not specified.

---

## configSchema

Defines the interactive prompts shown to consumers. Each key becomes a config parameter.

```json
{
  "configSchema": {
    "component": {
      "type": "select",
      "multiSelect": true,
      "label": "Which components do you want?",
      "options": [
        { "value": "dialog", "label": "Dialog" },
        { "value": "button", "label": "Button" },
        { "value": "tabs", "label": "Tabs" }
      ]
    },
    "styling": {
      "type": "select",
      "label": "Styling framework",
      "required": true,
      "options": [
        { "value": "panda", "label": "Panda CSS" },
        { "value": "tailwind", "label": "Tailwind CSS" }
      ],
      "default": "panda"
    },
    "darkMode": {
      "type": "boolean",
      "label": "Include dark mode styles?",
      "default": true
    }
  }
}
```

### Field Types

| Type | Prompt UI | Value Format |
|------|-----------|-------------|
| `"select"` | Single-choice dropdown | String: `"panda"` |
| `"select"` with `"multiSelect": true` | Checkbox list | Comma-separated string: `"dialog,button"` |
| `"boolean"` | Yes/No confirm | String: `"true"` or `"false"` |

### Schema Entry Properties

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"select"` or `"boolean"` | Required. Field type. |
| `label` | `string` | Display label for the prompt. |
| `options` | `array` | Required for `"select"`. Array of `{ "value": "...", "label": "..." }` objects or plain strings. |
| `multiSelect` | `boolean` | For `"select"` type. Shows checkbox UI instead of radio. |
| `required` | `boolean` | If `true`, consumer MUST provide a value. Prevents "include all" behavior for this param. |
| `default` | `any` | Default value. Can also be set in `defaultConfig`. |

---

## defaultConfig

Default values for config options. Used when the consumer doesn't provide a value.

```json
{
  "defaultConfig": {
    "component": "button",
    "styling": "panda",
    "darkMode": true
  }
}
```

---

## files

Array of file rules that control which files get copied and where.

```json
{
  "files": [
    {
      "src": "components/dialog/**",
      "dest": "components/ui/dialog/",
      "include": "when",
      "condition": { "component": "dialog" }
    },
    {
      "src": "lib/utils.js",
      "dest": "lib/utils.js",
      "include": "always"
    },
    {
      "src": "internal/test-utils.js",
      "include": "never"
    }
  ]
}
```

### File Rule Properties

| Property | Type | Description |
|----------|------|-------------|
| `src` | `string` | Source path relative to package root. Supports `**` glob for directories. |
| `dest` | `string` | Destination path relative to consumer's install target. If omitted, uses `src`. If ends with `/`, preserves filename. |
| `include` | `string` | `"always"` (default), `"never"`, or `"when"`. |
| `condition` | `object` | Required when `include` is `"when"`. Object of `{ "configKey": "expectedValue" }`. Multiple keys = AND logic. |

### Src Glob Patterns

| Pattern | Matches |
|---------|---------|
| `"components/dialog/Dialog.jsx"` | Exact file |
| `"components/dialog/**"` | All files recursively in directory |

### Include Behavior

| `include` | Behavior |
|-----------|----------|
| `"always"` | Always included (default if `include` is omitted) |
| `"never"` | Never included (exclude internal/test files) |
| `"when"` | Included only when condition matches |

### Condition Format

Conditions use simple key-value objects. No expression parsing, no eval.

```json
// Single condition
{ "component": "dialog" }

// Multiple conditions (AND logic — both must match)
{ "component": "dialog", "styling": "panda" }
```

---

## Include All by Default

**This is the most important behavior to understand.**

When a condition key is NOT provided by the consumer (not in URL params, not answered in prompts), the file is INCLUDED regardless of condition value.

| Consumer provides param? | Config value | File condition | Result |
|--------------------------|-------------|----------------|--------|
| No | (any/default) | (any) | **INCLUDED** |
| Yes | `"dialog"` | `"dialog"` | INCLUDED |
| Yes | `"dialog"` | `"button"` | EXCLUDED |
| Yes | `"dialog,button"` | `"dialog"` | INCLUDED |
| Yes | `"dialog,button"` | `"tooltip"` | EXCLUDED |

**Example:**
```bash
# No component filter → ALL components installed
lpm add "@lpm.dev/acme.kit?styling=panda"

# Only dialog installed
lpm add "@lpm.dev/acme.kit?component=dialog&styling=panda"

# Dialog + button installed
lpm add "@lpm.dev/acme.kit?component=dialog,button&styling=panda"
```

**Exception:** If a configSchema field has `"required": true`, the consumer must provide a value (or the default is used and treated as explicitly provided). This prevents "include all" for fields where including all doesn't make sense (e.g., `styling` where multiple files map to the same `dest`).

---

## dependencies

Conditional dependencies based on config choices.

```json
{
  "dependencies": {
    "styling": {
      "panda": ["@pandacss/dev"],
      "tailwind": ["tailwindcss", "autoprefixer"]
    },
    "iconLibrary": {
      "lucide": ["lucide-react"],
      "heroicons": ["@heroicons/react"],
      "custom": ["@lpm.dev/acme.icons"]
    }
  }
}
```

- Regular npm packages → installed via detected package manager (`npm install`, `pnpm add`, etc.)
- `@lpm.dev/` packages → shown as manual install commands (`lpm add @lpm.dev/...`)
- Dependencies are deduplicated automatically

---

## Import Alias (Smart Import Rewriting)

The `importAlias` field declares the import alias prefix the author uses during development. When a consumer runs `lpm add`, the CLI uses this to identify internal imports and rewrite them to match the consumer's project setup.

### How It Works

1. **Author declares their alias** in `lpm.config.json`:
   ```json
   { "importAlias": "@/" }
   ```

2. **Consumer runs `lpm add`** and is prompted:
   ```
   ? Import alias for this directory? @/components/design-system
   ```
   The CLI auto-detects from tsconfig/jsconfig and pre-fills the suggestion.

3. **CLI rewrites internal imports**:
   ```javascript
   // Author wrote:
   import { cn } from "@/lib/utils"
   import { DialogStyle } from "./Dialog.style"

   // Consumer gets (with alias @/components/design-system):
   import { cn } from "@/components/design-system/lib/utils"
   import { DialogStyle } from "@/components/design-system/components/dialog/Dialog.style"
   ```

4. **External imports are never touched**: `react`, `next/link`, `@radix-ui/dialog`, etc.

### When to Use

- Use when your package has multiple files that import each other
- Use when you use alias imports (like `@/`) during development
- Not needed for single-file packages or packages with no internal cross-references

### Without `importAlias`

If `importAlias` is not set, the CLI still handles the buyer's alias prompt:
- Relative imports between internal files are rewritten to use the buyer's alias
- If the buyer skips the alias prompt, relative imports are left as-is (they already work)
- The legacy `@/` prefix swap is used as a fallback when no aliases are configured

---

## Validation Rules

The CLI validates `lpm.config.json` and rejects invalid configs:

| Rule | Constraint |
|------|-----------|
| File size | Max 1 MB |
| File rules count | Max 1000 |
| `src` paths | Must be relative (no leading `/`), no path traversal (`..`) |
| `dest` paths | Must be relative (no leading `/`), no path traversal (`..`) |
| `configSchema` types | Must be `"select"` or `"boolean"` |
| `select` fields | Must have non-empty `options` array |
| `include` values | Must be `"always"`, `"never"`, or `"when"` |
| `include: "when"` | Must have a `condition` object |
| `importAlias` value | Must be a string ending with `/` (e.g., `"@/"`, `"~/"`) |

---

## File Placement

The `lpm.config.json` file goes in the package root, alongside `package.json`:

```
my-package/
├── package.json
├── lpm.config.json    ← HERE
├── components/
│   ├── dialog/
│   └── button/
├── styles/
│   ├── panda.config.js
│   └── tailwind.config.js
└── lib/
    └── utils.js
```

When the package is published with `lpm publish`, the `lpm.config.json` is included in the tarball and read by the CLI during `lpm add`.
