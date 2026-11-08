---
name: metabolomics-data-matrix-manipulation
description: Use when you need synthetic MNAR (left-censored) missingness in a complete
  targeted/un-targeted metabolomics matrix to benchmark imputation methods such as
  GSimp, QRILC, HM, or kNN-TN.
when_to_use_negative:
- The matrix already contains missing values (this skill requires a complete matrix
  as the ground-truth starting point)
- You need MCAR/MAR missingness rather than MNAR left-censoring — e.g., for the untargeted
  GC/MS data before manual missing retrieval, MCAR/MAR was assumed
- The goal is imputation itself rather than missingness generation — use pre_processing_GS_wrapper/GS_impute
  instead
tools:
- name: GSimp
  role: Provides MNAR_gen_imp in GSimp_evaluation.R, the function that performs MNAR
    left-censored masking of complete data matrices
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: Execution environment; source GSimp_evaluation.R, GSimp.R, and Trunc_KNN/Imput_funcs.r
    to run the MNAR generation
  repo: https://github.com/WandeRum/GSimp
- name: MVI-evaluation
  role: Companion repository containing the imputation evaluation pipeline (missing
    generation, imputations, evaluations) that consumes the generated MNAR matrices
  repo: https://github.com/WandeRum/MVI-evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:88b81567d54eb5d1341bfad08a11b7c2b6ae49cf585a6456c3eea8cc8fd73b7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - MVI-evaluation
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolomics-data-matrix-manipulation

## Summary

A procedure from the GSimp/MVI-evaluation pipeline (R) that transforms a complete, scaled metabolomics data matrix into a series of MNAR left-censored masked matrices via the MNAR_gen_imp function, for benchmarking missing-value imputation methods.

## When to use

Use when you need synthetic MNAR (left-censored) missingness in a complete targeted/un-targeted metabolomics matrix to benchmark imputation methods such as GSimp, QRILC, HM, or kNN-TN.

## When NOT to use

- The matrix already contains missing values (this skill requires a complete matrix as the ground-truth starting point)
- You need MCAR/MAR missingness rather than MNAR left-censoring — e.g., for the untargeted GC/MS data before manual missing retrieval, MCAR/MAR was assumed
- The goal is imputation itself rather than missingness generation — use pre_processing_GS_wrapper/GS_impute instead

## Inputs

- Complete (fully observed) scaled data matrix in R, loaded from real_data.csv, targeted_data.csv, untargeted_data.csv, or data_sim.csv
- Vector of missing proportions (mis_var_prop), e.g., seq(.1, .8, .1) for the simulation dataset or 0.1–0.6 step 0.05 for real-world datasets
- Quantile cut-off distribution specification (U(0.3, 0.6) for simulation, U(0.1, 0.5) for real-world data)

## Outputs

- List of MNAR-masked data matrices (e.g., sim_MNAR_list), one per requested missing-proportion level, with left-censored entries set to NA
- Per-level MNAR matrices saved as CSV files in an output directory

## How to apply

Source GSimp_evaluation.R from WandeRum/GSimp, then call MNAR_gen_imp on a complete, scaled data matrix (e.g., data_sim_sc from data_sim.csv, or real_data.csv, targeted_data.csv, untargeted_data.csv) with a vector of missing proportions (mis_var_prop = seq(.1, .8, .1) for the simulation dataset; 0.1 to 0.6 step 0.05 for real-world datasets). For each proportion level the function determines the fraction of variables to censor and draws a per-variable quantile cut-off from a uniform distribution — U(0.3, 0.6) for the simulation dataset or U(0.1, 0.5) for real-world datasets — then sets all values below each variable's cut-off to NA, emulating LOD/LOQ left-censoring. The result is a list of MNAR-masked matrices (e.g., sim_MNAR_list), one per missing-proportion level, which should be written out as CSV files per level. Rationale: left-censoring below a quantile reproduces the MNAR mechanism caused by detection limits in MS-based targeted metabolomics, which is the missingness pattern GSimp is designed to impute.

## Related tools

- **GSimp** (Provides MNAR_gen_imp in GSimp_evaluation.R, the function that performs MNAR left-censored masking of complete data matrices) — https://github.com/WandeRum/GSimp
- **R** (Execution environment; source GSimp_evaluation.R, GSimp.R, and Trunc_KNN/Imput_funcs.r to run the MNAR generation) — https://github.com/WandeRum/GSimp
- **MVI-evaluation** (Companion repository containing the imputation evaluation pipeline (missing generation, imputations, evaluations) that consumes the generated MNAR matrices) — https://github.com/WandeRum/MVI-evaluation

## Examples

```
source('GSimp_evaluation.R'); sim_MNAR_list <- MNAR_gen_imp(data_c = data_sim_sc, mis_var_prop = seq(.1, .8, .1))
```

## Evaluation signals

- For each output MNAR matrix, the realized proportion of missing variables matches the requested level in mis_var_prop
- No values above the per-variable quantile cut-offs were removed (only elements below the cut-off became NA)
- The NA pattern is left-censored: masked entries are confined to the lower tail of each variable's distribution
- The series of matrices spans the full requested range of missing proportions (e.g., 0.1–0.8 by 0.1 for the simulation dataset)
- Each generated matrix is saved as a CSV file with dimensions identical to the input complete matrix

## Limitations

- Quantile cut-offs are randomly drawn (U(0.1, 0.5) or U(0.3, 0.6)), so results vary between runs unless a random seed is fixed
- The simulation dataset's parameter choices (covariance from the diabetes dataset, P=76) are not fully justified in the paper
- The mask emulates LOD/LOQ-type MNAR only; it does not cover MCAR/MAR mechanisms relevant to pre-retrieval untargeted GC/MS data

## Evidence

- [methods] sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
- [readme] GSimp_evaluation.R contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline.: "**GSimp_evaluation.R** contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline."
- [readme] data_sim.csv is the simulated dataset (details could be found in simulation_generation.Rmd).: "**data_sim.csv** is the simulated dataset (details could be found in **simulation_generation.Rmd**)."
- [abstract] Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics studies: "Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics studies"
