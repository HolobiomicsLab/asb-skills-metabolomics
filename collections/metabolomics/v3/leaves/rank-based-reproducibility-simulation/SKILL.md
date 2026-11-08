---
name: rank-based-reproducibility-simulation
description: Use when benchmarking reproducibility methods on rank lists — specifically
  when generating synthetic replicate rank datasets to contrast the MaRR decision
  boundary with a parametric competitor's boundary on the rank plane.
when_to_use_negative:
- The goal is assessing reproducibility on real rank lists (e.g. SEQC RNA-seq data)
  rather than benchmarking methods on synthetic rank data
- A parametric copula mixture model fit is the desired end product rather than the
  comparison object — this skill simulates data specifically to expose that model's
  boundary behavior
- The replicate data are not rank lists (e.g. raw expression values or p-values on
  a non-rank scale), since MaRR's survival-function fitting and Mg statistic assume
  ranked inputs
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Implementation language for simulating benchmark rank datasets and computing
    the MaRR statistic
- name: NumPy
  role: Generates the Unif(1,5) and conditional normal rank draws and the bivariate
    normal noise for Simulations A and B
- name: SciPy
  role: Supports survival-function computations for the MSEn minimization and copula-related
    distributions
- name: Matplotlib
  role: Plots the MaRR and copula-model rejection regions on the (Rgx, Rgy) rank plane
- name: idr (copula mixture model, Li et al. 2011)
  role: Fits the competing copula mixture model with ten random initializations to
    extract its rejection region
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:88e344f75e213b9731062e3897c6fef1344b434c63d66437a44145831a0562ee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matplotlib
  - NumPy
  - Python
  - SciPy
  - idr (copula mixture model, Li et al. 2011)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rank-based-reproducibility-simulation

## Summary

Generates simulated replicate rank-list benchmarks (copula-model data and the r0=0.95, pi1=0.80 setting with gradually eroding correlation) and uses them to compare the MaRR square rejection boundary against the copula mixture model's Gaussian-tail-like boundary.

## When to use

Use when benchmarking reproducibility methods on rank lists — specifically when generating synthetic replicate rank datasets to contrast the MaRR decision boundary with a parametric competitor's boundary on the rank plane.

## When NOT to use

- The goal is assessing reproducibility on real rank lists (e.g. SEQC RNA-seq data) rather than benchmarking methods on synthetic rank data
- A parametric copula mixture model fit is the desired end product rather than the comparison object — this skill simulates data specifically to expose that model's boundary behavior
- The replicate data are not rank lists (e.g. raw expression values or p-values on a non-rank scale), since MaRR's survival-function fitting and Mg statistic assume ranked inputs

## Inputs

- Simulation parameters: r0=0.95, pi1=0.80, n=10,000 (Simulation B); muA, rho, pi1 and n=2000 (Simulation A); alpha=0.05
- Two synthetic replicate rank datasets Rgx, Rgy (one per gene/feature) generated in NumPy as described in the workflow
- Maximum rank statistics Mg = max(Rgx, Rgy) per dataset

## Outputs

- k-hat estimate (argmin of MSEn(i/n) over i = 0,...,floor(0.9n)) per dataset
- MaRR rejection region N-hat defining the square decision region {Mg <= N-hat} at alpha=0.05
- Copula mixture model (Li et al. 2011) rejection region on the rank scale from the highest-likelihood of ten fits
- Overlay figure of both rejection regions on the (Rgx, Rgy) rank plane for both simulated settings, saved as decision_boundaries.png

## How to apply

Simulate two benchmark rank datasets in Python/NumPy: a Simulation A dataset from the copula model's assumptions (bivariate normal reproducible signals with mean (muA, muA) and correlation rho, standard bivariate normal irreproducible signals, n=2000) and a Simulation B dataset with r0=0.95 and pi1=0.80 at n=10,000, drawing tg,1~Unif(1,5) and tg,2|tg,1~N(tg,1, 1-rg^2) with rg=(1-r0)/4*(tg,1-1)+r0, plus standard bivariate normal noise. For each dataset compute maximum rank statistics Mg=max(Rgx,Rgy) and estimate k-hat as argmin over i=0,...,floor(0.9n) of MSEn(i/n), the weighted mean squared error between the empirical survival function and (1-i/n)*S_{i/n}(x); then derive the MaRR rejection region via mFDR(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) <= alpha with alpha=0.05, yielding the square region {Mg <= N-hat}. Fit the copula mixture model of Li et al. (2011) with ten random initializations of (muA, rho, pi1) drawn from uniform domains, keeping the highest-likelihood fit, and extract its rank-scale rejection region. Plot both regions on the (Rgx, Rgy) rank plane with Matplotlib; the rationale is that Simulation B's near-perfectly correlated top signals and gradually eroding lower-rank correlation expose the copula model's overly narrow boundary.

## Related tools

- **Python** (Implementation language for simulating benchmark rank datasets and computing the MaRR statistic)
- **NumPy** (Generates the Unif(1,5) and conditional normal rank draws and the bivariate normal noise for Simulations A and B)
- **SciPy** (Supports survival-function computations for the MSEn minimization and copula-related distributions)
- **Matplotlib** (Plots the MaRR and copula-model rejection regions on the (Rgx, Rgy) rank plane)
- **idr (copula mixture model, Li et al. 2011)** (Fits the competing copula mixture model with ten random initializations to extract its rejection region)

## Examples

```
import numpy as np; t1 = np.random.uniform(1, 5, 10000); rg = (1-0.95)/4*(t1-1)+0.95; t2 = np.random.normal(t1, np.sqrt(1-rg**2))
```

## Evaluation signals

- Simulation B uses exactly r0=0.95 and pi1=0.80 at n=10,000, with rg=(1-r0)/4*(tg,1-1)+r0 decreasing linearly in tg,1
- k-hat equals the argmin of MSEn(i/n) over i=0,...,floor(0.9n), not an arbitrary threshold
- The MaRR rejection region is a consistent square {Mg <= N-hat} in both settings, controlled at alpha=0.05 via mFDR(i) = (i-k-hat)^2 / (Q(i)(n-k-hat))
- The copula mixture model boundary resembles a bivariate Gaussian tail and is overly narrow in Simulation B, omitting reproducible signals off the diagonal y=x
- The copula fit is the maximum-likelihood fit among ten random initializations of (muA, rho, pi1) from uniform domains

## Limitations

- In the near-perfect-correlation setting (r0=0.95, pi1=0.80), the copula mixture model's boundary is overly narrow, omitting reproducible signals somewhat off the diagonal y=x
- The copula mixture model requires ten random initializations with best-of-likelihood selection, reflecting sensitivity of the fit to starting values of (muA, rho, pi1)
- Simulation A data are drawn from the copula model's own parametric assumptions, so boundary comparisons there favor the parametric competitor by construction

## Evidence

- [other] max rank simulation benchmarks: [evidence span withheld — anchor status: not_found]
