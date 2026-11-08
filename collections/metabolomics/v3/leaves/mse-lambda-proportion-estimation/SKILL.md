---
name: mse-lambda-proportion-estimation
description: Use when you have rank statistics (or ranked signals) for metabolites
  across replicate sample pairs and need to estimate the reproducible proportion pi_1
  plus a data-driven rejection threshold controlling mFDR at nominal alpha.
when_to_use_negative:
- The true proportion of reproducible metabolites is expected to be very small — the
  paper reports MaRR tends to have high power except for smaller values of the proportion
  of reproducible metabolites
- The data are raw, unprocessed MS-metabolomics intensities — the MaRR procedure is
  applied after data processing (e.g., imputation/normalization), so preprocessing
  must happen first
- The goal is a comprehensive evaluation of pre-processing pipelines — the authors
  state their goal was not a comprehensive evaluation of pre-processing steps, but
  to show how MaRR can benchmark comparisons
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: marr
  role: Reference R/Bioconductor implementation of the Maximum Rank Reproducibility
    procedure; its Marr() function computes reproducibility of features per sample
    pair and sample pairs per feature with alpha controlling FDR, while source_MaRR.R
    in marr_paper_evaluations provides the paper's core MSE(lambda)/mFDR estimator
    scripts
  repo: http://bioconductor.org/packages/marr
- name: R
  role: Implementation language for the MaRR estimator scripts (SIM_B1.R etc.), simulation
    objects, and SIMPLOT_B-style validation plots
- name: xtable
  role: R package used by the paper's Table_R_Scripts to generate latex code for result
    tables derived from saved R objects
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:f2c3633b36e0a2fa58b945d2ca11d60efc6b96d35043318544fd321247417602
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - marr
  - xtable
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mse-lambda-proportion-estimation

## Summary

Implements the MaRR procedure's core nonparametric estimator: from maximal rank statistics of replicate pairs it estimates the proportion of reproducible metabolites (pi_1) by minimizing MSE(lambda) at tuning parameter lambda = 0.9, and derives an mFDR-based rejection threshold at a nominal FDR level alpha.

## When to use

Use when you have rank statistics (or ranked signals) for metabolites across replicate sample pairs and need to estimate the reproducible proportion pi_1 plus a data-driven rejection threshold controlling mFDR at nominal alpha.

## When NOT to use

- The true proportion of reproducible metabolites is expected to be very small — the paper reports MaRR tends to have high power except for smaller values of the proportion of reproducible metabolites
- The data are raw, unprocessed MS-metabolomics intensities — the MaRR procedure is applied after data processing (e.g., imputation/normalization), so preprocessing must happen first
- The goal is a comprehensive evaluation of pre-processing pipelines — the authors state their goal was not a comprehensive evaluation of pre-processing steps, but to show how MaRR can benchmark comparisons

## Inputs

- Matrix/data frame of metabolites (rows) by replicate sample pairs (columns) from MS-metabolomics data, post data-processing, from which maximal rank statistics are computed
- Nominal FDR level alpha (default 0.05 in the marr package)
- MSE tuning parameter lambda (paper uses lambda = 0.9)
- For validation: synthetic rank-pair data simulated under the ideal setting with a known reproducible proportion (e.g., pi_1 = 0.35, design SIM_B1)

## Outputs

- Estimated reproducible proportion pi_1-hat of metabolites
- mFDR curve mFDR-hat(l) over rejection regions (0, l)
- Rejection threshold N-hat at nominal FDR level alpha
- Saved R objects following the repo convention (e.g., B1_sim_results.Rdata) and SIMPLOT_B-style validation histograms/plots

## How to apply

