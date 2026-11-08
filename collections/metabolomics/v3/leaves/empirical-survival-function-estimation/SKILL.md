---
name: empirical-survival-function-estimation
description: Use when replicate metabolomics data have been reduced to per-metabolite
  rank pairs across sample pairs and you need a nonparametric estimate of the reproducible
  proportion pi_1 plus an mFDR-controlled rejection threshold.
when_to_use_negative:
- Input is raw, unprocessed MS-metabolomics abundance data — the MaRR procedure is
  applied only after data processing (imputation such as BPCA/kNN and normalization
  such as quantile, median, or RUV).
- The goal is a comprehensive evaluation of pre-processing pipelines rather than reproducibility
  estimation or using reproducibility as a benchmark for pipeline comparison.
- A parametric model of the distributions or dependence structures of reproducible
  metabolites is explicitly required — MaRR's advantage is precisely that it avoids
  such parametric assumptions.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Implementation environment for the MaRR functions (maximal rank statistic,
    empirical survival function, MSE(lambda) estimator, mFDR thresholding), e.g. sourced
    from src/source_MaRR.R
  repo: http://bioconductor.org/packages/marr
- name: marr
  role: Bioconductor package implementing the Maximum Rank Reproducibility approach;
    its Marr() function wraps the maximal rank statistic estimation with alpha for
    FDR control
  repo: https://github.com/Ghoshlab/marr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:d4b37de972d2aafc4a6760bc6ddc68336fe507d581b140cd8e4e588c58610ba0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - marr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# empirical-survival-function-estimation

## Summary

Estimates the empirical survival function of maximum rank statistics from replicate pairs, as implemented in the MaRR (Maximum Rank Reproducibility) procedure, to nonparametrically estimate the proportion of reproducible metabolites (pi_1) and construct mFDR-based rejection thresholds.

## When to use

Use when replicate metabolomics data have been reduced to per-metabolite rank pairs across sample pairs and you need a nonparametric estimate of the reproducible proportion pi_1 plus an mFDR-controlled rejection threshold.

## When NOT to use

- Input is raw, unprocessed MS-metabolomics abundance data — the MaRR procedure is applied only after data processing (imputation such as BPCA/kNN and normalization such as quantile, median, or RUV).
- The goal is a comprehensive evaluation of pre-processing pipelines rather than reproducibility estimation or using reproducibility as a benchmark for pipeline comparison.
- A parametric model of the distributions or dependence structures of reproducible metabolites is explicitly required — MaRR's advantage is precisely that it avoids such parametric assumptions.

## Inputs

- Rank-pair matrix of metabolites across replicate sample pairs (per-metabolite maximum rank statistics from MS-metabolomics replicate experiments)
- Tuning parameter lambda (default 0.9 in the MaRR simulations)
- Nominal FDR level alpha (default 0.05)

## Outputs

- Estimated reproducible proportion pi_1-hat
- Empirical survival function of ranks
- mFDR-hat(l) curve over candidate rejection regions (0, l)
- Rejection threshold N-hat for reproducible metabolites
- Saved R results object (e.g. B1_sim_results.Rdata following the SIM_B1.R convention)

## How to apply

First compute the maximal rank statistic for each sample pair (or feature) from the rank data, then form the empirical survival function of the ranks. Fit the reproducible proportion pi_1 via the MSE(lambda)-based estimator with tuning parameter lambda = 0.9, mirroring the source_MaRR.R module from the src folder of the marr_paper_evaluations repository. For any rejection region (0, l), compute the mFDR estimate mFDR-hat(l) = E[V_k-hat(l)]/Q(l) = (l - k-hat)^2 / (Q(l)(M - k-hat)), and derive the rejection threshold N-hat at the nominal FDR level alpha (e.g. alpha = 0.05 as in the marr package default). Validate by simulating a synthetic rank-pair dataset under the ideal setting with known pi_1 (e.g. pi_1 = 0.35 in SIM_B1) and checking that bias (estimated minus true reproducible proportion) is close to zero and the empirical FDR at N-hat is controlled at the nominal level.

## Related tools

- **R** (Implementation environment for the MaRR functions (maximal rank statistic, empirical survival function, MSE(lambda) estimator, mFDR thresholding), e.g. sourced from src/source_MaRR.R) — http://bioconductor.org/packages/marr
- **marr** (Bioconductor package implementing the Maximum Rank Reproducibility approach; its Marr() function wraps the maximal rank statistic estimation with alpha for FDR control) — https://github.com/Ghoshlab/marr

## Examples

```
Rscript -e 'source("src/source_MaRR.R"); out <- marrEstimate(rank_pairs, lambda = 0.9, alpha = 0.05)'
```

## Evaluation signals

- Bias (estimated minus true reproducible proportion pi_1) is close to zero on synthetic data with known pi_1 (e.g. pi_1 = 0.35 in simulation design SIM_B1).
- Empirical FDR at the rejection threshold N-hat is controlled at the nominal level alpha.
- The mFDR-hat(l) curve satisfies mFDR-hat(l) = (l - k-hat)^2 / (Q(l)(M - k-hat)) for every candidate region (0, l).
- The MSE(lambda) estimator uses the tuning parameter lambda = 0.9 as in the paper's simulation designs.
- Results object is saved with the expected naming convention (e.g. B1_sim_results.Rdata generated from SIM_B1.R).

## Limitations

- MaRR tends to have lower power (and higher bias) for smaller values of the proportion of reproducible metabolites.
- True reproducible signals may fall in undetectable regions due to lack of separation/overlap between the reproducible and irreproducible components.
- FDR control holds when there is gradual reduction in correlation between replicate pairs for less highly ranked signals; other dependence patterns are not guaranteed.

## Evidence

- [other] mFDR-hat(l) = E[V_k-hat(l)]/Q(l) = (l-k-hat)^2/(Q(l)(M-k-hat)): [evidence span withheld — anchor status: not_found]
- [other] the maximal rank statistic per sample pair, the empirical survival function of ranks, and the MSE(lambda)-based estimator: [evidence span withheld — anchor status: not_found]
- [readme] The MaRR functions ('source_MaRR.R') can be sourced from the folder 'src': "e) The MaRR functions ('source_MaRR.R') can be sourced from the folder 'src'."
- [methods] marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [abstract] Bias (i.e., the difference between the estimated and the true value of reproducible signal proportions) values for simulations are also close to zero: "Bias (i.e., the difference between the estimated and the true value of reproducible signal proportions) values for simulations are also close to zero"
