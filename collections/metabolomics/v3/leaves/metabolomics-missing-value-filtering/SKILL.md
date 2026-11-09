---
name: metabolomics-missing-value-filtering
description: Use when preprocessing heterogeneous metabolomics/xenobiotics feature-intensity
  tables where some variables show high missingness and you must decide per variable
  whether missingness means absence (zero) or data loss, and filter variables excessively
  missing across every study group.
when_to_use_negative:
- The dataset is untargeted MS data without pooled QC samples or study-group structure,
  since thresholds are computed per study group
- Missingness has already been resolved (features categorized as absent vs missing
  and filtered), so re-applying the 20%/70% rules would double-filter
- The goal is general QC (signal drift, carryover, outlier removal) rather than specifically
  missing-data handling — use the full QComics sequential workflow instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: QComics
  role: Implements the sequential multistep QC protocol, including the missing-data
    step that separates truly absent data from missing values and filters variables
    >20% missing in all study groups
  repo: github.com/ricoderks/QComics
- name: R
  role: Runtime environment for installing QComics and running the per-group missing-rate
    computation and filtering
- name: RStudio
  role: IDE from which the QComics Shiny app is launched (QComics::run_app())
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:64c66908e6e55a57535998e83da99d92052c1bd05d86871c0e20e31a7dd07d3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QComics
  - R
  - RStudio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolomics-missing-value-filtering

## Summary

QComics-based missing-data handling for targeted metabolomics that distinguishes truly absent data (features with >70% missingness treated as real zeros) from genuinely missing values, and removes variables whose missing rate exceeds 20% in all study groups before imputing the remainder.

## When to use

Use when preprocessing heterogeneous metabolomics/xenobiotics feature-intensity tables where some variables show high missingness and you must decide per variable whether missingness means absence (zero) or data loss, and filter variables excessively missing across every study group.

## When NOT to use

- The dataset is untargeted MS data without pooled QC samples or study-group structure, since thresholds are computed per study group
- Missingness has already been resolved (features categorized as absent vs missing and filtered), so re-applying the 20%/70% rules would double-filter
- The goal is general QC (signal drift, carryover, outlier removal) rather than specifically missing-data handling — use the full QComics sequential workflow instead

## Inputs

- Feature/metabolite intensity table (targeted metabolomics or lipidomics, e.g. Sciex MultiQuant txt export)
- Sample-to-study-group metadata mapping each sample to its study group
- Missing-value percentages per variable, computed separately within each study group

## Outputs

- Filtered feature table with variables missing >20% in all study groups removed and >70%-missing features recoded as true zeros
- Imputed data matrix for retained variables
- CSV report listing each variable as removed or retained with its per-group missing percentages

## How to apply

Load the metabolomics data table (feature/metabolite intensities plus sample-to-study-group metadata) into R after installing the QComics package via remotes::install_github("ricoderks/QComics"). For each variable, compute the missing-value rate separately within every study group, because QComics explicitly separates missing values from truly absent data to avoid losing relevant biological information. Categorize features with >70% missing values as truly absent (real zeros rather than missing). Apply the 20% rule: flag and remove any variable whose missing-value rate exceeds 20% in all study groups; impute the missing values of the remaining variables. Write a CSV report recording each variable as removed or retained along with its per-group missing percentages. The rationale is that a feature missing in every group is likely absent (biological zero), while a variable missing >20% in all groups cannot be reliably quantified, yet these two conditions must be judged separately to preserve biologically meaningful absent signals.

## Related tools

- **QComics** (Implements the sequential multistep QC protocol, including the missing-data step that separates truly absent data from missing values and filters variables >20% missing in all study groups) — github.com/ricoderks/QComics
- **R** (Runtime environment for installing QComics and running the per-group missing-rate computation and filtering)
- **RStudio** (IDE from which the QComics Shiny app is launched (QComics::run_app()))

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()
```

## Evaluation signals

- The removed/retained split in the CSV report exactly matches the rule that a variable is removed only when its missing rate exceeds 20% in every study group
- Features with >70% missing values are recorded as truly absent (real zeros), not as removed-missing variables
- Per-group missing percentages in the report are reproducible when recomputed within each study group from the input table
- The R script runs without error and the output CSV exists
- No variable retained for imputation has >20% missingness in all study groups

## Limitations

- QComics is labeled experimental (lifecycle badge) and expects QCpool samples measured at regular intervals and analyzed with Sciex MultiQuant (> v3.0.3) exported to txt format
- No version number, changelog, or quantitative validation benchmark for the protocol is provided
- Thresholds (20% removal, 70% absence) are predefined quality criteria adapted from bioanalytical method validation guidelines, which may not suit every study design

## Evidence

- [abstract] separately handling missing values and truly absent data: "such as the need of separately handling missing values and truly absent data to avoid losing relevant biological information"
- [full_text] 20% filter rule: [evidence span withheld — anchor status: not_found]
- [readme] installation: "You can install the development version of QComics like so:

``` r
remotes::install_github("ricoderks/QComics")
```"
- [intro] designed for heterogeneous endogenous + xenobiotic data: "it also becomes essential to adapt current QC practices for dealing with heterogeneous data sets comprising both endogenous metabolites and xenobiotics"
- [intro] predefined criteria provenance: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
