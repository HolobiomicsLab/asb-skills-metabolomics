---
name: missing-value-artificial-mask-generation
description: Use when you have one or more fully observed datasets (continuous, categorical,
  or mixed-type) and need controlled artificial missingness benchmarks to evaluate
  imputation methods against known ground truth.
when_to_use_negative:
- The data already contain genuinely missing values — artificial masking assumes a
  complete ground-truth matrix as the starting point
- Missingness is expected to be MNAR or MAR rather than MCAR — this mask generator
  removes values completely at random and will not reflect structured missingness
  mechanisms
- The goal is a real imputation application rather than a benchmark — masking destroys
  observed data and is only useful when ground-truth error evaluation (NRMSE/PFC)
  is the objective
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Statistical environment in which the complete datasets are loaded, values
    are removed completely at random at each missingness level, and masked replicate
    matrices are generated and written out
- name: missForest
  role: Downstream non-parametric imputation method (random-forest based) whose performance
    on the artificially masked replicates is evaluated via NRMSE and PFC against ground
    truth; R package available from CRAN
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:1f44e3150080687fe9ba43ca0e422f9f720ceae02fa97a13b63bab6dab80ed3c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# missing-value-artificial-mask-generation

## Summary

Generates benchmark matrices for imputation evaluation by removing values completely at random (MCAR) at 10%, 20% or 30% from complete source datasets, producing 50 independent masked replicates paired with their ground-truth complete matrices for NRMSE/PFC error computation.

## When to use

Use when you have one or more fully observed datasets (continuous, categorical, or mixed-type) and need controlled artificial missingness benchmarks to evaluate imputation methods against known ground truth.

## When NOT to use

- The data already contain genuinely missing values — artificial masking assumes a complete ground-truth matrix as the starting point
- Missingness is expected to be MNAR or MAR rather than MCAR — this mask generator removes values completely at random and will not reflect structured missingness mechanisms
- The goal is a real imputation application rather than a benchmark — masking destroys observed data and is only useful when ground-truth error evaluation (NRMSE/PFC) is the objective

## Inputs

- Complete (fully observed) source dataset, e.g. A. thaliana isoprenoid gene network (p=39 genes, n=118 expression profiles)
- Complete continuous gene expression matrix, e.g. insulin data (p=12,626 genes, n=110 muscle biopsies)
- Categorical datasets, e.g. musk molecule shapes (92 molecules) or Parkinson's voice measurements (31 individuals)
- Missingness proportions to induce: 10%, 20%, 30%
- Number of independent simulation replicates per setting (50; 10 for very high-dimensional data)

## Outputs

- Masked matrices with exactly 10%, 20% or 30% of entries removed completely at random, original variable types and scales retained
- Paired complete ground-truth matrices for each masked replicate
- Per-replicate metadata: dataset, variable-type setting (continuous/categorical/mixed), missingness fraction, replicate index
- Packaged simulation replicates (e.g. simulated_missing_datasets.zip)

## How to apply

Load each complete source dataset (e.g. the A. thaliana isoprenoid gene network with p=39 genes/n=118 profiles, insulin gene expression with p=12,626 genes/n=110 biopsies, musk molecules, or Parkinson's voice measurements) into the R statistical environment. Classify each dataset's variable-type composition as continuous-only, categorical-only, or mixed-type to define the evaluation settings. For each dataset and each missingness level (10%, 20%, 30%), run 50 independent simulations in which values are removed completely at random (MCAR), retaining the original variable types and scales in each masked matrix. Pair every artificially masked matrix with its corresponding complete ground-truth matrix so that NRMSE (continuous) and PFC (categorical) errors can later be computed. Exception: for very high-dimensional data such as the insulin gene expression set, reduce the replicate count (the article used 10 instead of 50 simulations due to computation time). Write out all replicates with per-replicate metadata (dataset, variable-type setting, missingness fraction, replicate index).

## Related tools

- **R** (Statistical environment in which the complete datasets are loaded, values are removed completely at random at each missingness level, and masked replicate matrices are generated and written out)
- **missForest** (Downstream non-parametric imputation method (random-forest based) whose performance on the artificially masked replicates is evaluated via NRMSE and PFC against ground truth; R package available from CRAN) — http://stat.ethz.ch/CRAN/

## Examples

```
Rscript generate_masked_datasets.R --dataset arabidopsis_isoprenoid.csv --levels 0.10,0.20,0.30 --n_sim 50 --mechanism MCAR --out simulated_missing_datasets.zip
```

## Evaluation signals

- Each generated replicate contains exactly the specified fraction of removed entries (10%, 20% or 30% of matrix cells)
- 50 independent replicates exist per dataset × variable-type × missingness-level setting (10 instead of 50 only where justified by computation time, as for the insulin data)
- Every masked matrix is paired with its corresponding complete ground-truth matrix so NRMSE/PFC can be computed
- Variable types and scales are retained unchanged between the masked matrix and the ground-truth matrix
- Masked positions are distributed completely at random (MCAR) across rows and columns, with different random patterns across the 50 replicates

## Limitations

- Only MCAR missingness is simulated; the article evaluates imputation under values 'removed completely at random' and does not cover MAR or MNAR mechanisms
- Masking from a complete ground truth requires datasets that were originally fully observed; real incomplete datasets (e.g. Children's Hospital data, which required removing observations with missing values) must be cleaned first
- High-dimensional datasets make the replicate design costly: for the insulin gene expression set (p=12,626) only 10 simulations were performed instead of 50 due to computation time
- Masking plus 50-replicate × 3-level design multiplies downstream imputation runs; competing methods can be computationally infeasible (e.g. MissPALasso could not be run on the insulin data)

## Evidence

- [results] This gene network includes p=39 genes each with n=118 gene expression profiles corresponding to different experimental conditions.: [evidence span withheld — anchor status: not_found]
