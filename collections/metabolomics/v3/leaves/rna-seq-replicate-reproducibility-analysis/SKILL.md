---
name: rna-seq-replicate-reproducibility-analysis
description: Use when you have read counts or rank lists for the same transcript/gene
  set from two replicate RNA-seq experiments (within-lab or between-lab, e.g. SEQC
  comparisons) and need to quantify reproducibility with error control at a target
  mFDR such as alpha = 0.01.
when_to_use_negative:
- Input is raw sequencing reads or unquantified FASTQ/BAM files — the skill operates
  on transcript-level read counts or rank lists, so reads must first be mapped to
  a common transcript set and counted
- The analysis requires explicit parametric modeling of the dependence structure or
  per-gene effect distributions — MaRR deliberately avoids parametric assumptions,
  unlike the copula mixture model
- Only a single experiment is available with no replicate pair — the maximum rank
  statistic Mg = max{Rgx, Rgy} requires two replicate rank lists
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Maximum rank reproducibility (MaRR) procedure
  role: 'Core statistical method: computes maximum rank statistics, estimates k-hat
    by MSE minimization of survival functions, derives mFDR-hat, and returns reproducibility
    counts, pi1-hat, and rho-hat'
- name: Python (NumPy/pandas)
  role: 'Implementation environment for the closed-form MaRR estimator: ranking, computing
    Mg, minimizing MSE to the weighted limiting survival function, and writing the
    results CSV'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:7f89849616ca354175032b903287a8ae5fdcd2d2e705d4b3f7e226f438bd9047
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Maximum rank reproducibility (MaRR) procedure
  - Python (NumPy/pandas implementation of the closed-form MaRR estimator)
  - Python (NumPy/pandas)
  - maximum rank reproducibility (MaRR) procedure
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# rna-seq-replicate-reproducibility-analysis

## Summary

Applies the Maximum Rank Reproducibility (MaRR) procedure — a nonparametric, tuning-parameter-free method — to rank lists from paired RNA-seq replicate experiments (e.g., SEQC multi-lab data over 43,919 transcripts), estimating the proportion of reproducible transcripts (pi1-hat), their rank correlation (rho-hat), and declaring reproducible signals at a target mFDR (alpha = 0.01).

## When to use

Use when you have read counts or rank lists for the same transcript/gene set from two replicate RNA-seq experiments (within-lab or between-lab, e.g. SEQC comparisons) and need to quantify reproducibility with error control at a target mFDR such as alpha = 0.01.

## When NOT to use

- Input is raw sequencing reads or unquantified FASTQ/BAM files — the skill operates on transcript-level read counts or rank lists, so reads must first be mapped to a common transcript set and counted
- The analysis requires explicit parametric modeling of the dependence structure or per-gene effect distributions — MaRR deliberately avoids parametric assumptions, unlike the copula mixture model
- Only a single experiment is available with no replicate pair — the maximum rank statistic Mg = max{Rgx, Rgy} requires two replicate rank lists

## Inputs

- Read-count matrix for the same transcript set (e.g. 43,919 transcripts) from two RNA-seq replicate experiments, or two precomputed rank lists over the same gene IDs
- Per-experiment replicate count data from a multi-lab study (e.g. SEQC GSE47774: Illumina HiSeq 2000, SOLiD 5500, Roche 454 GS FLX platforms)
- Target marginal false discovery rate alpha (e.g. 0.01)

## Outputs

- Number of transcripts declared reproducible at alpha = 0.01 per comparison pair
- Estimated proportion of reproducible transcripts pi1-hat per comparison
- Rank correlation rho-hat among declared-reproducible transcripts per comparison
- Summary table (e.g. seqc_marr_table5_results.csv) with one row per comparison

## How to apply

Download the replicate RNA-seq count data (e.g. SEQC GSE47774, all reads mapped to the same 43,919 transcripts) and form two-experiment comparison pairs, summing read counts over replicates per lab for between-lab comparisons. For each comparison, rank transcripts by read count within each experiment (rank 1 = most reads, ties broken by random assignment) and compute the maximum rank statistic Mg = max{Rgx, Rgy} for every gene. Estimate k-hat by minimizing the mean squared error between the empirical survival function and the weighted limiting survival function (1-lambda)*S_lambda over i = 0,...,floor(0.9n), then compute mFDR-hat(i) = (i-k-hat)^2 / (Q(i)*(n-k-hat)) and set the threshold N-hat = max{i : mFDR-hat(i) <= alpha}; declare all genes with Mg <= N-hat reproducible. Because MaRR makes no parametric assumptions about dependence structures or the distributions of reproducible genes, it requires no tuning parameters, unlike model-based alternatives such as the copula mixture model. Finally record per comparison the number of reproducible signals at alpha = 0.01, pi1-hat, and rho-hat among declared transcripts (e.g. SEQC comparisons 1, 2, 4, 5 gave pi1-hat = 0.851, 0.847, 0.900, 0.898).

## Related tools

- **Maximum rank reproducibility (MaRR) procedure** (Core statistical method: computes maximum rank statistics, estimates k-hat by MSE minimization of survival functions, derives mFDR-hat, and returns reproducibility counts, pi1-hat, and rho-hat)
- **Python (NumPy/pandas)** (Implementation environment for the closed-form MaRR estimator: ranking, computing Mg, minimizing MSE to the weighted limiting survival function, and writing the results CSV)

## Examples

```
Mg = np.maximum(Rgx, Rgy); k_hat = argmin_i mse(emp_surv, (1-lam)*S_lam)[:int(0.9*n)]; N_hat = max(i for i in range(n) if (i-k_hat)**2/(Q(i)*(n-k_hat)) <= 0.01)
```

## Evaluation signals

- pi1-hat for SEQC comparisons 1, 2, 4, and 5 reproduces 0.851, 0.847, 0.900, and 0.898 respectively (higher than the copula mixture model's 0.773, 0.767, 0.861, 0.649)
- Counts of reproducible transcripts at alpha = 0.01 agree with Table 5 / Figure 9 of the paper
- k-hat is estimated over i = 0,...,floor(0.9n) by minimizing MSE between the empirical survival function and (1-lambda)*S_lambda; verify mFDR-hat(i) <= 0.01 holds at N-hat but fails at N-hat+1
- Ties in ranking are broken by random assignment, and between-lab comparisons use read counts summed over all replicates per lab
- Output table contains exactly six comparison rows (three within-lab: Mayo Clinic, Penn State, NYU Medical Center; three between-lab same-platform: Illumina, Life Technologies, Roche 454) each with counts, pi1-hat, and rho-hat

## Limitations

- mFDR-hat is a marginal, not per-gene, false discovery rate, so individual declared transcripts carry no gene-specific error guarantee
- Random tie-breaking in rank assignment introduces stochasticity; results may vary slightly across runs unless a fixed random seed is used
- The procedure is rank-based and nonparametric, so it does not estimate per-transcript dependence or effect magnitudes and relies on survival-function fit up to floor(0.9n) rather than full-sample modeling

## Evidence
