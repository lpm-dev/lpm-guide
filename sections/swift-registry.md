# Swift Package Registry (SE-0292)

Guide for setting up and using LPM's native SPM integration. LPM implements SE-0292 so Swift Package Manager resolves LPM packages directly.

## Consumer Setup

### Normal install flow

```bash
lpm login
lpm install @lpm.dev/acme.swift-logger
```

`lpm install` is the primary workflow. Before it invokes `swift package resolve`, the CLI automatically:

1. Sets SPM's `lpmdev` scope to `https://lpm.dev/api/swift-registry`
2. Authenticates with the user's LPM token
3. Installs LPM's signing certificate to `~/.swiftpm/security/trusted-root-certs/lpm.der`
4. Configures signing trust for the `lpmdev` scope

### Repair or certificate refresh

```bash
lpm swift-registry
lpm swift-registry --force
```

Use the explicit command to repair Registry configuration. Use `--force` when the configuration or signing certificate is stale or corrupt, or after a documented certificate rotation. It is not a prerequisite for a normal install.

**Prerequisites:** Swift 5.9+ on PATH and `lpm login`. lpm.dev Swift package metadata and archives are access-controlled: private packages require owner/org access, Pool packages require Pool access, and Marketplace packages require a license.

### Manual setup

```bash
swift package-registry set --scope lpmdev https://lpm.dev/api/swift-registry
swift package-registry login https://lpm.dev/api/swift-registry --token $LPM_TOKEN --no-confirm
mkdir -p ~/.swiftpm/security/trusted-root-certs
curl -o ~/.swiftpm/security/trusted-root-certs/lpm.der https://lpm.dev/api/swift-registry/certificate
```

### Development (localhost)

```bash
lpm --registry http://localhost:3000 swift-registry
```

HTTP registries skip SPM login because SPM refuses auth over HTTP. `lpm swift-registry` still sets the scope and configures signing trust/certificate files for the local registry.

## Installing Packages

```bash
lpm install @lpm.dev/acme.swift-logger
```

The CLI automatically:
1. Fetches package metadata (version, SE-0292 identifier, product name)
2. Selects an eligible non-test target
3. Edits `Package.swift` — adds `.package(id:)` to dependencies and `.product(name:)` to the selected target
4. Configures the Registry scope, authentication, certificate, and signing trust when needed
5. Runs `swift package resolve`

One eligible target is selected automatically. With multiple eligible targets, a normal install prompts for the target. `-y` / `--yes` skips the prompt and selects the first eligible target, matching the selector's default.

### Result in Package.swift

```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "MyApp",
    platforms: [.macOS(.v12), .iOS(.v15)],
    dependencies: [
        .package(id: "lpmdev.acme_swift-logger", from: "1.0.0"),
        .package(url: "https://github.com/apple/swift-nio.git", from: "2.60.0"),
    ],
    targets: [
        .target(name: "MyApp", dependencies: [
            .product(name: "Logger", package: "lpmdev.acme_swift-logger"),
            .product(name: "NIOCore", package: "swift-nio"),
        ]),
    ]
)
```

You can also edit Package.swift manually — the `.package(id:)` syntax is standard SE-0292.

## Identity Mapping

LPM names map to SE-0292 identifiers:

| LPM                           | SE-0292                     |
| ----------------------------- | --------------------------- |
| `@lpm.dev/acme.swift-logger`  | `lpmdev.acme_swift-logger`  |
| `@lpm.dev/acme.design-system` | `lpmdev.acme_design-system` |

Pattern: `lpmdev.{owner}_{package-name}`. The underscore is reserved as the unambiguous owner/package separator.

## Package Signing

All published Swift packages are signed with CMS detached signatures (ECDSA P-256, `cms-1.0.0`).

- `lpm install` configures the certificate automatically when setup is needed
- `lpm swift-registry --force` repairs stale or corrupt configuration and refreshes the certificate
- Without cert: SPM warns `signer is not trusted` but still resolves
- To suppress: `swift package resolve --resolver-signing-entity-checking warn`

## Install vs Source Delivery

|                 | `lpm install`          | `lpm add`                            |
| --------------- | ---------------------- | ------------------------------------ |
| Use for         | Library dependencies   | UI components, templates             |
| Managed by      | SPM's resolver         | Developer (code copied into project) |
| Updates         | `swift package update` | Re-run `lpm add`                     |
| Transitive deps | Automatic              | Manual                               |

**Rule of thumb:** If you'd `import` it, use `lpm install`. If you'd modify it, use `lpm add`.

## Publishing Swift Packages

No extra steps needed. `lpm publish` from a directory with `Package.swift` automatically:

1. Detects `ecosystem: "swift"`
2. Extracts `Package.swift` metadata (targets, platforms, dependencies)
3. Creates a zip source archive for SPM download
4. Signs the archive with LPM's certificate

The published package works with both `lpm install` (SE-0292 registry) and `lpm add` (source delivery).

### Controlling Published Files

Swift build artifacts (`.build/`, `DerivedData/`) can be hundreds of megabytes. The CLI auto-excludes common Swift build directories, but **always add a `files` field** to `package.json` to explicitly whitelist what gets published:

```json
{
  "name": "@lpm.dev/owner.package-name",
  "version": "1.0.0",
  "files": [
    "Sources/",
    "Tests/",
    "Package.swift",
    "README.md",
    "LICENSE",
    "CHANGELOG.md",
    ".lpm/skills"
  ]
}
```

When `files` is present, **only listed paths** are included in the tarball (plus `package.json` which is always included). Without it, the CLI falls back to heuristic exclusions which may miss project-specific artifacts.

**What to include:**
- `Sources/` — required, your library code
- `Package.swift` — required, SPM manifest
- `Tests/` — recommended, demonstrates quality and lets consumers verify locally
- `README.md`, `LICENSE`, `CHANGELOG.md` — always auto-included, but listing them is explicit
- `.lpm/skills` — required if you have Agent Skills (otherwise they're silently excluded)

**What NOT to include:**
- `.build/` — Swift build artifacts (often 100MB+)
- `DerivedData/` — Xcode build cache
- `.swiftpm/` — local SPM state
- `*.xcworkspace/`, `*.xcodeproj/` — Xcode project files (consumers use Package.swift)
- `Pods/` — CocoaPods artifacts
- `node_modules/` — if using any JS tooling

## Troubleshooting

### "missing or invalid authentication credentials"

The normal `lpm install` flow configures authentication automatically. If the local Registry setup is stale or corrupt, run `lpm swift-registry --force`; otherwise check the inherited Swift output and `swift package-registry login`.

### "signer is not trusted" warning

Repair and refresh the signing certificate:

```bash
lpm swift-registry --force
```

### "no versions match the requirement"

Check that the package exists and has the version you need:

```bash
lpm info @lpm.dev/owner.package-name
```

### SPM login fails with "invalid URL"

SPM's `login` command requires HTTPS. For local development, use `.netrc` or skip login.
