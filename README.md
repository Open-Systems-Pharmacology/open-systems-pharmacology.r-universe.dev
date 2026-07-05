# Open Systems Pharmacology R-universe

This repository is the **R-universe registry** for the Open Systems Pharmacology R packages. It is not an R package itself; its only job is to hold `packages.json`, the list of package repositories that [R-universe](https://r-universe.dev) builds and publishes.

Once this repository lives in the `Open-Systems-Pharmacology` GitHub organization and the R-universe GitHub App is enabled, R-universe builds every listed package from source on Windows, macOS, and Linux and serves the resulting binaries from:

```
https://open-systems-pharmacology.r-universe.dev
```

## Repository naming

R-universe discovers a registry only when the repository is named `<organization>.r-universe.dev`. **This repository must be named `open-systems-pharmacology.r-universe.dev`** in the organization (all lowercase). The older `universe` name is deprecated and is no longer picked up for a new registry.

## Why R-universe

The OSP R packages (`rSharp`, `ospsuite`, and the rest) embed pre-compiled binaries in `inst/lib` (the .NET assemblies and the computational core). CRAN forbids binaries in source packages, so these packages cannot be submitted to CRAN as-is. R-universe has no such restriction and no package-size limit: it builds and hosts the packages with their binary payloads intact, and it resolves the cross-package dependency chain (`rSharp` -> `ospsuite.utils` -> `tlf` -> `ospsuite`) automatically.

R-universe distributes the packages. It does not install the external .NET runtime; that remains a prerequisite on the user's machine, and on the R-universe build environment (see the caveat below).

## Installing from this universe

Once the universe is live, users install with plain `install.packages()`:

```r
install.packages(
  "ospsuite",
  repos = c(
    OSP = "https://open-systems-pharmacology.r-universe.dev",
    CRAN = "https://cloud.r-project.org"
  )
)
```

`pak` works too, once the repository is configured:

```r
pak::repo_add(OSP = "https://open-systems-pharmacology.r-universe.dev")
pak::pak("ospsuite")
```

## What gets built

`packages.json` lists the OSP R packages that publish GitHub releases. Each entry tracks the package's latest release (`"branch": "*release"`).

| Package | Source repository |
| --- | --- |
| `rSharp` | `Open-Systems-Pharmacology/rSharp` |
| `ospsuite.utils` | `Open-Systems-Pharmacology/OSPSuite.RUtils` |
| `tlf` | `Open-Systems-Pharmacology/TLF-Library` |
| `ospsuite` | `Open-Systems-Pharmacology/OSPSuite-R` |
| `ospsuite.plots` | `Open-Systems-Pharmacology/OSPSuite.Plots` |
| `ospsuite.reportingengine` | `Open-Systems-Pharmacology/OSPSuite.ReportingEngine` |
| `ospsuite.parameteridentification` | `Open-Systems-Pharmacology/OSPSuite.ParameterIdentification` |
| `ospsuite.globalsensitivity` | `Open-Systems-Pharmacology/OSPSuite.GlobalSensitivity` |
| `ospsuite.reportingframework` | `Open-Systems-Pharmacology/OSPSuite.ReportingFramework` |

Some OSP R packages were left out for now because they do not publish GitHub releases, so they cannot be tracked with `*release`: `ospsuite.addins`, `ospsuite.qualificationplaneditor`, and `ospsuite.VBEToolbox`. Give a package a release, then add it here.

## Caveat: rSharp and the .NET runtime

`rSharp` (and therefore everything that depends on it) needs a .NET runtime to load. R-universe's build environment does not provide one, so until `rSharp` can install and load without a runtime present, its build (and the builds of the packages that import it) will fail on R-universe. This is a property of the packages and the build environment, not of this registry.

## Maintaining the list

- **Add a package**: append an object with its `package` name (the `Package:` field from the repository's `DESCRIPTION`) and its `url`. If the package does not cut GitHub releases, drop the `"branch"` field so R-universe builds its default branch instead of `*release`. If the app is installed with "only select repositories" scope, also grant it access to the new package repository.
- **Remove a package**: delete its object.
- **Change what is tracked**: edit or remove the `"branch"` field on that package's entry.

Every push to this repository triggers R-universe to re-read `packages.json` and reconcile what it builds. `packages.json` is validated as strict JSON; keep it well-formed (no comments, no trailing commas).
