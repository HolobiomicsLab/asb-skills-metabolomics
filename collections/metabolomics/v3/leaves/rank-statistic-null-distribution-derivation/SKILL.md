---
name: rank-statistic-null-distribution-derivation
description: Use when ranks of features (e.g. genes) are available from two or more
  independent replicate experiments and you need the theoretical null CDF of the maximum
  rank statistic to estimate pi1 or assign marginal FDRs to reproducible features.
when_to_use_negative:
- Input is raw expression/counts data rather than feature rank lists — MaRR's derivations
  operate on ranks and require per-replicate rankings first
- The dependence structure of reproducible features cannot satisfy the nonnegative
  rank-correlation assumption (assumption I2) or irreproducible features lack independent
  ranks (assumption I3), since the null CDF derivation relies on these assumptions
- A parametric, model-based joint distribution of reproducible features is preferred
  and its assumptions are justified — MaRR is explicitly nonparametric and makes no
  parametric assumptions on reproducible-gene dependence or distributions
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Implementation language for deriving the null CDFs, running the MSEn(lambda)
    grid minimization, computing mFDR estimates, and simulating three-replicate datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:9c0a3baa98d7c0e07ee98f1d04b369e7bac4bb15215d09b637dce708d930c1a6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# rank-statistic-null-distribution-derivation

## Summary

Derives the marginal null distribution of maximum-rank statistics (fn,0, the pi1-weighted CDF F_n,pi1, and its k-replicate generalization such as Mg(3)=max{Rgx,Rgy,Rgz}) that underpins the MaRR reproducibility procedure. These closed-form distributions enable nonparametric estimation of the reproducible signal proportion pi1 and marginal FDR control without tuning parameters.

## When to use

Use when ranks of features (e.g. genes) are available from two or more independent replicate experiments and you need the theoretical null CDF of the maximum rank statistic to estimate pi1 or assign marginal FDRs to reproducible features.

## When NOT to use

- Input is raw expression/counts data rather than feature rank lists — MaRR's derivations operate on ranks and require per-replicate rankings first
- The dependence structure of reproducible features cannot satisfy the nonnegative rank-correlation assumption (assumption I2) or irreproducible features lack independent ranks (assumption I3), since the null CDF derivation relies on these assumptions
- A parametric, model-based joint distribution of reproducible features is preferred and its assumptions are justified — MaRR is explicitly nonparametric and makes no parametric assumptions on reproducible-gene dependence or distributions

## Inputs

- Ranked lists of features (e.g. genes) from replicate high-throughput experiments, such as RNA-seq rank lists from SEQC consortium data
- Candidate pi1/reproducibility proportion values or a grid lambda in {0, 1/n, 2/n, ..., 0.9} for survival-function fitting
- Nominal FDR level alpha (e.g. 0.05 or 0.01)
- Optionally, three or more replicate rank lists (Rgx, Rgy, Rgz) for the order-statistic generalization

## Outputs

- Marginal null CDF/survival function of the maximum rank statistic (fn,0 or the k-replicate analog, e.g. P(Mg(3)<=i)=i^3/n^3 under independence)
- pi1-weighted theoretical distribution F_n,pi1 with boundary j_pi1=floor(n*pi1)
- Estimated change point k-hat and reproducibility proportion pi-hat_1 from MSEn(lambda) minimization
- Per-feature marginal FDR estimates E[V](i) and rejection set of features with N-hat = max{i: mFDR-hat(i) <= alpha}
- Aggregated simulation validation table (e.g. marr3_simulation_results.csv)

## How to apply

Start from the independent-rank assumption for irreproducible features: under H0 the CDF of the maximum rank Mh is P(Mh<=i)=(i/n)^k, which generalizes Proposition 1's fn,0 for k=2. Incorporate the reproducible proportion pi1 by deriving the mixture CDF F_n,pi1 analogously to Corollary 2, using j_pi1=floor(n*pi1) as the boundary between reproducible and irreproducible ranks. Fit the theoretical survival (1-lambda)*S_lambda to the empirical survival function of observed maximum-rank statistics by minimizing the MSEn(lambda) of Definition 2 over lambda in {0, 1/n, 2/n, ..., 0.9}, yielding k-hat (change point) and pi-hat_1. Then compute marginal FDRs from the null CDF, e.g. E[V](i)=(n-k-hat)*P(Mh<=i) for the k-replicate statistic, and choose the rejection threshold N-hat = max{i: mFDR-hat(i) <= alpha} (equations (8)-(10), alpha commonly 0.05 or 0.01). Validate by simulation (independent ranks for irreproducible genes, nonnegative rank correlation for reproducible genes, pi1 in {0.10, 0.30, 0.65, 0.80}, r0 in {0.30, 0.70, 0.95}, n modeled on 10,000 genes) confirming empirical mFDR stays at or below nominal alpha.

## Related tools

- **Python** (Implementation language for deriving the null CDFs, running the MSEn(lambda) grid minimization, computing mFDR estimates, and simulating three-replicate datasets)

## Examples

```
python run_marr3_sim.py --reps 3 --pi1 0.10,0.30,0.65,0.80 --r0 0.30,0.70,0.95 --n 10000 --alpha 0.05 --out marr3_simulation_results.csv
```

## Evaluation signals

- Across simulation settings (pi1 in {0.10, 0.30, 0.65, 0.80}; r0 in {0.30, 0.70, 0.95}; n modeled on 10,000), empirical mFDR (false calls / total declared reproducible) should be at or below nominal alpha = 0.05 and 0.01 — no anti-conservative settings analogous to the weak-and-infrequent case mu_A=1, pi1=0.1
- pi-hat_1 should recover the true pi1 without systematic overestimation across settings
- Under independent irreproducible ranks, the fitted theoretical survival (1-lambda)*S_lambda should closely track the empirical survival function at the minimizing lambda, yielding a stable k-hat near the true change point
- For the three-replicate statistic, verify the independence CDF algebraically: P(Mg(3)<=i) = i^3/n^3
- The mFDR estimator must satisfy E[V](i) = (n-k-hat)*P(Mh<=i) with N-hat = max{i: mFDR-hat(i) <= alpha}, matching equations (8)-(10)

## Limitations

- The derivation assumes irreproducible features have independent ranks across replicates (assumption I3); dependence among irreproducible features violates the null CDF
- Reproducible features are assumed to have nonnegative rank correlation (assumption I2); sign-flipped or negatively correlated reproducible signals fall outside the derivation
- The k-replicate extension (Mg(3) or second order statistic) was outlined rather than fully developed in the main text — the Mg(3) derivation is deferred to the supplementary materials
- Method behavior in weak-and-infrequent signal regimes (e.g. mu_A=1, pi1=0.1) can be anti-conservative, as noted for the original two-replicate procedure

## Evidence

- [supplementary] For three replicates with ranks Rgx, Rgy, Rgz, either the maximum order statistic Mg(3)=max{Rgx,Rgy,Rgz} or the second order statistic Mg(2) can be used: [evidence span withheld — anchor status: not_found]
- [supplementary] P(Mh<=i) = i^3/n^3 under independence: [evidence span withheld — anchor status: not_found]
