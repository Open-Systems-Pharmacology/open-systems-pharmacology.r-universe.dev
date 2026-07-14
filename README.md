# Open Systems Pharmacology R-universe

This repository is the **R-universe registry** for the Open Systems Pharmacology R packages. It is not an R package itself; its only job is to hold `packages.json`, the list of package repositories that [R-universe](https://r-universe.dev) builds and publishes to https://open-systems-pharmacology.r-universe.dev

## Why R-universe

The OSP R packages (`rSharp`, `ospsuite`, and the rest) embed pre-compiled binaries in `inst/lib` (the .NET assemblies and the computational core). CRAN forbids binaries in source packages, so these packages cannot be submitted to CRAN as-is. R-universe has no such restriction and no package-size limit: it builds and hosts the packages with their binary payloads intact, and it resolves the cross-package dependency chain (`rSharp` -> `ospsuite.utils` -> `tlf` -> `ospsuite`) automatically.

R-universe distributes the packages. It does not install the external .NET runtime; that remains a prerequisite on the user's machine, and on the R-universe build environment (see the caveat below).

## Availability does not require passing `R CMD check`

A package is **published and installable from this universe as soon as its source tarball builds** (`R CMD build` succeeds). Passing `R CMD check` is **not** required. R-universe is a publishing system, not a gatekeeper: it records each platform's check result (the `OK` / `NOTE` / `WARNING` / `ERROR` badges on the dashboard) for information, but a red check does **not** stop the binary from being built, stored, and served.

This matters for the OSP packages because the R-universe build runners have **no .NET runtime** (and their Linux image ships a `libxml2` whose soname differs from the one the bundled native libraries were linked against). As a result:

- The per-platform `R CMD check` jobs will typically show **ERROR** (tests, examples, and vignettes that call into .NET cannot run without the runtime). That is expected and does not block publishing.
- What must succeed is `R CMD build`: the package has to install and load far enough to produce a tarball. For that, its `.onLoad()` must not fail when the runtime is absent. `rSharp` and `ospsuite` degrade gracefully in that case (they load, record why the runtime is unavailable, and raise an actionable error only when a .NET call is actually made), so their tarballs build and they are published.

In short: **build green, check red, still installable.** A package only becomes unavailable if its `R CMD build` fails outright, or if one of its declared dependencies is not resolvable in the universe at a compatible version.

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

## Maintaining the list

- **Add a package**: append an object with its `package` name (the `Package:` field from the repository's `DESCRIPTION`) and its `url`. If the package does not cut GitHub releases, drop the `"branch"` field so R-universe builds its default branch instead of `*release`. If the app is installed with "only select repositories" scope, also grant it access to the new package repository.
- **Remove a package**: delete its object.
- **Change what is tracked**: edit or remove the `"branch"` field on that package's entry.

Every push to this repository triggers R-universe to re-read `packages.json` and reconcile what it builds. `packages.json` is validated as strict JSON; keep it well-formed (no comments, no trailing commas).
