---
name: group-difference-simulation
description: Use when a benchmark dataset with controlled two-group structure and
  realistic inter-variable covariance is needed to evaluate imputation methods on
  metabolomics data (e.g., MNAR left-censored missingness tests).
when_to_use_negative:
- The input is real-world measured metabolomics data (e.g., the FFA/BA targeted datasets)
  that already contains the group structure to be analyzed rather than synthetic benchmark
  data
- The goal is to generate MNAR missingness for evaluation — that is a separate step
  (e.g., MNAR_gen_imp) applied after the complete simulated matrix exists
- The goal is missing value imputation itself rather than constructing the ground-truth
  complete dataset on which imputation methods will be compared
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GSimp
  role: Repository containing simulation_generation.Rmd / simulation_generation.pdf
    with the simulation dataset generation algorithm and the resulting data_sim.csv
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: Environment for computing the covariance matrix, drawing from the multivariate
    normal, SVD, and writing data_sim.csv
  repo: https://github.com/WandeRum/GSimp
- name: SVD
  role: Used to keep/store the covariance matrix Cov during multivariate normal simulation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:c511223c437962f294d826e98ffc8d2dfdb5bd9716171fa0f5f6d6db417326df
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# group-difference-simulation

## Summary

Generates a synthetic two-group metabolomics benchmark by simulating 80x76 data matrices from a multivariate normal with covariance estimated from the real diabetes dataset (P=76 FFAs/BAs), then stacking two groups to preserve covariance and group differences for imputation-method evaluation.

## When to use

Use when a benchmark dataset with controlled two-group structure and realistic inter-variable covariance is needed to evaluate imputation methods on metabolomics data (e.g., MNAR left-censored missingness tests).

## When NOT to use

- The input is real-world measured metabolomics data (e.g., the FFA/BA targeted datasets) that already contains the group structure to be analyzed rather than synthetic benchmark data
- The goal is to generate MNAR missingness for evaluation — that is a separate step (e.g., MNAR_gen_imp) applied after the complete simulated matrix exists
- The goal is missing value imputation itself rather than constructing the ground-truth complete dataset on which imputation methods will be compared

## Inputs

- Real-world diabetes targeted metabolomics dataset (P=76 variables containing FFAs and BAs) used to estimate covariance
- Simulation parameters: two groups of 80 observations, per-variable means drawn from N(0, 0.5^2)

## Outputs

- data_sim.csv: complete 160x76 simulated data matrix with row names, containing two 80-sample groups
- Two simulated biological groups with small p-values (on the order of 1e-6 to 1e-11) from t-tests for a subset of variables

## How to apply

Load the whole real-world diabetes dataset (P=76 variables containing FFAs and BAs) in R and compute its covariance matrix Cov. Draw per-variable sample means from N(0, 0.5^2) and generate an 80-observation data matrix from a multivariate normal with covariance Cov, keeping Cov via singular value decomposition (SVD). Repeat to generate a second 80-observation matrix for the other biological group, then column-wise (horizontally) stack the two 80x76 matrices into a complete 160x76 matrix so group differences and covariance structure are preserved. Write the result to data_sim.csv with row names; the full procedure is documented in simulation_generation.Rmd in the GSimp repository.

## Related tools

- **GSimp** (Repository containing simulation_generation.Rmd / simulation_generation.pdf with the simulation dataset generation algorithm and the resulting data_sim.csv) — https://github.com/WandeRum/GSimp
- **R** (Environment for computing the covariance matrix, drawing from the multivariate normal, SVD, and writing data_sim.csv) — https://github.com/WandeRum/GSimp
- **SVD** (Used to keep/store the covariance matrix Cov during multivariate normal simulation)

## Examples

```
Rscript -e "rmarkdown::render('simulation_generation.Rmd')"  # writes data_sim.csv (160x76)
```

## Evaluation signals

- Output file data_sim.csv exists with exactly 160 rows and 76 columns (P=76)
- t-tests between the two simulated groups of 80 samples yield small p-values for a subset of variables (on the order of 1e-6 to 1e-11 as reported in the vignette)
- Per-variable sample means follow the N(0, 0.5^2) drawing distribution
- Covariance structure is preserved (Cov kept via SVD from the whole diabetes dataset) in the stacked 160x76 matrix

## Limitations

- Simulation parameter choices (e.g., N(0, 0.5^2) means, covariance from the diabetes dataset) are not fully justified in the paper's discussion
- The simulated data inherits the covariance of the single real-world diabetes dataset (P=76), so covariance realism is bounded by that source dataset
- The 160x76 complete matrix has no missing values; missingness must be generated separately before imputation evaluation

## Evidence

- [methods] Covariance estimated from real data: "we first calculated the covariance matrix Cov based on the whole diabetes dataset (*P*=76)"
- [discussion] Simulation parameters: "the sample mean of each variable was drawn from a normal distribution N(0, 0.52) and Cov was kept using SVD"
- [readme] Simulation algorithm location: "**simulation_generation.Rmd** contains a simulation dataset generation algorithm."
- [readme] Simulated output file: "**data_sim.csv** is the simulated dataset (details could be found in **simulation_generation.Rmd**)."
