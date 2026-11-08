---
name: r-package-biocmanager-installation
description: Use when a workflow requires the JPA R package or its xcms/MSnbase dependencies
  and they are not yet installed in the R environment (R >= 4.0.0).
when_to_use_negative:
- Running under R < 4.0.0, which the JPA documentation explicitly states is insufficient
  for installation
- The needed packages are already installed and requireNamespace('JPA') succeeds —
  reinstalling is unnecessary
edam_operation: http://edamontology.org/operation_2985
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3172
tools:
- name: JPA
  role: Target package being installed from GitHub; provides XCMS.featureTable(),
    find.level3features(), and other feature-extraction functions
  repo: https://github.com/HuanLab/JPA.git
- name: BiocManager
  role: Installed from CRAN first to manage Bioconductor dependencies such as xcms
    and MSnbase
- name: devtools
  role: Used to install JPA directly from GitHub via install_github()
- name: RStudio
  role: Recommended IDE for completing the JPA installation and usage
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:02e2b11c3b321d6224d70c2e503eae3d5deac0162d7aadfa7d8b791ee44d78f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - BiocManager
  - JPA
  - MSnbase
  - R
  - RStudio
  - devtools
  edam_operation: http://edamontology.org/operation_2985
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# r-package-biocmanager-installation

## Summary

Installs the JPA R package (HuanLab/JPA) and its Bioconductor dependencies (xcms, MSnbase) using BiocManager and devtools under R 4.0.0 or above, as a prerequisite for JPA-based LC-MS metabolic feature extraction.

## When to use

Use when a workflow requires the JPA R package or its xcms/MSnbase dependencies and they are not yet installed in the R environment (R >= 4.0.0).

## When NOT to use

- Running under R < 4.0.0, which the JPA documentation explicitly states is insufficient for installation
- The needed packages are already installed and requireNamespace('JPA') succeeds — reinstalling is unnecessary

## Inputs

- R environment version 4.0.0 or above
- CRAN package repositories (for BiocManager and devtools)
- GitHub repository HuanLab/JPA source code

## Outputs

- Installed JPA R package with xcms and MSnbase dependencies loaded and callable

## How to apply

Confirm R 4.0.0 or above is available, ideally within RStudio as the paper recommends. Install 'BiocManager' from CRAN, then install 'devtools' from CRAN, and use devtools::install_github('HuanLab/JPA') to pull the package source from GitHub. Load the package with library(JPA) and load xcms/MSnbase so the embedded XCMS peak-picking functions used by XCMS.featureTable() and related calls are available. Verify installation by checking that requireNamespace('JPA') returns TRUE before running any feature-extraction step.

## Related tools

- **JPA** (Target package being installed from GitHub; provides XCMS.featureTable(), find.level3features(), and other feature-extraction functions) — https://github.com/HuanLab/JPA.git
- **BiocManager** (Installed from CRAN first to manage Bioconductor dependencies such as xcms and MSnbase)
- **devtools** (Used to install JPA directly from GitHub via install_github())
- **RStudio** (Recommended IDE for completing the JPA installation and usage)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); library(devtools); if (!requireNamespace("JPA", quietly=TRUE)) install_github("HuanLab/JPA"); library(JPA)
```

## Evaluation signals

- requireNamespace('JPA', quietly = TRUE) returns TRUE after install_github('HuanLab/JPA')
- library(JPA) loads without error and BiocManager-managed xcms/MSnbase packages are present
- R version reported by getRversion() is 4.0.0 or above as required by the JPA docs
- help('XCMS.featureTable') resolves, confirming function-level documentation is installed

## Limitations

- Requires R >= 4.0.0; older R versions will fail to install the package
- JPA accepts only raw data formats compatible with MS-Convert, so vendor conversion is a separate prerequisite
- BiocManager/devtools must be installed from CRAN before install_github can be used

## Evidence

- [methods] R version requirement: "To install `JPA` package R version 4.0.0 or above is required"
- [methods] Install from GitHub via devtools: "Install "JPA" from Github using "devtools".
if (!requireNamespace("JPA", quietly = TRUE)){
  install_github("HuanLab/JPA")"
- [methods] BiocManager installation: "Install "BiocManager" package from CRAN if you do not already have it installed."
- [methods] RStudio recommendation: "we recommend using RStudio to complete the installation and usage of `JPA`"
- [intro] Source availability: "'JPA' is written in R and its source code is publicly available at <https://github.com/HuanLab/JPA.git>"
