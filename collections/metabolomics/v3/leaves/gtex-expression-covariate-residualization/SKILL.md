---
name: gtex-expression-covariate-residualization
description: Use when you need human-expression-variability-normalized effect sizes
  for cross-species DE comparisons, e.g. dividing exercise log2FCs by GTEx per-gene,
  per-tissue standard deviations.
when_to_use_negative:
- Your analysis uses the default GTEx inverse-normal-transformed expression values
  and does not require variance-standardized (SD-unit) effect sizes
- The gene x tissue pair fails the IHW alpha = 0.10 heritability significance filter,
  so its SDgeno estimate is not confidently heritable
- You need qualitative sex-specific exercise responses rather than sex-consistent
  standardized effect sizes (the article notes sex-specificity 'fell outside the scope
  considered here')
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GTEx Pipeline
  role: Source of GTEx v8 expression processing whose default inverse-normal transform
    is replaced by a pseudolog log2(x+1) transform
  repo: https://github.com/broadinstitute/gtex-pipeline/
- name: fitdistrplus
  role: fitdist() maximum goodness-of-fit estimator for the inverse-gamma prior on
    per-gene, per-tissue expression variances
- name: Plink
  role: Genotype processing upstream of GCTA-GRM heritability estimation
- name: GCTA-GRM
  role: Estimates h_SNP^2 of each gene's expression using the same covariates, bounded
    between 0 and 1
  repo: https://yanglab.westlake.edu.cn/software/gcta/#Overview
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:f8911d601611c793d87cd54469fb15d9e0b5c9bf4a760dc713a202b729b246c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/gtex-pipeline
  tools:
  - GCTA
  - GCTA-GRM
  - GTEx Pipeline
  - Plink
  - fitdistrplus
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# gtex-expression-covariate-residualization

## Summary

Reprocess GTEx v8 expression by replacing the pipeline's default inverse-normal transform with a pseudolog (log2(x+1)) transform and residualizing sex, genotyping PCs, sequencing platform/protocol, and PEER factors, so that exercise log2FCs from MoTrPAC rat training can be expressed as standardized effect sizes in units of human phenotypic (SDpheno) and genetic (SDgeno) expression variability.

## When to use

Use when you need human-expression-variability-normalized effect sizes for cross-species DE comparisons, e.g. dividing exercise log2FCs by GTEx per-gene, per-tissue standard deviations.

## When NOT to use

- Your analysis uses the default GTEx inverse-normal-transformed expression values and does not require variance-standardized (SD-unit) effect sizes
- The gene x tissue pair fails the IHW alpha = 0.10 heritability significance filter, so its SDgeno estimate is not confidently heritable
- You need qualitative sex-specific exercise responses rather than sex-consistent standardized effect sizes (the article notes sex-specificity 'fell outside the scope considered here')

## Inputs

- GTEx v8 expression matrices (gene x sample per tissue)
- GTEx covariate table (sex, top 5 genotyping PCs, sequencing platform, sequencing protocol, GTEx-suggested number of PEER factors)
- 8-week sex-consistent exercise differential-expression log2FC estimates (MoTrPAC rat EET, mapped to human orthologs)
- PLINK binary genotype files for GTEx samples

## Outputs

- standardized_effect_sizes.csv with SDpheno- and SDgeno-standardized effect sizes per gene-tissue (~49,500 pairs after IHW alpha=0.10 heritability filter)
- Empirical-Bayes posterior mean log2-expression variances per gene x tissue (inverse-gamma conjugate prior, fitted separately per sex)
- GCTA-GRM h_SNP^2 estimates per gene's expression, bounded to [0, 1]

## How to apply

Load GTEx v8 expression matrices and modify the GTEx pipeline processing to apply a pseudolog log2(x+1) transform instead of the default inverse-normal transform; residualize the provided covariates (sex, top 5 genotyping PCs, sequencing platform, sequencing protocol, and GTEx-suggested number of PEER factors) with base-R lm(). Estimate per-gene, per-tissue expression variances pooled across sex, fit an inverse-gamma distribution to these variances separately per sex with fitdistrplus fitdist() maximum goodness-of-fit, and derive Empirical-Bayes posterior mean variances; the square root of the posterior mean gives SDpheno. Estimate SNP-heritability (h_SNP^2) of each gene's expression with Plink and GCTA-GRM using the same covariates, bounded between 0 and 1, then SDgeno = sqrt(h_SNP^2 * variance). Divide the 8-week sex-consistent exercise log2FCs by SDpheno and SDgeno, and threshold on IHW significance (alpha = 0.10, tissue as covariate) to retain confidently heritable gene x tissue pairs.

## Related tools

- **GTEx Pipeline** (Source of GTEx v8 expression processing whose default inverse-normal transform is replaced by a pseudolog log2(x+1) transform) — https://github.com/broadinstitute/gtex-pipeline/
- **fitdistrplus** (fitdist() maximum goodness-of-fit estimator for the inverse-gamma prior on per-gene, per-tissue expression variances)
- **Plink** (Genotype processing upstream of GCTA-GRM heritability estimation)
- **GCTA-GRM** (Estimates h_SNP^2 of each gene's expression using the same covariates, bounded between 0 and 1) — https://yanglab.westlake.edu.cn/software/gcta/#Overview

## Examples

```
resid <- residuals(lm(expr ~ sex + PC1 + PC2 + PC3 + PC4 + PC5 + platform + protocol + PEER, data = gtex_v8))
```

## Evaluation signals

- Output table contains approximately 49,500 gene x tissue pairs after the IHW alpha = 0.10 heritability filter (the filter removes ~92% of 632,738 pairs)
- Output includes both SDpheno- and SDgeno-standardized effect-size columns
- Illustrative values match reported ranges, e.g. APOB at roughly +9.7 SDpheno in male lung
- Counts of genes exceeding 2 SD match the paper: average 1 gene per tissue (range 0-10) vs 2 SD of genetic variability, and average 52 genes per tissue (range 1-586) vs 2 SD of overall variability
- h_SNP^2 estimates are bounded between 0 and 1, and posterior variances are fitted separately for male- and female-coded individuals

## Limitations

- The IHW alpha = 0.10 heritability filter discards ~92% of 632,738 gene x tissue pairs, so only confidently heritable genetically regulated expression is covered by SDgeno
- h_SNP^2 estimates are bounded between 0 and 1, which caps the genetic component of SDgeno
- GTEx samples are mostly European-descendant individuals, limiting portability of the human variability estimates to other populations
- Most GTEx v8 humans were aged 50+ while trained F344 rats were under eight months, so age is a confounder in the cross-species comparison

## Evidence

- [methods] we used the software Plink53 and GCTA54, specifically GCTA-GRM55: "we used the software Plink53 and GCTA54, specifically GCTA-GRM55"
- [methods] we used the GTEx database (version 8)34: "we used the GTEx database (version 8)34"
- [methods] we thresholded on significance (IHW α = 0.10, with tissue as a covariate) to focus on confidently heritable genetically regulated expression.: "we thresholded on significance (IHW α = 0.10, with tissue as a covariate) to focus on confidently heritable genetically regulated expression."
- [other] Load GTEx v8 expression matrices and modify the GTEx pipeline processing to apply a pseudolog (log2(x + 1)) transform in place of the default inverse-normal transform: [evidence span withheld — anchor status: not_found]
