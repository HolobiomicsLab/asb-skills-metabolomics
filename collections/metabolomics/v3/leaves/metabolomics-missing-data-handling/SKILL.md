---
name: metabolomics-missing-data-handling
description: Use when a metabolomics or lipidomics feature table contains widespread
  missing intensities and the analysis must decide, per feature, whether gaps reflect
  measurement dropout to impute or genuine absence to zero-fill — e.g. heterogeneous
  datasets mixing endogenous metabolites and xenobiotics.
when_to_use_negative:
- The input is already a clean feature table with no missing or zero intensity entries
  — there is nothing to classify
- The dataset is not MS-based metabolomics (e.g. transcriptomics without analogous
  intensity-missingness semantics) — QComics was initially designed for MS-based metabolomics,
  though the authors note it may extend to other omics/MS data
- You need a fully automated, scriptable batch QC pipeline — QComics runs as a Shiny
  app (QComics::run_app()) and is intended for interactive use without advanced programming
  skills
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: QComics
  role: Implements the sequential QC protocol whose missing-data step classifies >70%-missing
    features as truly absent and zero-fills them; runs as an R Shiny app
  repo: github.com/ricoderks/QComics
- name: R
  role: Runtime environment for the QComics Shiny app; installation is via remotes::install_github("ricoderks/QComics")
  repo: github.com/ricoderks/QComics
- name: RStudio
  role: Used to run the app locally via QComics::run_app() or QComics::run_in_browser()
  repo: github.com/ricoderks/QComics
- name: shiny server
  role: 'Alternative deployment: install QComics on a shiny server and launch via
    an app.R containing QComics::run_app()'
  repo: github.com/ricoderks/QComics
- name: Microsoft Excel
  role: The article notes the QC protocol can be performed manually in Excel, but
    the 70% true-absence classification is implemented in the QComics tool
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:a5b57a752d57409243485046c353f17b860798855e1688e35053ac4359c96243
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Microsoft Excel
  - QComics
  - R
  - RStudio
  - dplyr
  - readr
  - shiny server
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# metabolomics-missing-data-handling

## Summary

A QComics-derived skill for distinguishing missing values (imputable signal dropout) from truly absent metabolites (genuine real zeros) in MS-based metabolomics data, so that appropriate statistical treatment preserves biological information rather than discarding it.

## When to use

Use when a metabolomics or lipidomics feature table contains widespread missing intensities and the analysis must decide, per feature, whether gaps reflect measurement dropout to impute or genuine absence to zero-fill — e.g. heterogeneous datasets mixing endogenous metabolites and xenobiotics.

## When NOT to use

- The input is already a clean feature table with no missing or zero intensity entries — there is nothing to classify
- The dataset is not MS-based metabolomics (e.g. transcriptomics without analogous intensity-missingness semantics) — QComics was initially designed for MS-based metabolomics, though the authors note it may extend to other omics/MS data
- You need a fully automated, scriptable batch QC pipeline — QComics runs as a Shiny app (QComics::run_app()) and is intended for interactive use without advanced programming skills

## Inputs

- Sample-by-feature metabolomics intensity table (e.g. exported from Sciex MultiQuant as txt format)
- Per-feature missingness fractions computed across all samples

## Outputs

- A per-feature classification separating 'truly absent' features (real zero values) from features with imputable missing values
- Zero-filled intensity matrix for the truly absent features

## How to apply

Load the sample-by-feature intensity table and compute per-feature missingness across samples. Classify features whose missingness exceeds 70% as truly absent: the article notes that the true-positive rate in statistical testing is rapidly restored above roughly 70% missingness (following Armitage et al.), so such features are treated as containing real zero values rather than imputable dropout. Replace the missing entries of these flagged features with real zero values, keeping them separate from ordinary missing values, which are handled by the other QComics missing-data steps. For exogenous metabolites in targeted metabolomics (e.g. Sciex MultiQuant-exported txt intensity tables), missing values are regarded as truly absent outright. The rationale is to separately handle missing values and truly absent data so that relevant biological information is not lost. Run the workflow from within R via the QComics Shiny app, since the package implements the sequential QC steps interactively rather than as a scripted batch call.

## Related tools

- **QComics** (Implements the sequential QC protocol whose missing-data step classifies >70%-missing features as truly absent and zero-fills them; runs as an R Shiny app) — github.com/ricoderks/QComics
- **R** (Runtime environment for the QComics Shiny app; installation is via remotes::install_github("ricoderks/QComics")) — github.com/ricoderks/QComics
- **RStudio** (Used to run the app locally via QComics::run_app() or QComics::run_in_browser()) — github.com/ricoderks/QComics
- **shiny server** (Alternative deployment: install QComics on a shiny server and launch via an app.R containing QComics::run_app()) — github.com/ricoderks/QComics
- **Microsoft Excel** (The article notes the QC protocol can be performed manually in Excel, but the 70% true-absence classification is implemented in the QComics tool)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- The 70% missingness threshold is applied per feature — verify that the fraction of samples with missing intensity is computed column-wise and that only features above 0.7 are zero-filled
- Truly absent features must be separated from ordinary missing values — confirm the output keeps the two categories distinct rather than uniformly imputing or zero-filling everything
- For targeted/exogenous metabolites, missing values should be marked truly absent without applying the 70% fraction check
- The resulting dataset should retain biological signal: after zero-filling, downstream statistical tests should show restored true-positive rates consistent with the Armitage et al. rationale cited in the article
- The package lifecycle badge is 'experimental' and coverage is tracked on Codecov (branch main) — successful installation and app launch (no errors on QComics::run_app()) is the basic run-signal

## Limitations

- The 70% cutoff is a heuristic adapted from bioanalytical method validation guidelines and existing QC literature, not a validated universal threshold — datasets with different noise structure may need adjustment
- QComics is labeled 'experimental' (lifecycle badge in the README), so API stability is not guaranteed
- The README requires QCpool samples to be analysed with Sciex Multiquant > v3.0.3 and exported to txt format; inputs from other vendor software are not documented
- The README and article do not state the version number of QComics or a changelog for the repository
- The article cites prior applications (refs 43−50) but does not report quantitative benchmark metrics for the missing-data step itself in the provided material

## Evidence

- [abstract] distinguishing missing values from truly absent metabolites and 70% zero-fill threshold: "such as the need of separately handling missing values and truly absent data to avoid losing relevant biological information"
- [abstract] missing-data step in the sequential workflow: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover, (ii) detect signal drifts and “out-of-control” observations, (iii) deal with missing data,"
- [intro] sequential workflow includes handling missing values and truly absent data: "(ii) handling missing values and truly absent data"
- [intro] QComics designed for implementability without advanced programming: "without the need of advanced statistical and programming skills"
- [readme] Sciex MultiQuant txt export requirement: "These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt` format."
- [readme] install from GitHub via remotes: "remotes::install_github("ricoderks/QComics")"
