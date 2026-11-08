---
name: rsd-precision-estimation-from-qc-pools
description: Use when a metabolomics or lipidomics study has QCpool samples injected
  at regular intervals during one or more sequences and you need tabulated per-feature
  RSD precision estimates to judge and report overall data quality.
when_to_use_negative:
- No pooled QC samples were measured at regular intervals during the sequence — RSD
  estimation from QC pools is not possible
- Input is non-MS omics or non-MS metabolomics data without a pooled-QC injection
  structure
- You only need a single-step QC fix (e.g. only drift correction) rather than the
  sequential protocol whose final step is precision assessment
tools:
- name: QComics
  role: Sequential multistep R/Shiny QC tool that computes and exports tabulated RSD
    estimations and report figures for QCpool-based precision assessment
  repo: https://github.com/ricoderks/QComics
- name: Sciex MultiQuant
  role: Upstream acquisition/processing software (> v3.0.3) that must have processed
    and exported the QCpool peaks to txt before RSD estimation
- name: R / RStudio
  role: Runtime environment from which the QComics Shiny app is launched to produce
    the RSD tables and figures
  repo: https://github.com/ricoderks/QComics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:c2a92de807595a2e4b0fb9b37ff560a241a7a94f167f10c1e0508a9a1599c848
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - QComics
  - R
  - R / RStudio
  - RStudio
  - Sciex MultiQuant
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# RSD precision estimation from QC pools

## Summary

Final assessment step of the QComics multistep QC protocol: estimate per-feature relative standard deviation (RSD) across pooled QC samples (measured at regular intervals in Sciex MultiQuant-exported txt data) and export tabulated RSD estimations alongside QC report figures as the package's reproducible quality report.

## When to use

Use when a metabolomics or lipidomics study has QCpool samples injected at regular intervals during one or more sequences and you need tabulated per-feature RSD precision estimates to judge and report overall data quality.

## When NOT to use

- No pooled QC samples were measured at regular intervals during the sequence — RSD estimation from QC pools is not possible
- Input is non-MS omics or non-MS metabolomics data without a pooled-QC injection structure
- You only need a single-step QC fix (e.g. only drift correction) rather than the sequential protocol whose final step is precision assessment

## Inputs

- QCpool intensity table exported to txt format from Sciex MultiQuant software (version > v3.0.3)
- Cleaned QCpool feature matrix produced by the preceding QComics correction/drift/outlier modules

## Outputs

- Tabulated RSD estimations per feature (CSV-style table)
- QC report figures (PNG/PDF, e.g. Figures 2–4 style) documenting precision
- Precision-based pass/fail decision for overall data quality

## How to apply

After the upstream QComics modules (background noise/carryover correction, drift and out-of-control detection, missing-value handling, outlier removal, quality-marker monitoring) have produced a cleaned QCpool intensity matrix, compute per-feature RSD as the dispersion of QCpool signal across the sequence. Because QComics reports 'overall data quality in terms of precision and accuracy', use these RSDs together with accuracy metrics to decide whether feature-level variability is acceptable. Export the results in the package's two documented output forms: report figures (Figures 2–4 style) and tabulated RSD estimations per feature, which the authors state should be annexed to scientific publications for consistent reporting of data quality. The RSD thresholds used internally were adapted from guidelines for validating conventional bioanalytical methods and existing metabolomics QC literature, so a feature passing QC here has been judged against harmonized quality criteria rather than ad-hoc lab-specific cutoffs. Run the Shiny app (QComics::run_app() or run_in_browser()) and let it render the figures and CSV of RSD estimations rather than hand-computing them, so the QComics harmonized criteria are applied.

## Related tools

- **QComics** (Sequential multistep R/Shiny QC tool that computes and exports tabulated RSD estimations and report figures for QCpool-based precision assessment) — https://github.com/ricoderks/QComics
- **Sciex MultiQuant** (Upstream acquisition/processing software (> v3.0.3) that must have processed and exported the QCpool peaks to txt before RSD estimation)
- **R / RStudio** (Runtime environment from which the QComics Shiny app is launched to produce the RSD tables and figures) — https://github.com/ricoderks/QComics

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_in_browser()
```

## Evaluation signals

- Output contains a per-feature table with an RSD column for every feature retained after outlier removal and missing-data handling
- Tabulated RSD estimations are accompanied by the QC report figures the README/paper document (Figures 2–4 form)
- RSD values were computed only after the sequential correction steps (carryover, drift, missing values, outliers) were applied, so the numbers reflect cleaned intensities
- Precision results can be traced to harmonized quality criteria adapted from bioanalytical method validation guidelines, not ad-hoc lab cutoffs
- The exported artifacts are annexed to the study's report in the form the paper recommends for consistent reporting of data quality

## Limitations

- QComics is marked 'Lifecycle: experimental', so behavior and output formats may change without notice
- Requires QCpool samples measured at regular intervals and pre-processed with Sciex MultiQuant > v3.0.3; it will not work on mzML, NetCDF, or vendor-agnostic raw files
- The paper does not state a quantitative RSD cutoff in the provided text; thresholds are internal to the package and adapted from bioanalytical validation guidelines
- Although initially designed for MS-based metabolomics, its stated scope covers other omics and MS data — non-MS or non-pooled designs are out of scope

## Evidence

- [abstract] assess overall data quality in terms of precision and accuracy: "assess overall data quality in terms of precision and accuracy"
- [discussion] tabulated RSD estimations: "in the form of figures (e.g., Figures 2–4) and tables (e.g., tabulated RSD estimations)"
- [readme] The goal of the QComics package is to have a quick overview of the quality of a metabolomics or lipidomics study. For this, a pooled sample (QCpool) needs to measured in regular intervals: "The goal of the `QComics` package is to have a quick overview of the quality of a metabolomics or lipidomics study. For this, a pooled sample (QCpool) needs to measured in regular intervals"
- [readme] These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to txt format: "These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt` format."
- [intro] The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
- [readme] QComics::run_app(): [evidence span withheld — anchor status: not_found]
