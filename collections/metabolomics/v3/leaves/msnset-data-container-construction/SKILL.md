---
name: msnset-data-container-construction
description: Use when metabolomics or proteomics peak/feature intensity tables and
  their sample metadata arrive as two separate CSVs and must be assembled into an
  S4 container before POMAShiny/POMA preprocessing.
when_to_use_negative:
- Input is already a single S4/MSnSet container from POMA — no further assembly is
  needed before preprocessing.
- Data is raw MS spectra (mzML/RAW files); POMAShiny expects pre-processed feature
  and metadata tables rather than spectral files.
tools:
- name: POMAShiny
  role: Web application whose data-upload panel consumes the assembled S4 container
    and drives the four analysis panels.
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package providing the S4 container constructors and preprocessing
    functions that operate on it outside the web environment.
  repo: https://github.com/nutrimetabolomics/POMA
- name: MSnbase
  role: Back-end package supplying the S4 data-container infrastructure POMAShiny
    is built on.
  repo: https://github.com/nutrimetabolomics/POMAShiny
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:447e44b4e483dc1d1a91c0768d375dfa3e7fef0fb8bfa88807b654fe301c4254
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - MSnbase
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# msnset-data-container-construction

## Summary

Constructs the S4 data container required by POMAShiny/POMA: combine a target (metadata) file and a features file into an MSnbase 'MSnSet'-style S4 object so downstream missing-value imputation, normalization, EDA and statistical panels can operate on a coherent single object.

## When to use

Use when metabolomics or proteomics peak/feature intensity tables and their sample metadata arrive as two separate CSVs and must be assembled into an S4 container before POMAShiny/POMA preprocessing.

## When NOT to use

- Input is already a single S4/MSnSet container from POMA — no further assembly is needed before preprocessing.
- Data is raw MS spectra (mzML/RAW files); POMAShiny expects pre-processed feature and metadata tables rather than spectral files.

## Inputs

- features file (CSV, metabolomics/proteomics intensities)
- target / metadata file (CSV, one row per sample)

## Outputs

- S4 data container (MSnbase-style) holding assays, feature metadata and sample metadata
- object ready as input to POMA preprocessing functions

## How to apply

POMAShiny ingests two comma-separated value files — a target (metadata) file and a features file — and its back-end uses the MSnbase and tidyverse R packages; assemble these two inputs into the S4 container (e.g. via POMA/MSnbase constructors) that the app expects. Validate that the feature table's column order matches the target's row names, since sample alignment is a prerequisite for the later preprocessing panel (missing-value filtering at the 20% default threshold per study group, imputation and normalization). Once the container is built, hand it to POMA functions for subsequent steps; the POMA R/Bioconductor package can be used outside the web environment for reproducibility. Keep the container structure intact through the four POMAShiny panels (data upload, preprocessing, EDA, statistical analysis).

## Related tools

- **POMAShiny** (Web application whose data-upload panel consumes the assembled S4 container and drives the four analysis panels.) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package providing the S4 container constructors and preprocessing functions that operate on it outside the web environment.) — https://github.com/nutrimetabolomics/POMA
- **MSnbase** (Back-end package supplying the S4 data-container infrastructure POMAShiny is built on.) — https://github.com/nutrimetabolomics/POMAShiny

## Examples

```
Rscript -e 'library(POMA); PomaDeploy()'
```

## Evaluation signals

- The container exposes a valid S4 structure (assay, featureData and pData slots) as expected by POMA functions.
- Row names of the target metadata table match the column names of the features table sample-for-sample.
- The resulting object can be loaded by the POMAShiny data-upload panel without error.
- Preprocessing steps invoked on the container (e.g. 20% missingness filter) run without sample-mismatch warnings.

## Limitations

- POMAShiny requires the two CSV inputs exactly; format deviations from the expected target/features file structure will fail container assembly.
- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions of bs4Dash due to substantial breaking changes in the API, so local deployment must pin that dependency.

## Evidence

- [results] POMAShiny requires two CSV input files: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] Back-end packages: [evidence span withheld — anchor status: not_found]
- [abstract] Reproducibility outside the web environment: "it is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of analyses outside the web environment"
- [readme] README deployment and dependencies: "POMAShiny was built with `bs4Dash` version 0.5.0 and is not compatible with newer versions of `bs4Dash` due to substantial breaking changes in the API"
- [readme] README local run: "shiny::runApp(appDir = "app")"
