---
name: twas-deg-enrichment-intersection-analysis
description: Use when you have TWAS/PrediXcan gene-level association results and a
  set of condition-responsive DEGs, and need to quantify whether the DEGs are enriched
  (or depleted) among significant trait-tissue-gene associations with uncertainty
  quantification.
when_to_use_negative:
- When the DEG set lacks sex-consistent replicate states or genes have missing values
  across conditions (the z-score matrix was filtered to genes with no missing values
  across all conditions)
- When you need single-tissue, single-trait answers without partial pooling — a simple
  2×2 Fisher test suffices and the hierarchical model's correlation estimation (bivariate
  probit + nearPD) is unnecessary
- When the GWAS traits cannot be mapped to GTEx v8 tissues with sufficient sample
  overlap to estimate trait-level correlation matrices
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: S-PrediXcan
  role: Provides gene-by-tissue-level association estimates (TWAS hits) whose IHW-significant
    results form the enriched set being tested
  repo: https://github.com/hakyimlab/MetaXcan
- name: DESeq2
  role: Performs differential expression analysis (likelihood ratio tests per sex
    and tissue) producing the exercise-responsive DEGs being tested for enrichment
- name: CmdStanR
  role: R interface to Stan used to fit the hierarchical binomial enrichment model
    with non-centered parameterization
- name: posterior
  role: R package used to diagnose MCMC output (r̂ < 1.01, bulk/tail ESS > 500, divergences)
- name: fgsea
  role: Frequentist GSEA per trait×tissue pair using -log10(PrediXcan p-values) as
    ranking metric
