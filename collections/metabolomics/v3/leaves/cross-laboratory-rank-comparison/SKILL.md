---
name: cross-laboratory-rank-comparison
description: Use when two or more replicate experiments (e.g., within-lab biological
  replicates or between-lab same-platform comparisons of RNA-seq read counts) need
  reproducibility assessment as gene rank lists with error control at a target FDR.
when_to_use_negative:
- When parametric dependence modeling of replicate data is acceptable and a model-based
  alternative such as the copula mixture model is preferred (MaRR estimates pi1-hat
  higher than the copula mixture model's 0.773, 0.767, 0.861, 0.649 on the same comparisons)
- When raw expression values are needed rather than rank-based reproducibility assessment
  — MaRR operates only on gene rank lists
- When replicate experiments do not yield a common set of features that can be consistently
  ranked across both experiments
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MaRR (maximum rank reproducibility procedure)
  role: Core statistical procedure computing the maximum rank statistic, k-hat via
    MSE minimization, mFDR-hat, and pi1-hat/rho-hat estimates
- name: Python (NumPy/pandas)
  role: Implementation of the closed-form MaRR estimator and generation of the seqc_marr_table5_results.csv
    summary table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:ecb0442242ae53f17604d2cecc697917d43fa19dc51ac9a8df762238999061fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MaRR (maximum rank reproducibility procedure)
  - Python (NumPy/pandas implementation of the closed-form MaRR estimator)
  - Python (NumPy/pandas)
  - maximum rank reproducibility (MaRR) procedure
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-laboratory-rank-comparison

## Summary

Applies the nonparametric Maximum Rank Reproducibility (MaRR) procedure to rank lists from replicate high-throughput experiments, declaring reproducible signals via a maximum rank statistic with marginal FDR control and estimating the proportion of reproducible signals (pi1-hat) and rank correlation (rho-hat).

## When to use

Use when two or more replicate experiments (e.g., within-lab biological replicates or between-lab same-platform comparisons of RNA-seq read counts) need reproducibility assessment as gene rank lists with error control at a target FDR.

## When NOT to use

- When parametric dependence modeling of replicate data is acceptable and a model-based alternative such as the copula mixture model is preferred (MaRR estimates pi1-hat higher than the copula mixture model's 0.773, 0.767, 0.861, 0.649 on the same comparisons)
- When raw expression values are needed rather than rank-based reproducibility assessment — MaRR operates only on gene rank lists
- When replicate experiments do not yield a common set of features that can be consistently ranked across both experiments

## Inputs

- Replicate rank lists of 43,919 transcripts ranked by RNA-seq read count (e.g., SEQC GSE47774 data from 13 laboratories on Illumina HiSeq 2000, SOLiD 5500, and Roche 454 GS FLX platforms)
- Six replicate comparisons: within-lab biological replicate pairs (Mayo Clinic, Penn State, NYU Medical Center) and between-lab same-platform comparisons (Illumina, Life Technologies, Roche 454)
- Target FDR level alpha (e.g., alpha = 0.01)

## Outputs

- Number of transcripts declared reproducible at alpha = 0.01 per comparison
- Estimated proportion of reproducible signals pi1-hat per comparison (e.g., 0.851, 0.847, 0.900, 0.898 for comparisons 1, 2, 4, 5)
- Rank correlation rho-hat among declared-reproducible transcripts per comparison
- Summary table (e.g., seqc_marr_table5_results.csv)

## How to apply

Rank the ~43,919 transcripts by read count within each experiment (rank 1 = most reads, ties broken by random assignment), then compute the maximum rank statistic Mg = max{Rgx, Rgy} for every gene in each replicate comparison. Estimate k-hat by minimizing the mean squared error between the empirical survival function of Mg and the weighted limiting survival function (1-lambda)*S_lambda over i = 0,...,floor(0.9n); the procedure requires no tuning parameters. Compute the marginal false discovery rate mFDR-hat(i) = (i-k-hat)^2 / (Q(i)*(n-k-hat)) and set the threshold N-hat = max{i : mFDR-hat(i) <= alpha}, declaring all genes with Mg <= N-hat reproducible. For each comparison record the number of declared-reproducible transcripts, pi1-hat, and rho-hat among declared signals. Rationale: the nonparametric procedure identifies where the change from reproducible to irreproducible signals begins without parametric assumptions on dependence structures or distributions of reproducible genes.

## Related tools

- **MaRR (maximum rank reproducibility procedure)** (Core statistical procedure computing the maximum rank statistic, k-hat via MSE minimization, mFDR-hat, and pi1-hat/rho-hat estimates)
- **Python (NumPy/pandas)** (Implementation of the closed-form MaRR estimator and generation of the seqc_marr_table5_results.csv summary table)

## Examples

```
python marr_reproducibility.py --counts seqc_gse47774_counts.csv --comparisons 6 --alpha 0.01 --out seqc_marr_table5_results.csv
```

## Evaluation signals

- Reproduce reported pi1-hat values 0.851, 0.847, 0.900, 0.898 for SEQC comparisons 1, 2, 4, 5 at alpha = 0.01
- Counts of declared-reproducible transcripts at alpha = 0.01 agree with Table 5 / Figure 9 of the article
- MaRR pi1-hat estimates should be higher than the copula mixture model's estimates of 0.773, 0.767, 0.861, 0.649 for comparisons 1, 2, 4, 5
- Sanity check that k-hat minimization was performed over i = 0,...,floor(0.9n) and mFDR-hat(i) <= 0.01 holds at the chosen threshold N-hat
- Ranking ties in the 43,919-transcript lists are broken by random assignment as specified in the workflow

## Limitations

- Ranking ties among equal read counts are broken by random assignment, which introduces stochasticity in Mg and downstream estimates
- MaRR makes no parametric assumptions on dependence structures or distributions of reproducible genes, so it may yield higher (more liberal) pi1-hat estimates than model-based approaches like the copula mixture model
- Threshold search is truncated at floor(0.9n), so extremely large k values beyond 90% of transcripts are not considered

## Evidence

- [abstract] MaRR effectively controls false discovery rates: "the MaRR procedure effectively controls false discovery rates, has desirable power properties, and compares well to existing methods"
- [intro] Nonparametric rank-list reproducibility without tuning parameters: "we introduce a non-parametric procedure to assess the reproducibility of gene rank lists. In contrast to the model-based method, this procedure does not make parametric assumptions"
- [intro] Change-point identification via MSE minimization of survival functions: "Based on a maximum rank statistic, our procedure identifies where the change from reproducible to irreproducible signals begins by minimizing the mean squared error"
- [intro] mFDR calculation from irreproducible maximum rank distribution: "The marginal false discovery rates for each gene are then calculated based on the distribution of irreproducible maximum rank statistics. The procedure requires no tuning parameters"
