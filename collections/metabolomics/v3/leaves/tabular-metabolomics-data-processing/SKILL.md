---
name: tabular-metabolomics-data-processing
description: Use when a metabolomics or lipidomics feature intensity table (with sample-to-study-group
  metadata) contains missing values and variables must be separated into truly absent
  features vs. missing data before imputation and downstream QC.
when_to_use_negative:
- When the goal is signal-drift correction, carryover/background-noise correction,
  or outlier detection — these are separate upstream QComics steps and do not involve
  the missing-data criterion
- When the input is not a heterogeneous endogenous-metabolite/xenobiotic intensity
  table with study-group labels (e.g. already-normalized omics matrices without group
  metadata)
- When a different predefined missingness threshold is mandated by the study design;
  the 20%/70% cutoffs are QComics-specific quality criteria
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: QComics
  role: R/Shiny package implementing the sequential QC workflow including handling
    of missing values and truly absent data
  repo: github.com/ricoderks/QComics
- name: R
  role: Runtime environment; QComics is installed via remotes and run from RStudio
  repo: github.com/ricoderks/QComics
- name: RStudio
  role: IDE from which the QComics app is launched (QComics::run_app())
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:8cce95443c5b377411747d170b99fcc2575c46bb8604c0a12a4b070cd2a63b8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QComics
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tabular-metabolomics-data-processing

## Summary

Applies the QComics missing-data handling criterion to feature/metabolite intensity tables: classify features with >70% missingness as truly absent (real zeros), remove variables whose missing rate exceeds 20% in every study group, and impute the remaining missing values.

## When to use

Use when a metabolomics or lipidomics feature intensity table (with sample-to-study-group metadata) contains missing values and variables must be separated into truly absent features vs. missing data before imputation and downstream QC.

## When NOT to use

- When the goal is signal-drift correction, carryover/background-noise correction, or outlier detection — these are separate upstream QComics steps and do not involve the missing-data criterion
- When the input is not a heterogeneous endogenous-metabolite/xenobiotic intensity table with study-group labels (e.g. already-normalized omics matrices without group metadata)
- When a different predefined missingness threshold is mandated by the study design; the 20%/70% cutoffs are QComics-specific quality criteria

## Inputs

- Feature/metabolite intensity table (tabular, from targeted MS metabolomics, e.g. Sciex MultiQuant txt export)
- Sample-to-study-group metadata mapping samples to study groups

## Outputs

- CSV report recording each variable as removed or retained with its per-study-group missing percentages
- Filtered intensity table with >70%-missing features reclassified as real zeros, >20%-missing-in-all-groups variables removed, and remaining missing values imputed

## How to apply

Load the feature/metabolite intensity table plus sample-to-study-group metadata into R. For each variable (endogenous metabolite or xenobiotic), compute the missing-value rate separately within every study group, because QComics explicitly distinguishes missing values from truly absent data to avoid losing relevant biological information. Categorize features with >70% missing values as truly absent (real zeros). Flag and remove any variable whose missing-value rate exceeds 20% in all study groups; impute the remaining missing values. This criterion is part of the QComics sequential multistep QC workflow (step ii: handling missing values and truly absent data), whose quality criteria were adapted from bioanalytical method validation guidelines.

## Related tools

- **QComics** (R/Shiny package implementing the sequential QC workflow including handling of missing values and truly absent data) — github.com/ricoderks/QComics
- **R** (Runtime environment; QComics is installed via remotes and run from RStudio) — github.com/ricoderks/QComics
- **RStudio** (IDE from which the QComics app is launched (QComics::run_app()))

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- The removed/retained split exactly matches the rule that a variable is removed only when its missing rate exceeds 20% in every study group
- Features with >70% missing values are classified as truly absent (real zeros), not removed as missing
- The output CSV exists and records per-group missing percentages for each variable
- The R script runs without error after remotes::install_github("ricoderks/QComics")

## Limitations

- QComics is experimental (lifecycle badge) and expects QCpool samples measured in regular intervals and analysed with Sciex MultiQuant (> v3.0.3) exported to txt format
- Quality criteria are QComics-specific adaptations from bioanalytical validation guidelines; no quantitative benchmark or validation metrics are reported in the provided material
- No version number, changelog, or dependency requirements are stated for the QComics package

## Evidence

- [abstract] separately handling missing values and truly absent data: "such as the need of separately handling missing values and truly absent data to avoid losing relevant biological information"
- [intro] handling missing values and truly absent data: "(ii) handling missing values and truly absent data"
- [intro] quality criteria adapted from guidelines: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21"
- [methods] installation via remotes: "remotes::install_github("ricoderks/QComics")"
- [readme] run app locally: [evidence span withheld — anchor status: not_found]
- [readme] MultiQuant txt export requirement: "These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt` format."
