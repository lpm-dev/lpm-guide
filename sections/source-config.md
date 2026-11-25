# Source Package Config (`lpm.config.json`)

Write `lpm.config.json` so `lpm add` can ask the consumer questions, copy only the matching files, add dependencies and rewrite imports. Put it at the root of the tarball. Every field, rule and limit is in [config-spec.md](../references/config-spec.md). This file covers only the workflow.

`lpm add` copies source from any registry the resolver can reach: lpm.dev, npmjs.org, or a registry declared in `.npmrc`.

## Workflow

1. List the package files. Decide which choices consumers make, such as components, styling or tests.
2. Write the config:
   - `$schema`
   - one `configSchema` field per choice
   - ordered `files` rules that use `include: "when"` conditions
   - `dependencies` keyed by each choice
3. Validate with `lpm publish --check`. `lpm schema lpm.config.json` only prints the schema.

## Example

The author's `tsconfig` maps `@/*` to the package root, so a source file imports `@/lib/tokens`.

```json
{
  "$schema": "https://cli.lpm.dev/schemas/lpm.config.json",
  "importAlias": "@/",
  "configSchema": {
    "component": {
      "type": "select", "multiSelect": true, "label": "Components",
      "options": [{ "value": "dialog", "label": "Dialog" }, { "value": "button", "label": "Button" }]
    },
    "styling": {
      "type": "select", "required": true, "label": "Styling", "default": "panda",
      "options": ["panda", "tailwind"]
    }
  },
  "files": [
    { "src": "components/dialog/**", "dest": "dialog/", "include": "when", "condition": { "component": "dialog" } },
    { "src": "components/button/**", "dest": "button/", "include": "when", "condition": { "component": "button" } },
    { "src": "lib/tokens.js", "dest": "lib/tokens.js" },
    { "src": "styles/panda.config.js", "dest": "styles/config.js", "include": "when", "condition": { "styling": "panda" } },
    { "src": "styles/tailwind.config.js", "dest": "styles/config.js", "include": "when", "condition": { "styling": "tailwind" } }
  ],
  "dependencies": {
    "styling": { "panda": ["@pandacss/dev"], "tailwind": ["tailwindcss"] }
  }
}
```

## Design rules

- **Make a choice `required` when copying every option would clash.** The two style configs above both write `styles/config.js`. Give a required field a `default` so `--yes` still works.
- **Shared code goes in a rule without `include`**, which defaults to `always`. To leave files out, don't match them. `include: "never"` only turns off its own rule.
- **End `importAlias` with `/` and map it to the tarball root** ([details](../references/config-spec.md#import-alias)). With no alias, use relative imports.
- **Put an `lpm add <exact-name>` code block in the README.** It counts as the install example for source packages.
- Unset keys, `dest`, and the `dependencies` fallback: see [config-spec.md](../references/config-spec.md).

## Consumer commands

```bash
lpm add @lpm.dev/acme.ui-kit                                         # prompts
lpm add "@lpm.dev/acme.ui-kit?component=dialog,button&styling=panda" # preset values
lpm add @lpm.dev/acme.ui-kit --yes                                   # defaults, no prompts
lpm add lpm-source-package                                           # npm-published source package
```

Useful flags:

- `--path <dir>` sets the destination directory.
- `--alias <prefix>` sets the consumer's import alias.
- `--pm lpm|npm|pnpm|yarn|bun|auto` picks the package manager.
- `--dry-run` shows what would happen without changing anything.
- `--force` overwrites existing files without asking.
- `--no-install-deps` skips the consumer's dependency changes.
- `--target <NAME>` names the Swift target.
