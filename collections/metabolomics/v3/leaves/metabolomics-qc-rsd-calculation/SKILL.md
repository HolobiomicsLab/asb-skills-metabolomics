---
name: metabolomics-qc-rsd-calculation
description: Use when a metabolomics/lipidomics sequence contains repeated pooled
  QC (QCpool) injections and you need to quantitatively assess overall data quality
  in terms of precision and accuracy per chemical descriptor before reporting the
  study.
when_to_use_negative:
- The sequence lacks repeated pooled QC (QCpool) injections — precision and accuracy
  cannot be estimated without them
- You are performing earlier QComics steps (background/carryover correction, drift
  detection, missing-value handling, outlier removal) rather than the final overall
  data-quality assessment
- Data are non-MS omics outputs where intensity/RT/peak-width descriptors and ppm
  m/z error do not apply without redefining criteria
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: QComics
  role: Shiny app that implements the sequential multistep QC protocol including the
    final precision/accuracy assessment with predefined acceptance criteria
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime environment for launching the QComics app and scripting RSD/ppm computations
    with dplyr
- name: RStudio
  role: IDE from which the app is run via QComics::run_app()
- name: dplyr
  role: Per-descriptor RSD computation across repeated QCpool injections
- name: ggplot2
  role: Time-series QC plots per descriptor with tolerance-window bands
- name: DT
  role: Rendering the per-descriptor pass/fail QC table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:d6179b5e24e562c60e14689d7e65b7cc5e6efa79da9a76de15b5470962caa705
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - DT
  - QComics
  - R
  - RStudio
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolomics-qc-rsd-calculation

## Summary

Computes per-descriptor precision (%RSD of peak intensity, retention time, and peak width) and mass accuracy (ppm m/z error) across repeated pooled-QC injections, then applies the QComics acceptance criteria (RSD < 30% intensity, RSD < 2% RT, RSD < 15% peak width, m/z error < 10 ppm for HRMS) to flag each descriptor pass/fail as the final overall data-quality assessment.

## When to use

Use when a metabolomics/lipidomics sequence contains repeated pooled QC (QCpool) injections and you need to quantitatively assess overall data quality in terms of precision and accuracy per chemical descriptor before reporting the study.

## When NOT to use

- The sequence lacks repeated pooled QC (QCpool) injections — precision and accuracy cannot be estimated without them
- You are performing earlier QComics steps (background/carryover correction, drift detection, missing-value handling, outlier removal) rather than the final overall data-quality assessment
- Data are non-MS omics outputs where intensity/RT/peak-width descriptors and ppm m/z error do not apply without redefining criteria

## Inputs

- QCpool feature table (data_df) exported from Sciex MultiQuant (> v3.0.3) as txt, with per-injection peak intensity, retention time, peak width, and m/z
- Injection sequence/order metadata identifying the repeated QCpool injections

## Outputs

- Per-descriptor pass/fail QC assessment table (e.g., qc_descriptor_assessment.csv) with %RSD intensity, %RSD RT, %RSD peak width, and ppm m/z error flags
- Time-series QC plots per descriptor with tolerance-window bands

## How to apply

Load the QCpool feature table (data_df) with its injection sequence into the QComics Shiny app launched via QComics::run_app() or QComics::run_in_browser() in R/RStudio. For each descriptor, compute the relative standard deviation (%RSD) of peak intensity, retention time, and peak width across the repeated QCpool injections (e.g., with dplyr), and for HRMS descriptors compute the m/z error in ppm. Apply the QComics acceptance criteria adapted from bioanalytical method validation guidelines: RSD < 30% for peak intensity, RSD < 2% for retention time, RSD < 15% for peak width, and m/z error < 10 ppm (HRMS only), assigning a pass/fail flag per descriptor. Visualize each descriptor as a time-series QC plot with predefined tolerance-window bands (ggplot2) so drifts or out-of-control injections are visible, and export the per-descriptor pass/fail table for reporting. This is the concluding step (vi) of the QComics multistep workflow: 'assess overall data quality in terms of precision and accuracy'.

## Related tools

- **QComics** (Shiny app that implements the sequential multistep QC protocol including the final precision/accuracy assessment with predefined acceptance criteria) — https://github.com/ricoderks/QComics
- **R** (Runtime environment for launching the QComics app and scripting RSD/ppm computations with dplyr)
- **RStudio** (IDE from which the app is run via QComics::run_app())
- **dplyr** (Per-descriptor RSD computation across repeated QCpool injections)
- **ggplot2** (Time-series QC plots per descriptor with tolerance-window bands)
- **DT** (Rendering the per-descriptor pass/fail QC table)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- Every descriptor in the output table carries a pass/fail flag against all four criteria (RSD intensity < 30%, RSD RT < 2%, RSD width < 15%, m/z error < 10 ppm for HRMS)
- Computed %RSD values are reproducible from the QCpool feature table (e.g., sd/mean * 100 per descriptor)
- Time-series plots show each descriptor against its predefined tolerance window, making out-of-control injections visually identifiable
- The pass/fail table (qc_descriptor_assessment.csv) covers all descriptors present in the input feature table with no missing rows
- HRMS-only descriptors additionally report ppm m/z error; non-HRMS descriptors do not require an m/z accuracy flag

## Limitations

- QComics is lifecycle-experimental software; QCpool data must be analysed with Sciex MultiQuant (> v3.0.3) and exported to txt format
- Acceptance criteria were adapted from bioanalytical method validation guidelines and may need justification when transferred to other omics/MS data types
- m/z error criterion (< 10 ppm) applies only to HRMS descriptors

## Evidence

- [abstract] assess overall data quality in terms of precision and accuracy: "(vi) assess overall data quality in terms of precision and accuracy"
- [intro] quality criteria adapted from guidelines: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
- [readme] run the app from RStudio: "From Rstudio run the app as:

``` r
QComics::run_app()
```

or if you want the app to run in the browser use:

``` r
QComics::run_in_browser()"
- [readme] QCpool analysed with Sciex MultiQuant and exported to txt: "These QCpool samples need to be analysed with Sciex
Multiquant (> v3.0.3) software and exported to `txt` format."
