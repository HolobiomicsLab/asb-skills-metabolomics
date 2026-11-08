---
name: gene-set-enrichment-analysis
description: Use when testing whether a defined gene set (e.g., exercise-responsive
  DEGs) is enriched for trait/tissue associations in TWAS/S-PrediXcan results across
  many tissue-by-trait cells, with partial pooling and direction-of-effect testing.
when_to_use_negative:
- The question concerns SNP-level heritability enrichment rather than gene-level TWAS
  hit enrichment — use LDSC or MESC instead, which accommodate linkage disequilibrium
  and eQTL-mediated heritability.
- Few or no tissue-by-trait cells have nonzero DEG-PrediXcan intersect counts, leaving
  the hierarchical binomial model underpowered to partial-pool.
- The gene set lacks sex-consistent state assignments (no repfdr F1_M1 / F-1_M-1 nodes),
  so direction-of-effect enrichment cannot be defined as in this workflow.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CmdStanR
  role: Interface to fit the hierarchical binomial partial-pooling enrichment model
    in Stan (4 chains, 2500 warmup/2500 sampling, delta = 0.95, non-centered parameterization).
- name: posterior
  role: 'Diagnose MCMC output: r-hat < 1.01, bulk and tail ESS > 500, < 0.05% divergent
    iterations; extract marginal posterior distributions of difference effects.'
- name: fgsea
  role: Frequentist GSEA per trait-by-tissue pair ranked by -log10(PrediXcan p-values)
    with Bonferroni correction and harmonic-mean aggregation of subtests.
