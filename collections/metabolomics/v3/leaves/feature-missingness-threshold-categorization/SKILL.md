---
name: feature-missingness-threshold-categorization
description: Use when a metabolomics feature table (sample-by-feature intensity matrix,
  e.g. from Sciex MultiQuant export) contains missing values that must be distinguished
  from true absence before imputation or statistical testing.
when_to_use_negative:
- Input is already a zero-filled or imputed feature table — the distinction between
  missing and truly absent values has already been collapsed
- Feature table comes from an untargeted workflow without a pooled QC sample measured
  at regular intervals, since QComics assumes QCpool-based sequences
- Data are quality-control metrics (RSD, drift, carryover) rather than sample-level
  metabolite intensities requiring missingness classification
edam_operation: http://edamontology.org/operation_3667
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: QComics
  role: Implements the sequential QC protocol step that classifies features with >70%
    missing values as truly absent and zero-fills them; installable as an R package
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime environment in which the QComics package is installed and executed
    (remotes::install_github)
- name: RStudio
  role: IDE used to launch the QComics shiny app locally via QComics::run_app()
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:15b89a3ca6c3876810e8f4138ac2810be386cecf29bd683cc28af57276930888
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - QComics
  - R
  - RStudio
  - dplyr
  - readr
  edam_operation: http://edamontology.org/operation_3667
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# feature-missingness-threshold-categorization

## Summary

Categorizes molecular features in metabolomics data by their missing-value fraction, flagging those with more than 70% missingness as truly absent (real zeros) versus those with lower missingness treated as imputable missing data, following the QComics protocol.

## When to use

Use when a metabolomics feature table (sample-by-feature intensity matrix, e.g. from Sciex MultiQuant export) contains missing values that must be distinguished from true absence before imputation or statistical testing.

## When NOT to use

- Input is already a zero-filled or imputed feature table — the distinction between missing and truly absent values has already been collapsed
- Feature table comes from an untargeted workflow without a pooled QC sample measured at regular intervals, since QComics assumes QCpool-based sequences
- Data are quality-control metrics (RSD, drift, carryover) rather than sample-level metabolite intensities requiring missingness classification

## Inputs

- Sample-by-feature metabolomics intensity table (e.g. txt exported from Sciex MultiQuant > v3.0.3)
- Per-feature missing-value counts

## Outputs

- Truly absent features list (features with >70% missingness, zero-filled)
- Feature classification (truly absent vs. ordinary missing)

## How to apply

Compute the per-feature missingness fraction across all samples in the sample-by-feature intensity table. Classify every molecular feature whose missingness exceeds 70% as truly absent — its missing values represent real zero concentrations rather than imputable dropout, since the true-positive rate in statistical testing is rapidly restored above this level (Armitage et al.) — and replace its missing entries with real zeros. Features with ≤70% missingness are retained as ordinary missing values for separate downstream handling (e.g. imputation or filtering by other QComics rules). For exogenous/xenobiotic metabolites in targeted metabolomics, treat their missing values as truly absent regardless of threshold. The 70% boundary aligns with the QComics rationale for separately handling missing values and truly absent data to avoid losing relevant biological information.

## Related tools

- **QComics** (Implements the sequential QC protocol step that classifies features with >70% missing values as truly absent and zero-fills them; installable as an R package) — https://github.com/ricoderks/QComics
- **R** (Runtime environment in which the QComics package is installed and executed (remotes::install_github))
- **RStudio** (IDE used to launch the QComics shiny app locally via QComics::run_app())

## Examples

```
Rscript -e 'remotes::install_github("ricoderks/QComics"); QComics::run_app()'
```

## Evaluation signals

- Every flagged truly absent feature must have missingness fraction strictly greater than 0.70 across all samples
- Zero-filled values replace missing entries only in the >70% missingness group; features at or below 70% retain their missing values unmodified
- Output CSV of truly absent features exists and contains only features meeting the threshold
- The classification boundary (70%) matches the expert-reviewed QComics true-absence step rather than an ad hoc cutoff
- Exogenous/targeted metabolites are classified as truly absent regardless of their missingness fraction, consistent with the QComics protocol

## Limitations

- The QComics package is marked as experimental (lifecycle badge) — no stable version guarantee
- Sciex MultiQuant version must be > v3.0.3 for the txt export that feeds the workflow
- The 70% threshold is a heuristic adapted from bioanalytical method validation guidelines; no quantitative benchmark is reported in the article
- Requires a pooled QC (QCpool) measured at regular intervals; not applicable to sequences without QCpool samples

## Evidence

- [results] QComics classifies molecular features with more than 70% missing values as metabolites likely to contain real zero values (truly absent), replacing their missing data with real zero values: [evidence span withheld — anchor status: not_found]
- [results] follows the rationale that the true-positive rate in statistical testing is rapidly restored above ~70% missingness (Armitage et al.): [evidence span withheld — anchor status: not_found]
- [readme] The goal of the `QComics` package is to have a quick overview of the quality of a metabolomics or lipidomics study. For this, a pooled sample (QCpool) needs to measured in regular intervals: "The goal of the `QComics` package is to have a quick overview of the quality of a metabolomics or lipidomics study. For this, a pooled sample (QCpool) needs to measured in regular intervals during"
