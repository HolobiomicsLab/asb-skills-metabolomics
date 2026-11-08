---
name: maximum-rank-statistic-implementation
description: Use when a replicate-pair metabolomics (or other high-dimensional biological)
  dataset needs a nonparametric estimate of the proportion of reproducible metabolites
  (pi_1) plus a data-driven rejection threshold controlling mFDR at level alpha.
when_to_use_negative:
- When a ready-made package call suffices — use the Marr() function from the Bioconductor
  marr package with thresholds pSamplepairs/pFeatures (default 0.75) instead of reconstructing
  the estimator
- When the data lack a replicate-pair structure (no technical or biological replicates),
  since the maximal rank statistic is defined per sample pair
- When the goal is a comprehensive evaluation of pre-processing steps (missing value
  imputation, normalization); the MaRR approach was designed only to benchmark comparisons
  between pipelines
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Implementation language for the MaRR estimator functions (source_MaRR.R),
    simulation scripts (SIM_B1.R), and plotting scripts (SIMPLOT_B.R)
- name: marr
  role: Bioconductor package implementing the Maximum Rank Reproducibility procedure;
    reference for the Marr() interface, accessor methods, and default alpha = 0.05
  repo: https://github.com/Ghoshlab/marr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:ae04ed4374568ee5631c4188eeddb5cd3d7ee4a28bfe153708685e1629594eb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ghoshlab/marr
  tools:
  - R
  - marr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# maximum-rank-statistic-implementation

## Summary

Reimplementation of the MaRR (Maximum Rank Reproducibility) core estimator as reusable R functions: the maximal rank statistic per replicate sample pair, the MSE(lambda)-based estimator of the reproducible metabolite proportion, and the mFDR thresholding step at a nominal FDR level alpha.

## When to use

Use when a replicate-pair metabolomics (or other high-dimensional biological) dataset needs a nonparametric estimate of the proportion of reproducible metabolites (pi_1) plus a data-driven rejection threshold controlling mFDR at level alpha.

## When NOT to use

- When a ready-made package call suffices — use the Marr() function from the Bioconductor marr package with thresholds pSamplepairs/pFeatures (default 0.75) instead of reconstructing the estimator
- When the data lack a replicate-pair structure (no technical or biological replicates), since the maximal rank statistic is defined per sample pair
- When the goal is a comprehensive evaluation of pre-processing steps (missing value imputation, normalization); the MaRR approach was designed only to benchmark comparisons between pipelines

## Inputs

-  metabolite-by-sample matrix of ranks (or a matrix of metabolite abundances across replicate samples that can be converted to ranks per sample pair)
- Nominal FDR level alpha (e.g., 0.05) and tuning parameter lambda (default 0.9) for the MSE(lambda)-based pi_1 estimator
- Synthetic rank-pair data with a known reproducible proportion (e.g., pi_1 = 0.35, SIM_B1 design) for validation

## Outputs

- Estimated reproducible proportion pi_1-hat of metabolites
- mFDR curve mFDR-hat(l) over candidate rejection regions (0, l)
- Rejection threshold N-hat at nominal FDR level alpha
- Saved R object such as B1_sim_results.Rdata following the SIM_B1.R -> B1_sim_results.Rdata convention
- Validation histogram/plot of reproducibility outputs (SIMPLOT_B-style)

## How to apply

Source the MaRR functions from 'src/source_MaRR.R' in the Ghoshlab/marr_paper_evaluations repository, then feed a metabolite-by-sample rank matrix from replicate pairs. For each sample pair compute the maximal rank statistic and its empirical survival function; estimate the reproducible proportion pi_1 via the MSE(lambda) criterion with tuning parameter lambda = 0.9. Compute the mFDR estimate for any rejection region (0, l) as mFDR-hat(l) = E[V_k-hat(l)]/Q(l) = (l-k-hat)^2/(Q(l)(M-k-hat)) and derive the rejection threshold N-hat at the nominal FDR level alpha (default 0.05 in the marr package). Validate on synthetic rank-pair data with known pi_1 (e.g., 0.35 as in simulation design SIM_B1): bias should be close to zero and empirical FDR at N-hat should be controlled at alpha.

## Related tools

- **R** (Implementation language for the MaRR estimator functions (source_MaRR.R), simulation scripts (SIM_B1.R), and plotting scripts (SIMPLOT_B.R))
- **marr** (Bioconductor package implementing the Maximum Rank Reproducibility procedure; reference for the Marr() interface, accessor methods, and default alpha = 0.05) — https://github.com/Ghoshlab/marr

## Examples

```
Rscript SIM_B1.R  # sources src/source_MaRR.R and saves B1_sim_results.Rdata
```

## Evaluation signals

- Bias (estimated minus true reproducible proportion) on simulated data with known pi_1 = 0.35 is close to zero
- Empirical FDR at the rejection threshold N-hat is controlled at the nominal level alpha
- The output R object (e.g., B1_sim_results.Rdata) matches the SIM_B1.R -> B1_sim_results.Rdata naming convention of the marr_paper_evaluations repository
- The empirical survival function of the maximal rank statistics is monotone non-increasing over rank values
- The mFDR-hat(l) curve is well-defined for every candidate rejection region (0, l) with Q(l) > 0

## Limitations

- True signals may fall in undetectable regions because of overlap between the reproducible and irreproducible components
- MaRR tends to have lower power for detecting reproducible metabolites when the true proportion of reproducible metabolites is small
- Results depend on the MSE(lambda) tuning parameter choice (lambda = 0.9 in the paper's simulations)

## Evidence

- [methods] The MaRR functions ('source_MaRR.R') can be sourced from the folder 'src': "The MaRR functions ('source_MaRR.R') can be sourced from the folder 'src'"
- [methods] SIM_B1 script convention: "The scripts 'SIM_B1.R',.. , 'SIM_C1.R',..., 'SIM_D4.R' can generate the corresponding R objects 'B1_sim_results.Rdata'"
- [readme] marr package description: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [abstract] Bias validation signal: "Bias (i.e., the difference between the estimated and the true value of reproducible signal proportions) values for simulations are also close to zero"
- [abstract] FDR control signal: "the MaRR procedure effectively controls the False Discovery Rate (FDR) when there was a gradual reduction in correlation between replicate pairs for less highly ranked signals"
