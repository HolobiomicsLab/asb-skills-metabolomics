---
name: bayesian-multilevel-binomial-modeling
description: Use when testing whether a gene set (e.g., 8W sex-consistent exercise
  DEGs from repfdr nodes 8w_F1_M1 and 8w_F-1_M-1) is enriched among IHW α = 0.05 PrediXcan
  hits across tissues, traits, and trait categories using cell-level intersect counts.
when_to_use_negative:
- When individual-level genotype and phenotype data are available for direct gene-trait
  association (use PrediXcan.py rather than the summary-statistics/hierarchical enrichment
  route)
- When analyzing qualitative sex-specificity of exercise adaptation — the article
  states this 'fell outside the scope considered here'
- When no gene-wise or LD correlation structure can be estimated (the model requires
  PSD-projected correlation matrices, e.g., from a bivariate probit or GCTA-GRM)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3053
tools:
- name: CmdStanR
  role: R interface to Stan for fitting the hierarchical binomial model with non-centered
    parameterization, 4 chains, δ = 0.95
- name: posterior
  role: MCMC convergence diagnostics (r̂ < 1.01, bulk/tail ESS > 500, divergence rate)
    and posterior summary extraction
- name: Matrix
  role: nearPD() projection of estimated correlation matrices to the nearest positive
    semi-definite matrix (Higham's algorithm)
- name: GCTA-GRM
  role: Estimates the G_SNP genetic relatedness matrix used as the SNP-correlation
    input to the direction-of-effect model
  repo: https://yanglab.westlake.edu.cn/software/gcta/#Overview
- name: S-PrediXcan
  role: Provides gene-by-tissue-level trait associations and directions of effect
    whose IHW-significant hits form the count basis for enrichment
  repo: https://github.com/hakyimlab/MetaXcan
- name: DESeq2
  role: Differential expression analysis per sex and tissue whose results feed the
    repfdr-derived 8W sex-consistent DEG sets
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:64fc863a4880e57fcc95e35aba51e0bbed8f792f5e2782080579171502b583f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CmdStanR
  - DESeq2
  - GCTA-GRM
  - Matrix
  - Plink
  - PrediXcan
  - S-PrediXcan
  - Stan
  - fgsea
  - posterior
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# bayesian-multilevel-binomial-modeling

## Summary

Hierarchical Bayesian partial-pooling binomial modeling (Stan/CmdStanR) that tests whether sex-consistent 8-week exercise DEGs are enriched or depleted among IHW-significant S-PrediXcan hits per tissue, trait, and tissue×trait cell, with correlated random effects and posterior-probability-based enrichment calls.

## When to use

Use when testing whether a gene set (e.g., 8W sex-consistent exercise DEGs from repfdr nodes 8w_F1_M1 and 8w_F-1_M-1) is enriched among IHW α = 0.05 PrediXcan hits across tissues, traits, and trait categories using cell-level intersect counts.

## When NOT to use

- When individual-level genotype and phenotype data are available for direct gene-trait association (use PrediXcan.py rather than the summary-statistics/hierarchical enrichment route)
- When analyzing qualitative sex-specificity of exercise adaptation — the article states this 'fell outside the scope considered here'
- When no gene-wise or LD correlation structure can be estimated (the model requires PSD-projected correlation matrices, e.g., from a bivariate probit or GCTA-GRM)

## Inputs

- S-PrediXcan outputs filtered at IHW α = 0.05 with tissue×trait pairs as covariate
- Sex-consistent 8-week DEG sets (repfdr nodes 8w_F1_M1 and 8w_F-1_M-1)
- Per-cell intersect counts (99 of 114 traits with nonzero intersect)
- Gene-wise correlation matrices (Σi, Σj, Σi×j) or GCTA-GRM-derived G_SNP matrix
- Standardized exercise effect sizes and direction signs (DE direction × PrediXcan direction)

## Outputs

- Posterior summary tables (CSV) for tissue (βi), trait (γj), and trait-category (μk) difference effects
- Enrichment/depletion flags per tissue×trait cell (> 95% posterior mass on one side of 0; red/blue)
- Ranked trait-level direction-of-effect posterior means
- MCMC diagnostic report (r̂, ESS, divergences) from the posterior package

## How to apply

Compute intersect counts per tissue×trait cell between the DEG set and IHW α = 0.05-filtered S-PrediXcan hits, and estimate tissue (Σi), trait (Σj), and tissue×trait (Σi×j) gene-wise correlation matrices under a bivariate probit via nlm (stats/optimx), projecting each to the nearest positive semi-definite matrix with Matrix::nearPD() (Higham's algorithm). Fit the hierarchical binomial model in Stan via CmdStanR with a non-centered parameterization: 4 randomly initialized chains, 2500 warmup and 2500 sampling iterations each, target acceptance δ = 0.95. For the direction-of-effect variant, compare the observed frequency of positive effects against a null log-odds of 0, using a G_SNP correlation matrix from GCTA-GRM (projected to PSD and averaged with the identity matrix under theta ~ Beta(1,1)). Diagnose MCMC output with the posterior package (r̂ < 1.01, bulk and tail ESS > 500, < 0.05% divergent iterations) before extracting marginal posteriors for difference effects. Flag tissue×trait cells with > 95% posterior mass on one side of 0 as enriched or depleted; interpret effect sizes on the log-odds scale, where the study found effects predominantly within 0 ± 0.3 (≈7.5% maximum difference on the probability scale).

## Related tools

- **CmdStanR** (R interface to Stan for fitting the hierarchical binomial model with non-centered parameterization, 4 chains, δ = 0.95)
- **posterior** (MCMC convergence diagnostics (r̂ < 1.01, bulk/tail ESS > 500, divergence rate) and posterior summary extraction)
- **Matrix** (nearPD() projection of estimated correlation matrices to the nearest positive semi-definite matrix (Higham's algorithm))
- **GCTA-GRM** (Estimates the G_SNP genetic relatedness matrix used as the SNP-correlation input to the direction-of-effect model) — https://yanglab.westlake.edu.cn/software/gcta/#Overview
- **S-PrediXcan** (Provides gene-by-tissue-level trait associations and directions of effect whose IHW-significant hits form the count basis for enrichment) — https://github.com/hakyimlab/MetaXcan
- **DESeq2** (Differential expression analysis per sex and tissue whose results feed the repfdr-derived 8W sex-consistent DEG sets)

## Examples

```
fit <- model$sample(data = stan_data, chains = 4, iter_warmup = 2500, iter_sampling = 2500, adapt_delta = 0.95)
```

## Evaluation signals

- All Stan parameters pass r̂ < 1.01, bulk and tail ESS > 500, and < 0.05% divergent iterations
- Posterior summaries reproduce the reported confident positive enrichments: colon, kidneys, small intestines, spleen, hippocampus, lungs, heart (decreasing posterior mean) and Endocrine and Cardiometabolic trait categories
- Direction-of-effect posterior means rank body fat percentage, asthma, and BMI as strongest positive and standing height and high cholesterol as strongest depletions
- Every flagged cell has > 95% posterior mass on one side of 0; unflagged difference parameters are reported as not confident
- Effect sizes remain predominantly within 0 ± 0.3 on the log-odds scale (≈7.5% maximum difference on the probability scale)

## Limitations

- Correlation matrices estimated under the bivariate probit may not be positive semi-definite and require Higham nearPD projection, which can distort the intended covariance structure
- Some marginal posterior distributions for difference parameters do not reach the 95% posterior mass threshold (e.g., no individual tissues were confidently enriched in positive effects for asthma)
- Enrichment/depletion effects are small (±0.3 log-odds), so cell-level calls are sensitive to count sparsity — only 99 of 114 traits had nonzero intersects

## Evidence

- [methods] IHW filtering of PrediXcan hits: "filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate)"
- [methods] GRM estimation software: "we used the software Plink53 and GCTA54, specifically GCTA-GRM55"
- [discussion] Log-odds effect size scale: "predominately varying within 0 ± 0.3 on the log-odds scale"
- [discussion] Non-confident asthma enrichment: "Though no individual tissues were found to be confidently enriched in positive effects for this phenotype"
- [readme] Repository hosting R scripts: "It primarily hosts scripts written in the R programming language, but interfacing with several other languages, that were used to"
