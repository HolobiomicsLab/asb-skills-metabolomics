---
name: decision-boundary-visualization-on-rank-scale
description: Use when comparing discovery regions of reproducibility procedures (e.g.,
  MaRR vs copula mixture model) on paired rank lists, especially to diagnose boundary
  shape when top signals are near-perfectly correlated and correlation erodes for
  lower ranks.
when_to_use_negative:
- Input data are raw expression intensities or p-values rather than paired rank lists
  — the MaRR statistic Mg=max(Rgx,Rgy) and the copula rejection region are both defined
  on the rank scale.
- The goal is to fit a parametric dependence model to obtain discoveries rather than
  to diagnose/compare boundary shapes; under near-perfect correlation (Simulation
  B, r0=0.95) the copula-model boundary is overly narrow and omits reproducible signals
  off the y=x diagonal.
- There are more than two replicate experiments — the visualization compares bivariate
  rejection regions in the (Rgx, Rgy) plane.
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NumPy
  role: Simulation of the two benchmark rank datasets and computation of Mg=max(Rgx,Rgy)
- name: SciPy
  role: Survival-function and bivariate normal calculations for the simulations and
    boundary derivation
- name: Matplotlib
  role: Plotting both rejection regions on the (Rgx, Rgy) rank plane and saving decision_boundaries.png
- name: idr (copula mixture model, Li et al. 2011)
  role: Fitting the copula mixture model with ten random initializations of (muA,
    rho, pi1) to extract its rank-scale rejection region
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:7a79c98dfd882673de8f9e94ae9d403741fcd0d9c0c69c82190712deb916ca77
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matplotlib
  - NumPy
  - Python
  - SciPy
  - idr (copula mixture model, Li et al. 2011)
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# decision-boundary-visualization-on-rank-scale

## Summary

Simulates two benchmark rank datasets, derives the MaRR square rejection region from the maximum rank statistic Mg=max(Rgx,Rgy), fits the Li et al. (2011) copula mixture model, and overlays both rejection regions on the (Rgx,Rgy) rank plane to expose how model-based boundaries become overly narrow under near-perfect signal correlation.

## When to use

Use when comparing discovery regions of reproducibility procedures (e.g., MaRR vs copula mixture model) on paired rank lists, especially to diagnose boundary shape when top signals are near-perfectly correlated and correlation erodes for lower ranks.

## When NOT to use

- Input data are raw expression intensities or p-values rather than paired rank lists — the MaRR statistic Mg=max(Rgx,Rgy) and the copula rejection region are both defined on the rank scale.
- The goal is to fit a parametric dependence model to obtain discoveries rather than to diagnose/compare boundary shapes; under near-perfect correlation (Simulation B, r0=0.95) the copula-model boundary is overly narrow and omits reproducible signals off the y=x diagonal.
- There are more than two replicate experiments — the visualization compares bivariate rejection regions in the (Rgx, Rgy) plane.

## Inputs

- Two paired rank lists (Rgx, Rgy) on the rank scale, from simulated replicates (Simulation A: n=2000; Simulation B: n=10000, r0=0.95, pi1=0.80)
- False discovery rate threshold alpha=0.05

## Outputs

- MaRR square rejection region {Mg <= N-hat} with N-hat chosen so mFDR(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) <= 0.05
- Copula mixture model rejection region extracted on the rank scale (highest-likelihood of ten random (muA, rho, pi1) starts)
- Overlaid boundary plot of both rejection regions on the (Rgx, Rgy) rank plane, saved as decision_boundaries.png

## How to apply

