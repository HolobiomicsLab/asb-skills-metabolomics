---
name: metabolomics-raw-data-preprocessing
description: Use when raw targeted LC–MS runs from public repositories (e.g., MetaboLights
  MTBLS8735, potentially from different vendors) must be integrated against a target
  list and QC-clustered PCA is the exploratory endpoint.
when_to_use_negative:
- Input is already a preprocessed feature/peak table — TARDIS performs targeted integration
  on raw files
- The goal is untargeted peak picking — use untargeted tools such as xcms, MZmine
  or OpenMS instead
- Absolute quantification validated against vendor software (e.g., TraceFinder, Xcalibur)
  is required — this vignette workflow only supports targeted preprocessing and exploratory
  analysis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: TARDIS
  role: R package performing targeted raw data integration of the MTBLS8735 LC–MS
    files against the vignette target list and computing the five quality metrics
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment for TARDIS installation and for running prcomp PCA on
    the integrated AUC matrix
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:5c616a277c9705f05837ca69a0fe5ae55804b1e0bb74b96b32b13540058272c6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/UGent-LIMET/TARDIS
  repo_url: https://github.com/UGent-LIMET/TARDIS
  tools:
  - R
  - TARDIS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-raw-data-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the TARDIS case_study vignette workflow: targeted integration of public multi-vendor LC–MS raw files (MetaboLights MTBLS8735) with the TARDIS R package, extracting per-target AUC quality metrics and validating data quality via QC-clustered exploratory PCA.

## When to use

Use when raw targeted LC–MS runs from public repositories (e.g., MetaboLights MTBLS8735, potentially from different vendors) must be integrated against a target list and QC-clustered PCA is the exploratory endpoint.

## When NOT to use

- Input is already a preprocessed feature/peak table — TARDIS performs targeted integration on raw files
- The goal is untargeted peak picking — use untargeted tools such as xcms, MZmine or OpenMS instead
- Absolute quantification validated against vendor software (e.g., TraceFinder, Xcalibur) is required — this vignette workflow only supports targeted preprocessing and exploratory analysis

## Inputs

- Raw LC–MS files from MetaboLights study MTBLS8735 (multi-vendor)
- Target list distributed with the TARDIS case_study vignette
- R environment (version >= 4.4.0; BiocManager version 3.20 for latest TARDIS)

## Outputs

- data.frame of AUC per target per run (extracted from the TARDIS results list)
- tibble with average quality metrics per target in QC runs
- EIC diagnostic plots saved in the output folder
- PCA score plot of integrated AUCs with samples colored by run type

## How to apply

Install TARDIS from GitHub in R (>= 4.4.0) via BiocManager with dependencies = TRUE, raising the R timeout (options(timeout = 300)) since bundled example data often causes connection timeouts. Download the MTBLS8735 raw LC–MS files and the target list shipped with the case_study vignette (RShowDoc("case_study", package = "TARDIS")), then run TARDIS targeted peak integration over the raw files, which computes the five built-in quality metrics: area under the curve, maximum intensity, points over the peak, custom signal-to-noise ratio, and bell-curve similarity. Extract the AUC data.frame (one row per target per run) from the TARDIS results list — it also contains a tibble of average metrics per target in QC runs — and export it as the processed AUC table. Run exploratory PCA (e.g., prcomp) on the integrated AUC matrix and draw a score plot colored by run type. Decision point: integration quality is confirmed only if QC runs form a tightly clustered group while sample runs show variance; inspect the EIC diagnostic plots saved in the output folder to visually verify peak integration.

## Related tools

- **TARDIS** (R package performing targeted raw data integration of the MTBLS8735 LC–MS files against the vignette target list and computing the five quality metrics) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment for TARDIS installation and for running prcomp PCA on the integrated AUC matrix) — https://cloud.r-project.org/index.html

## Examples

```
BiocManager::install("pablovgd/TARDIS", build_vignettes = TRUE, dependencies = TRUE)
```

## Evaluation signals

- In the PCA score plot of integrated AUCs, QC runs form a tightly clustered group while sample runs show variance — the paper's stated expectation for a high-quality data set
- The TARDIS results object is a list containing a data.frame with the AUC of each target in each run plus a tibble of average metrics for each target in QC runs
- All five quality metrics (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) are present in the output
- EIC diagnostic plots for QC components are generated, saved in the output folder, and visually inspected

## Limitations

- Package installation often hits connection timeouts because the package contains example data; increase the R timeout setting (options(timeout = 300))
- Platform prerequisites: Rtools on Windows; Xcode and a GNU Fortran compiler on macOS
- No TARDIS package version number is stated in the article; the latest version requires BiocManager 3.20
- Applicability of TARDIS itself was validated at large scale (n = 1786) on stool, saliva and urine across two LC–MS instruments; MTBLS8735 is a vignette-level demonstration

## Evidence

- [readme] Minimum R version required: [evidence span withheld — anchor status: external_doc]
- [readme] case_study vignette available as tutorial: [evidence span withheld — anchor status: external_doc]