- name: Matrix
  role: Project tissue, trait, and tissue-by-trait correlation matrices to the nearest
    PSD matrix via Matrix::nearPD() (Higham's algorithm) before Stan fitting.
- name: S-PrediXcan
  role: Source of gene-by-tissue trait association p-values and effect directions
    that define the enrichment hit sets per tissue-by-trait cell.
  repo: https://github.com/hakyimlab/MetaXcan
- name: repfdr
  role: EM assignment of genes to sex-consistent DEG states (F1_M1, F-1_M-1) that
    constitute the gene sets tested for enrichment.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:4e2382408886eb28cd50b6486c93d9af1b3ef48e921e852ae225015daabb7672
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
  - repfdr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gene-set-enrichment-analysis

## Summary

Tests whether exercise training-responsive gene sets (e.g., sex-consistent 8-week DEGs) are over- or under-represented among trait-associated genes from S-PrediXcan output, using both a hierarchical Bayesian partial-pooling binomial model in Stan and frequentist fgsea() per trait-by-tissue pair.

## When to use

Use when testing whether a defined gene set (e.g., exercise-responsive DEGs) is enriched for trait/tissue associations in TWAS/S-PrediXcan results across many tissue-by-trait cells, with partial pooling and direction-of-effect testing.

## When NOT to use

- The question concerns SNP-level heritability enrichment rather than gene-level TWAS hit enrichment — use LDSC or MESC instead, which accommodate linkage disequilibrium and eQTL-mediated heritability.
- Few or no tissue-by-trait cells have nonzero DEG-PrediXcan intersect counts, leaving the hierarchical binomial model underpowered to partial-pool.
- The gene set lacks sex-consistent state assignments (no repfdr F1_M1 / F-1_M-1 nodes), so direction-of-effect enrichment cannot be defined as in this workflow.

## Inputs

- S-PrediXcan gene-by-trait association outputs filtered at IHW alpha = 0.05 with tissue x trait pairs as covariate (99 of 114 traits with nonzero DEG intersect)
- Sex-consistent 8-week DEG sets from repfdr EM states 8w_F1_M1 (up in both sexes) and 8w_F-1_M-1 (down in both sexes)
- Trait-to-category mappings (e.g., Endocrine, Cardiometabolic) for the 114 GTEx/GWAS traits

## Outputs

- Posterior summary CSV tables for tissue (beta_i), trait (gamma_j), and trait-category (mu_k) difference effects
- Tissue-by-trait enrichment flags: cells with >95% posterior mass on one side of 0 marked enriched (red) or depleted (blue)
- Per-pair fgsea() results with Bonferroni-adjusted p-values and harmonic-mean aggregated subtest p-values at FWER alpha = 0.05

## How to apply

Load S-PrediXcan outputs filtered at IHW alpha = 0.05 (with tissue-by-trait pairs as covariate) and the sex-consistent 8W DEG sets (repfdr nodes 8w_F1_M1 and 8w_F-1_M-1), then compute intersect counts per tissue-by-trait cell for the 99 of 114 traits with nonzero intersect. Estimate tissue, trait, and tissue-by-trait gene-wise correlation matrices under a bivariate probit via the nlm algorithm, projecting each to the nearest positive semi-definite matrix with Higham's algorithm (Matrix::nearPD()). Fit the hierarchical binomial model in Stan via CmdStanR with a non-centered parameterization: 4 randomly initialized chains, 2.5x10^3 warmup and 2.5x10^3 sampling iterations each, and target acceptance rate delta = 0.95. Extract marginal posteriors for tissue (beta_i), trait (gamma_j), and trait-category (mu_k) difference effects, flagging cells with >95% posterior mass on one side of 0 as enriched (red) or depleted (blue). As a frequentist complement, run fgsea() per trait-by-tissue pair using -log10(PrediXcan p-values) as the ranking metric with Bonferroni correction, aggregating interdependent subtests by harmonic-mean p-values at FWER alpha = 0.05.

## Related tools

- **CmdStanR** (Interface to fit the hierarchical binomial partial-pooling enrichment model in Stan (4 chains, 2500 warmup/2500 sampling, delta = 0.95, non-centered parameterization).)
- **posterior** (Diagnose MCMC output: r-hat < 1.01, bulk and tail ESS > 500, < 0.05% divergent iterations; extract marginal posterior distributions of difference effects.)
- **fgsea** (Frequentist GSEA per trait-by-tissue pair ranked by -log10(PrediXcan p-values) with Bonferroni correction and harmonic-mean aggregation of subtests.)
- **Matrix** (Project tissue, trait, and tissue-by-trait correlation matrices to the nearest PSD matrix via Matrix::nearPD() (Higham's algorithm) before Stan fitting.)
- **S-PrediXcan** (Source of gene-by-tissue trait association p-values and effect directions that define the enrichment hit sets per tissue-by-trait cell.) — https://github.com/hakyimlab/MetaXcan
- **repfdr** (EM assignment of genes to sex-consistent DEG states (F1_M1, F-1_M-1) that constitute the gene sets tested for enrichment.)

## Examples

```
fgseaRes <- fgsea(pathways = trait_tissue_gene_sets, stats = -log10(spredixcan_hits$p_value))
```

## Evaluation signals

- All Stan parameters pass r-hat < 1.01, bulk and tail ESS > 500, and < 0.05% divergent iterations under the posterior package diagnostics.
- Posterior summaries reproduce the reported confident (>95% posterior probability) positive enrichments: colon, kidneys, small intestines, spleen, hippocampus, lungs, heart (in decreasing posterior mean), plus Endocrine and Cardiometabolic trait categories.
- Effect sizes fall predominately within 0 +/- 0.3 on the log-odds scale (maximum difference of about 7.5% on the probability scale).
- Frequentist GSEA reproduces the high-cholesterol test result: enrichment score ~0.63 and harmonic-mean adjusted p ~0.064 at FWER alpha = 0.05.
- Intersect counts are computed only for the 99 of 114 traits with nonzero overlap between IHW-filtered PrediXcan hits and the 8W sex-consistent DEG sets.

## Limitations

- Enrichment/depletion effect sizes were small, predominately within 0 +/- 0.3 on the log-odds scale, so confident detections require large counts per tissue-by-trait cell.
- For asthma, no individual tissues were found to be confidently enriched in positive effects — no marginal posterior for difference parameters there reached the 95% posterior mass threshold.
- Results depend on GTEx v8 (mostly European-descendant, aged 50+) being linked to rat MoTrPAC EET DEGs, limiting portability and cross-species interpretation.

## Evidence

- [methods] filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate): "filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate)"
- [intro] we leveraged published S-PrediXcan output: "we leveraged published S-PrediXcan12,13 output, which estimates gene by tissue-level associations and directions of effect for specific diseases"
- [discussion] Enrichment/depletion effect sizes were small: [evidence span withheld — anchor status: not_found]
- [methods] repfdr EM state assignment defines the gene sets: "the expectation-maximization (EM) process of the repfdr algorithm was used to assign one of three simplified states to each z-score"
- [full_text] fgsea ranking metric and Bonferroni correction: [evidence span withheld — anchor status: not_found]
