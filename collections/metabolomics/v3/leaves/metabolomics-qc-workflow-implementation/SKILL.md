---
name: metabolomics-qc-workflow-implementation
description: Use when processing pooled-QC (QCpool)-injected metabolomics or lipidomics
  datasets — e.g., Sciex MultiQuant .txt exports — that require a standardized, reportable
  QC workflow before statistical analysis.
when_to_use_negative:
- Input is untargeted MS raw spectra (e.g. mzML) rather than a pre-integrated, blank/QCpool-annotated
  feature table
- The study contains no procedural blanks or regular pooled-QC injections, so the
  blank-comparison and drift-monitoring steps cannot be applied
- Features are already cleaned and only a single downstream statistical test (e.g.
  p<0.05) is needed rather than the full multistep QC chain
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: QComics
  role: Primary R/Shiny package implementing the sequential multistep QC protocol
    (contaminant flagging, drift/outlier detection, missing-data handling, quality-marker
    monitoring)
  repo: github.com/ricoderks/QComics
- name: R / RStudio
  role: 'Runtime environment: install the package with remotes::install_github("ricoderks/QComics")
    and run the app locally or on a Shiny server'
- name: dplyr / readr / stats
  role: Used to compute blank-sample means, RSDs, apply the 3× blank and >15% RSD
    thresholds, and export filter decision tables as CSV
- name: Sciex MultiQuant
  role: 'Downstream-analyses prerequisite: QCpool chromatograms are analysed and exported
    to txt format before loading into QComics'
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:bd20ccff0af760e7fd5fe285e81cd7ffa3f43e0d93ac9c6622b1731b7a84249f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QComics
  - R
  - R / RStudio
  - RStudio
  - Sciex MultiQuant
  - dplyr
  - dplyr / readr / stats
  - readr
  - stats
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# metabolomics-qc-workflow-implementation

## Summary

Implements the QComics sequential multistep quality-control protocol for targeted metabolomics/lipidomics studies, covering background-noise and carryover correction, drift and out-of-control detection, missing-data handling, outlier removal, and quality-marker monitoring.

## When to use

Use when processing pooled-QC (QCpool)-injected metabolomics or lipidomics datasets — e.g., Sciex MultiQuant .txt exports — that require a standardized, reportable QC workflow before statistical analysis.

## When NOT to use

- Input is untargeted MS raw spectra (e.g. mzML) rather than a pre-integrated, blank/QCpool-annotated feature table
- The study contains no procedural blanks or regular pooled-QC injections, so the blank-comparison and drift-monitoring steps cannot be applied
- Features are already cleaned and only a single downstream statistical test (e.g. p<0.05) is needed rather than the full multistep QC chain

## Inputs

- Feature/metabolite intensity table (data_df) with sample-to-study-group metadata
- Procedural blank injection intensity table
- QCpool sample annotations exported as txt from Sciex MultiQuant (> v3.0.3)

## Outputs

- Flagged-contaminant and blank-variability filter decision tables (e.g. flagged_features.csv, blank_variability_filter_results.csv)
- Missing-data removal/imputation report (per-variable per-group missing percentages, removed vs retained labels)
- Outlier and out-of-control sample list
- Quality-marker monitoring summary and overall precision/accuracy assessment

## How to apply

Install the QComics R package from github.com/ricoderks/QComics and launch the Shiny app with QComics::run_app() or run_in_browser(), feeding it a feature-intensity table (e.g. data_df) plus pooled-QC annotations. The workflow proceeds sequentially: (1) flag features whose mean in real samples is ≤3× their mean in procedural blanks as potential contaminants, discarding only those with blank RSD >15%; (2) correct for background noise/carryover and detect signal drifts and out-of-control observations; (3) treat features with >70% missing values as truly absent (real zeros) and remove variables with >20% missing values in all study groups, imputing the rest; (4) remove outlying samples; (5) monitor quality markers to detect improper collection, preprocessing, or storage; (6) assess overall precision and accuracy. Quality criteria are adapted from bioanalytical method validation guidelines, and each step's decisions should be tabulated (e.g. flagged_features.csv, blank_variability_filter_results.csv) for reproducible reporting.

## Related tools

- **QComics** (Primary R/Shiny package implementing the sequential multistep QC protocol (contaminant flagging, drift/outlier detection, missing-data handling, quality-marker monitoring)) — github.com/ricoderks/QComics
- **R / RStudio** (Runtime environment: install the package with remotes::install_github("ricoderks/QComics") and run the app locally or on a Shiny server)
- **dplyr / readr / stats** (Used to compute blank-sample means, RSDs, apply the 3× blank and >15% RSD thresholds, and export filter decision tables as CSV)
- **Sciex MultiQuant** (Downstream-analyses prerequisite: QCpool chromatograms are analysed and exported to txt format before loading into QComics)

## Examples

```
Rscript -e 'remotes::install_github("ricoderks/QComics"); QComics::run_app()'
```

## Evaluation signals

- Every flagged-contaminant feature satisfies sample mean ≤ 3× blank mean, and every discarded feature among those has blank RSD > 15%
- Every variable marked removed has missing rate > 20% in all study groups; features with >70% missing values are labelled truly absent, not missing
- Outlier removal and out-of-control detection outputs are consistent across QCpool batches and follow the sequential order (contaminants → drift → missing data → outliers)
- The final report includes per-step decision tables (CSV) that match the 3× / 15% / 70% / 20% thresholds applied to the same input
- Script executes without error and the package launches via run_app() from RStudio or a Shiny server

## Limitations

- QComics is marked experimental in its lifecycle badge and carries no changelog; no version number, license details, or quantitative benchmark metrics are stated in the article
- The protocol requires a pooled QC (QCpool) injected at regular intervals and Sciex MultiQuant (> v3.0.3) txt exports, so it cannot be applied without that QC scheme
- The pipeline assumes pre-integrated feature tables (endogenous metabolites and xenobiotics) and was designed for MS-based metabolomics, though it is extendable to other omics/MS data

## Evidence

- [abstract] The protocol operates in various sequential steps aimed to correct for background noise and carryover: [evidence span withheld — anchor status: not_found]
- [abstract] monitor quality markers: "monitor quality markers to identify samples affected by improper collection, preprocessing, or storage"
- [abstract] The tool considers important issues often neglected along quality control: "such as the need of separately handling missing values and truly absent data to avoid losing relevant biological information"
- [readme] Install package: "remotes::install_github("ricoderks/QComics")"
- [readme] Run the app: "From Rstudio run the app as:

``` r
QComics::run_app()
```"
- [readme] Analysis of pooled samples: "For this, a pooled sample
(QCpool) needs to measured in regular intervals during one or more
sequences. These QCpool samples need to be analysed with Sciex
Multiquant (> v3.0.3) software and exported"
