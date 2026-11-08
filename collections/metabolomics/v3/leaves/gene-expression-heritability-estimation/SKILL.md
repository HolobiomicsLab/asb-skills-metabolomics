---
name: gene-expression-heritability-estimation
description: Use when standardizing cross-species (rat-to-human) differential expression
  effect sizes against GTEx v8 per-gene, per-tissue expression variability and its
  genetic component via h_SNP^2.
when_to_use_negative:
- The analysis goal is trait heritability enrichment or mediated heritability from
  GWAS summary statistics — use LDSC or MESC instead; this skill operates on individual-level
  GTEx expression and genotype data per gene x tissue.
- The expression data have not been re-processed with the pseudolog transform — the
  default GTEx inverse-normal transform removes the variance scale needed to compute
  meaningful SDpheno/SDgeno units.
- Tissue-specificity or cross-tissue exercise adaptation patterns are not needed;
  a simpler DE analysis (DESeq2 per sex and tissue) suffices without the heritability
  machinery.
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: Plink
  role: Genotype data preparation feeding GCTA-GRM for heritability estimation of
    each gene's expression
- name: GCTA-GRM (GCTA 1.93.2)
  role: Estimates h_SNP^2 of each gene's expression per gene x tissue, with estimates
    bounded between 0 and 1
  repo: https://yanglab.westlake.edu.cn/software/gcta/#Overview
- name: GTEx Pipeline
  role: Base expression processing whose default inverse-normal transform is replaced
    with a pseudolog log2(x+1) transform
  repo: https://github.com/broadinstitute/gtex-pipeline/
- name: fitdistrplus
  role: Maximum goodness-of-fit estimation of inverse-gamma priors on per-gene, per-tissue
    expression variances for the Empirical-Bayes posterior mean variance
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:762fbbeafa962ed1fcdbb00ce04f66dfdb5a7e7a523bb52bb8334660d3d7101b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCTA
  - GCTA-GRM
  - GCTA-GRM (GCTA 1.93.2)
  - GTEx Pipeline
  - Plink
  - fitdistrplus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-expression-heritability-estimation

## Summary

Estimates the SNP-heritability (h_SNP^2) of each gene's expression at the gene-by-tissue level in GTEx v8 using GCTA-GRM, then combines it with Empirical-Bayes posterior expression variances to standardize exercise-induced log2FC effect sizes in SDgeno and SDpheno units. This identifies where training-induced expression changes can exceed genetic or standing human expression variability.

## When to use

Use when standardizing cross-species (rat-to-human) differential expression effect sizes against GTEx v8 per-gene, per-tissue expression variability and its genetic component via h_SNP^2.

## When NOT to use

- The analysis goal is trait heritability enrichment or mediated heritability from GWAS summary statistics — use LDSC or MESC instead; this skill operates on individual-level GTEx expression and genotype data per gene x tissue.
- The expression data have not been re-processed with the pseudolog transform — the default GTEx inverse-normal transform removes the variance scale needed to compute meaningful SDpheno/SDgeno units.
- Tissue-specificity or cross-tissue exercise adaptation patterns are not needed; a simpler DE analysis (DESeq2 per sex and tissue) suffices without the heritability machinery.

## Inputs

- GTEx v8 expression matrices (per tissue, in raw/processed form suitable for re-processing)
- GTEx covariates: sex, top 5 genotyping principal components, sequencing platform, sequencing protocol, GTEx-suggested number of PEER factors
- 8-week sex-consistent exercise log2FC differential expression estimates from MoTrPAC rat EET data (mapped to human orthologs)

## Outputs

- standardized_effect_sizes.csv containing ~49,500 gene x tissue pairs with SDpheno- and SDgeno-standardized exercise effect sizes
- Per-gene, per-tissue h_SNP^2 estimates (bounded between 0 and 1) from GCTA-GRM
- Empirical-Bayes posterior mean log2-expression variances and derived within-population phenotypic SDs per gene x tissue

## How to apply

Modify the GTEx v8 pipeline processing to apply a pseudolog log2(x+1) transform instead of the default inverse-normal transform, then residualize covariates (sex, top 5 genotyping PCs, sequencing platform, sequencing protocol, and the GTEx-suggested number of PEER factors) with base-R lm(). Estimate per-gene, per-tissue pooled sample variances, fit inverse-gamma priors with fitdistrplus fitdist() separately for male- and female-coded individuals, and derive Empirical-Bayes posterior mean variances; their square root is the phenotypic SD (SDpheno). Estimate h_SNP^2 of each gene's expression with Plink and GCTA, specifically GCTA-GRM, using the same covariates, bounding estimates between 0 and 1; the genetic SD is sqrt(h_SNP^2 * expression variance) (SDgeno). Divide the 8-week sex-consistent exercise log2FC estimates by SDpheno and SDgeno to obtain standardized effect sizes, and threshold on IHW significance (alpha = 0.10, with tissue as a covariate) to retain confidently heritable gene x tissue pairs.

## Related tools

- **Plink** (Genotype data preparation feeding GCTA-GRM for heritability estimation of each gene's expression)
- **GCTA-GRM (GCTA 1.93.2)** (Estimates h_SNP^2 of each gene's expression per gene x tissue, with estimates bounded between 0 and 1) — https://yanglab.westlake.edu.cn/software/gcta/#Overview
- **GTEx Pipeline** (Base expression processing whose default inverse-normal transform is replaced with a pseudolog log2(x+1) transform) — https://github.com/broadinstitute/gtex-pipeline/
- **fitdistrplus** (Maximum goodness-of-fit estimation of inverse-gamma priors on per-gene, per-tissue expression variances for the Empirical-Bayes posterior mean variance)

## Examples

```
fitdistrplus::fitdist(pooled_vars, distr = "invgamma", method = "mge")
```

## Evaluation signals

- After the IHW alpha = 0.10 heritability filter (tissue as covariate), the output contains approximately 49,500 gene x tissue pairs out of the original 632,738 (~92% removed).
- All h_SNP^2 estimates are bounded within [0, 1].
- The standardized_effect_sizes.csv table includes both SDpheno- and SDgeno-standardized columns.
- Illustrative values match the reported figure ranges: e.g., APOB at roughly +9.7 SDpheno in male lung; average of 52 genes per tissue exceeding 2 SDpheno (range 1-586).
- The output uses residualized log2(x+1) expression rather than inverse-normal-transformed GTEx expression, verifiable by re-computing variances from the processed matrices.

## Limitations

- Portability is limited to European-descendant populations because the GTEx sample comprises mostly European-descendant individuals.
- Age mismatch between GTEx humans (aged 50+) and trained F344 rats (under eight months) limits comparison of exercise-independent age and sex effects.
- The IHW alpha = 0.10 heritability filter discards ~92% of gene x tissue pairs, so standardized effect sizes are only available for confidently heritable genetically regulated expression.

## Evidence

- [methods] GCTA-GRM heritability estimation: [evidence span withheld — anchor status: not_found]
- [methods] IHW heritability filter: "we thresholded on significance (IHW α = 0.10, with tissue as a covariate) to focus on confidently heritable genetically regulated expression."
- [methods] hSNP2 bounding and covariates: [evidence span withheld — anchor status: not_found]
- [readme] GTEx pipeline role: "This repository contains analysis pipelines for:

* RNA-seq alignment, quantification, and quality control"
- [intro] MESC contrast (not this skill's method): [evidence span withheld — anchor status: not_found]
