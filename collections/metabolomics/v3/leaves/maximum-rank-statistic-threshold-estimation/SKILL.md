---
name: maximum-rank-statistic-threshold-estimation
description: Use when you have two replicate rank lists (e.g., RNA-seq gene rankings
  from technical or biological replicates such as SEQC platform data) and need to
  detect the reproducible-to-irreproducible change point with FDR control, without
  parametric copula assumptions.
when_to_use_negative:
- When parametric copula mixture modeling (Li et al. 2011) is specifically required
  and parametric dependence assumptions on reproducible genes are acceptable
- When the data are not replicate rank lists (e.g., unranked feature intensities that
  have not been converted to gene rank lists)
- When the goal requires rejection regions off the diagonal y=x under near-perfect
  signal correlation and a model-based boundary shape is expected rather than the
  square MaRR boundary
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Python
  role: Implementation language for computing maximum rank statistics, MSE minimization,
    and mFDR thresholds
- name: NumPy
  role: Array computation of Mg = max(Rgx, Rgy) and survival-function comparison
- name: SciPy
  role: Empirical and theoretical survival function evaluation
- name: Matplotlib
  role: Plotting rejection regions (decision boundaries) on the rank plane as decision_boundaries.png
- name: idr
  role: Copula mixture model (Li et al. 2011) fit with ten random initializations
    for boundary comparison against MaRR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:26d02f539fc22542b75c0ccb78bcce26a874f5ba08309aaed196a1943132e14c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matplotlib
  - NumPy
  - Python
  - SciPy
  - idr
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# maximum-rank-statistic-threshold-estimation

## Summary

Nonparametric MaRR (Maximum Rank Reproducibility) procedure that estimates where reproducible signals end in replicate rank lists by minimizing the mean squared error between observed and theoretical survival functions of maximum rank statistics, then derives gene-level marginal FDR calls with no tuning parameters.

## When to use

Use when you have two replicate rank lists (e.g., RNA-seq gene rankings from technical or biological replicates such as SEQC platform data) and need to detect the reproducible-to-irreproducible change point with FDR control, without parametric copula assumptions.

## When NOT to use

- When parametric copula mixture modeling (Li et al. 2011) is specifically required and parametric dependence assumptions on reproducible genes are acceptable
- When the data are not replicate rank lists (e.g., unranked feature intensities that have not been converted to gene rank lists)
- When the goal requires rejection regions off the diagonal y=x under near-perfect signal correlation and a model-based boundary shape is expected rather than the square MaRR boundary

## Inputs

- Two replicate rank lists (rank vectors Rgx, Rgy) over n genes from replicate high-throughput experiments (e.g., SEQC RNA-seq platform data)
- Target FDR level alpha (e.g., 0.05)
- Optional simulation settings for validation: Simulation A (bivariate normal reproducible signals, mean (muA, muA), correlation rho, n=2000) or Simulation B (r0=0.95, pi1=0.80, n=10,000)

## Outputs

- Estimated change point k-hat (number of reproducible signals)
- Rejection threshold N-hat defining the reproducible-gene call set {Mg <= N-hat}
- Per-gene marginal false discovery rates mFDR(i)
- Decision-boundary plot of the rejection region on the (Rgx, Rgy) rank plane (e.g., decision_boundaries.png via Matplotlib)

## How to apply

Compute the maximum rank statistic Mg = max(Rgx, Rgy) for each gene across the two replicate rank lists. Estimate the change point k-hat as the argmin over i = 0,...,floor(0.9n) of MSEn(i/n), the weighted mean squared error between the empirical survival function and the theoretical survival function S_{i/n}(x) scaled by (1 - i/n); the 0.9n cap limits the search to the irreproducible-dominated tail. Derive the rejection threshold N-hat = max{k-hat < i <= n : mFDR(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) <= alpha} with alpha = 0.05, giving the square decision region {Mg <= N-hat} in the (Rgx, Rgy) rank plane. The rationale for the nonparametric MSE-based estimation is that it avoids the parametric dependence assumptions of the copula mixture model (Li et al. 2011), which can produce overly narrow, Gaussian-tail-shaped rejection regions that miss reproducible signals off the diagonal (e.g., in near-perfect-correlation settings with r0 = 0.95, pi1 = 0.80).

## Related tools

- **Python** (Implementation language for computing maximum rank statistics, MSE minimization, and mFDR thresholds)
- **NumPy** (Array computation of Mg = max(Rgx, Rgy) and survival-function comparison)
- **SciPy** (Empirical and theoretical survival function evaluation)
- **Matplotlib** (Plotting rejection regions (decision boundaries) on the rank plane as decision_boundaries.png)
- **idr** (Copula mixture model (Li et al. 2011) fit with ten random initializations for boundary comparison against MaRR)

## Examples

```
Mg = np.maximum(Rgx, Rgy); k_hat = np.argmin([MSEn(i/n, Mg) for i in range(int(0.9*n))]); N_hat = max(i for i in range(k_hat+1, n+1) if mFDR(i, k_hat) <= 0.05)
```

## Evaluation signals

- k-hat minimizes MSEn(i/n) over i = 0,...,floor(0.9n), not over the full range up to n
- All reported discoveries satisfy mFDR(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) <= alpha (alpha = 0.05)
- The resulting rejection region is square: {Mg <= N-hat} in the (Rgx, Rgy) plane, unlike the Gaussian-tail-shaped copula boundary
- FDR control and power verified on simulations and SEQC data, where MaRR 'compares well to existing methods'
- Boundary plot saved as decision_boundaries.png shows MaRR's consistent square boundary in both Simulation A and Simulation B (r0=0.95, pi1=0.80)

## Limitations

- The rejection region is always a square {Mg <= N-hat}, so reproducible signals somewhat off the diagonal y=x may be handled differently than by the copula mixture model's Gaussian-tail-shaped (narrower) boundary
- The change-point search is capped at floor(0.9n), so the reproducible fraction cannot exceed ~90% of genes
- The procedure is nonparametric and does not model the dependence structure of reproducible genes, unlike the copula model of Li et al. (2011)

## Evidence