Simulate two settings: Simulation A from the copula model's assumptions (bivariate normal reproducible signals with mean (muA, muA) and correlation rho, standard bivariate normal irreproducible signals, n=2000) and Simulation B with r0=0.95 and pi1=0.80 at n=10,000, where tg,1~Unif(1,5), tg,2|tg,1~N(tg,1, 1-rg^2), rg=(1-r0)/4*(tg,1-1)+r0, plus standard bivariate normal noise. For each dataset compute Mg=max(Rgx,Rgy) and estimate k-hat as argmin over i=0,...,floor(0.9n) of MSEn(i/n), the weighted MSE between the empirical survival function and (1-i/n)*S_{i/n}(x). Derive the MaRR rejection region N-hat = max{k-hat < i <= n : mFDR(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) <= alpha} with alpha=0.05, yielding the square region {Mg <= N-hat}. Fit the Li et al. (2011) copula mixture model with ten random initializations of (muA, rho, pi1) from uniform domains, retain the highest-likelihood fit, and extract its rejection region on the rank scale. Plot both regions in Matplotlib to show that MaRR retains a consistent square boundary while the copula boundary mimics a bivariate Gaussian tail.

## Related tools

- **NumPy** (Simulation of the two benchmark rank datasets and computation of Mg=max(Rgx,Rgy))
- **SciPy** (Survival-function and bivariate normal calculations for the simulations and boundary derivation)
- **Matplotlib** (Plotting both rejection regions on the (Rgx, Rgy) rank plane and saving decision_boundaries.png)
- **idr (copula mixture model, Li et al. 2011)** (Fitting the copula mixture model with ten random initializations of (muA, rho, pi1) to extract its rank-scale rejection region)

## Examples

```
import matplotlib.pyplot as plt; M = np.maximum(Rgx, Rgy); plt.plot(Rgx, Rgy, '.'); plt.contourf(M <= N_hat); plt.savefig('decision_boundaries.png')
```

## Evaluation signals

- MaRR's rejection region must be exactly square on the rank plane: {Mg <= N-hat}, identical in shape across Simulation A and Simulation B.
- In Simulation B (r0=0.95, pi1=0.80) the copula mixture model boundary should visibly resemble a bivariate Gaussian tail and be narrower than MaRR's, omitting reproducible signals somewhat off the diagonal y=x.
- k-hat equals the argmin over i=0,...,floor(0.9n) of MSEn(i/n), the weighted MSE between the empirical survival function and (1-i/n)*S_{i/n}(x).
- Every point in the reported MaRR region satisfies mFDR(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) <= alpha=0.05.
- The copula fit retained is the maximum-likelihood among ten random initializations of (muA, rho, pi1) drawn from uniform domains.
- Figure saved as decision_boundaries.png contains both regions overlaid on the (Rgx, Rgy) rank plane for both settings.

## Limitations

- The copula mixture model's rejection boundary becomes overly narrow under near-perfect signal correlation, omitting reproducible signals somewhat off the diagonal y=x (Simulation B, r0=0.95, pi1=0.80).
- MaRR's k-hat search is truncated at floor(0.9n); the boundary depends on this tuning-free but fixed search range.
- The visualization compares two-procedure boundaries in the bivariate rank plane; it does not by itself assess FDR control or power.

## Evidence

- [other] In the decision-boundary comparison, MaRR always has a consistent, square decision boundary, while the copula mixture model's boundary has a shape similar to a bivariate Gaussian tail.: [evidence span withheld — anchor status: not_found]
- [other] expert review confirms MaRR exhibits a consistent square decision boundary in both settings, while the copula mixture model boundary resembles a bivariate Gaussian tail and is overly narrow, omitting reproducible signals somewhat off the diagonal y=x: [evidence span withheld — anchor status: not_found]
- [intro] Based on a maximum rank statistic, our procedure identifies where the change from reproducible to irreproducible signals begins by minimizing the mean squared error: "Based on a maximum rank statistic, our procedure identifies where the change from reproducible to irreproducible signals begins by minimizing the mean squared error between observed and theoretical"
- [abstract] the MaRR procedure effectively controls false discovery rates, has desirable power properties, and compares well to existing methods: "the MaRR procedure effectively controls false discovery rates, has desirable power properties, and compares well to existing methods"