- name: Matrix
  role: nearPD() projects estimated correlation matrices to the nearest positive semi-definite
    matrix (Higham's algorithm)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:0e6ce77f2618a2886588c3a1057e415e8a29c8364d7b31178eb86261cb819ba2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hakyimlab/MetaXcan
  tools:
  - CmdStanR
  - DESeq2
  - Matrix
  - PrediXcan
  - S-PrediXcan
  - Stan
  - fgsea
  - posterior
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# twas-deg-enrichment-intersection-analysis

## Summary

A hierarchical Bayesian partial-pooling enrichment model that tests whether exercise training-responsive genes (sex-consistent 8-week DEGs) are enriched for IHW-significant S-PrediXcan TWAS hits across tissues, traits, and trait-by-tissue cells, complemented by frequentist fgsea GSEA per trait×tissue pair.

## When to use

Use when you have TWAS/PrediXcan gene-level association results and a set of condition-responsive DEGs, and need to quantify whether the DEGs are enriched (or depleted) among significant trait-tissue-gene associations with uncertainty quantification.

## When NOT to use

- When the DEG set lacks sex-consistent replicate states or genes have missing values across conditions (the z-score matrix was filtered to genes with no missing values across all conditions)
- When you need single-tissue, single-trait answers without partial pooling — a simple 2×2 Fisher test suffices and the hierarchical model's correlation estimation (bivariate probit + nearPD) is unnecessary
- When the GWAS traits cannot be mapped to GTEx v8 tissues with sufficient sample overlap to estimate trait-level correlation matrices

## Inputs

- S-PrediXcan gene-by-tissue association outputs filtered at IHW α = 0.05 (tissue×trait pairs as covariate)
- Sex-consistent 8-week DEG sets (repfdr EM states 8w_F1_M1 up-regulated and 8w_F-1_M-1 down-regulated in both sexes)
- Tissue and trait metadata mapping 114 GWAS traits to GTEx v8 tissues and trait categories (e.g., Endocrine, Cardiometabolic)
- Gene-wise correlation structure inputs for bivariate probit estimation (gene expression/GWAS directionality per cell)

## Outputs

- Posterior summary tables (CSV) for tissue (βi), trait (γj), and trait-category (μk) difference effects
- Tissue×trait cell enrichment flags: > 95% posterior mass on one side of 0 marked enriched (red) or depleted (blue)
- fgsea GSEA results per trait×tissue pair with Bonferroni-adjusted p-values and harmonic-mean aggregated subtest p-values
- MCMC diagnostics summary (r̂, bulk/tail ESS, divergent iteration counts)

## How to apply

Load S-PrediXcan outputs filtered at IHW α = 0.05 with tissue×trait pairs as covariate and the sex-consistent 8W DEG sets (repfdr nodes 8w_F1_M1 and 8w_F-1_M-1), then compute intersect counts per tissue×trait cell for the 99 of 114 traits with nonzero intersect. Estimate tissue (Σi), trait (Σj), and tissue×trait (Σi×j) gene-wise correlation matrices under a bivariate probit via the nlm algorithm (stats/optimx), projecting each to the nearest positive semi-definite matrix with Matrix::nearPD() (Higham's algorithm). Fit the hierarchical binomial model in Stan via CmdStanR using a non-centered parameterization with 4 chains (2.5×10³ warmup and 2.5×10³ sampling iterations each, δ = 0.95), then extract marginal posteriors for βi, γj, and trait-category μk effects, flagging tissue×trait cells with > 95% posterior mass on one side of 0 as enriched (red) or depleted (blue). In parallel, run frequentist fgsea() per trait×tissue pair using -log10(PrediXcan p-values) as the ranking metric with Bonferroni correction, aggregating interdependent subtests by harmonic-mean p-values at FWER α = 0.05.

## Related tools

- **S-PrediXcan** (Provides gene-by-tissue-level association estimates (TWAS hits) whose IHW-significant results form the enriched set being tested) — https://github.com/hakyimlab/MetaXcan
- **DESeq2** (Performs differential expression analysis (likelihood ratio tests per sex and tissue) producing the exercise-responsive DEGs being tested for enrichment)
- **CmdStanR** (R interface to Stan used to fit the hierarchical binomial enrichment model with non-centered parameterization)
- **posterior** (R package used to diagnose MCMC output (r̂ < 1.01, bulk/tail ESS > 500, divergences))
- **fgsea** (Frequentist GSEA per trait×tissue pair using -log10(PrediXcan p-values) as ranking metric)
- **Matrix** (nearPD() projects estimated correlation matrices to the nearest positive semi-definite matrix (Higham's algorithm))

## Examples

```
fit <- model$sample(data = stan_data, chains = 4, iter_warmup = 2500, iter_sampling = 2500, adapt_delta = 0.95)
```

## Evaluation signals

- All Stan parameters pass r̂ < 1.01, bulk and tail ESS > 500, and < 0.05% divergent iterations
- Posterior summaries reproduce the reported confident (>95% posterior probability) positive enrichments in colon, kidneys, small intestines, spleen, hippocampus, lungs, and heart (in decreasing posterior mean), plus Endocrine and Cardiometabolic trait categories
- fgsea reproduces the high-cholesterol test result (ES ≈ 0.63, adjusted p ≈ 0.064) and harmonic-mean aggregated subtests pass FWER α = 0.05
- Intersect counts are computable for 99 of 114 traits (nonzero intersect per tissue×trait cell)
- All three correlation matrices (Σi, Σj, Σi×j) project to valid positive semi-definite matrices via nearPD() without failure

## Limitations

- Effect sizes are small, predominately within 0 ± 0.3 on the log-odds scale (≈7.5% maximum difference on the probability scale)
- Some marginal posterior distributions for difference parameters (e.g., for asthma) did not reach the 95% posterior mass threshold
- GTEx sample is mostly European-descendant individuals, limiting portability, and GTEx humans (aged 50+) vs trained F344 rats (under eight months) create age and species mismatches in interpreting enrichment

## Evidence

- [results] The hierarchical Bayesian enrichment model identified confident (>95% posterior probability) positive enrichments in the colon, kidneys, small intestines, spleen, hippocampus, lungs, and heart: [evidence span withheld — anchor status: not_found]
- [methods] filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate): "filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate)"
- [discussion] predominately varying within 0 ± 0.3 on the log-odds scale ... This corresponds to a maximum difference of ≈7.5% on the probability scale: [evidence span withheld — anchor status: not_found]
- [intro] we leverage published S-PrediXcan12,13 output, which estimates gene by tissue-level associations and directions of effect for specific diseases: [evidence span withheld — anchor status: not_found]
- [readme] S-PrediXcan is an extension that infers PrediXcan's results using only summary statistics, implemented in SPrediXcan.py: [evidence span withheld — anchor status: not_found]
