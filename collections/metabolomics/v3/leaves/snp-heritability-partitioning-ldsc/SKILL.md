---
name: snp-heritability-partitioning-ldsc
description: Use when GWAS summary statistics (here 114 published traits/diseases)
  must be tested for SNP-heritability enrichment in predefined gene sets — e.g., tissue-specific
  DE genes from exercise training — via LD score regression rather than simple overlap
  tests.
when_to_use_negative:
- The goal is expression-mediated heritability (proportion of hSNP2 mediated by gene
  expression) — use MESC with GWAS + eQTL summary statistics instead of LDSC enrichment
- The question is gene-level trait association with direction of effect rather than
  heritability enrichment — S-PrediXcan/TWAS is the appropriate approach
- Gene sets are not tissue/gene-set stratified annotations of common variants with
  appropriate LD Scores (e.g., rare-variant or sequencing-based signals), which LD
  score regression does not accommodate
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3053
tools:
- name: LDSC 1.0.1
  role: Estimates SNP-heritability proportion and enrichment captured by each trait
    x tissue DE gene set annotation, run per the 'Cell type specific analyses' tutorial
  repo: https://github.com/bulik/ldsc
- name: MESC
  role: Complementary estimator of the proportion of hSNP2 mediated by gene expression
    using GWAS and eQTL summary statistics
  repo: https://github.com/douglasyao/mesc
- name: GCTA-GRM (GCTA 1.93.2)
  role: Used upstream to estimate hSNP2 of each gene's expression for the eQTL heritability
    thresholding that feeds gene selection
  repo: https://yanglab.westlake.edu.cn/software/gcta/#Overview
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:369394dedee6a76c7fe168528da2d5ef7fca1e2db4302a89b2e56f2eade8d978
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bulik/ldsc
  tools:
  - GCTA-GRM (GCTA 1.93.2)
  - LDSC
  - LDSC 1.0.1
  - MESC
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# snp-heritability-partitioning-ldsc

## Summary

Estimate SNP-heritability (hSNP2) enrichment of trait GWAS signal in gene sets (e.g., sex-consistent 8-week exercise DE genes per tissue) using LDSC partitioned/cell-type-specific heritability regression, with a random-gene-set simulation as negative control.

## When to use

Use when GWAS summary statistics (here 114 published traits/diseases) must be tested for SNP-heritability enrichment in predefined gene sets — e.g., tissue-specific DE genes from exercise training — via LD score regression rather than simple overlap tests.

## When NOT to use

- The goal is expression-mediated heritability (proportion of hSNP2 mediated by gene expression) — use MESC with GWAS + eQTL summary statistics instead of LDSC enrichment
- The question is gene-level trait association with direction of effect rather than heritability enrichment — S-PrediXcan/TWAS is the appropriate approach
- Gene sets are not tissue/gene-set stratified annotations of common variants with appropriate LD Scores (e.g., rare-variant or sequencing-based signals), which LD score regression does not accommodate

## Inputs

- GWAS summary statistics for 114 published traits and diseases (distributed across several phenotypic and plausibly exercise-responsive categories)
- Sex-consistent 8-week differential expression gene sets per tissue (from MoTrPAC rat EET study, rat-human orthology-mapped)
- LDSC baseline annotation with 53 functional categories
- Per-tissue annotation of all expressed genes
- LD Scores and matching SNP weights/allele frequencies for the GWAS population (GTEx/GWAS, mostly European ancestry)

## Outputs

- Per-trait, per-tissue hSNP2 proportions, enrichment factors, and regression p-values (CSV tables)
- List of traits with significant LDSC heritability enrichment at Bonferroni-adjusted alpha = 0.05
- Null p-value distributions from random gene set simulations of equivalent size

## How to apply

Build stratified annotations comprising loci within 100 kb windows of sex-consistent 8-week DE genes (F1_M1/F-1_M-1 states) in each tissue, alongside the LDSC baseline annotation of 53 functional categories and an annotation of all expressed genes per tissue. Run LDSC 1.0.1 following the 'Cell type specific analyses' tutorial to estimate the proportion of hSNP2 and enrichment per trait x tissue DE gene set. Re-run LDSC with annotations for all other tissues included alongside the baseline to obtain enrichment conditional on tissue-gene-set overlaps. As a negative control, sample random gene sets of equivalent size per tissue and re-run LDSC to derive null p-value distributions. Call enrichments significant at Bonferroni-adjusted alpha = 0.05, and compile per-trait, per-tissue enrichment proportions, enrichment factors, and p-values into CSV tables.

## Related tools

- **LDSC 1.0.1** (Estimates SNP-heritability proportion and enrichment captured by each trait x tissue DE gene set annotation, run per the 'Cell type specific analyses' tutorial) — https://github.com/bulik/ldsc
- **MESC** (Complementary estimator of the proportion of hSNP2 mediated by gene expression using GWAS and eQTL summary statistics) — https://github.com/douglasyao/mesc
- **GCTA-GRM (GCTA 1.93.2)** (Used upstream to estimate hSNP2 of each gene's expression for the eQTL heritability thresholding that feeds gene selection) — https://yanglab.westlake.edu.cn/software/gcta/#Overview

## Examples

```
git clone https://github.com/bulik/ldsc.git && cd ldsc && python ldsc.py --h2 <trait>.sumstats.gz --ref-ld-chr baseline.,<tissue_DE_genes> --w-ld-chr weights_hm3_no_hla. --frqfile-chr 1000G.EUR.
```

## Evaluation signals

- Reproduced analysis yields significant enrichments in 43 of 114 traits at Bonferroni-adjusted alpha = 0.05 across tissue-specific 8-week sex-consistent DE gene sets
- The largest enrichment factor most often corresponds to spleen (22/43 traits ≈ 51%), with average enrichment factor ≈ 2.85
- Spleen DE genes show the greatest enrichment for asthma: nearly four times baseline expectation, accounting for ≈ 10% of trait heritability
- Randomly sampled gene sets of equivalent size produce p-value distributions highly similar to real sets — matching the null simulation indicates the regression machinery is unbiased
- Output CSVs contain per-trait, per-tissue enrichment proportions, enrichment factors, p-values, and simulation p-value distributions with complete trait x tissue coverage

## Limitations

- Because random gene sets of equivalent size yielded highly similar p-value distributions, enrichment results are best interpreted descriptively rather than as strong evidence of trait-specific tissue relevance
- Rats are separated from humans by nearly 140 million years of evolution, complicating interpretation of cross-species gene-set transfer
- GTEx v8 sample comprises mostly European-descendant individuals, limiting portability of LDSC-heritability results to non-European populations; GTEx humans were aged 50+ while trained rats were under eight months
- LDSC 1.0.1 is the pinned version (Python 2 codebase); a Python 3 reimplementation exists but was not what the paper validated

## Evidence

- [intro] We applied LDSC9, which can accommodate linkage disequilibrium to estimate SNP-heritability captured by sets of exercise adaption genes: "We applied LDSC9, which can accommodate linkage disequilibrium to estimate SNP-heritability captured by sets of exercise adaption genes"
- [discussion] DE genes in the spleen ... emerged as having the greatest enrichment in hSNP2, comprising nearly four times baseline expectation and accounting for ≈10% of trait heritability overall: [evidence span withheld — anchor status: not_found]
- [readme] ldsc is a command line tool for estimating heritability and genetic correlation from GWAS summary statistics. ldsc also computes LD Scores.: [evidence span withheld — anchor status: not_found]
- [methods] [LDSC](https://github.com/bulik/ldsc) (1.0.1): "[LDSC](https://github.com/bulik/ldsc) (1.0.1)"
