---
name: maximum-rank-statistic-computation
description: Use when two gene rank lists from replicate high-throughput experiments
  (e.g., SEQC RNA-seq comparisons of 43,919 transcripts) must be converted to per-gene
  maximum rank statistics M_g = max(R_gx, R_gy) as input to MaRR thresholding and
  mFDR estimation.
when_to_use_negative:
- When the data plausibly satisfy the copula mixture model's parametric assumptions
  and a full model-based fit is preferred — in Simulation A, MaRR was anti-conservative
  (inflated FDR) for weak, infrequent signals (muA=1, pi1=0.1) and systematically
  underestimated pi1 for a large weakly correlated group (muA in {1,2}, rho=0.4, pi1=0.8)
- When the two experiments cannot be ranked on a common gene universe (e.g., transcripts
  absent from one platform's mapping), since M_g requires paired ranks for every gene
- When continuous effect-size estimates or per-gene dependence strength (rho-hat)
  from a generative model are required rather than rank-based reproducibility calls
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python (NumPy/pandas implementation of the closed-form MaRR estimator)
  role: Implements ranking, M_g = max(R_gx, R_gy) computation, survival-function MSE
    minimization for k-hat, and mFDR-hat thresholding
- name: R idr package
  role: Used as the copula mixture model comparison method fit to the same ranked
    data via EM with ten random restarts
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:b9e81a7a5ab543f8d4ad8ccf762386ee8e0c2cb654454e597039f4404511e12f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  - Python (NumPy/pandas implementation of the closed-form MaRR estimator)
  - R idr package
  - maximum rank reproducibility (MaRR) procedure
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# maximum-rank-statistic-computation

## Summary

Computes the maximum rank statistic M_g = max(R_gx, R_gy) for every gene from paired rank lists of replicate experiments, the core statistic of the nonparametric MaRR reproducibility procedure. It quantifies how extreme a gene is in both experiments simultaneously without parametric assumptions on dependence or distribution.

## When to use

Use when two gene rank lists from replicate high-throughput experiments (e.g., SEQC RNA-seq comparisons of 43,919 transcripts) must be converted to per-gene maximum rank statistics M_g = max(R_gx, R_gy) as input to MaRR thresholding and mFDR estimation.

## When NOT to use

- When the data plausibly satisfy the copula mixture model's parametric assumptions and a full model-based fit is preferred — in Simulation A, MaRR was anti-conservative (inflated FDR) for weak, infrequent signals (muA=1, pi1=0.1) and systematically underestimated pi1 for a large weakly correlated group (muA in {1,2}, rho=0.4, pi1=0.8)
- When the two experiments cannot be ranked on a common gene universe (e.g., transcripts absent from one platform's mapping), since M_g requires paired ranks for every gene
- When continuous effect-size estimates or per-gene dependence strength (rho-hat) from a generative model are required rather than rank-based reproducibility calls

## Inputs

- Two per-gene rank vectors R_gx and R_gy (rank 1 = most extreme) derived from replicate experiment read counts or test statistics over the same n genes/transcripts
- Raw count matrix for replicate experiments (e.g., SEQC RNA-seq counts over 43,919 transcripts) to be ranked within each experiment
- Nominal FDR level alpha (e.g., 0.05 or 0.01)

## Outputs

- Vector of maximum rank statistics M_g = max(R_gx, R_gy) for all n genes
- Empirical survival function S_n_hat(x/n) of the maximum rank statistic
- Threshold N-hat and list of genes declared reproducible at the chosen alpha
- mFDR-hat(i) values for i = k_hat+1, ..., n

## How to apply

Rank the 43,919 transcripts (or all n genes) by read count within each experiment, assigning rank 1 to the most extreme signal and breaking ties by random assignment; compute M_g = max(R_gx, R_gy) for every gene. The empirical pmf of M_g/n on irreproducible-only data (two independent uniform rank permutations) must match (2i-1)/n^2 per Proposition 1, which validates the implementation. The M_g vector and its empirical survival function S_n_hat(x/n) then feed the k-hat estimator, which minimizes MSEn(i/n) over i = 0,...,floor(0.9n), and the marginal false discovery rate mFDR-hat(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)), where Q(i) counts genes with M_g <= i. Genes with M_g <= N-hat, where N-hat = max{i : mFDR-hat(i) <= alpha}, are declared reproducible at alpha = 0.05 or 0.01. The procedure requires no tuning parameters because it relies only on rank order, so preserving exact ranks (including the randomized tie-breaking) is critical.

## Related tools

- **Python (NumPy/pandas implementation of the closed-form MaRR estimator)** (Implements ranking, M_g = max(R_gx, R_gy) computation, survival-function MSE minimization for k-hat, and mFDR-hat thresholding)
- **R idr package** (Used as the copula mixture model comparison method fit to the same ranked data via EM with ten random restarts)

## Examples

```
Mg = np.maximum(Rgx, Rgy); S_hat = np.array([np.mean(Mg <= i) for i in range(1, n+1)])
```

## Evaluation signals

- On irreproducible-only data (two independent uniform rank permutations), the empirical probability mass function of M_g/n matches (2i-1)/n^2 from Proposition 1
- On ideal-setting data (e.g., pi1 = 0.35, n = 1000, assumptions (I1)-(I3)), pi1-hat converges to pi1 as n grows (Theorem 2)
- All computed M_g values are integers in {1, ..., n} and the empirical survival function S_n_hat is non-increasing
- Empirical FDR of MaRR in Simulation A settings with weak infrequent signals (muA=1, pi1=0.1) should exceed nominal 0.05 (documented anti-conservatism), and pi1-hat should fall below 0.8 for (muA in {1,2}, rho=0.4, pi1=0.8)
- On SEQC data at alpha = 0.01, declared-reproducible counts and pi1-hat = 0.851, 0.847, 0.900, 0.898 for comparisons 1, 2, 4, 5 reproduce Table 5 / Figure 9

## Limitations

- k-hat under-estimates n*pi1, so n-k-hat over-estimates the true number of irreproducible genes, which shapes the conservative/anti-conservative behavior of mFDR-hat
- The procedure is anti-conservative (inflated FDR) when signals are weak and infrequent (muA=1, pi1=0.1) and underestimates pi1 for large weakly correlated reproducible groups
- Rank-based only: no parametric dependence structure or distribution is modeled, so per-gene posterior probabilities or correlation estimates (rho-hat from a fitted model) are not produced

## Evidence
