---
name: exercise-de-effect-size-standardization
description: Use when you need to express exercise training-induced differential expression
  effect sizes in units of human phenotypic or SNP-heritable expression variability
  (GTEx v8) to identify gene x tissue pairs where exercise can overcome standing or
  genetic variation.
when_to_use_negative:
- When the goal is disease-trait association or enrichment of exercise-responsive
  genes (e.g. LDSC/MESC heritability enrichment or S-PrediXcan TWAS) rather than standardizing
  DE effect sizes against expression variability.
- When gene x tissue pairs cannot pass the IHW alpha = 0.10 heritability filter, since
  unconfidently heritable pairs (~92% of the 632,738 tested) are deliberately removed.
- When comparing across populations other than GTEx (results have limited portability
  to non-European populations, as the GTEx sample comprises mostly European-descendant
  individuals).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: GTEx Pipeline
  role: Source of expression processing pipeline, modified to use a pseudolog (log2(x+1))
    transform in place of the default inverse-normal transform before covariate residualization
  repo: https://github.com/broadinstitute/gtex-pipeline/
- name: fitdistrplus
  role: Fits inverse-gamma distribution to per-gene, per-tissue expression variances
    via fitdist() maximum goodness-of-fit estimation for the Empirical-Bayes prior
- name: Plink
  role: Used with GCTA to estimate SNP-heritability of each gene's expression from
    GTEx genotypes and covariates
- name: GCTA-GRM
  role: GCTA (1.93.2) genomic-relatedness-matrix estimator of h_SNP^2 of each gene's
    expression, bounded between 0 and 1
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:3834f486cecff20854cde20dbc979649e3fe4ba943dab0e39d61895258ff6374
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# exercise-de-effect-size-standardization

## Summary

Standardizes 8-week sex-consistent exercise log2FC estimates from MoTrPAC rat DE into SDpheno (within-population phenotypic SD) and SDgeno (genetic component of expression SD) units, enabling comparison of exercise effects against GTEx human expression variability at the gene-by-tissue level.

## When to use

Use when you need to express exercise training-induced differential expression effect sizes in units of human phenotypic or SNP-heritable expression variability (GTEx v8) to identify gene x tissue pairs where exercise can overcome standing or genetic variation.

## When NOT to use

- When the goal is disease-trait association or enrichment of exercise-responsive genes (e.g. LDSC/MESC heritability enrichment or S-PrediXcan TWAS) rather than standardizing DE effect sizes against expression variability.
- When gene x tissue pairs cannot pass the IHW alpha = 0.10 heritability filter, since unconfidently heritable pairs (~92% of the 632,738 tested) are deliberately removed.
- When comparing across populations other than GTEx (results have limited portability to non-European populations, as the GTEx sample comprises mostly European-descendant individuals).

## Inputs

- GTEx v8 expression matrices (per tissue)
- GTEx v8 covariates: sex, top 5 genotyping principal components, sequencing platform, sequencing protocol, GTEx-suggested number of PEER factors
- 8-week sex-consistent exercise DE log2FC estimates from the MoTrPAC rat endurance exercise training (EET) study (PASS1B)
- Genotype data for GTEx samples (for GCTA-GRM h_SNP^2 estimation)

## Outputs

- standardized_effect_sizes.csv containing per gene-tissue SDpheno- and SDgeno-standardized exercise effect sizes
- Per-gene, per-tissue posterior mean expression variances (inverse-gamma Empirical-Bayes) and within-population phenotypic SDs
- Per-gene, per-tissue h_SNP^2 estimates bounded between 0 and 1

## How to apply

Load GTEx v8 expression matrices, replace the GTEx pipeline's default inverse-normal transform with a pseudolog (log2(x + 1)) transform, and residualize covariates (sex, top 5 genotyping PCs, sequencing platform, sequencing protocol, and GTEx-suggested number of PEER factors) using base-R lm(). Compute per-gene, per-tissue sample variances pooled across sex, fit an inverse-gamma distribution to these variances separately for male- and female-coded individuals with the fitdistrplus fitdist() maximum goodness-of-fit estimator, and derive Empirical-Bayes posterior mean expression variances from the conjugate prior; the square root of the posterior mean yields SDpheno. Estimate SNP-heritability (h_SNP^2) of each gene's expression with Plink and GCTA-GRM (1.93.2) using the same covariates, bounding estimates between 0 and 1, then compute SDgeno = sqrt(h_SNP^2 * expression variance). Divide the 8-week sex-consistent exercise log2FC estimates by SDpheno and SDgeno, threshold on IHW significance (alpha = 0.10, with tissue as covariate) to retain confidently heritable gene x tissue pairs, and write standardized_effect_sizes.csv.

## Related tools

- **GTEx Pipeline** (Source of expression processing pipeline, modified to use a pseudolog (log2(x+1)) transform in place of the default inverse-normal transform before covariate residualization) — https://github.com/broadinstitute/gtex-pipeline/
- **fitdistrplus** (Fits inverse-gamma distribution to per-gene, per-tissue expression variances via fitdist() maximum goodness-of-fit estimation for the Empirical-Bayes prior)
- **Plink** (Used with GCTA to estimate SNP-heritability of each gene's expression from GTEx genotypes and covariates)
- **GCTA-GRM** (GCTA (1.93.2) genomic-relatedness-matrix estimator of h_SNP^2 of each gene's expression, bounded between 0 and 1)

## Evaluation signals

- Output table contains approximately 49,500 gene x tissue pairs after the IHW alpha = 0.10 heritability filter (removing ~92% of 632,738 pairs).
- Output includes both SDpheno and SDgeno standardized columns per gene-tissue pair.
- Illustrative values match reported figure ranges, e.g. APOB at roughly +9.7 SDpheno in male lung.
- Per-tissue counts of genes with exercise effect sizes exceeding 2 SDpheno average ~52 (range 1-586); counts exceeding 2 SDgeno average ~1 (range 0-10) at 8 weeks.
- All h_SNP^2 estimates are bounded between 0 and 1.

## Limitations

- GTEx v8 sample comprises mostly European-descendant individuals, so results have limited portability to non-European populations.
- Qualitative sex-specificity of exercise adaptation fell outside the scope of this study; standardization uses sex-consistent 8-week effects and sex-pooled (or sex-coded) variance fitting.
- The SDgeno comparison depends on confidently heritable gene x tissue pairs; genes with low h_SNP^2 are excluded by the IHW alpha = 0.10 filter.

## Evidence

- [methods] Pseudolog transform and residualization of covariates: [evidence span withheld — anchor status: not_found]
- [methods] Heritability significance filter: "we thresholded on significance (IHW α = 0.10, with tissue as a covariate) to focus on confidently heritable genetically regulated expression."
- [readme] Repository identity and publication: "This is the GitHub repository associated with the publication:"
