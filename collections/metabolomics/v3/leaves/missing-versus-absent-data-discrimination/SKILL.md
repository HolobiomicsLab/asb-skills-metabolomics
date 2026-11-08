---
name: missing-versus-absent-data-discrimination
description: Use when a metabolomics intensity table contains features with intermittent
  missing signals and you must decide which are real zeros versus missing data before
  imputation or filtering.
when_to_use_negative:
- When the data are not MS-based metabolomics/lipidomics intensities with pooled QC
  (QCpool) samples measured at regular intervals — the QComics workflow assumes this
  design
- When missingness is purely random (MCAR) and no distinction between absent compounds
  and failed measurements is needed — a simple global imputation suffices
- When data are heterogeneous omics types other than MS-based metabolomics without
  adaptation — although QComics is extendable to other omics/MS data, thresholds were
  designed for metabolomics
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: QComics
  role: Implements the sequential multistep QC protocol including handling of missing
    values vs. truly absent data; installed from GitHub and run as a Shiny app
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime environment in which the QComics package runs and where per-group
    missing-rate computation/filtering is scripted
- name: RStudio
  role: IDE from which the QComics app is launched via QComics::run_app()
- name: shiny server
  role: 'Alternative hosting mode: create an app.R containing QComics::run_app() to
    run the QC workflow in a browser'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:fc1659c3106744ff5a0ac0ae295f4d05f0b2f795e98d9b66df9608155cf9b51b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - QComics
  - R
  - RStudio
  - shiny server
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# missing-versus-absent-data-discrimination

## Summary

QComics-based missing-data handling that distinguishes truly absent features (treated as real zeros) from genuine missing values, then filters variables with excessive missingness per study group.

## When to use

Use when a metabolomics intensity table contains features with intermittent missing signals and you must decide which are real zeros versus missing data before imputation or filtering.

## When NOT to use

- When the data are not MS-based metabolomics/lipidomics intensities with pooled QC (QCpool) samples measured at regular intervals — the QComics workflow assumes this design
- When missingness is purely random (MCAR) and no distinction between absent compounds and failed measurements is needed — a simple global imputation suffices
- When data are heterogeneous omics types other than MS-based metabolomics without adaptation — although QComics is extendable to other omics/MS data, thresholds were designed for metabolomics

## Inputs

- Feature/metabolite intensity table (rows = variables including endogenous metabolites and xenobiotics, columns = samples) exported from Sciex Multiquant as txt
- Sample-to-study-group metadata mapping (e.g., study group / batch annotations)

## Outputs

- Filtered intensity matrix with variables exceeding the missingness threshold removed and truly absent features recoded as real zeros
- Per-variable missingness classification (removed vs. retained vs. truly absent) usable for reporting

## How to apply

Load the feature/metabolite intensity table with sample-to-study-group metadata into R (QComics is installed via remotes::install_github('ricoderks/QComics')). For each variable, compute the missing-value rate separately within every study group rather than globally. Apply QComics' two-tier criterion: features with >70% missing values across the data are categorized as truly absent data and recoded as real zeros to avoid losing relevant biological information (e.g., xenobiotics absent from most samples), while variables whose missing-value rate exceeds 20% in all study groups are flagged and removed. Remaining missing values are then handled by imputation. The rationale, adapted from bioanalytical method validation guidelines, is that treating absent compounds as 'missing' would corrupt downstream statistics for heterogeneous data sets containing both endogenous metabolites and xenobiotics.

## Related tools

- **QComics** (Implements the sequential multistep QC protocol including handling of missing values vs. truly absent data; installed from GitHub and run as a Shiny app) — https://github.com/ricoderks/QComics
- **R** (Runtime environment in which the QComics package runs and where per-group missing-rate computation/filtering is scripted)
- **RStudio** (IDE from which the QComics app is launched via QComics::run_app())
- **shiny server** (Alternative hosting mode: create an app.R containing QComics::run_app() to run the QC workflow in a browser)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- A variable is removed only if its missing rate exceeds 20% in every study group — verify the removed/retained split exactly matches this rule
- Features with >70% missing values are classified as truly absent (real zeros), not counted toward removal or imputation
- Per-group (not pooled) missing percentages are computed for each variable
- The R script runs without error (remotes::install_github('ricoderks/QComics') succeeds; QComics::run_app() launches)
- Reported missingness counts are consistent with the original intensity table before imputation

## Limitations

- QComics is lifecycle-experimental and its quality criteria, while adapted from bioanalytical validation guidelines, imply thresholds (20%, 70%) that may need justification for specific studies
- Missing-value handling is designed to be implementable without advanced statistical or programming skills, but the 20% 'in all study groups' rule requires well-defined study-group metadata; mislabeled groups will distort per-group rates
- Sciex Multiquant (> v3.0.3) txt export of QCpool measurements is the assumed upstream input format

## Evidence

- [abstract] The tool considers important issues often neglected along quality control, such as separately handling missing values and truly absent data: "such as the need of separately handling missing values and truly absent data to avoid losing relevant biological information"
- [abstract] The protocol operates in various sequential steps aimed to correct for background noise and carryover, detect signal drifts and out-of-control observations, deal with missing data, remove outliers, monitor quality markers, and assess overall data quality: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover, (ii) detect signal drifts and “out-of-control” observations, (iii) deal with missing data,"
- [intro] Initial data exploration and handling missing values and truly absent data are early steps of the QComics workflow: "(ii) handling missing values and truly absent data"
- [intro] Quality criteria adapted from bioanalytical method validation guidelines: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
- [intro] Heterogeneous data sets comprising both endogenous metabolites and xenobiotics require adapted QC practices: "it also becomes essential to adapt current QC practices for dealing with heterogeneous data sets comprising both endogenous metabolites and xenobiotics, which are normally present at very different"
- [methods] Installation of QComics from GitHub: "remotes::install_github("ricoderks/QComics")"
- [readme] Running the QComics app locally from RStudio: "From Rstudio run the app as:

``` r
QComics::run_app()
```"
- [readme] QComics requires pooled QC samples measured at regular intervals and Sciex Multiquant txt export: "These QCpool samples need to be analysed with Sciex
Multiquant (> v3.0.3) software and exported to `txt` format."
