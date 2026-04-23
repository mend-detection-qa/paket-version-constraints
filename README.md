# paket-version-constraints

## Probe metadata

| Field | Value |
|---|---|
| Pattern | paket-version-constraints |
| Target framework | net8.0 |
| NuGet source | https://api.nuget.org/v3/index.json |
| Storage mode | none |
| Dependency groups | Main only (no named groups) |
| Generated | 2026-04-22 |

## Feature exercised

This probe exercises all four version constraint syntaxes supported by Paket in a single
`paket.dependencies` file: exact pin (`==`), pessimistic operator (`~>`), open range
(`>= X < Y`), and the `prerelease` keyword. The `paket.lock` always records exact resolved
versions regardless of which constraint syntax was used in `paket.dependencies`. The probe
validates that Mend's UA engine parses `paket.lock` correctly for all constraint varieties —
the constraint syntax itself must not cause parsing failures or missed packages.

## Expected dependency tree

### Direct dependencies (Main group)

| Package | Constraint in paket.dependencies | Resolved version | Constraint type | Has transitives |
|---|---|---|---|---|
| Newtonsoft.Json | `== 13.0.3` | 13.0.3 | Exact pin | No |
| Serilog | `~> 3.1` | 3.1.1 | Pessimistic (>= 3.1.0 && < 3.2.0) | No |
| Microsoft.Extensions.Logging | `>= 8.0.0 < 9.0.0` | 8.0.1 | Range | Yes |
| Microsoft.Bcl.AsyncInterfaces | `>= 8.0.0 prerelease` | 8.0.0 | Prerelease-allowed | No |

### Transitive dependencies (resolved by lockfile)

| Package | Resolved version | Required by |
|---|---|---|
| Microsoft.Extensions.DependencyInjection.Abstractions | 8.0.1 | Microsoft.Extensions.Logging, Microsoft.Extensions.Logging.Abstractions, Microsoft.Extensions.Options |
| Microsoft.Extensions.Logging.Abstractions | 8.0.1 | Microsoft.Extensions.Logging |
| Microsoft.Extensions.Options | 8.0.0 | Microsoft.Extensions.Logging |
| Microsoft.Extensions.Primitives | 8.0.0 | Microsoft.Extensions.Options |

### Detection expectations

- Mend should detect **8 unique packages** in total from `paket.lock`.
- All packages belong to the `Main` (default) group.
- Source is `https://api.nuget.org/v3/index.json` for all packages.
- `storage: none` must not prevent detection — Mend reads `paket.lock`, not the local packages folder.
- The exact pin (`==`), pessimistic (`~>`), range (`>= X < Y`), and `prerelease` constraint
  syntaxes in `paket.dependencies` must not interfere with lockfile parsing.
- Transitive packages appear as children of their direct-dependency parents, not as top-level entries.
- `Microsoft.Bcl.AsyncInterfaces 8.0.0` has no transitive dependencies and appears as a leaf node.

## File structure

```
paket-version-constraints/
├── paket.dependencies       # Main group, 4 direct deps with 4 different constraint syntaxes
├── paket.lock               # Fully resolved lockfile (8 packages)
├── expected-tree.json       # Expected Mend dependency tree (probe format)
├── README.md                # This file
└── src/
    └── MyProject/
        ├── MyProject.csproj # SDK-style net8.0 project
        └── paket.references # 4 direct package references
```
