---
name: empirical-bayes-posterior-variance-estimation
description: Use when standardizing exercise (or other condition-induced) log2 fold-changes
  against within-population expression variability at the gene-by-tissue level, e.g.
  asking how many genes' 8-week endurance-training effect sizes exceed 2 SD of GTEx
  expression variance.
when_to_use_negative:
- Expression data are raw counts or already inverse-normal transformed — the pipeline
  specifically requires the pseudolog log2(x + 1) scale for variance estimation
- The goal is TWAS/PrediXcan disease association rather than effect-size standardization
  against human expression variability (use S-PrediXcan/MESC workflows instead)
- Too few samples or genes per tissue to support fitting an inverse-gamma distribution
  to variances separately for male- and female-coded individuals
edam_topics:
- http://edamontology.org/topic_3489
- http://edamontology.org/topic_3308
tools:
- name: fitdistrplus
  role: fitdist() with the maximum goodness-of-fit (MGE) estimator fits the inverse-gamma
    distribution to pooled per-gene per-tissue expression variances
- name: GTEx Pipeline
  role: source of GTEx v8 expression processing whose inverse-normal transform is
    replaced with the pseudolog (log2(x + 1)) transform
  repo: https://github.com/broadinstitute/gtex-pipeline/
- name: base-R lm()
  role: residualizes covariates (sex, genotyping PCs, sequencing platform/protocol,
    PEER factors) from pseudolog-transformed expression
- name: GCTA
  role: GCTA-GRM estimates h_SNP^2 of each gene's expression (bounded 0-1) so variances
    can be converted to SDgeno units alongside SDpheno
  repo: https://yanglab.westlake.edu.cn/software/gcta/#Overview
- name: Plink
  role: prepares genotype input used together with GCTA-GRM for expression SNP-heritability
    estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:b4cbf2086a43bc660f9da72f9201fd64b07268c47d3ab30bdf71d89be5f7cffc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/gtex-pipeline
  tools:
  - GCTA
  - GCTA-GRM
  - GTEx Pipeline
  - Plink
  - base-R lm()
  - fitdistrplus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3489
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# empirical-bayes-posterior-variance-estimation

## Summary

Empirical-Bayes estimation of within-population phenotypic standard deviation (SDpheno) of GTEx log2 gene expression per gene-by-tissue pair: an inverse-gamma conjugate prior is fit to pooled sample variances, and exercise differential expression is then standardized in SDpheno units. This converts raw log2FC of exercise training-induced DE into effect sizes comparable to human expression variability.

## When to use

Use when standardizing exercise (or other condition-induced) log2 fold-changes against within-population expression variability at the gene-by-tissue level, e.g. asking how many genes' 8-week endurance-training effect sizes exceed 2 SD of GTEx expression variance.

## When NOT to use

- Expression data are raw counts or already inverse-normal transformed — the pipeline specifically requires the pseudolog log2(x + 1) scale for variance estimation
- The goal is TWAS/PrediXcan disease association rather than effect-size standardization against human expression variability (use S-PrediXcan/MESC workflows instead)
- Too few samples or genes per tissue to support fitting an inverse-gamma distribution to variances separately for male- and female-coded individuals

## Inputs

- GTEx v8 per-tissue expression matrices (raw counts transformed with pseudolog log2(x + 1))
- GTEx covariates: sex, top 5 genotyping principal components, sequencing platform, sequencing protocol, GTEx-suggested number of PEER factors
- 8-week sex-consistent exercise differential expression log2FC estimates (MoTrPAC rat EET, human-orthologous genes)

## Outputs

- Empirical-Bayes posterior mean log2-expression variances per gene x tissue (separately for male- and female-coded individuals)
- SDpheno = sqrt(posterior mean variance) per gene x tissue
- Exercise log2FC standardized in SDpheno units (and optionally SDgeno units via sqrt(h_SNP^2 x variance))
- standardized_effect_sizes.csv with SDpheno and SDgeno standardized columns per gene-tissue

## How to apply