Source the MaRR functions ('source_MaRR.R') from the 'src' folder of Ghoshlab/marr_paper_evaluations (or use the Bioconductor marr package's Marr() function with alpha set to the nominal FDR level, default 0.05). For each replicate sample pair compute the maximal rank statistic, then build the empirical survival function of ranks across the M metabolites. Estimate pi_1 as the minimizer of the MSE(lambda) criterion with tuning parameter lambda = 0.9, then compute the mFDR estimate for any rejection region (0, l) as mFDR-hat(l) = E[V_k-hat(l)]/Q(l) = (l-k-hat)^2/(Q(l)(M-k-hat)) and derive the rejection threshold N-hat at the nominal level alpha. Validate on a synthetic rank-pair dataset with known true pi_1 (e.g., simulation design SIM_B1 with pi_1 = 0.35): the bias (estimated minus true reproducible proportion) should be close to zero and the empirical FDR at N-hat should be controlled at alpha, per the paper's simulation convention SIM_B1.R -> B1_sim_results.Rdata.

## Related tools

- **marr** (Reference R/Bioconductor implementation of the Maximum Rank Reproducibility procedure; its Marr() function computes reproducibility of features per sample pair and sample pairs per feature with alpha controlling FDR, while source_MaRR.R in marr_paper_evaluations provides the paper's core MSE(lambda)/mFDR estimator scripts) — http://bioconductor.org/packages/marr
- **R** (Implementation language for the MaRR estimator scripts (SIM_B1.R etc.), simulation objects, and SIMPLOT_B-style validation plots)
- **xtable** (R package used by the paper's Table_R_Scripts to generate latex code for result tables derived from saved R objects)

## Examples

```
Rscript SIM_B1.R  # sources src/source_MaRR.R, simulates rank-pair data with true pi_1 = 0.35, and saves B1_sim_results.Rdata
```

## Evaluation signals

- Bias (estimated minus true reproducible proportion pi_1) on simulated data is close to zero, as reported for the paper's simulations
- Empirical FDR at the computed threshold N-hat is controlled at the nominal level alpha
- MSE(lambda) criterion is minimized at the paper's tuning value lambda = 0.9
- Simulation runs reproduce the repo convention: SIM_B1.R yields B1_sim_results.Rdata, and SIMPLOT_B.R generates the corresponding validation plots
- On real COPDGene MS-metabolomics replicate datasets, estimated reproducibility is higher for technical replicates than biological replicates across the Tech, BioTech, and Bio datasets

## Limitations

- MaRR has reduced power for smaller values of the proportion of reproducible metabolites
- True signals may fall in undetectable regions because of lack of separation (overlap) between the reproducible and irreproducible components
- The estimator is nonparametric and assumes replicate-pair rank data; it does not make parametric assumptions on distributions or dependence structures, so it should not be expected to output effect-size or correlation estimates
- The paper's pre-processing comparisons (BPCA/kNN imputation; quantile/median/RUV normalization) were benchmarks only, not a comprehensive evaluation

## Evidence

- [methods] marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [methods] The scripts 'SIM_B1.R',.. , 'SIM_C1.R',..., 'SIM_D4.R' can generate the corresponding R objects 'B1_sim_results.Rdata',..,'C1_sim_results.Rdata',...,'D4_sim_results.Rdata': "The scripts 'SIM_B1.R',.. , 'SIM_C1.R',..., 'SIM_D4.R' can generate the corresponding R objects 'B1_sim_results.Rdata',..,'C1_sim_results.Rdata',...,'D4_sim_results.Rdata'"
- [abstract] Bias (i.e., the difference between the estimated and the true value of reproducible signal proportions) values for simulations are also close to zero: "Bias (i.e., the difference between the estimated and the true value of reproducible signal proportions) values for simulations are also close to zero"
- [discussion] the real signals might fall in the undetectable regions because of overlapping between the reproducible and irreproducible components: "the real signals might fall in the undetectable regions because of overlapping between the reproducible and irreproducible components"
- [readme] marr measures the reproducibility of features per sample pair and sample pairs per feature in high-dimensional biological replicate experiments.: "marr measures the reproducibility of features per sample pair and sample pairs per feature in high-dimensional biological replicate experiments."
