---
name: covariance-matrix-svd-preservation
description: Use when a missing-value imputation benchmark needs a complete simulated
  metabolomics data matrix with realistic covariance and group structure, so MNAR
  patterns can be injected and imputation accuracy be measured against known truth.
when_to_use_negative:
- When the goal is to impute already missing values in a real dataset — this skill
  only generates a complete simulation with no missing values.
- When the target data structure is not covariance-based multivariate normal (e.g.
  count data or compositional data), since the generator assumes a multivariate normal
  draw with covariance Cov.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language in which the covariance matrix is computed and the multivariate-normal
    samples are drawn; simulation_generation.Rmd is run with R/knitr.
  repo: https://github.com/WandeRum/GSimp
- name: SVD
  role: Used to keep the covariance matrix Cov during generation of the simulated
    data blocks.
- name: GSimp
  role: Repository context providing simulation_generation.Rmd and the evaluation
    pipeline that consumes the simulated dataset.
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:da5cd2d66d29168fdf0d75f8a7005c0699b9b8d1ae83d7eaef682ce9cd4edc69
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
yes_rate: 1.0
n_claims: 5
---

# covariance-matrix-svd-preservation

## Summary

Generates a simulated metabolomics dataset by computing a covariance matrix from a real targeted metabolomics dataset (P=76 FFAs and BAs), then repeatedly drawing multivariate-normal sample blocks with per-variable means N(0, 0.5^2) while preserving the covariance via SVD, producing a complete data_sim.csv matrix without missing values.

## When to use

Use when a missing-value imputation benchmark needs a complete simulated metabolomics data matrix with realistic covariance and group structure, so MNAR patterns can be injected and imputation accuracy be measured against known truth.

## When NOT to use

- When the goal is to impute already missing values in a real dataset — this skill only generates a complete simulation with no missing values.
- When the target data structure is not covariance-based multivariate normal (e.g. count data or compositional data), since the generator assumes a multivariate normal draw with covariance Cov.

## Inputs

- Real-world diabetes targeted metabolomics dataset with P=76 variables (FFAs and BAs), e.g. real_data.csv
- Per-variable sample means drawn from N(0, 0.5^2)
- Choice of sample size per group (80 observations) and number of variables (P=76)

## Outputs

- data_sim.csv: complete simulated 160x76 metabolomics data matrix with row names, containing two simulated biological groups of 80 samples each
- Preserved covariance structure (via SVD) inherited from the real diabetes dataset

## How to apply

In R, load the whole real-world diabetes targeted metabolomics dataset (P=76 variables covering FFAs and BAs) and compute its covariance matrix Cov. Draw the sample mean of each variable from N(0, 0.5^2), then generate an 80-observation multivariate-normal data block whose covariance is Cov, kept via singular value decomposition (SVD). Repeat to obtain a second 80-observation block representing the other biological group, and column-wise (horizontally) stack the two 80x76 matrices into a complete 160x76 matrix so group differences and covariance structure are preserved. Write the result with row names to data_sim.csv; the algorithm lives in simulation_generation.Rmd in the GSimp repository. Rationale: starting from the empirical covariance of real targeted metabolomics data gives the simulated matrix realistic correlation structure, and the complete matrix serves as ground truth when missing values are later generated for the MVI evaluation pipeline.

## Related tools

- **R** (Language in which the covariance matrix is computed and the multivariate-normal samples are drawn; simulation_generation.Rmd is run with R/knitr.) — https://github.com/WandeRum/GSimp
- **SVD** (Used to keep the covariance matrix Cov during generation of the simulated data blocks.)
- **GSimp** (Repository context providing simulation_generation.Rmd and the evaluation pipeline that consumes the simulated dataset.) — https://github.com/WandeRum/GSimp

## Examples

```
Rscript -e "rmarkdown::render('simulation_generation.Rmd')"
```

## Evaluation signals

- data_sim.csv exists with exactly 160 rows (80 per simulated group) and 76 columns matching P=76 of the diabetes dataset.
- t-tests between the two simulated 80-sample groups yield small p-values on the order of 1e-6 to 1e-11 for a subset of variables, confirming preserved group differences as reported in the vignette.
- Column covariances of the simulated matrix reproduce the covariance structure computed from the whole diabetes dataset (preserved via SVD).
- The output file contains row names and no missing values, since it is intended as complete ground truth for later MNAR generation.
- The generation script corresponds to simulation_generation.Rmd / simulation_generation.pdf in the GSimp repository, so re-knitting should reproduce data_sim.csv.

## Limitations

- The parameter choices — sample means from N(0, 0.5^2) and covariance taken from the diabetes dataset — are not fully justified in the paper's discussion.
- The simulation preserves the covariance of a single real dataset; correlation structures of other metabolomics platforms (e.g. GC/MS profiling) are not necessarily represented.
- The generated matrix is complete (no missing values), so it cannot directly reflect real-world MNAR patterns until missing values are injected by the evaluation pipeline.

## Evidence

- [methods] Covariance matrix from diabetes dataset: "we first calculated the covariance matrix Cov based on the whole diabetes dataset (*P*=76)"
- [discussion] Means drawn from N(0, 0.5^2): "the sample mean of each variable was drawn from a normal distribution N(0, 0.52) and Cov was kept using SVD"
- [methods] Output file data_sim.csv: "**data_sim.csv** is the simulated dataset (details could be found in **simulation_generation.Rmd**)."
- [readme] README describes data_sim.csv: "**data_sim.csv** is the simulated dataset (details could be found in **simulation_generation.Rmd**)."
- [readme] Simulation generation file: "**simulation_generation.Rmd** contains a simulation dataset generation algorithm."