Load GTEx v8 expression matrices and modify the GTEx pipeline processing to apply a pseudolog (log2(x + 1)) transform instead of the default inverse-normal transform, then residualize covariates (sex, top 5 genotyping principal components, sequencing platform, sequencing protocol, and the GTEx-suggested number of PEER factors) with base-R lm(). Compute per-gene, per-tissue sample variances pooled across sex, and fit an inverse-gamma distribution to these variances separately for male- and female-coded individuals using the fitdistrplus fitdist() maximum goodness-of-fit (MGE) estimator, exploiting the conjugacy of the inverse-gamma prior for Gaussian variance to derive Empirical-Bayes posterior mean expression variances. Take the square root of the posterior mean of log2-expression variance to obtain SDpheno, and divide the 8-week sex-consistent exercise log2FC estimates by SDpheno to produce standardized effect sizes (per gene-tissue). The same variance estimates can be combined with GCTA-GRM SNP-heritability (h_SNP^2, bounded 0-1) as sqrt(h_SNP^2 x variance) to also yield SDgeno units; confident heritable pairs are retained by thresholding on IHW significance (alpha = 0.10, tissue as covariate). Write both standardized columns to standardized_effect_sizes.csv.

## Related tools

- **fitdistrplus** (fitdist() with the maximum goodness-of-fit (MGE) estimator fits the inverse-gamma distribution to pooled per-gene per-tissue expression variances)
- **GTEx Pipeline** (source of GTEx v8 expression processing whose inverse-normal transform is replaced with the pseudolog (log2(x + 1)) transform) — https://github.com/broadinstitute/gtex-pipeline/
- **base-R lm()** (residualizes covariates (sex, genotyping PCs, sequencing platform/protocol, PEER factors) from pseudolog-transformed expression)
- **GCTA** (GCTA-GRM estimates h_SNP^2 of each gene's expression (bounded 0-1) so variances can be converted to SDgeno units alongside SDpheno) — https://yanglab.westlake.edu.cn/software/gcta/#Overview
- **Plink** (prepares genotype input used together with GCTA-GRM for expression SNP-heritability estimation)

## Examples

```
library(fitdistrplus); invgamma_fit <- fitdist(var_pooled_per_gene_tissue, distr = "invgamma", method = "mge")
```

## Evaluation signals

- standardized_effect_sizes.csv contains approximately 49,500 gene x tissue pairs after the IHW alpha = 0.10 heritability filter (removing ~92% of 632,738 candidate pairs)
- Output table includes both SDpheno and SDgeno standardized columns for every gene-tissue pair
- Posterior mean variances are finite and positive for both the male- and female-coded inverse-gamma fits; SDpheno is the square root of the posterior mean of log2-expression variance
- Illustrative values match reported figure ranges, e.g. APOB at roughly +9.7 SDpheno in male lung
- Count of genes exceeding 2 SDpheno per tissue falls in the reported range (average 52 genes/tissue, range 1-586)

## Limitations

- GTEx v8 donors are mostly aged 50+ and of European descent, limiting portability of the variance and standardized effect estimates to non-European populations and to younger populations
- The IHW alpha = 0.10 heritability filter discards ~92% of 632,738 gene-tissue pairs, so SDgeno-based standardization is only defined for the confidently heritable subset
- h_SNP^2 estimates are bounded between 0 and 1; genes with low heritability yield unstable SDgeno values even when the EB variance estimate is well fit

## Evidence

- [methods] Compute per-gene, per-tissue sample variances pooled across sex, fit an inverse-gamma distribution to these variances separately for male- and female-coded individuals using the fitdistrplus fitdist(): [evidence span withheld — anchor status: not_found]
- [methods] Take the square root of the posterior mean of log2-expression variance to obtain the within-population phenotypic standard deviation: [evidence span withheld — anchor status: not_found]
- [methods] we thresholded on significance (IHW α = 0.10, with tissue as a covariate) to focus on confidently heritable genetically regulated expression.: "we thresholded on significance (IHW α = 0.10, with tissue as a covariate) to focus on confidently heritable genetically regulated expression."
- [results] an average of 52 genes per tissue (range 1-586) exceeded 2 SD of overall expression variability: [evidence span withheld — anchor status: not_found]
- [discussion] most humans in the GTEx v8 dataset were aged 50+34,35: "most humans in the GTEx v8 dataset were aged 50+34,35 while trained F344 rats were uniformly under eight months of age"
