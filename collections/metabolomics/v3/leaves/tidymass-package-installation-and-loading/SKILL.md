---
name: tidymass-package-installation-and-loading
description: Use when beginning an LC-MS untargeted metabolomics analysis in R and
  the tidyMass packages are not yet available in the environment, or when a reproducible
  docker-based environment (e.g. the tidymass-case-study image) must be provisioned
  before data processing.
when_to_use_negative:
- When tidyMass is already installed and the goal is to convert an existing mass_dataset
  object to SummarizedExperiment, mzTab-M, or MetDNA format — use the massdataset
  interoperability converters instead.
- When the analysis pipeline is not R-based (e.g. Python or standalone tools like
  MS-DIAL/MZmine alone) and no mass_dataset workflow is needed.
- When raw vendor file processing via Bioconductor's xcms3 API alone is required without
  the tidyMass object-oriented layer.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: tidyMass
  role: The R package being installed and loaded; umbrella package organizing all
    tidyMass project packages
  repo: https://github.com/tidymass/tidymass
- name: R
  role: Runtime environment in which tidyMass packages are installed and loaded
- name: xcmsrocker
  role: Alternative rocker/docker image providing an R-based metabolomics environment
    (RStudio, xcms) that can host tidyMass workflows reproducibly
  repo: https://github.com/yufree/xcmsrocker
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:b69ed6add1513d333fb7727cab221fc32d187d3f40ec1b3b42493ae3fc94c788
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - MetaboAnnotation
  - R
  - tidyMass
  - xcmsrocker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tidymass-package-installation-and-loading

## Summary

Installs and loads the tidyMass R package ecosystem — the object-oriented, reproducible framework for LC-MS-based untargeted metabolomics whose kernel is the mass_dataset class — as a prerequisite for any downstream tidyMass workflow.

## When to use

Use when beginning an LC-MS untargeted metabolomics analysis in R and the tidyMass packages are not yet available in the environment, or when a reproducible docker-based environment (e.g. the tidymass-case-study image) must be provisioned before data processing.

## When NOT to use

- When tidyMass is already installed and the goal is to convert an existing mass_dataset object to SummarizedExperiment, mzTab-M, or MetDNA format — use the massdataset interoperability converters instead.
- When the analysis pipeline is not R-based (e.g. Python or standalone tools like MS-DIAL/MZmine alone) and no mass_dataset workflow is needed.
- When raw vendor file processing via Bioconductor's xcms3 API alone is required without the tidyMass object-oriented layer.

## Inputs

- R environment (all platforms supported) with remotes installed
- Optional: docker runtime for the tidymass-case-study image

## Outputs

- Installed and loadable tidymass package ecosystem in R
- Access to the mass_dataset object class and tidyMass functions (data processing, annotation, statistics, format conversion)

## How to apply

Install the umbrella package `tidymass` with the `remotes` package, either from GitLab (remotes::install_gitlab("tidymass/tidymass")) or GitHub (remotes::install_github("tidymass/tidymass")); first install remotes from CRAN if missing (install.packages("remotes")). tidyMass is an ecosystem of R packages sharing a common design philosophy, grammar, and data structure, so installing the umbrella package organizes all constituent packages in one step; for sub-packages used for specific steps (e.g. massdataset for interoperability conversion to SummarizedExperiment, mzTab-M, or MetDNA formats), install them from their tidyMass GitLab/GitHub repositories. For fully reproducible runs, pull the provided docker image (hub.docker.com/r/jaspershen/tidymass-case-study) which bundles data, code, and the analysis environment. Verify the installation by loading the package in R and confirming access to the mass_dataset class before proceeding to data processing steps such as peak picking, data cleaning, QC, metabolite annotation, statistical analysis, or format conversion.

## Related tools

- **tidyMass** (The R package being installed and loaded; umbrella package organizing all tidyMass project packages) — https://github.com/tidymass/tidymass
- **R** (Runtime environment in which tidyMass packages are installed and loaded)
- **xcmsrocker** (Alternative rocker/docker image providing an R-based metabolomics environment (RStudio, xcms) that can host tidyMass workflows reproducibly) — https://github.com/yufree/xcmsrocker

## Examples

```
if(!require(remotes)){install.packages("remotes")}; remotes::install_gitlab("tidymass/tidymass"); library(tidymass)
```

## Evaluation signals

- remotes::install_gitlab("tidymass/tidymass") completes without error and library(tidymass) loads in a fresh R session.
- The installed package exposes the mass_dataset class (object-oriented kernel of tidyMass) so a mass_dataset object can be constructed or loaded.
- The installation matches the citation version: Shen et al., Nat Commun 13, 4365 (2022).
- When using docker, the tidymass-case-study image pulls and runs, providing the data, code, and analysis environment as documented.
- Downstream tidyMass vignette steps (e.g. the massdataset interoperability vignette at massdataset.tidymass.org) execute without missing-dependency errors.

## Limitations

- tidyMass is R-based, so the skill only applies within R (or R within docker images like xcmsrocker); it is not a Python solution.
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; for those, mass_dataset objects must be converted to other eligible data structures instead.
- Installation from GitLab or GitHub via remotes requires internet access and build toolchains; lifecycle status of the umbrella package is marked experimental.

## Evidence

- [abstract] TidyMass is an ecosystem of R packages that share an underlying design philosophy, grammar, and data structure: "TidyMass is an ecosystem of R packages that share an underlying design philosophy, grammar, and data structure"
- [methods] You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass) or [GitHub](https://github.com/tidymass/tidymass): "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [readme] `tidymass` is a R package which is used to manage and organize all the pacakges in tidyMass project: [evidence span withheld — anchor status: external_doc]
- [abstract] TidyMass can be installed on all platforms: "1) Cross-platform utility. TidyMass can be installed on all platforms"
- [intro] A docker image containing the data, code, and analysis environment is also provided: [evidence span withheld — anchor status: not_found]
