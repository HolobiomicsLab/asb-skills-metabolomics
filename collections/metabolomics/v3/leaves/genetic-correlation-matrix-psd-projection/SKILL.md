---
name: genetic-correlation-matrix-psd-projection
description: Use when a multilevel/Bayesian model needs a correlation matrix among
  trait- or tissue-level effects (e.g. G_SNP built from GCTA-GRM output) that is empirically
  not positive semi-definite and would break the sampler.
when_to_use_negative:
- The correlation matrix is already positive semi-definite (e.g. from a validated
  LD panel), so Higham projection would only perturb it
- The analysis is univariate or per-gene (e.g. LDSC/MESC heritability estimation per
  gene) and no cross-trait correlation covariance is needed
- Correlations come from a small number of groups where shrinkage toward the identity
  via theta ~ Beta(1,1) would dominate the empirical signal
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: GCTA-GRM
  role: Estimates the pairwise genetic relationship / heritability structure from
    which the G_SNP correlation matrix is built
  repo: https://yanglab.westlake.edu.cn/software/gcta/#Overview
- name: Plink
  role: Used alongside GCTA-GRM to compute the GRM underlying the correlation matrix
- name: Matrix
  role: Provides Matrix::nearPD() (Higham's algorithm) to project G_SNP to positive
    semi-definiteness
- name: CmdStanR
  role: Fits the binomial Bayesian multilevel model using the projected, identity-blended
    covariance matrix as prior
- name: posterior
  role: Verifies convergence of the fitted model (rhat, bulk/tail ESS, divergences)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:97205a639c564c48d73d88f771bf1cdd84e9c967c5ab0a027825f53e4dc0cac5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CmdStanR
  - GCTA-GRM
  - Matrix
  - Plink
  - S-PrediXcan
  - Stan
  - posterior
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# genetic-correlation-matrix-psd-projection

## Summary

Projects a genetic correlation matrix (here G_SNP derived from GCTA-GRM estimates of expression heritability) to the nearest positive semi-definite matrix with Matrix::nearPD() (Higham's algorithm), then blends it with the identity matrix via a mixing weight theta ~ Beta(1, 1) for use as a covariance prior in a Bayesian multilevel Stan model of trait/tissue effect directions.

## When to use

Use when a multilevel/Bayesian model needs a correlation matrix among trait- or tissue-level effects (e.g. G_SNP built from GCTA-GRM output) that is empirically not positive semi-definite and would break the sampler.

## When NOT to use

- The correlation matrix is already positive semi-definite (e.g. from a validated LD panel), so Higham projection would only perturb it
- The analysis is univariate or per-gene (e.g. LDSC/MESC heritability estimation per gene) and no cross-trait correlation covariance is needed
- Correlations come from a small number of groups where shrinkage toward the identity via theta ~ Beta(1,1) would dominate the empirical signal

## Inputs

- Pairwise genetic correlation estimates among trait x tissue groups derived from GCTA-GRM (GCTA 1.93.2) and Plink outputs
- Square correlation matrix G_SNP (possibly indefinite) to be projected
- Posterior MCMC output from /data/MCMC_output of the MoTrPAC_Complex_Traits repository
- Standardized exercise effect sizes with direction signs (product of DE direction and PrediXcan association direction)

## Outputs

- Positive semi-definite correlation matrix (nearPD output with unit diagonal)
- Blended covariance matrix averaged with the identity matrix via theta ~ Beta(1, 1)
- Converged CmdStanR posterior samples (rhat < 1.01, bulk/tail ESS > 500, < 0.05% divergences) of direction-of-effect deviations per trait x tissue

## How to apply

Derive the pairwise genetic correlation estimates (in this paper, from GCTA-GRM 1.93.2 / Plink estimates of expression heritability, supplied alongside posterior MCMC output in /data/MCMC_output of the MoTrPAC_Complex_Traits repository). Assemble the square correlation matrix G_SNP among trait x tissue groups and run Matrix::nearPD() with corr=TRUE (Higham's alternating-projections algorithm) so that all eigenvalues are >= 0 while the diagonal remains 1; this is required because empirically estimated correlation matrices are frequently indefinite and would invalidate a multivariate normal covariance. To avoid over-trusting the projection, average G_SNP with the identity matrix using a weight theta ~ Beta(1, 1), which shrinks the prior toward independence. Feed the resulting blended matrix into the binomial Bayesian multilevel model fit in Stan via CmdStanR with four randomly initialized chains (2.5e3 warmup, 2.5e3 sampling iterations, target acceptance delta = 0.95, non-centered parameterization), then verify convergence with the posterior package.

## Related tools

- **GCTA-GRM** (Estimates the pairwise genetic relationship / heritability structure from which the G_SNP correlation matrix is built) — https://yanglab.westlake.edu.cn/software/gcta/#Overview
- **Plink** (Used alongside GCTA-GRM to compute the GRM underlying the correlation matrix)
- **Matrix** (Provides Matrix::nearPD() (Higham's algorithm) to project G_SNP to positive semi-definiteness)
- **CmdStanR** (Fits the binomial Bayesian multilevel model using the projected, identity-blended covariance matrix as prior)
- **posterior** (Verifies convergence of the fitted model (rhat, bulk/tail ESS, divergences))

## Examples

```
G_snp_pd <- Matrix::nearPD(G_snp, corr = TRUE)$mat  # Higham projection before blending with identity via theta ~ Beta(1,1) in the CmdStanR model
```

## Evaluation signals

- After Matrix::nearPD(corr=TRUE), all eigenvalues of the projected G_SNP are >= 0 and the diagonal is 1
- CmdStanR fit passes convergence checks: rhat < 1.01, bulk and tail ESS > 500, and < 0.05% divergent transitions across four randomly initialized chains
- The blended covariance lies between G_SNP and the identity, with theta ~ Beta(1, 1) posterior concentrated near 0.5 absent strong evidence
- Marginal posterior means of direction-of-effect deviations reproduce the reported trait ranking: strongest positive enrichments for body fat percentage, asthma, and BMI; strongest depletions for standing height and high cholesterol
- Enrichment/depletion posterior means remain within 0 +/- 0.3 on the log-odds scale (max ~7.5% on the probability scale), matching the reported effect-size range

## Limitations

- Empirical correlation matrices built from GCTA-GRM estimates are not guaranteed positive semi-definite, so projection changes the matrix; downstream inference depends on this approximation plus Beta(1,1) shrinkage
- No individual tissues were confidently enriched in positive effects for some phenotypes (e.g. asthma), and some marginal posterior distributions for difference parameters did not reach the 95% posterior mass threshold
- Downstream trait associations rely on GTEx/S-PrediXcan results with limited portability to non-European populations, which propagates into any G_SNP-based covariance

## Evidence

- [methods] we used the software Plink53 and GCTA54, specifically GCTA-GRM55: "we used the software Plink53 and GCTA54, specifically GCTA-GRM55"
- [methods] GCTA version: "[GCTA](https://yanglab.westlake.edu.cn/software/gcta/#Overview) (1.93.2)"
- [discussion] effect sizes on the log-odds scale: "predominately varying within 0 ± 0.3 on the log-odds scale"
- [readme] LDSC estimates genetic correlation: [evidence span withheld — anchor status: not_found]
- [discussion] asthma posterior-mass caveat: "though no marginal posterior distributions for difference parameters there reached our 95% posterior mass threshold"
