---
name: gene-set-enrichment-score-ranking
description: Use when the goal is to rank predefined gene-sets (e.g., KEGG or Biocarta
  pathways) by enrichment score in a two-group comparison and assess their significance
  by random label permutation.
when_to_use_negative:
- When the aim is to obtain pathway activity levels per sample (as in PLAGE) rather
  than an aggregate ES and rank per comparison; GSEA produces an ES for a gene-set
  across samples, not a continuous activity level for each individual sample
- When the gene-sets of interest are not predefined (e.g., discovery of novel pathways
  from data alone); this skill requires an a priori gene-set collection
- When the comparison is not between two pre-defined groups of samples (e.g., survival
  analysis or continuous covariates), since the ES and permutation scheme as described
  are formulated for pairwise group comparisons
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: GSEA
  role: Used to compute enrichment scores and rank gene-sets in pairwise comparisons
    for comparison with PLAGE-derived pathway activity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:e2deda83ff933fe5f4c169f8e04e1ac5b6286031edb31eb2be0dbd7a7a1a2fe0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSEA
  - PLAGE
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Gene-set enrichment score ranking

## Summary

This skill applies the Gene Set Enrichment Analysis (GSEA) method to rank predefined pathways/gene-sets by an enrichment score in two-group comparisons, providing a baseline against which pathway-level SVD methods like PLAGE can be compared.

## When to use

Use when the goal is to rank predefined gene-sets (e.g., KEGG or Biocarta pathways) by enrichment score in a two-group comparison and assess their significance by random label permutation.

## When NOT to use

- When the aim is to obtain pathway activity levels per sample (as in PLAGE) rather than an aggregate ES and rank per comparison; GSEA produces an ES for a gene-set across samples, not a continuous activity level for each individual sample
- When the gene-sets of interest are not predefined (e.g., discovery of novel pathways from data alone); this skill requires an a priori gene-set collection
- When the comparison is not between two pre-defined groups of samples (e.g., survival analysis or continuous covariates), since the ES and permutation scheme as described are formulated for pairwise group comparisons

## Inputs

- A predefined collection of gene-sets (e.g., KEGG or Biocarta pathways/complexes) with known gene memberships
- A gene-by-sample expression matrix (e.g., type 2 diabetes expression data or cigarette smoke airway epithelia data) with sample labels defining the groups to compare

## Outputs

- For each gene-set and each comparison: an enrichment score (ES), a permutation-based p-value, and the resulting rank of the gene-set among all tested sets
- A comparison of ES-based rankings with SVD-derived pathway activity metrics (e.g., first-eigenvalue fraction) for the same gene-sets

## How to apply

Predefine a collection of gene-sets (e.g., ~400 KEGG/Biocarta pathways and complexes) and restrict analysis to those. For each two-group comparison, rank all genes by the signal-to-noise ratio metric. Then compute an enrichment score (ES) for each gene-set and, to establish significance, repeat the comparison 1000 times with randomly permuted sample labels, recording the fraction of permutations whose ES exceeds the observed one. In the article's benchmark, GSEA was run on the same pathway collection used for PLAGE, and the resulting ES ranks were compared directly to pathways' first-eigenvalue fractions; e.g., the ribosome pathway's high coordination (first-eigenvalue fraction ~0.53 in the smoking data) corresponded to the highest ES in three of four comparisons, even when its between-group separation was not biologically the most informative.

## Related tools

- **GSEA** (Used to compute enrichment scores and rank gene-sets in pairwise comparisons for comparison with PLAGE-derived pathway activity)

## Examples

```
gsea_cli.sh -res data.gct -cls labels.cls#comparison_v1 -gmx pathways.gmt -permute phenotype -rnd_seed timestamp -collapse No -metric Signal2Noise -nperm 1000
```

## Evaluation signals

- The top-ranked gene-set by ES should coincide with the gene-set whose genes are most coordinately expressed, as measured by the first-eigenvalue fraction of its restricted SVD (e.g., ribosome ~0.53 in the smoking data, ~0.23 for Oxidative Phosphorylation, ~0.05 for size-matched Gaussian noise)
- The permutation-based p-value should decrease as the number of permutations increases, converging to a stable estimate; with 1000 permutations the minimum resolvable p-value is ~1/1000 = 0.001
- In the article's benchmark, the ribosome gene-set achieved the highest enrichment score in three of the four two-group comparisons, and this should be reproducible when the same data and pathway collections are used
- The reported ES values (e.g., ribosome ES=162 p=0.17 current-vs-never; ribosome ES=304 p=0.047 former-vs-never) should fall within the expected ranges when rerunning with the same inputs

## Limitations

- GSEA's enrichment score can be dominated by pathways whose genes are coordinately expressed (e.g., the ribosome set with ~53% of variation in the first eigengene) regardless of whether the pathway shows a consistent group difference, making it less effective than methods that separate pathway activity levels per sample
- GSEA does not produce a per-sample pathway activity level; it yields a single aggregate ES per comparison, which limits downstream uses (e.g., regression, clustering, or patient-level stratification) that require pathway-level sample profiles
- The method requires a predefined gene-set collection, so results depend on the version and composition of the pathway database used

## Evidence

- [results] In the smoking data set the ribosome gene-set's first metagene accounts for 0.53 (over fifty percent) of total variation, and the ribosome gene-set had the highest enrichment score in three of the four GSEA comparisons performed.: [evidence span withheld — anchor status: not_found]
- [intro] We demonstrate the approach using the same expression data analyzed by Mootha et al. in their study of type 2 diabetes, and also with expression data from airway epithelia of smokers and non-smokers.: [evidence span withheld — anchor status: not_found]
- [abstract] A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided.: "A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided."
- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
