---
name: metagene-gene-weight-inspection
description: Use when a PLAGE/SVD pathway activity result is significant (e.g. current
  vs never smokers, p < 0.0001) and you need to identify which pathway genes dominate
  the first metagene w by absolute weight ranking.
when_to_use_negative:
- The analysis goal is individual-gene differential expression rather than interpreting
  which genes drive a pathway-level (metagene) activity signal
- Genes in the pathway cannot be identified or the pathway-restricted matrix cannot
  be standardized across samples, so no SVD metagene can be computed
- The pathway activity comparison itself is not yet established; run the PLAGE activity-level
  t test with permutation significance first before inspecting weights
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway analysis; provides the pathway
    activity levels and first-metagene weights that this skill inspects
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:125e5863ee87c3594201224c27ad85fe2dfc1d1daf8ddd83bdc8dc3ebb45278f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# metagene-gene-weight-inspection

## Summary

Inspect the first-metagene (singular vector) weights from a PLAGE SVD of a pathway-restricted expression matrix to rank pathway genes by absolute weight and identify the genes driving a significant pathway activity difference, e.g. GCLM and GCLC in the KEGG glutathione metabolism pathway.

## When to use

Use when a PLAGE/SVD pathway activity result is significant (e.g. current vs never smokers, p < 0.0001) and you need to identify which pathway genes dominate the first metagene w by absolute weight ranking.

## When NOT to use

- The analysis goal is individual-gene differential expression rather than interpreting which genes drive a pathway-level (metagene) activity signal
- Genes in the pathway cannot be identified or the pathway-restricted matrix cannot be standardized across samples, so no SVD metagene can be computed
- The pathway activity comparison itself is not yet established; run the PLAGE activity-level t test with permutation significance first before inspecting weights

## Inputs

- Standardized (zero mean, unit variance per gene) expression matrix restricted to pathway genes, e.g. the AGED airway epithelium profiles from 75 subjects restricted to KEGG glutathione metabolism genes
- Pathway gene membership list (e.g. KEGG or Biocarta pathway gene set)
- Sample group labels (e.g. current vs never smoker)

## Outputs

- First-metagene weight vector w (one weight per pathway gene) ranked by absolute weight
- Gene weight ranking table (e.g. glutathione_pathway_verification.tsv) including driver genes such as GCLM and GCLC
- Observed two-group t statistic and permutation-derived p-value for the pathway activity comparison

## How to apply

Restrict the standardized expression matrix to the genes of the pathway of interest (e.g. KEGG glutathione metabolism) with each gene standardized to zero mean and unit variance across samples. Compute the singular value decomposition Y = WDC of the pathway-restricted matrix and extract the first metagene w and its coefficient vector c, which serve as the pathway activity levels. Rank the pathway genes by the absolute value of their first-metagene weight; in the Spira et al. airway epithelium dataset (75 subjects: 34 current, 18 former, 23 never smokers), GCLM and GCLC — the two subunits of glutamate cysteine ligase — emerge as the two highest-weight genes. Pair the weight ranking with the pathway-level test (two-group t statistic on activity levels, significance assessed by 10,000 permutations recording the maximum t-statistic per permutation, p < 0.0001) so the driver genes are interpreted in the context of a validated pathway activity difference. Write the observed t-statistic, permutation-derived p-value, and gene weight ranking to an output table such as glutathione_pathway_verification.tsv.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway analysis; provides the pathway activity levels and first-metagene weights that this skill inspects) — http://dulci.biostat.edu/pathways

## Examples

```
import numpy as np; W, D, CT = np.linalg.svd(Y_pathway, full_matrices=False); weights = W[:, 0]; ranking = sorted(zip(pathway_genes, weights), key=lambda x: -abs(x[1]))
```

## Evaluation signals

- GCLM and GCLC appear as the two genes with the highest absolute first-metagene weights in the KEGG glutathione metabolism pathway analysis
- Permutation-derived p-value (10,000 permutations, maximum t-statistic per permutation) is below 0.0001 for current vs never smoker pathway activity
- Weight ranking is derived from the first singular vector of the pathway-restricted matrix, and the accompanying coefficient vector c shows the expected activity difference between groups
- Observed t-statistic, p-value, and gene weight ranking are all written to the verification TSV, and the ranking is consistent with the t-statistic direction

## Limitations

- KEGG and Biocarta pathway definitions evolve over time; the article's pathway versions were available at the PLAGE website but no specific version numbers are stated, so gene membership may differ across releases
- The pathway-based result is complementary to individual-gene-focused analyses and should not be read as a substitute for per-gene statistics
- Pathway activity levels and weights are specific to the pathway-restricted, standardized matrix; different pathway definitions or standardization choices will yield different weight rankings

## Evidence

- [abstract] PLAGE abstract: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
- [intro] SVD-derived activity levels: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [discussion] quantifying pathway activity: "the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons"
- [discussion] group-level robustness: "Features defined by groups of genes will tend to be more robust in the face of variation at the level of individual genes."
- [abstract] identified pathways in smoker comparisons: "A number of interesting pathways are identified in comparisons between smokers and non-smokers including ones related to nicotine metabolism, mucus production, and glutathione metabolism."
