---
name: reproducibility-assessment-high-throughput-rank-lists
description: Use when you have gene rank lists (e.g., from RNA-seq SEQC-style replicate
  experiments) from three replicates and need to quantify reproducibility, estimate
  pi1, and call reproducible genes with FDR control.
when_to_use_negative:
- Input consists of raw expression measurements rather than gene rank lists — MaRR
  operates on ranks and does not make parametric assumptions on distributions, so
  ranks must be formed first
- Only one replicate is available — reproducibility assessment requires at least two
  rank lists (original MaRR) and this extension requires three
- A parametric, model-based dependence analysis of replicate signal intensities is
  explicitly preferred over the nonparametric rank-based procedure
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Used to implement the Mg(3) null-CDF derivation, the MSE_n(lambda) minimization,
    the mFDR estimator, the three-replicate simulation design, and to write marr3_simulation_results.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:08172d578dda60166ac4f6ef307996f0fd4900b7fa4bcf5b3396847bb1ecd376
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# reproducibility-assessment-high-throughput-rank-lists

## Summary

A nonparametric MaRR-based procedure for assessing reproducibility of gene rank lists across replicate high-throughput experiments, extended to three replicates via the maximum order statistic Mg(3)=max{Rgx,Rgy,Rgz}. It estimates the reproducible signal proportion pi1 and controls marginal FDR without tuning parameters.

## When to use

Use when you have gene rank lists (e.g., from RNA-seq SEQC-style replicate experiments) from three replicates and need to quantify reproducibility, estimate pi1, and call reproducible genes with FDR control.

## When NOT to use

- Input consists of raw expression measurements rather than gene rank lists — MaRR operates on ranks and does not make parametric assumptions on distributions, so ranks must be formed first
- Only one replicate is available — reproducibility assessment requires at least two rank lists (original MaRR) and this extension requires three
- A parametric, model-based dependence analysis of replicate signal intensities is explicitly preferred over the nonparametric rank-based procedure

## Inputs

- Gene rank lists Rgx, Rgy, Rgz across three replicate experiments (e.g., rank lists derived from SEQC RNA-seq replicate data)
- Number of genes n (e.g., n = 10,000)
- Lambda grid {0, 1/n, 2/n, ..., 0.9} for MSE_n(lambda) minimization (Definition 2)
- Nominal FDR level alpha (e.g., 0.05 or 0.01)

## Outputs

- Estimated reproducible proportion pi-hat_1 and change-point k-hat
- Per-gene marginal FDR estimates E[V](i)=(n-k-hat)*P(Mh(3)<=i) and rejection threshold N-hat = max{i: mFDR-hat(i) <= alpha}
- Set of genes declared reproducible and the number declared reproducible per dataset
- Aggregated per-setting simulation results table (marr3_simulation_results.csv)

## How to apply

Rank genes in each replicate, then compute the maximum rank statistic Mg(3)=max{Rgx,Rgy,Rgz}; for irreproducible genes with independent ranks its marginal null CDF is P(Mh<=i)=i^3/n^3, generalized analogously to Proposition 1 and Corollary 2 with j_pi1=floor(n*pi1). Minimize MSE_n(lambda) over lambda in {0, 1/n, 2/n, ..., 0.9} by comparing the empirical survival function of the maximum-rank statistics against the weighted theoretical survival (1-lambda)*S_lambda to obtain k-hat and pi-hat_1, thereby identifying where the change from reproducible to irreproducible signals begins. Estimate mFDR per equations (8)-(10) adapted to the three-replicate null CDF, with E[V](i)=(n-k-hat)*P(Mh(3)<=i), and set the rejection threshold N-hat = max{i: mFDR-hat(i) <= alpha}. Validate by simulation with irreproducible genes (independent ranks) and reproducible genes with nonnegative rank correlation, using pi1 in {0.10, 0.30, 0.65, 0.80}, r0 in {0.30, 0.70, 0.95}, n=10,000, 100 datasets per setting, and nominal alpha = 0.05 and 0.01.

## Related tools

- **Python** (Used to implement the Mg(3) null-CDF derivation, the MSE_n(lambda) minimization, the mFDR estimator, the three-replicate simulation design, and to write marr3_simulation_results.csv)

## Examples

```
python run_marr3.py --replicates 3 --statistic max --alpha 0.05 --n 10000 --output marr3_simulation_results.csv
```

## Evaluation signals

- Across simulation settings (pi1 in {0.10, 0.30, 0.65, 0.80}; r0 in {0.30, 0.70, 0.95}) the empirical mFDR (false calls divided by total declared reproducible) is at or below the nominal alpha (0.05 and 0.01), with no anti-conservative settings analogous to the weak-and-infrequent mu_A=1, pi1=0.1 case in Figure 6
- pi-hat_1 recovers the true pi1 without systematic overestimation
- k-hat correctly locates the change from reproducible to irreproducible signals via minimization of the MSE between observed and theoretical survival functions
- The survival comparison converges: empirical survival of the three-replicate maximum-rank statistics matches (1-lambda)*S_lambda at the minimizing lambda
- The per-setting results table is written to marr3_simulation_results.csv with 100 datasets per setting recorded (pi-hat_1, number declared reproducible, empirical mFDR)

## Limitations

- The Mg(3) derivation is outlined only in the supplementary materials, and the second order statistic Mg(2) is an alternative summary rank statistic — choice of order statistic affects the null CDF used
- Control of the empirical mFDR relies on the simulation assumptions: irreproducible genes have independent ranks, and reproducible genes have nonnegative rank correlation; the weak-and-infrequent signal case (mu_A=1, pi1=0.1 in Figure 6) illustrates where anti-conservative behavior can arise
- The procedure assesses reproducibility of rank lists only; it does not model expression magnitudes or parametric dependence structures

## Evidence

- [other] Source card describes the three-replicate extension using the maximum order statistic and its derived null distribution.: [evidence span withheld — anchor status: not_found]
