---
name: tidymass-package-installation-and-version-checking
description: Use when an R environment must be set up for a tidyMass LC–MS metabolomics
  pipeline, or when the installed tidyMass package versions must be confirmed to match
  a published workflow before reprocessing mzML data.
when_to_use_negative:
- A tidyMass environment is already installed and verified — rerunning installation
  risks version drift instead of fixing a downstream analysis error
- The pipeline requires a non-tidyMass tool such as MZmine, MS-DIAL, XCMS, OpenMS,
  Galaxy-M, or eMZed as the primary processing engine rather than an interoperable
  `mass_dataset` conversion target
- The target platform lacks R/Git access and cannot pull the docker image, since tidyMass
  still requires an R-based runtime
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0091
tools:
- name: tidymass
  role: Umbrella package that manages and organizes all packages in the tidyMass project;
    installed via remotes and inspected for versions
  repo: https://github.com/tidymass/tidymass
- name: R
  role: Runtime language of the tidyMass ecosystem, installable on all platforms
- name: tidyMass docker image
  role: Prebuilt docker image containing the case-study data, code, and analysis environment
    for reproducible setup
  repo: https://hub.docker.com/r/jaspershen/tidymass-case-study
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:1c654a0106b3de6f242b36177ab4ffd54e2cf29175a0e45f24896b4df8530f02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - massprocesser
  - remotes
  - tidyMass
  - tidyMass docker image
  - tidymass
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tidymass-package-installation-and-version-checking

## Summary

Installs the tidyMass R package ecosystem (the metID/massProcesser/massDataset toolkit for LC-MS untargeted metabolomics) from GitLab/GitHub via remotes and verifies that the installed package set and versions match the published tidyMass workflow.

## When to use

Use when an R environment must be set up for a tidyMass LC–MS metabolomics pipeline, or when the installed tidyMass package versions must be confirmed to match a published workflow before reprocessing mzML data.

## When NOT to use

- A tidyMass environment is already installed and verified — rerunning installation risks version drift instead of fixing a downstream analysis error
- The pipeline requires a non-tidyMass tool such as MZmine, MS-DIAL, XCMS, OpenMS, Galaxy-M, or eMZed as the primary processing engine rather than an interoperable `mass_dataset` conversion target
- The target platform lacks R/Git access and cannot pull the docker image, since tidyMass still requires an R-based runtime

## Inputs

- R session (cross-platform: Windows/macOS/Linux)
- CRAN `remotes` package (installed if missing)
- GitLab or GitHub access to the tidymass repositories
- Optionally, the jaspershen/tidymass-case-study docker image for a prebuilt environment

## Outputs

- Installed `tidymass` umbrella package with the constituent tidyMass packages (e.g. massprocesser, massdataset, metID)
- Version report from check_tidymass_version()/tidymass_packages() confirming the tidyMass ecosystem version in use

## How to apply

Install the umbrella `tidymass` package with `remotes::install_gitlab("tidymass/tidymass")` from GitLab or `remotes::install_github("tidymass/tidymass")` from GitHub, installing `remotes` from CRAN first if absent. Because tidyMass is an ecosystem of R packages sharing a common design philosophy, grammar, and data structure (the `mass_dataset` class as the kernel), the umbrella install organizes the constituent packages rather than a single tool. After installation, run the ecosystem inspection helpers (e.g. `check_tidymass_version()` / `tidymass_packages()`) to confirm which tidyMass packages are loaded and at which versions, so the environment matches the version used by the published workflow. For fully reproducible environments, the article also provides a docker image (jaspershen/tidymass-case-study) bundling the data, code, and analysis environment, and cross-platform installation is supported on all operating systems.

## Related tools

- **tidymass** (Umbrella package that manages and organizes all packages in the tidyMass project; installed via remotes and inspected for versions) — https://github.com/tidymass/tidymass
- **R** (Runtime language of the tidyMass ecosystem, installable on all platforms)
- **tidyMass docker image** (Prebuilt docker image containing the case-study data, code, and analysis environment for reproducible setup) — https://hub.docker.com/r/jaspershen/tidymass-case-study

## Examples

```
if(!require(remotes)){install.packages("remotes")}; remotes::install_gitlab("tidymass/tidymass"); tidymass::tidymass_packages()
```

## Evaluation signals

- check_tidymass_version() returns without warning, indicating installed tidyMass packages are mutually consistent
- tidymass_packages() lists the expected constituent packages (massprocesser, massdataset, metID, etc.) with versions matching the published workflow
- `library(tidymass)` loads cleanly from the remotes-installed GitLab/GitHub source
- A trivial `mass_dataset` object can be constructed, confirming the kernel data structure of the object-oriented workflow is functional

## Limitations

- tidyMass does not wrap all widely used metabolomics tools; for those, the `mass_dataset` class must be converted to other tools' data structures instead
- The umbrella `tidymass` package is marked experimental on CRAN, so version instability across releases is possible
- Package versions must be recorded explicitly, since tracing prior analysis steps and parameters depends on the uniform data format capturing processing parameters

## Evidence

- [readme] You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass): "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [readme] remotes::install_github("tidymass/tidymass"): "remotes::install_github("tidymass/tidymass")"
- [abstract] TidyMass is an ecosystem of R packages that share an underlying design philosophy, grammar, and data structure: "TidyMass is an ecosystem of R packages that share an underlying design philosophy, grammar, and data structure"
- [readme] `tidymass` is a R package which is used to manage and organize all the pacakges in [tidyMass project]: [evidence span withheld — anchor status: external_doc]
- [abstract] TidyMass can be installed on all platforms: "1) Cross-platform utility. TidyMass can be installed on all platforms"
- [intro] A docker image containing the data, code, and analysis environment is also provided: [evidence span withheld — anchor status: not_found]
