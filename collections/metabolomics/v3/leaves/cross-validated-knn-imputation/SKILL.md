---
name: cross-validated-knn-imputation
description: Use when a benchmark requires KNNimpute error rates on data with artificially
  introduced missing values (10%, 20% or 30% MCAR) but no a priori neighbour count
  k is given, so k_ must be chosen by cross-validation.
when_to_use_negative:
- 'The goal is a single off-the-shelf imputation without benchmarking: the cross-validation
  over k (Algorithm 2) exists only to give KNNimpute a fair, parameter-tuned comparison
  against missForest, which has no k-like parameter'
- The data are mixed-type and you need a method that handles variable types natively
  without the {-1,1} dummy-coding workaround — missForest or MICE are the paper's
  choices there
- The data contain no missing values (or missingness is not MCAR in the 10–30% range
  the evaluation assumes)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KNNimpute
  role: Baseline nearest-neighbour imputation algorithm (Algorithm 2) whose neighbour
    count k_best is tuned by cross-validation; run on standardized continuous data
    or on {-1,1} dummy-coded categorical data
- name: R
  role: Implementation environment in which the cross-validated KNNimpute baseline
    is run alongside missForest, MissPALasso and MICE for the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:89cd2c0d054dacd34dee1043bf0fcbaf385afbb0574d4a2b6d3fa30a86b09bb4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KNNimpute
  - MICE
  - MissPALasso
  - R
  - missForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-validated-knn-imputation

## Summary

Reproduces the KNNimpute baseline (Troyanskaya et al. 2001) from Stekhoven & Bühlmann (2011) as a cross-validated competitor: the neighbour count k_best is selected by cross-validation (Algorithm 2) over a candidate range K, imputation is done on standardized data, and the imputed matrix is retransformed to the original scales before computing NRMSE (continuous data) or PFC (dummy-coded categorical data).

## When to use

Use when a benchmark requires KNNimpute error rates on data with artificially introduced missing values (10%, 20% or 30% MCAR) but no a priori neighbour count k is given, so k_best must be chosen by cross-validation.

## When NOT to use

- The goal is a single off-the-shelf imputation without benchmarking: the cross-validation over k (Algorithm 2) exists only to give KNNimpute a fair, parameter-tuned comparison against missForest, which has no k-like parameter
- The data are mixed-type and you need a method that handles variable types natively without the {-1,1} dummy-coding workaround — missForest or MICE are the paper's choices there
- The data contain no missing values (or missingness is not MCAR in the 10–30% range the evaluation assumes)

## Inputs

- Continuous data matrix with artificially introduced missing values (e.g. A. thaliana isoprenoid gene expression n=118/p=39, Parkinson's voice measures, musk molecule shapes, Insulin gene expression p=12,626/n=110, Gaucher proteomics, C. elegans GFOP peptides), standardized for imputation
- Categorical/mixed-type data matrix (e.g. SPECT binary features, E. coli promoter sequences, lymphography, Children's Hospital data with 48 continuous and 76 categorical variables) coded into {-1,1} dummies for the dummy-coded KNNimpute variant
- Candidate range K of neighbour counts k for cross-validation

## Outputs

- Selected neighbour count k_best from cross-validation over K
- Completed data matrix retransformed to the original variable scales (and dummies coded back to categories for the dummy-coded variant)
- Per-simulation NRMSE values over the continuous missing entries (and PFC over categorical missing entries), fed into imputation_comparison.csv with paired Wilcoxon test outcomes versus missForest

## How to apply

Center all variables at zero and scale them to unit standard deviation so that no variable dominates the neighbour distance by its variance. Run Algorithm 2: hold out observed entries, impute them with candidate k values from a candidate range K, and select k_best minimizing the held-out error, since KNNimpute itself provides no parameter-free error estimate. Impute the truly missing entries with k_best and retransform the imputed matrix to the original scales so error is computed on the original units. For categorical or mixed-type data, first code each categorical variable with m levels into m dichotomous {-1,1} dummy variables, standardize, run the same cross-validated Algorithm 2, then retransform and code the dummies back to categories. Compute NRMSE over the continuous missing entries (and PFC over categorical ones) per simulation and compare against missForest with a paired Wilcoxon test of the error rates.

## Related tools

- **KNNimpute** (Baseline nearest-neighbour imputation algorithm (Algorithm 2) whose neighbour count k_best is tuned by cross-validation; run on standardized continuous data or on {-1,1} dummy-coded categorical data)
- **R** (Implementation environment in which the cross-validated KNNimpute baseline is run alongside missForest, MissPALasso and MICE for the benchmark)

## Evaluation signals

- NRMSE values for every method fall on the paper's scale (good performance near 0, bad performance around 1)
- The selected k_best lies within the candidate range K and is the cross-validation minimizer of held-out imputation error (Algorithm 2)
- Errors are computed on the original scales: the imputed matrix is retransformed after imputation (and dummies are coded back to categories for categorical data) before NRMSE/PFC computation
- Aggregated means and paired Wilcoxon test outcomes of competitor error rates versus missForest's are recorded per comparison in imputation_comparison.csv
- Across the 50 (10 for Insulin) simulations, KNNimpute is competitive to or worse than missForest, e.g. missForest reduces average NRMSE by up to 25% relative to KNNimpute on continuous data and above 50% on musk molecules

## Limitations

- Evaluation relies on artificially introduced missing values (10% to 30%, completely at random); performance under other missingness mechanisms is not assessed
- For categorical variables the {-1,1} dummy coding plus retransformation is an ad-hoc workaround rather than native handling, and results must be judged accordingly
- missForest's own OOB error estimates remove the need for such cross-validation; this skill's CV machinery exists only because KNNimpute has a tunable k parameter

## Evidence

- [intro] Here we compare our method with k nearest neighbour imputation (KNNimpute, Troyanskaya et al. (2001)): [evidence span withheld — anchor status: not_found]
- [methods] Since our method includes no such parameter we implement a cross-validation (see Algorithm 2) to obtain a suitable k.: "Since our method includes no such parameter we implement a cross-validation (see Algorithm 2) to obtain a suitable k."
- [methods] To avoid variance based weighting of the variables we scale them to a unit standard deviation. We also center the variables at zero.: "To avoid variance based weighting of the variables we scale them to a unit standard deviation. We also center the variables at zero."
- [methods] This is done by coding a categorical variable Xj into m dichotomous variables: [evidence span withheld — anchor status: not_found]
