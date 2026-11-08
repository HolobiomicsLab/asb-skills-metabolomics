---
name: simulation-dataset-generation
description: Reference for constructing a synthetic, fully observed metabolomics dataset
  with realistic covariance and simulated group differences when no complete ground-truth
  data exist for benchmarking imputation methods.
when_to_use_negative:
- When a fully observed real dataset already exists — the article's goal is a synthetic
  surrogate because real complete data are unavailable for left-censored MNAR evaluation.
- When imputation must be evaluated on targeted metabolomics data with real group
  labels — this simulation only preserves covariance and creates synthetic group differences,
  not real biological annotations.
- When the data are non-targeted GC/MS profiles treated as MCAR/MAR after missing
  retrieval — the simulation is tailored to targeted (P=76 FFAs/BA) covariance structure.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Language used to compute the covariance matrix Cov from the diabetes dataset
    and generate the multivariate-normal simulated matrices
  repo: https://github.com/WandeRum/GSimp
- name: GSimp
  role: Repository (GSimp.R, MVI_global.R, simulation_generation.Rmd) that houses
    the simulation generation script and downstream MNAR_gen_imp masking used with
    the simulated matrix
  repo: https://github.com/WandeRum/GSimp
- name: SVD
  role: Singular value decomposition used to keep/re-decompose the covariance matrix
    Cov when constructing each 80-observation multivariate-normal draw
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:691317c43cf9b6cc66a3064f8b7e1c8b42d083a4864129b4bc9b8a723f131f7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - R
  - SVD
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# simulation-dataset-generation

## Summary

Generates a complete simulated targeted metabolomics matrix (data_sim.csv, 160x76) by drawing multivariate-normal samples whose covariance is preserved from the real diabetes FFAs/BAs dataset, to serve as ground truth for MNAR imputation evaluation.

## When to use

Reference for constructing a synthetic, fully observed metabolomics dataset with realistic covariance and simulated group differences when no complete ground-truth data exist for benchmarking imputation methods.

## When NOT to use

- When a fully observed real dataset already exists — the article's goal is a synthetic surrogate because real complete data are unavailable for left-censored MNAR evaluation.
- When imputation must be evaluated on targeted metabolomics data with real group labels — this simulation only preserves covariance and creates synthetic group differences, not real biological annotations.
- When the data are non-targeted GC/MS profiles treated as MCAR/MAR after missing retrieval — the simulation is tailored to targeted (P=76 FFAs/BA) covariance structure.

## Inputs

- Real-world diabetes targeted metabolomics dataset with P=76 variables (FFAs and BAs) for covariance estimation
- R environment with package dependencies: magrittr, knitr (for simulation_generation.Rmd)

## Outputs

- data_sim.csv — complete simulated data matrix with 160 rows (80 samples x 2 groups) and 76 columns (P=76 variables), written with row names
- Series of MNAR-masked matrices generated downstream from data_sim.csv by MNAR_gen_imp at missing proportions 0.1 to 0.8 step 0.1

## How to apply

Load the real-world diabetes dataset (P=76 variables containing FFAs and BAs) with R and compute the covariance matrix Cov from the whole dataset. Draw per-variable sample means from N(0, 0.5^2) and construct the first 80-observation data matrix from a multivariate normal distribution with covariance Cov, keeping Cov via singular value decomposition (SVD). Repeat to generate a second 80-observation matrix for the other biological group, then column-wise (horizontally) stack the two 80x76 matrices into a complete 160x76 data matrix so group differences and covariance are preserved. Write the result to data_sim.csv with row names. The rationale is that preserving the empirical covariance of the real targeted metabolomics data makes the simulated left-censored MNAR masking (via MNAR_gen_imp) and subsequent GSimp/QRILC/kNN-TN benchmarking meaningful, while the two-group design yields t-test p-values on the order of 1e-6 to 1e-11 that validate simulated group separation.

## Related tools

- **R** (Language used to compute the covariance matrix Cov from the diabetes dataset and generate the multivariate-normal simulated matrices) — https://github.com/WandeRum/GSimp
- **GSimp** (Repository (GSimp.R, MVI_global.R, simulation_generation.Rmd) that houses the simulation generation script and downstream MNAR_gen_imp masking used with the simulated matrix) — https://github.com/WandeRum/GSimp
- **SVD** (Singular value decomposition used to keep/re-decompose the covariance matrix Cov when constructing each 80-observation multivariate-normal draw)

## Examples

```
Rscript -e "source('simulation_generation.Rmd'); Cov <- cov(real_data); mean_vec <- rnorm(76, 0, 0.5); m1 <- MASS::mvrnorm(80, mean_vec, Cov); write.csv(data_sim, 'data_sim.csv', row.names=TRUE)"
```

## Evaluation signals

- Output file data_sim.csv exists with exactly 160 rows and 76 columns (P=76).
- Two-sample t-tests between the two simulated groups of 80 samples yield small p-values (on the order of 1e-6 to 1e-11) for a subset of variables, confirming simulated group differences.
- Covariance preservation: the covariance matrix used for the multivariate-normal draws matches the covariance computed from the whole diabetes dataset (checked via SVD round-trip).
- When passed to MNAR_gen_imp, each resulting MNAR-masked matrix has a missing-variable proportion matching the requested level (0.1–0.8 step 0.1) and no values above the per-variable quantile cut-offs were removed.

## Limitations

- Simulation parameter choices (sample means drawn from N(0, 0.5^2), covariance from the diabetes dataset) are not fully justified in the article's discussion — they are heuristic, not calibrated to a generative model of metabolomics noise.
- The simulation preserves the covariance of a single real dataset (P=76 FFAs/BA), so covariance structure from other platforms (e.g., non-targeted GC/MS) is not represented.
- The generated matrix is complete (no missing values); realistic left-censoring must be added separately via MNAR_gen_imp with per-variable quantile cut-offs drawn from U(0.3, 0.6).

## Evidence

- [methods] Covariance estimation from diabetes dataset: "we first calculated the covariance matrix Cov based on the whole diabetes dataset (*P*=76)"
- [readme] Simulation dataset description: "**data_sim.csv** is the simulated dataset (details could be found in **simulation_generation.Rmd**)."
- [methods] Covariance-based generation algorithm: "the sample mean of each variable was drawn from a normal distribution N(0, 0.52) and Cov was kept using SVD"
- [methods] MNAR masking on simulated data: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
