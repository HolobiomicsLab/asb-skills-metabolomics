---
name: completely-at-random-missingness-simulation
description: Use when evaluating a missing-value imputation method on complete source
  data and you need reproducible MCAR-masked test matrices paired with ground truth
  at fixed missingness fractions.
when_to_use_negative:
- The input data are already incomplete — the skill requires a complete ground-truth
  matrix against which imputation error is measured
- The evaluation design calls for missingness mechanisms other than MCAR (the article's
  simulations remove values completely at random only)
- The goal is error estimation without a ground truth (e.g. using missForest's out-of-bag
  error estimates instead of artificial masking)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Statistical environment in which the complete datasets are loaded, masked
    at 10%/20%/30% MCAR, and the 50 replicates per setting are generated and written
    out
  repo: http://stat.ethz.ch/CRAN/
- name: missForest
  role: Downstream iterative random-forest imputation method whose benchmark evaluation
    (NRMSE/PFC) consumes the MCAR-masked matrices produced by this skill
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:5cfa41bbb6e426df01c1b3bafb50683be5220d4ffb10234e6baf73cce1f6458f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# completely-at-random-missingness-simulation

## Summary

Generates benchmark matrices with artificially introduced missing values (10%, 20%, 30%) removed completely at random (MCAR) from complete biological datasets, with 50 independent replicates per setting, so that imputation methods can be scored against ground truth via NRMSE (continuous) and PFC (categorical).

## When to use

Use when evaluating a missing-value imputation method on complete source data and you need reproducible MCAR-masked test matrices paired with ground truth at fixed missingness fractions.

## When NOT to use

- The input data are already incomplete — the skill requires a complete ground-truth matrix against which imputation error is measured
- The evaluation design calls for missingness mechanisms other than MCAR (the article's simulations remove values completely at random only)
- The goal is error estimation without a ground truth (e.g. using missForest's out-of-bag error estimates instead of artificial masking)

## Inputs

- Complete (fully observed) source matrices in R: e.g. A. thaliana isoprenoid gene network (p=39, n=118), insulin gene expression (p=12,626, n=110), musk molecule shapes (92 molecules), Parkinson's voice measurements (31 individuals)
- Per-dataset variable-type classification (continuous-only, categorical-only, mixed-type)
- Target missingness fractions (10%, 20%, 30%)

## Outputs

- Artificially masked (MCAR) matrices retaining original variable types and scales, per missingness level and replicate
- Paired complete ground-truth matrices for subsequent NRMSE/PFC error computation
- Per-replicate metadata (dataset, variable-type setting, missingness fraction, replicate index), packaged e.g. as simulated_missing_datasets.zip

## How to apply

Load each complete source dataset into the R statistical environment — e.g. the A. thaliana isoprenoid gene network (p=39 genes, n=118 expression profiles), insulin gene expression (p=12,626 genes, n=110 muscle biopsies), musk molecule shapes (92 molecules), and Parkinson's patient voice measurements (31 individuals). Classify each dataset by variable-type composition (continuous-only, categorical-only, or mixed-type) to define the evaluation settings. For each dataset and each missingness level (10%, 20%, 30%), run 50 independent simulations in which values are removed completely at random, retaining the original variable types and scales in each masked matrix. Pair every artificially masked matrix with its complete ground-truth counterpart so NRMSE and PFC error rates can be computed later. Verify that each replicate contains exactly the specified fraction of removed entries and that the intended number of independent replicates exists per setting.

## Related tools

- **R** (Statistical environment in which the complete datasets are loaded, masked at 10%/20%/30% MCAR, and the 50 replicates per setting are generated and written out) — http://stat.ethz.ch/CRAN/
- **missForest** (Downstream iterative random-forest imputation method whose benchmark evaluation (NRMSE/PFC) consumes the MCAR-masked matrices produced by this skill) — http://stat.ethz.ch/CRAN/

## Evaluation signals

- Each generated replicate contains exactly the specified fraction of removed entries (10%, 20% or 30%)
- The design's replicate count is met: 50 independent simulations per dataset/variable-type/missingness setting (exception: 10 simulations for the Insulin dataset in the published evaluation)
- Each masked matrix retains the original variable types and scales of the source dataset
- Every masked matrix is paired with its corresponding complete ground-truth matrix for later NRMSE/PFC error computation
- Per-replicate metadata (dataset, variable-type setting, missingness fraction, replicate index) is present and consistent across the archive

## Limitations

- The published evaluation reduced the replicate count for the high-dimensional Insulin dataset: 'Due to computation time we only perform 10 simulations instead of 50.'
- Only MCAR missingness is simulated; the skill does not cover MAR or MNAR mechanisms.
- Masking is applied to datasets judged complete after preprocessing (e.g. observations with missing values were removed from the Children's Hospital data), so the ground truth is an idealization rather than truly observed completeness.

## Evidence

- [abstract] Evaluation was performed on multiple datasets ... at 10%, 20% and 30% MCAR: "Evaluation is performed on multiple datasets coming from a diverse selection of biological fields with artificially introduced missing values ranging from 10% to 30%."
- [results] Design specifies 50 independent simulations with values removed completely at random: "Due to computation time we only perform 10 simulations instead of 50."
- [results] NRMSE/PFC error computation against artificially masked data underlies the evaluation: "We perform a paired Wilcoxon test of the error rates of the compared methods versus the error rates of missForest."
- [results] Ground-truth completeness is itself constructed by filtering incomplete observations: [evidence span withheld — anchor status: not_found]
