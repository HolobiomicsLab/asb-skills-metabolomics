---
name: multivariate-normal-sampling
description: Use when a benchmark needs a fully-observed synthetic data matrix with
  realistic covariance and group structure, e.g., to evaluate imputation methods under
  controlled MNAR masking before injecting missing values.
when_to_use_negative:
- The goal is to impute an already-missing real dataset (e.g., targeted_data.csv or
  untargeted_data.csv) rather than to generate synthetic fully-observed benchmark
  data
- The data are non-normal and cannot be reasonably modeled by a multivariate normal
  without prior log-transformation
- A benchmark requires real missingness patterns (LOD/LOQ-driven MNAR from actual
  MS acquisition) instead of simulated group structure
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language used to compute the covariance matrix Cov from the diabetes dataset
    and to draw the two multivariate-normal 80-observation groups
  repo: https://github.com/WandeRum/GSimp
- name: SVD
  role: Singular value decomposition used to retain/store the covariance matrix Cov
    during simulation generation
- name: GSimp
  role: Repository (simulation_generation.Rmd) containing the simulation dataset generation
    algorithm whose output, data_sim.csv, feeds the imputation evaluation pipeline
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:b72275ffb83844f31981959265b8cd158fa292a2cf8d37d9c93430d19392d52f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - R
  - SVD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# multivariate-normal-sampling

## Summary

Generates a simulated metabolomics dataset by drawing two 80-observation groups from a multivariate normal distribution whose covariance matrix Cov is estimated from a real targeted metabolomics dataset (the diabetes dataset with P=76 FFAs and BAs), preserving realistic covariance structure and between-group differences for imputation benchmarking.

## When to use

Use when a benchmark needs a fully-observed synthetic data matrix with realistic covariance and group structure, e.g., to evaluate imputation methods under controlled MNAR masking before injecting missing values.

## When NOT to use

- The goal is to impute an already-missing real dataset (e.g., targeted_data.csv or untargeted_data.csv) rather than to generate synthetic fully-observed benchmark data
- The data are non-normal and cannot be reasonably modeled by a multivariate normal without prior log-transformation
- A benchmark requires real missingness patterns (LOD/LOQ-driven MNAR from actual MS acquisition) instead of simulated group structure

## Inputs

- Real-world diabetes targeted metabolomics dataset with P=76 variables (FFAs and BAs), used to estimate the empirical covariance matrix Cov
- Simulation parameters: number of observations per group (80 per group, 160 total), number of variables (P=76), mean distribution N(0, 0.5^2)

## Outputs

- Complete simulated 160x76 data matrix written to data_sim.csv (with row names)
- Two simulated groups of 80 samples each with preserved covariance and detectable group differences

## How to apply

Load the real-world diabetes dataset (P=76 variables containing FFAs and BAs) in R and compute the covariance matrix Cov over all 76 variables. Draw each variable's sample mean independently from N(0, 0.5^2), then generate an 80-observation matrix from a multivariate normal with that mean vector and covariance Cov, retaining Cov via singular value decomposition (SVD). Repeat the draw for a second 80-observation matrix representing the other biological group. Horizontally (column-wise) stack the two 80x76 matrices into a complete 160x76 data matrix, and write it to data_sim.csv with row names. The rationale is that grounding the simulation in the empirical Cov and injecting group-differentiating means yields a complete matrix on which controlled MNAR (left-censored) missing patterns can be generated to benchmark GSimp against QRILC and kNN-TN.

## Related tools

- **R** (Language used to compute the covariance matrix Cov from the diabetes dataset and to draw the two multivariate-normal 80-observation groups) — https://github.com/WandeRum/GSimp
- **SVD** (Singular value decomposition used to retain/store the covariance matrix Cov during simulation generation)
- **GSimp** (Repository (simulation_generation.Rmd) containing the simulation dataset generation algorithm whose output, data_sim.csv, feeds the imputation evaluation pipeline) — https://github.com/WandeRum/GSimp

## Examples

```
rmarkdown::render('simulation_generation.Rmd')  # generates data_sim.csv (160x76) as described in the GSimp vignette
```

## Evaluation signals

- The output file data_sim.csv exists with exactly 160 rows and 76 columns (P=76)
- Two-sample t-tests between the two simulated groups of 80 samples yield small p-values for a subset of variables (on the order of 1e-6 to 1e-11 as reported in the vignette), confirming preserved group differences
- The covariance matrix Cov used for sampling equals the covariance computed from the whole diabetes dataset, retained via SVD
- Variable sample means match draws from the specified N(0, 0.5^2) distribution

## Limitations

- Simulation parameter choices (e.g., N(0, 0.5^2) means, covariance taken from the diabetes dataset) are not fully justified in the paper
- The multivariate-normal assumption may not hold for raw targeted metabolomics intensities; the paper's pipeline recommends log-transformation for non-normal data before related processing
- The generated data are fully observed by design; MNAR missing patterns must be injected separately (e.g., via MNAR_gen_imp with mis_var_prop=seq(.1, .8, .1))

## Evidence

- [methods] Covariance from diabetes dataset: "we first calculated the covariance matrix Cov based on the whole diabetes dataset (*P*=76)"
- [discussion] Mean draws and SVD retention: "the sample mean of each variable was drawn from a normal distribution N(0, 0.52) and Cov was kept using SVD"
- [readme] Simulation source file: "**simulation_generation.Rmd** contains a simulation dataset generation algorithm."
- [readme] Simulated output file: "**data_sim.csv** is the simulated dataset (details could be found in **simulation_generation.Rmd**)."
