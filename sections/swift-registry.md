# Swift Package Registry (SE-0292)

LPM implements SE-0292, so Swift Package Manager resolves lpm.dev packages natively. Requires a compatible Swift toolchain on `PATH` (Xcode projects: select the intended developer directory).

## Install

From the directory with `Package.swift`, `.xcodeproj`, or `.xcworkspace`:

```bash
lpm login                                   # only for Private, Pool, Marketplace packages
lpm install @lpm.dev/acme.swift-logger      # @^1.2.0 saves from:, @1.2.0 saves exact:; --yes skips the target prompt
```

`lpm install` sets up the registry automatically, then edits the manifest and runs resolution:

1. Points the `lpmdev` scope at `https://lpm.dev/api/swift-registry` (project scope; Xcode projects also get the user-wide scope)
2. Logs SwiftPM in with the current LPM credential (skipped for Free releases and HTTP)
3. Installs the signing roots bundled in LPM CLI (no certificate download) into `<SwiftPM dir>/security/lpm-trusted-roots/lpm.dev` and removes the legacy `trusted-root-certs/lpm.der`
4. Writes the user-wide trust policy in `<SwiftPM dir>/configuration/registries.json`: for lpm.dev, `onUnsigned` and `onUntrustedCertificate` are `error` and only the bundled roots are trusted. It also sets SwiftPM's default policy to `error`, so unsigned packages from other registries fail too
5. Adds `.package(id:)` and `.product(name:package:)` to one eligible target (regular, executable, or macro; prompts when several, `--yes` picks the first)

SwiftPM dir: macOS `~/Library/org.swift.swiftpm`; Linux/Windows `$XDG_CONFIG_HOME/swiftpm` or `~/.swiftpm`.

Result:

```swift
.package(id: "lpmdev.acme_swift-logger", from: "1.0.0"),
// in the target:
.product(name: "Logger", package: "lpmdev.acme_swift-logger"),
```

Xcode projects: LPM creates a `Packages/LPMDependencies` wrapper package linked into the app target. Commit the wrapper, project changes, and `Package.resolved`.

Use `lpm install` for Swift libraries; transitive LPM and Git dependencies resolve through SwiftPM. `lpm add` copies source files into the project instead and adds no dependency or manifest entry.

**Repair:** `lpm swift-registry` re-applies scope, login, roots, and trust policy. `--force` also rewrites signing roots even when they already match. Manual `swift package-registry set` does not configure LPM's signing trust, so use the command.

**CI:** provide a read-scoped `LPM_TOKEN` (see [Deploy](./deploy.md)) and run `lpm swift-registry --json` before `swift package resolve` or `xcodebuild -resolvePackageDependencies`. A CLI token alone does not log SwiftPM in.

**Local registry:** `lpm swift-registry --registry http://localhost:3000`. HTTP works only for loopback addresses, skips authentication, installs no pinned roots, and accepts any signer.

## Identity Mapping

`@lpm.dev/{owner}.{package-name}` → `lpmdev.{owner}_{package-name}` (e.g. `@lpm.dev/acme.swift-logger` → `lpmdev.acme_swift-logger`). The dot becomes an underscore; hyphens stay.

## Signing

LPM.dev Registry signs each Swift source archive with a detached CMS signature (ECDSA P-256, SHA-256) under an offline LPM.dev root. Unsigned or untrusted archives **fail to resolve**. The pinned roots cover source archives only; SwiftPM accepts manifests signed by any signer. The signature proves the Registry signed the archive bytes, not which author made it.

## Publishing

Needs a root `Package.swift` and a `package.json` with the `@lpm.dev/owner.package-name` name and version (template: [Scaffold → Swift Package](./scaffold.md#swift-package)).

```bash
swift build && swift test
lpm publish --lpm --check
lpm publish --lpm --yes --wait
```

- LPM CLI inspects `Package.swift` with the local toolchain and uploads the archive. LPM.dev Registry inspects it, builds the Swift source ZIP, and signs it server-side; publishers never hold the signing key. `--wait` reports a rejection (e.g. compiler output); a release rejected during preparation can be republished with the same version.
- `files` controls inclusion. Literal `Sources`/`Tests` entries skip matching directories under `.build`. Without `files`, `.npmignore` or `.gitignore` decides. `package.json`, README, and license files always ship; list `CHANGELOG.md` and any resources explicitly.
- Valid Agent Skills in `lpm-skills/` ship automatically regardless of `files`; see [Skills](./skills.md).
- New packages start Private; see [Distribution](./distribution.md) before choosing Free, Pool, or Marketplace.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `401` / "missing or invalid authentication credentials" | `lpm login`, then rerun `lpm swift-registry` (or `lpm install`) |
| "signer is not trusted" for an lpm.dev release | Update LPM CLI (a newer root needs a newer CLI), then rerun `lpm swift-registry` |
| Setup refuses a project with its own signing settings | Remove the `security` section from the named `.swiftpm/configuration/registries.json` (or Xcode `xcshareddata` copy) |
| Roots path missing or not a directory | `lpm swift-registry` |
| Malformed or linked SwiftPM config | Fix the reported file, then rerun |
| "no versions match the requirement" | `lpm info @lpm.dev/owner.package-name` to confirm the version; check access (subscription, license, org) |
| Swift not found / no compatible library product | Install a toolchain on `PATH`; check the package's products and tools version |
