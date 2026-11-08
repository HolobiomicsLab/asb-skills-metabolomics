---
name: inverse-gamma-variance-regularization
description: Use when per-gene expression variances estimated from limited samples
  (e.g., GTEx v8 tissue subsets) are too noisy to standardize effect sizes, and you
  need Empirical-Bayes posterior mean variances under an inverse-gamma conjugate prior.
when_to_use_negative:
- When expression values have already been inverse-normal transformed (the GTEx default),
  since variance shrinkage on that scale is not comparable to the pseudolog log2(x+1)
  scale used to standardize exercise log2FC
- When the goal is significance testing of differential expression rather than effect-size
  standardization — DESeq2 likelihood ratio tests with 5% IHW alpha are the article's
  tool for DEA
- When sample sizes per gene x tissue are large enough that raw pooled variances are
  stable and Empirical-Bayes shrinkage adds little
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: fitdistrplus
  role: fitdist() maximum goodness-of-fit estimator used to fit the inverse-gamma
    distribution to per-gene, per-tissue expression variances, separately for male-
    and female-coded individuals
- name: GTEx Pipeline
  role: source of expression processing whose default inverse-normal transform is
    replaced with a pseudolog (log2(x + 1)) transform before variance estimation
  repo: https://github.com/broadinstitute/gtex-pipeline/
- name: GCTA-GRM
  role: estimates h_SNP^2 of each gene's expression (via Plink and GCTA) used to build
    the genetic denominator SDgeno = sqrt(h_SNP^2 x variance)
- name: Plink
  role: used alongside GCTA-GRM for genetic relatedness/heritability estimation of
    expression traits
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:333cf834f79d341801e9f8962606abc8af9bfad9228fda658d698158d0e4e2d9
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# inverse-gamma-variance-regularization

## Summary

An Empirical-Bayes shrinkage procedure that regularizes per-gene, per-tissue gene expression variances by fitting an inverse-gamma conjugate prior (separately per sex) and using the posterior mean variance as the denominator for standardized exercise effect sizes (SDpheno units) in the MoTrPAC/GTEx analysis.

## When to use

Use when per-gene expression variances estimated from limited samples (e.g., GTEx v8 tissue subsets) are too noisy to standardize effect sizes, and you need Empirical-Bayes posterior mean variances under an inverse-gamma conjugate prior.

## When NOT to use

- When expression values have already been inverse-normal transformed (the GTEx default), since variance shrinkage on that scale is not comparable to the pseudolog log2(x+1) scale used to standardize exercise log2FC
- When the goal is significance testing of differential expression rather than effect-size standardization — DESeq2 likelihood ratio tests with 5% IHW alpha are the article's tool for DEA
- When sample sizes per gene x tissue are large enough that raw pooled variances are stable and Empirical-Bayes shrinkage adds little

## Inputs

- GTEx v8 per-tissue gene expression matrices (raw counts or TPM, prior to inverse-normal transform)
- GTEx covariates table (sex, top 5 genotyping PCs, sequencing platform, sequencing protocol, GTEx-suggested number of PEER factors)
- 8-week sex-consistent exercise log2FC estimates (from DESeq2 differential expression in MoTrPAC rat tissues)
- Per-gene, per-tissue sample variances pooled across sex, computed separately for male- and female-coded individuals

## Outputs

- Inverse-gamma prior parameter estimates (male and female) from fitdist() maximum goodness-of-fit
- Empirical-Bayes posterior mean log2-expression variances and derived within-population phenotypic SD (SDpheno) per gene x tissue
- SDpheno-standardized and SDgeno-standardized exercise effect sizes written to standardized_effect_sizes.csv (~49,500 heritable gene x tissue pairs after IHW alpha = 0.10 filtering)

## How to apply

First transform expression with a pseudolog (log2(x + 1)) instead of the GTEx default inverse-normal transform, then residualize covariates (sex, top 5 genotyping PCs, sequencing platform, sequencing protocol, and the GTEx-suggested number of PEER factors) with base-R lm(). Compute per-gene, per-tissue sample variances pooled across sex, and fit an inverse-gamma distribution to these variances separately for male- and female-coded individuals using fitdistrplus's fitdist() maximum goodness-of-fit estimator. Take the square root of the Empirical-Bayes posterior mean of the log2-expression variance to get the within-population phenotypic standard deviation (SDpheno), and divide the 8-week sex-consistent exercise log2FC estimates by SDpheno to produce standardized effect sizes. A parallel genetic denominator (SDgeno = sqrt(h_SNP^2 x variance)) is computed with GCTA-GRM heritability estimates and IHW alpha = 0.10 filtering.

## Related tools

- **fitdistrplus** (fitdist() maximum goodness-of-fit estimator used to fit the inverse-gamma distribution to per-gene, per-tissue expression variances, separately for male- and female-coded individuals)
- **GTEx Pipeline** (source of expression processing whose default inverse-normal transform is replaced with a pseudolog (log2(x + 1)) transform before variance estimation) — https://github.com/broadinstitute/gtex-pipeline/
- **GCTA-GRM** (estimates h_SNP^2 of each gene's expression (via Plink and GCTA) used to build the genetic denominator SDgeno = sqrt(h_SNP^2 x variance))
- **Plink** (used alongside GCTA-GRM for genetic relatedness/heritability estimation of expression traits)

## Examples

```
library(fitdistrplus); ig <- fitdist(v_gene_tissue, distr="invgamma", method="mge")
```

## Evaluation signals

- Output table standardized_effect_sizes.csv contains approximately 49,500 gene x tissue pairs after the IHW alpha = 0.10 heritability filter (removing ~92% of the original 632,738 pairs)
- Output includes both SDpheno- and SDgeno-standardized effect size columns for each gene-tissue pair
- Illustrative values match reported ranges, e.g., APOB at roughly +9.7 SDpheno in male lung
- Gene counts exceeding 2 SDpheno fall in the reported range of 1-586 per tissue (average 52), and counts exceeding 2 SDgeno in the range 0-10 (average 1)
- Inverse-gamma fits converge separately for male- and female-coded individuals via the fitdistrplus maximum goodness-of-fit estimator without errors

## Limitations

- Results may have limited portability to non-European populations, as the GTEx sample comprises mostly European-descendant individuals
- Age mismatch (GTEx humans mostly aged 50+ vs trained F344 rats uniformly under eight months) limits comparison of exercise-independent age and sex effects
- Variance shrinkage is applied per sex (separate inverse-gamma fits for male- and female-coded individuals), so sex-specific variance estimates depend on the conjugate prior assumption holding within each sex

## Evidence

- [other] fit inverse-gamma distribution to pooled expression variances: [evidence span withheld — anchor status: not_found]
- [other] pseudolog transform replaces inverse-normal: [evidence span withheld — anchor status: not_found]
- [methods] GCTA-GRM heritability of expression: [evidence span withheld — anchor status: not_found]
- [methods] IHW heritability significance threshold: "we thresholded on significance (IHW α = 0.10, with tissue as a covariate) to focus on confidently heritable genetically regulated expression."
