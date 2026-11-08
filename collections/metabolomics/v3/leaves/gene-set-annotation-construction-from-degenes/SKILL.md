---
name: gene-set-annotation-construction-from-degenes
description: Use when GWAS summary statistics must be stratified by SNP-heritability
  enrichment for a gene set (e.g., tissue DE genes from exercise training) and you
  need to convert a gene list into a partitioned/binary LDSC annotation.
when_to_use_negative:
- You need expression-mediated heritability (proportion of hSNP2 mediated by expression)
  rather than annotation-based enrichment — use MESC instead
- The gene set is not a coherent gene list (e.g., per-gene trait-tissue-gene triplets
  or TWAS hits) — S-PrediXcan/FUSION-style TWAS is the appropriate skill
- The DE gene list lacks a matched expressed-gene background annotation, making enrichment
  uninterpretable relative to baseline expression
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: LDSC
  role: Estimates SNP-heritability enrichment (hSNP2) captured by the constructed
    DE-gene annotations, run per the 'Cell type specific analyses' tutorial
  repo: https://github.com/bulik/ldsc
- name: LDSC 1.0.1
  role: Specific version (1.0.1) used to partition heritability across the baseline
    and DE-gene annotations
  repo: https://github.com/bulik/ldsc
- name: DESeq2
  role: Upstream tool producing the sex- and tissue-specific differential expression
    results from which the DE gene sets are selected
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:85af306cd53fb31cd98dabd8e0f43108c9d4fb68418c3b5a17c1b634c9e97de0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bulik/ldsc
  tools:
  - DESeq2
  - LDSC
  - LDSC 1.0.1
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-set-annotation-construction-from-degenes

## Summary

Constructs LDSC-ready SNP annotations from tissue-specific differentially expressed gene sets (e.g., sex-consistent 8-week exercise DE genes) by assigning SNPs/loci within 100kb windows of the DE genes, alongside the LDSC baseline annotation and an all-expressed-genes control annotation, enabling heritability enrichment estimation per trait x tissue gene set.

## When to use

Use when GWAS summary statistics must be stratified by SNP-heritability enrichment for a gene set (e.g., tissue DE genes from exercise training) and you need to convert a gene list into a partitioned/binary LDSC annotation.

## When NOT to use

- You need expression-mediated heritability (proportion of hSNP2 mediated by expression) rather than annotation-based enrichment — use MESC instead
- The gene set is not a coherent gene list (e.g., per-gene trait-tissue-gene triplets or TWAS hits) — S-PrediXcan/FUSION-style TWAS is the appropriate skill
- The DE gene list lacks a matched expressed-gene background annotation, making enrichment uninterpretable relative to baseline expression

## Inputs

- List of tissue-specific, sex-consistent 8-week DE genes (e.g., from MoTrPAC rat EET DESeq2 analysis at 5% IHW alpha)
- 114 published GWAS summary statistics (traits/diseases)
- LDSC baseline annotation (53 functional categories)
- List of all expressed genes per tissue (for the control annotation)
- LD reference panel / LD scores compatible with LDSC 1.0.1

## Outputs

- Per-tissue binary SNP annotation files marking loci within 100kb of DE genes, all-expressed genes, and random size-matched gene sets
- Per-trait, per-tissue LDSC heritability (proportion of hSNP2) and enrichment factor estimates with p-values
- Conditional enrichment results when all other tissue annotations are included
- CSV tables of enrichment proportions, enrichment factors, p-values, and random-gene-set simulation p-value distributions

## How to apply

Start from sex-consistent 8-week DE genes per tissue (selected at 5% IHW alpha, e.g., genes in the F1_M1 or F-1_M-1 repfdr states, up- or down-regulated in both sexes). For each tissue, build a binary annotation marking loci within 100kb of the tissue's DE genes, and combine it with the LDSC baseline annotation (53 functional categories) plus an annotation of loci within 100kb of all expressed genes in that tissue so the DE-gene annotation is tested against an expressed-gene background. Use LDSC 1.0.1 following the 'Cell type specific analyses' tutorial to estimate enrichment of SNP-heritability (hSNP2) for each trait x tissue gene set; optionally re-run with annotations for all other tissues included to obtain enrichment conditional on tissue-gene-set overlaps. As a negative control, sample random gene sets of equivalent size per tissue, build matched annotations, and re-run LDSC to derive null p-value distributions.

## Related tools

- **LDSC** (Estimates SNP-heritability enrichment (hSNP2) captured by the constructed DE-gene annotations, run per the 'Cell type specific analyses' tutorial) — https://github.com/bulik/ldsc
- **LDSC 1.0.1** (Specific version (1.0.1) used to partition heritability across the baseline and DE-gene annotations) — https://github.com/bulik/ldsc
- **DESeq2** (Upstream tool producing the sex- and tissue-specific differential expression results from which the DE gene sets are selected)

## Examples

```
python ldsc.py --h2 trait.sumstats --ref-ld-chr baseline,@,de_spleen,expressed_spleen --w-ld-chr weights --out trait_spleen
```

## Evaluation signals

- Significant LDSC heritability enrichments should appear in 43 traits at Bonferroni-adjusted alpha = 0.05 across the tissue-specific sex-consistent 8-week DE gene sets
- The largest enrichment factor should most often correspond to the spleen (22/43 ≈ 51% of traits) with an average enrichment factor of ≈2.85
- Spleen DE genes should show the greatest enrichment for asthma — nearly four times baseline expectation, accounting for ≈10% of trait heritability
- Randomly sampled gene sets of equivalent size should yield highly similar p-value distributions, indicating results must be interpreted descriptively
- Each annotation must include the LDSC baseline (53 functional categories) and the all-expressed-genes-per-tissue control so enrichment is interpretable against an expression background

## Limitations

- Random-gene-set simulations produced p-value distributions highly similar to the real DE gene sets, so enrichment results are best interpreted descriptively rather than as strong evidence
- Rats are separated from humans by nearly 140 million years of evolution, complicating interpretation of rat-derived DE gene sets in human GWAS
- Limited portability to non-European populations because the GTEx/GWAS sample comprises mostly European-descendant individuals

## Evidence

- [methods] LDSC annotations from DE gene sets: [evidence span withheld — anchor status: not_found]
- [intro] LDSC applied to exercise adaptation genes: "We applied LDSC9, which can accommodate linkage disequilibrium to estimate SNP-heritability captured by sets of exercise adaption genes"
- [methods] LDSC version and source: "[LDSC](https://github.com/bulik/ldsc) (1.0.1)"
- [discussion] Spleen enrichment finding: [evidence span withheld — anchor status: not_found]
- [readme] LDSC tool description: [evidence span withheld — anchor status: not_found]
