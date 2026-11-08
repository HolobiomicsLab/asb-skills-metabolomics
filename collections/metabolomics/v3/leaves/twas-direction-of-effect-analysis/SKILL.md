---
name: twas-direction-of-effect-analysis
description: Use when you have per-gene direction-of-effect calls (product of DE direction
  and PrediXcan association direction) aggregated into trait–tissue gene sets and
  need to test enrichment/depletion of positive effects against a null log-odds of
  0 with correlated-trait correction.
when_to_use_negative:
- When the question is single-gene differential expression (use DESeq2 per sex/tissue
  with likelihood ratio tests instead), not the aggregate direction of trait-associated
  gene effects.
- When you lack per-gene direction-of-effect calls linking rat DE direction to human
  PrediXcan association direction — a binomial proportion model on directions is meaningless
  without signed gene sets.
- 'When interpreting qualitative sex-specific effects: the paper explicitly states
  sex-specificity of exercise adaptation ''fell outside the scope considered here.'''
tools:
- name: Stan
  role: Probabilistic programming language for the binomial Bayesian multilevel direction-of-effect
    model
- name: CmdStanR
  role: R interface used to fit the Stan model with 4 chains, 2.5e3 warmup/sampling
    iterations, adapt_delta = 0.95, non-centered parameterization
- name: posterior
  role: Convergence diagnostics (rhat, bulk/tail ESS) and extraction of marginal posterior
    means
- name: Matrix
  role: nearPD() (Higham's algorithm) to project the G_SNP correlation matrix to positive
    semi-definiteness
- name: GCTA-GRM
  role: Source of the correlation matrix among traits from gene expression heritability
    estimates
  repo: https://yanglab.westlake.edu.cn/software/gcta/#Overview
- name: S-PrediXcan
  role: Provides gene-by-tissue associations and directions of effect for specific
    diseases used to define positive vs. negative gene directions
  repo: https://github.com/hakyimlab/MetaXcan
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:47d8305b5feac90b620bba51647ef200c87a426791f12c64d5c9a43008312bfc
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# twas-direction-of-effect-analysis

## Summary

Bayesian multilevel binomial model (Stan/CmdStanR) that tests whether the proportion of exercise-induced, disease-association direction-positive genes in trait–tissue PrediXcan–DEG intersecting gene sets deviates from a null log-odds of 0, accounting for G_SNP correlation among traits. It ranks traits by direction-of-effect posterior means and traces high-ranking traits back to candidate genes.

## When to use

Use when you have per-gene direction-of-effect calls (product of DE direction and PrediXcan association direction) aggregated into trait–tissue gene sets and need to test enrichment/depletion of positive effects against a null log-odds of 0 with correlated-trait correction.

## When NOT to use

- When the question is single-gene differential expression (use DESeq2 per sex/tissue with likelihood ratio tests instead), not the aggregate direction of trait-associated gene effects.
- When you lack per-gene direction-of-effect calls linking rat DE direction to human PrediXcan association direction — a binomial proportion model on directions is meaningless without signed gene sets.
- When interpreting qualitative sex-specific effects: the paper explicitly states sex-specificity of exercise adaptation 'fell outside the scope considered here.'

## Inputs

- Posterior MCMC output (/data/MCMC_output) from the repository
- Standardized exercise training effect sizes per gene (rat, MoTrPAC EET)
- S-PrediXcan gene-by-tissue association directions of effect for human traits
- GCTA-GRM-derived gene expression heritability correlation matrix (G_SNP) per trait
- repfdr-assigned differential-expression states for the 8W sex-consistent gene set (and 1-, 2-, 4-week predecessors)

## Outputs

- Marginal posterior means of the direction-of-effect deviation per trait x tissue, with trait rankings
- Trait-level enrichment/depletion calls with 95% posterior mass thresholds
- Candidate gene table (cholesterol: NDUFA13, FADS2, PNKD, AAMP, OGDH, TMBIM1, APOB, ABCG8; asthma: BAG6, CCNG, CRAT, PTPA, FAM89B, ATP6V1G2, ENDOU, CCNF) with tissue, +/- direction sign, and standardized effect size

## How to apply

Load the posterior MCMC output (/data/MCMC_output) and standardized exercise effect sizes, then fit a binomial Bayesian multilevel model in Stan via CmdStanR comparing the observed frequency of positive direction-of-effect genes per trait against a null log-odds of 0. Derive the trait correlation matrix G_SNP from GCTA-GRM estimates, project it to positive semi-definiteness with Matrix::nearPD() (Higham's algorithm), and average it with the identity matrix using weight theta ~ Beta(1, 1). Run four randomly initialized chains with 2.5e3 warmup and 2.5e3 sampling iterations, target acceptance adapt_delta = 0.95, and a non-centered parameterization. Verify convergence with the posterior package (rhat < 1.01, bulk/tail ESS > 500, < 0.05% divergences), extract marginal posterior means of the direction-of-effect deviation per trait x tissue, and rank traits. For top non-anthropometric traits (e.g., cholesterol and asthma), trace the proportion of positive effects of the 8W sex-consistent (F1_M1 / F-1_M-1) gene set backwards through the 1-, 2-, and 4-week time points, assigning 0.5 to genes not assigned to a differential-expression node, and compile a candidate gene table with tissue, +/- sign, and standardized effect size.

## Related tools

- **Stan** (Probabilistic programming language for the binomial Bayesian multilevel direction-of-effect model)
- **CmdStanR** (R interface used to fit the Stan model with 4 chains, 2.5e3 warmup/sampling iterations, adapt_delta = 0.95, non-centered parameterization)
- **posterior** (Convergence diagnostics (rhat, bulk/tail ESS) and extraction of marginal posterior means)
- **Matrix** (nearPD() (Higham's algorithm) to project the G_SNP correlation matrix to positive semi-definiteness)
- **GCTA-GRM** (Source of the correlation matrix among traits from gene expression heritability estimates) — https://yanglab.westlake.edu.cn/software/gcta/#Overview
- **S-PrediXcan** (Provides gene-by-tissue associations and directions of effect for specific diseases used to define positive vs. negative gene directions) — https://github.com/hakyimlab/MetaXcan

## Examples

```
fit <- mod$sample(data = stan_data, chains = 4, iter_warmup = 2500, iter_sampling = 2500, adapt_delta = 0.95, parallel_chains = 4)
```

## Evaluation signals

- Convergence diagnostics pass: rhat < 1.01, bulk and tail ESS > 500, and < 0.05% divergent transitions across the four chains.
- Posterior mean trait ranking reproduces the reported pattern: strongest positive enrichments for body fat percentage, asthma, and BMI; strongest mean depletions for standing height and high cholesterol.
- Every candidate gene row carries a tissue, a +/- direction sign (product of DE direction and PrediXcan association direction), and a standardized effect size consistent with Fig. 2b.
- Enrichment/depletion effect sizes fall within the reported range of 0 ± 0.3 on the log-odds scale (≈7.5% maximum difference on the probability scale).

## Limitations

- No individual tissues were confidently enriched in positive effects for the asthma phenotype — the signal is trait-level, not tissue-resolved.
- Some marginal posterior distributions for difference parameters did not reach the 95% posterior mass threshold.
- The analysis used the 8W sex-consistent gene set; genes not assigned to a differential-expression node were assigned a neutral 0.5 proportion when tracing back through earlier time points.
- Results have limited portability to non-European populations, as the underlying GTEx-based PrediXcan sample comprises mostly European-descendant individuals.

## Evidence

- [results] Direction-of-effect finding (trait ranking): "the strongest confident mean enrichments for positive effects were observed in body fat percentage, asthma, and body mass index (BMI), and the strongest mean depletions in standing height and high"
- [methods] Model fitting and convergence workflow: [evidence span withheld — anchor status: not_found]
- [methods] Trait correlation matrix construction: [evidence span withheld — anchor status: not_found]
- [discussion] Asthma trait-level caveat: "Though no individual tissues were found to be confidently enriched in positive effects for this phenotype"
- [readme] Repository provenance: "This is the GitHub repository associated with the publication:"
