---
name: signal-to-noise-gene-ranking
description: Use when performing GSEA-style gene-set enrichment on a two-group expression
  matrix (e.g., DM2 vs NGT, current vs never smokers) and genes must be ranked before
  computing pathway enrichment scores and permutation p-values.
when_to_use_negative:
- You want sample-level pathway activity scores via singular value decomposition —
  that is the article's own PLAGE method, which the authors contrast with GSEA's ranking-based
  approach
- A group has no replicates, so within-group standard deviations (the denominator
  of the signal-to-noise ratio) are undefined or zero
- Genes are already ranked or ranked by another statistic (e.g., fold change alone),
  which would not reproduce GSEA's reported ES and p-values
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: The SVD-based pathway analysis program developed and used in this article;
    it is the method against which GSEA's signal-to-noise-ranking-based enrichment
    results are compared
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:7541effeb021cd5c4cca645dea5f289a97be7b4ff9cd9f46acd03e8bce07ecd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# signal-to-noise-gene-ranking

## Summary

Ranks all genes in a two-group microarray comparison by the signal-to-noise ratio (absolute difference of group means divided by the sum of within-group standard deviations), the ranking procedure used by GSEA and reproduced in Mootha et al.'s type 2 diabetes and airway epithelia datasets for gene-set enrichment scoring.

## When to use

Use when performing GSEA-style gene-set enrichment on a two-group expression matrix (e.g., DM2 vs NGT, current vs never smokers) and genes must be ranked before computing pathway enrichment scores and permutation p-values.

## When NOT to use

- You want sample-level pathway activity scores via singular value decomposition — that is the article's own PLAGE method, which the authors contrast with GSEA's ranking-based approach
- A group has no replicates, so within-group standard deviations (the denominator of the signal-to-noise ratio) are undefined or zero
- Genes are already ranked or ranked by another statistic (e.g., fold change alone), which would not reproduce GSEA's reported ES and p-values

## Inputs

- Gene expression matrix for a two-group comparison (e.g., type 2 diabetes profiles: 17 DM2 vs 17 NGT; airway epithelia: 34 current vs 23 never smokers)
- Collection of ~400 predefined KEGG and Biocarta pathway/complex gene-sets

## Outputs

- Per-gene signal-to-noise ranking for each pairwise comparison
- Enrichment score (ES) for every gene-set per comparison
- Permutation-based p-values (1000 permutations, max-ES null) written to gsea_results.csv

## How to apply

Load the expression matrix (e.g., the 43-sample type 2 diabetes dataset with 17 DM2, 17 NGT, 9 IGT, or the 75-sample airway epithelia dataset with 34 current, 18 former, 23 never smokers) and the ~400 predefined KEGG/Biocarta pathway gene-sets. For each pairwise comparison, rank all genes by the signal-to-noise ratio: the absolute value of the difference of group means divided by the sum of the within-group standard deviations, following the Mootha et al. procedure. Compute the enrichment score (ES) for each gene-set from its member genes' positions in this ranking. Permute sample labels 1000 times, recomputing rankings each time and recording the maximum ES across all gene-sets to build the null distribution; each pathway's p-value is the fraction of permutation maximum ES values exceeding the observed ES. Write all ES and p-values to gsea_results.csv.

## Related tools

- **PLAGE** (The SVD-based pathway analysis program developed and used in this article; it is the method against which GSEA's signal-to-noise-ranking-based enrichment results are compared) — http://dulci.biostat.edu/pathways

## Examples

```
snr = np.abs(X[a].mean(1)-X[b].mean(1))/(X[a].std(1,ddof=1)+X[b].std(1,ddof=1)); ranked = np.argsort(-snr)
```

## Evaluation signals

- Reproduced ribosome (KEGG) gene-set ES ≈ 162 with p ≈ 0.17 for current vs never smokers
- Reproduced ribosome gene-set ES ≈ 304 with p ≈ 0.047 (1000 permutations) for former vs never smokers, i.e., significance in that comparison
- Ribosome ES = 278, p = 0.073 for DM2 vs NGT, with oxidative phosphorylation ES = 261, p = 0.092 in the same comparison
- Ribosome gene-set obtains the highest ES in three of the four pairwise comparisons
- Lectin Induced Complement Pathway (Biocarta) ES = 128, p = 0.52 for current vs former smokers

## Limitations

- p-values carry permutation-sampling variability; reproduced values must match reported ones only within this variability, and 1000 permutations give limited resolution (e.g., p = 0.073)
- KEGG and Biocarta pathways continue to expand and evolve; the versions used in the paper are available at the PLAGE website but no version numbers or accessions are stated
- Significance is assessed against the maximum ES over all gene-sets per permutation, a null that controls for the number of pathways tested but makes p-values conservative

## Evidence

- [intro] objective: present a new pathway-based approach similar to GSEA: "In this article we present a new pathway based approach to the analysis of gene expression that, while similar in spirit to GSEA, has a number of potential advantages."
- [abstract] A comparison with gene-set enrichment analysis results is provided: "A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided."
- [discussion] permutation null uses the most significant pathway's statistic: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] gene-sets come from KEGG and Biocarta (~400 pathways and complexes): "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
