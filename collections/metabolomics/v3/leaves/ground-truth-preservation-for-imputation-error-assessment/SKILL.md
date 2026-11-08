---
name: ground-truth-preservation-for-imputation-error-assessment
description: Use when complete source datasets are available and a controlled imputation
  benchmark is needed — create MCAR-masked replicates at fixed missingness fractions,
  retaining the complete matrix as the error-assessment reference.
when_to_use_negative:
- The source data already contains real (non-artificial) missing values, so no complete
  ground-truth matrix exists for NRMSE/PFC computation — as in the Children's Hospital
  data where observations with missing values had to be removed first
- The missingness mechanism of interest is not MCAR (the simulations remove values
  completely at random, so MAR/MNAR scenarios are out of scope)
- The matrix to be masked is high-dimensional such that the replicate budget is infeasible
  — the article reduced insulin simulations from 50 to 10 due to computation time
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Statistical environment in which complete datasets are loaded, MCAR masks
    are generated, replicates and metadata are written out, and NRMSE/PFC errors are
    later computed against ground truth
  repo: http://stat.ethz.ch/CRAN/
- name: missForest
  role: R package whose complete/masked data workflow motivates this skill; masking
    must preserve original variable types and scales so errors are computed on the
    original scales
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:dc0e1f41a00bd0d5debd529de61f2b2ba0056de900be962c925e1218671aa0e8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ground-truth-preservation-for-imputation-error-assessment

## Summary

Generate artificially masked versions of complete reference datasets (MCAR removal of 10%, 20% or 30% of entries) while preserving the untouched complete matrix as ground truth, so that imputation error can later be quantified with NRMSE (continuous) and PFC (categorical) metrics.

## When to use

Use when complete source datasets are available and a controlled imputation benchmark is needed — create MCAR-masked replicates at fixed missingness fractions, retaining the complete matrix as the error-assessment reference.

## When NOT to use

- The source data already contains real (non-artificial) missing values, so no complete ground-truth matrix exists for NRMSE/PFC computation — as in the Children's Hospital data where observations with missing values had to be removed first
- The missingness mechanism of interest is not MCAR (the simulations remove values completely at random, so MAR/MNAR scenarios are out of scope)
- The matrix to be masked is high-dimensional such that the replicate budget is infeasible — the article reduced insulin simulations from 50 to 10 due to computation time

## Inputs

- Complete (unmasked) source dataset, e.g. A. thaliana isoprenoid gene network (p=39 genes, n=118 expression profiles)
- Complete insulin gene expression matrix (p=12,626 genes, n=110 muscle biopsies)
- Complete musk molecule shape data (92 molecules, 47 musks / 45 non-musks)
- Complete Parkinson's voice measurement data (31 individuals, 23 with PD)
- Missingness levels to simulate: 10%, 20%, 30%; number of independent replicates per setting: 50

## Outputs

- simulated_missing_datasets.zip containing all masked simulation replicates
- Per-replicate metadata: dataset, variable-type setting (continuous/categorical/mixed), missingness fraction, replicate index
- Paired ground-truth (complete) matrix and artificially masked matrix per replicate, with original variable types and scales retained

## How to apply

Load each complete source dataset (e.g. A. thaliana isoprenoid gene network p=39/n=118, insulin expression p=12,626/n=110, musk molecules 92 shapes, Parkinson's voice measurements from 31 individuals) into R, and classify each variable-type composition as continuous-only, categorical-only, or mixed-type. For each dataset and each missingness level (10%, 20%, 30%), run 50 independent simulations in which values are removed completely at random (MCAR). Retain the original variable types and scales in each masked matrix — after imputation the error must be computed on the original scales — and pair every masked matrix with its corresponding complete ground-truth matrix for later NRMSE/PFC computation. Write out all replicates with per-replicate metadata (dataset, variable-type setting, missingness fraction, replicate index), packaged as simulated_missing_datasets.zip.

## Related tools

- **R** (Statistical environment in which complete datasets are loaded, MCAR masks are generated, replicates and metadata are written out, and NRMSE/PFC errors are later computed against ground truth) — http://stat.ethz.ch/CRAN/
- **missForest** (R package whose complete/masked data workflow motivates this skill; masking must preserve original variable types and scales so errors are computed on the original scales) — http://stat.ethz.ch/CRAN/

## Examples

```
R: naidx <- sample(seq_along(x), size = round(0.2*length(x))); x[naidx] <- NA  # MCAR 20% mask, keep complete copy as ground truth
```

## Evaluation signals

- Verify each generated replicate contains exactly the specified fraction of removed entries (10%, 20% or 30%)
- Verify 50 independent replicates exist per dataset/variable-type/missingness setting (exception: only 10 for the insulin data due to computation time)
- Verify the masked matrix retains the original variable types and scales of the complete matrix (no standardization/dummy-coding baked into the stored masked data)
- Verify every artificially masked matrix is stored paired with its corresponding complete ground-truth matrix
- Confirm MCAR removal: the deleted-entry positions should be uniformly random, matching 'removed completely at random'

## Limitations

- For the insulin dataset (p=12,626) only 10 simulations instead of 50 could be performed due to computation time, reducing statistical power at that setting
- MissPALasso could not be run on the insulin data because the high dimension made computation infeasible, so ground-truth-based comparisons are incomplete for that dataset
- On the Children's Hospital data (n=55, p=124 with 48 continuous and 76 categorical variables), observations with missing values had to be removed and nearly dependent variables were removed after each introduction of missing values, so ground-truth fidelity was imperfect for MICE comparisons

## Evidence

- [results] Ground-truth fidelity issue on Children's Hospital data: [evidence span withheld — anchor status: not_found]
