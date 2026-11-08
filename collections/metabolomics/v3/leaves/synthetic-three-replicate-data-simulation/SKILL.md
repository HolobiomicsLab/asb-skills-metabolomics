---
name: synthetic-three-replicate-data-simulation
description: 'Use when validating a k>2 replicate extension of MaRR: you need controlled
  synthetic rank lists (pi1, r0 grid, n≈10,000) to check empirical mFDR against nominal
  alpha and unbiased pi1 estimation.'
when_to_use_negative:
- The goal is reproducibility assessment on real replicate data (e.g., SEQC RNA-seq
  rank lists) rather than simulation-based validation of the estimator
- Only two replicates are available and the original two-replicate MaRR with the fn,0
  null from Proposition 1 already applies
- Irreproducible genes are known to have dependent ranks, violating the independence
  assumption (P(Mh<=i) = i^3/n^3) on which the Mg(3) null rests
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Python
  role: Implements the three-replicate rank simulation, Mg(3) null CDF and MSEn(lambda)
    minimization, and writes marr3_simulation_results.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:14c6186ada2b4d25099502b23311637ddd71079146c800fa9f509e272cc78815
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# synthetic-three-replicate-data-simulation

## Summary

Generates synthetic three-replicate rank-list datasets under the MaRR simulation design — irreproducible genes with independent ranks and reproducible genes with nonnegative rank correlation — to validate mFDR control and pi1 recovery of an extended three-replicate MaRR procedure.

## When to use

Use when validating a k>2 replicate extension of MaRR: you need controlled synthetic rank lists (pi1, r0 grid, n≈10,000) to check empirical mFDR against nominal alpha and unbiased pi1 estimation.

## When NOT to use

- The goal is reproducibility assessment on real replicate data (e.g., SEQC RNA-seq rank lists) rather than simulation-based validation of the estimator
- Only two replicates are available and the original two-replicate MaRR with the fn,0 null from Proposition 1 already applies
- Irreproducible genes are known to have dependent ranks, violating the independence assumption (P(Mh<=i) = i^3/n^3) on which the Mg(3) null rests

## Inputs

- Simulation setting grid: pi1 in {0.10, 0.30, 0.65, 0.80}, r0 in {0.30, 0.70, 0.95}
- Dataset size parameter n (modeled on 10,000 genes)
- Nominal FDR levels alpha in {0.05, 0.01}
- Number of Monte Carlo replicates per setting (100)
- Rank-correlation structure spec for reproducible genes (assumption I2) and independence spec for irreproducible genes (assumption I3)

## Outputs

- Synthetic three-replicate rank matrices (Rgx, Rgy, Rgz) per gene for each dataset
- Per-dataset MaRR outputs: k-hat, pi-hat_1, rejection threshold N-hat, number of genes declared reproducible
- Empirical mFDR per setting (false calls / total declared reproducible)
- Aggregated per-setting results table marr3_simulation_results.csv

## How to apply

Simulate three-replicate datasets following the simulation B design: for irreproducible genes draw ranks Rgx, Rgy, Rgz independently (Section 3 assumption I3), and for reproducible genes impose nonnegative rank correlation (assumption I2). Sweep pi1 in {0.10, 0.30, 0.65, 0.80} crossed with r0 in {0.30, 0.70, 0.95}, with dataset sizes modeled on n = 10,000 genes. For each setting generate 100 datasets, run the three-replicate MaRR procedure (using Mg(3)=max{Rgx,Rgy,Rgz} null distribution, MSEn(lambda) minimization over lambda in {0, 1/n, ..., 0.9}, and the mFDR estimator of equations (8)-(10) with E[V](i) = (n-k-hat)*P(Mh(3)<=i)) at nominal alpha = 0.05 and 0.01. Record pi-hat_1, the number of genes declared reproducible, and empirical mFDR (false calls divided by total declared reproducible), then aggregate per setting into marr3_simulation_results.csv. The rationale is that the marginal null CDF of Mg(3) generalizes fn,0 via P(Mh<=i) = i^3/n^3 under independence, so simulation is the direct way to confirm the Corollary 2-style F_n,pi1 approximation holds in the three-replicate case.

## Related tools

- **Python** (Implements the three-replicate rank simulation, Mg(3) null CDF and MSEn(lambda) minimization, and writes marr3_simulation_results.csv)

## Examples

```
import numpy as np; n=10000; R = np.column_stack([np.random.permutation(n) for _ in range(3)])  # irreproducible genes: independent ranks, P(M<=i)=i^3/n^3
```

## Evaluation signals

- Empirical mFDR is at or below the nominal alpha (0.05 and 0.01) in every simulation setting — no anti-conservative settings analogous to the weak-and-infrequent case mu_A=1, pi1=0.1 in Figure 6
- pi-hat_1 recovers the true pi1 across the {0.10, 0.30, 0.65, 0.80} grid without systematic overestimation
- Each per-setting row aggregates exactly 100 datasets; the number of declared reproducible genes is finite and consistent with the mFDR-hat threshold N-hat = max{i: mFDR-hat(i) <= alpha}
- The empirical survival function of Mg(3) closely tracks the weighted theoretical survival (1-lambda)*S_lambda at the minimizing lambda, confirming the i^3/n^3 null approximation
- marr3_simulation_results.csv contains one row per (pi1, r0, alpha) setting with pi-hat_1, declared-reproducible counts, and empirical mFDR columns

## Limitations

- The marginal null derivation for Mg(3) is only outlined in the supplementary materials, so the three-replicate CDF (including the j_pi1 = floor(n*pi1) truncation of Corollary 2) must be re-derived and verified numerically
- The two-replicate MaRR procedure was anti-conservative in the weak-and-infrequent signal case (mu_A=1, pi1=0.1); the three-replicate simulation must explicitly check whether such settings recur
- Simulation validity depends on assumption I3 (independent ranks for irreproducible genes); real replicate rank lists may violate this

## Evidence

- [methods] three-replicate simulation design parameters: [evidence span withheld — anchor status: not_found]
- [supplementary] Mg(3) null derivation location: [evidence span withheld — anchor status: not_found]
