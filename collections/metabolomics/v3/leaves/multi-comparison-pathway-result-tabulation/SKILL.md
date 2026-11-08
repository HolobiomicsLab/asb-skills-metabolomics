---
name: multi-comparison-pathway-result-tabulation
description: Use when reproducing or reporting gene-set enrichment results (ES plus
  permutation p-values) for specific pathways across multiple pairwise comparisons
  of microarray groups, e.g. DM2 vs NGT and smoker vs never-smoker.
when_to_use_negative:
- The goal is to compute pathway activity levels via SVD (the PLAGE approach) rather
  than to reproduce GSEA-style ES and permutation p-values for named gene-sets
- Only a single comparison is of interest with no need to cross-compare pathway ES/p-values
  across groups
- The gene-set collection or expression dataset differs from the KEGG/Biocarta ~400
  pathway sets and the diabetes/smoking microarray datasets, so the published ES values
  are no longer valid reproduction targets
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: SVD-based pathway-level analysis whose results are compared against the tabulated
    gene-set enrichment (GSEA) ES and p-values
  repo: http://dulci.biostat.edu/pathways
- name: KEGG
  role: Source of pathway gene-sets (e.g. ribosome, oxidative phosphorylation) scored
    and tabulated
- name: Biocarta
  role: Source of pathway gene-sets (e.g. Lectin Induced Complement Pathway) scored
    and tabulated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:a8c288576bc65331d4f8c6e2c32f4bf1a8b5083d59fa261d6f6be23ac62efd8f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biocarta
  - KEGG
  - PLAGE
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-comparison-pathway-result-tabulation

## Summary

Tabulates gene-set enrichment scores (ES) and permutation-based p-values for target pathways (e.g., KEGG ribosome, KEGG oxidative phosphorylation, Biocarta Lectin Induced Complement Pathway) across several pairwise group comparisons of expression data, following the GSEA-style procedure compared against PLAGE in Tomfohr et al. 2005.

## When to use

Use when reproducing or reporting gene-set enrichment results (ES plus permutation p-values) for specific pathways across multiple pairwise comparisons of microarray groups, e.g. DM2 vs NGT and smoker vs never-smoker.

## When NOT to use

- The goal is to compute pathway activity levels via SVD (the PLAGE approach) rather than to reproduce GSEA-style ES and permutation p-values for named gene-sets
- Only a single comparison is of interest with no need to cross-compare pathway ES/p-values across groups
- The gene-set collection or expression dataset differs from the KEGG/Biocarta ~400 pathway sets and the diabetes/smoking microarray datasets, so the published ES values are no longer valid reproduction targets

## Inputs

- Type 2 diabetes expression profiles from the Whitehead Institute deposit (43 samples: 17 DM2, 17 NGT, 9 IGT)
- Airway epithelia expression profiles from the AGED database (75 samples: 34 current, 18 former, 23 never smokers)
- Group/phenotype labels defining the pairwise comparisons
- ~400 pathway and complex gene-sets assembled from the KEGG and Biocarta collections

## Outputs

- Table of enrichment scores (ES) per gene-set per pairwise comparison
- Permutation p-values (out of 1000 permutations) per gene-set per comparison
- Ranking of gene-sets by ES within each comparison (e.g. ribosome highest in DM2 vs NGT, oxidative phosphorylation next)

## How to apply

For each pairwise comparison (e.g. current vs never smokers, former vs never smokers, current vs former smokers, DM2 vs NGT), rank all genes by the signal-to-noise ratio — the absolute difference of group means divided by the sum of within-group standard deviations, following the Mootha et al. procedure. Compute the enrichment score for each of the ~400 KEGG/Biocarta pathway and complex gene-sets from its member genes' rankings. Assess significance via 1000 random permutations of sample labels, comparing each pathway's ES against the maximum ES over permuted labels to yield a p-value. Tabulate the ES and p-value for the target gene-sets (KEGG ribosome: current vs never ES 162 p 0.17, former vs never ES 304 p 0.047, DM2 vs NGT ES 278 p 0.073; Biocarta Lectin Induced Complement Pathway: current vs former ES 128 p 0.52; KEGG Oxidative Phosphorylation: DM2 vs NGT ES 261 p 0.092). The rationale, per the article, is that GSEA-style gene-set enrichment results serve as the reference point against which the SVD-based PLAGE activity levels are compared, so the tabulated ES/p-values must be exactly reproducible.

## Related tools

- **PLAGE** (SVD-based pathway-level analysis whose results are compared against the tabulated gene-set enrichment (GSEA) ES and p-values) — http://dulci.biostat.edu/pathways
- **KEGG** (Source of pathway gene-sets (e.g. ribosome, oxidative phosphorylation) scored and tabulated)
- **Biocarta** (Source of pathway gene-sets (e.g. Lectin Induced Complement Pathway) scored and tabulated)

## Evaluation signals

- KEGG ribosome gene-set obtains the highest ES in three of the four comparisons and significance (p = 0.047, 1000 permutations) in the former vs never smokers comparison
- Reproduced ES and p-values match the reported values within permutation-sampling variability: ribosome ES 162/304/278, OxPhos ES 261, Lectin Complement ES 128
- p-values are quantized in multiples of 1/1000 given the 1000-permutation scheme (e.g. 0.073, 0.092, 0.17, 0.52)
- Permutation p-values reflect the max-ES-over-permuted-labels correction, not a per-pathway null
- Each of the four pairwise comparisons yields a complete ES/p row for every target gene-set

## Limitations

- Significance is limited by permutation count: with 1000 permutations, ribosome in DM2 vs NGT reaches only p = 0.073 and oxidative phosphorylation p = 0.092 despite top ES ranks
- KEGG and Biocarta pathways continue to expand and evolve, so gene-set versions must match those used in the article (available at the PLAGE website) for exact reproduction
- The p-value procedure records the most significant pathway per permutation, coupling p-values across the ~400 gene-sets tested

## Evidence

- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [abstract] A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided.: "A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided."
- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [abstract] expression data from a study of type 2 diabetes: "expression data from a study of type 2 diabetes"
