---
name: microarray-expression-data-processing
description: Use when a two-group microarray comparison (e.g., current vs. never smokers,
  diabetic vs. non-diabetic) must be analyzed by ranking all genes by signal-to-noise
  and scoring ~400 predefined KEGG/Biocarta pathway gene-sets with permutation-based
  enrichment p-values.
when_to_use_negative:
- The goal is pathway 'activity' levels per sample via singular value decomposition
  (the PLAGE approach itself), not gene-ranking-based enrichment scores — the article
  presents the SVD method as an alternative that is only 'similar in spirit to GSEA'.
- The analysis is a single-sample or multi-group setting with no two-group comparison,
  since the signal-to-noise ranking is defined specifically as the absolute difference
  of group means divided by the sum of within-group standard deviations.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway-level analysis whose results
    are compared against the GSEA ranking/ES reconstruction; the KEGG and Biocarta
    pathway versions used are available at the PLAGE website.
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:3bc6cbbe12d73100089cefe2803debc853d2c60aca798f0b9d4209f3d7da388a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# microarray-expression-data-processing

## Summary

Reconstructs the GSEA-style analysis of microarray expression data: genes are ranked by the signal-to-noise ratio in a two-group comparison, pathway enrichment scores are computed from those rankings, and significance is estimated by permuting sample labels. It provides a reproducible baseline against which SVD-based pathway methods such as PLAGE can be compared.

## When to use

Use when a two-group microarray comparison (e.g., current vs. never smokers, diabetic vs. non-diabetic) must be analyzed by ranking all genes by signal-to-noise and scoring ~400 predefined KEGG/Biocarta pathway gene-sets with permutation-based enrichment p-values.

## When NOT to use

- The goal is pathway 'activity' levels per sample via singular value decomposition (the PLAGE approach itself), not gene-ranking-based enrichment scores — the article presents the SVD method as an alternative that is only 'similar in spirit to GSEA'.
- The analysis is a single-sample or multi-group setting with no two-group comparison, since the signal-to-noise ranking is defined specifically as the absolute difference of group means divided by the sum of within-group standard deviations.

## Inputs

- gene expression matrix (microarray data, e.g., type 2 diabetes or airway epithelia samples)
- two-group sample labels for the comparison (e.g., current vs. never smokers, former vs. never smokers)
- collection of ~400 predefined pathway gene-sets from KEGG and Biocarta websites

## Outputs

- gene rankings per two-group comparison (ordered by signal-to-noise ratio)
- enrichment scores (ES) for each KEGG/Biocarta pathway gene-set
- permutation p-values per pathway (fraction of permuted max-ES values exceeding the observed ES)
- gsea_results.csv containing all enrichment scores and permutation p-values

## How to apply

Load the gene expression matrix (e.g., the type 2 diabetes dataset studied by Mootha et al. or the cigarette-smoke airway epithelia dataset) together with the collection of ~400 KEGG/Biocarta pathways and complexes. For each two-group comparison, rank all genes by the signal-to-noise ratio, defined as the absolute value of the difference of the group means divided by the sum of the within-group standard deviations. From these rankings compute the enrichment score (ES) for each gene-set. Permute the sample labels 1000 times; for each permutation recompute the gene rankings and record the maximum ES across all gene-sets to build the null distribution. Estimate each pathway's p-value as the fraction of maximum ES values from permutations that exceed the observed ES, and write all enrichment scores and permutation p-values to gsea_results.csv.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway-level analysis whose results are compared against the GSEA ranking/ES reconstruction; the KEGG and Biocarta pathway versions used are available at the PLAGE website.) — http://dulci.biostat.edu/pathways

## Evaluation signals

- Running the implementation on current vs. never smokers must place the KEGG ribosome gene-set at the highest ES (~162) with a permutation p-value near the reported 0.17.
- Running on former vs. never smokers must give ribosome ES ~304 with p ≈ 0.047.
- The null distribution must record the maximum ES across all gene-sets per permutation (not per-pathway ES values).
- The number of sample-label permutations must be exactly 1000, and each permutation must recompute the gene rankings from the permuted groups.
- The S2N ranking must equal |mean(group1) − mean(group2)| / (std(group1) + std(group2)) for every gene; verify rankings change appropriately after each label permutation.
- gsea_results.csv must contain one ES and one permutation p-value for each of the ~400 gene-sets.

## Limitations

- The KEGG and Biocarta pathways continue to expand and evolve; the versions used for the article are only available at the PLAGE website and no specific version numbers or accessions are stated.
- The article's own PLAGE permutation analysis used 10,000 permutations and recorded the t-statistic of the most significant pathway, whereas this GSEA-style reconstruction uses 1000 permutations and the maximum enrichment score — permutation counts and null statistics differ between the two methods being compared.
- The comparison analysis reports an uncorrected permutation p-value per pathway; the reported ribosome p-values (0.17 and 0.047) should not be interpreted as family-wise-corrected significance across the ~400 gene-sets.
- The t-statistic formula used for PLAGE's pathway significance testing is referenced as an equation placeholder and is not fully rendered in the available text.

## Evidence

- [abstract] The null distribution and max-ES permutation scheme mirrors the gene-set framework against which PLAGE is compared.: "A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided."
- [abstract] The article's SVD-based analysis on the same datasets identified smoking-related pathways, contextualizing the airway epithelia comparisons reconstructed by this skill.: "A number of interesting pathways are identified in comparisons between smokers and non-smokers including ones related to nicotine metabolism, mucus production, and glutathione metabolism."
- [discussion] Defines the ~400 KEGG/Biocarta gene-set collection used as pathway input.: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [discussion] Shows the article's own permutation null construction (10,000 permutations, max statistic across pathways), the analogue of the 1000-permutation max-ES null reconstructed here.: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
