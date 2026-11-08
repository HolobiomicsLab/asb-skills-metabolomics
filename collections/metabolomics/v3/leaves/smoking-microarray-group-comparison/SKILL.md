---
name: smoking-microarray-group-comparison
description: Use when comparing predefined pathway activity between smoker groups
  in airway epithelium microarray expression data (e.g. 34 current vs 23 never smokers),
  especially for KEGG glutathione metabolism genes.
when_to_use_negative:
- The goal is single-gene differential expression rather than pathway-level activity;
  the authors position pathway-based results as complementary to analyses 'focused
  on individual genes'
- No predefined pathway gene set (KEGG/Biocarta membership) is available, since the
  method is restricted to a pathway-defined submatrix
- The dataset lacks labeled smoker groups to compare, since the skill hinges on pairwise
  between-group comparison of mean activity levels
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway level analysis of gene expression
    used to derive pathway activity levels and compare smoker groups
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:be8df535d91935c8eba3e39c717feb6e1a9660c6952d6acda8ae1580938a3054
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# smoking-microarray-group-comparison

## Summary

Quantifies KEGG pathway activity from airway epithelium microarray data via PLAGE (SVD-derived pathway activity levels) and tests whether the glutathione metabolism pathway differs between current and never smokers, with permutation-based significance.

## When to use

Use when comparing predefined pathway activity between smoker groups in airway epithelium microarray expression data (e.g. 34 current vs 23 never smokers), especially for KEGG glutathione metabolism genes.

## When NOT to use

- The goal is single-gene differential expression rather than pathway-level activity; the authors position pathway-based results as complementary to analyses 'focused on individual genes'
- No predefined pathway gene set (KEGG/Biocarta membership) is available, since the method is restricted to a pathway-defined submatrix
- The dataset lacks labeled smoker groups to compare, since the skill hinges on pairwise between-group comparison of mean activity levels

## Inputs

- Airway epithelium microarray expression matrix (75 subjects: 34 current, 18 former, 23 never smokers), standardized per gene
- KEGG glutathione metabolism pathway gene membership list
- Sample group labels (current / former / never smoker)

## Outputs

- Pathway activity levels (first SVD metagene coefficient vector per sample)
- Observed two-group t statistic (current vs never smokers)
- Permutation-derived p-value from 10,000 max-t-statistic permutations
- Gene weight ranking by absolute first-metagene weight (e.g. GCLM, GCLC)
- glutathione_pathway_verification.tsv summarizing t-statistic, p-value, and gene weight ranking

## How to apply

Obtain the airway epithelium expression profiles (75 subjects: 34 current, 18 former, 23 never smokers) and restrict the standardized expression matrix to genes of the KEGG glutathione metabolism pathway. Standardize each gene's expression to zero mean and unit variance across samples, compute the SVD Y = WDC of the pathway-restricted matrix, and take the first metagene w and its coefficient vector c as the pathway activity levels. Compute a two-group t statistic comparing mean activity between current and never smokers, then assess significance with 10,000 permutations of sample labels, recording the maximum t-statistic per permutation and taking the p-value as the fraction of maxima exceeding the observed value. Extract first-metagene weights per pathway gene, rank by absolute weight, and write the t-statistic, permutation p-value, and gene ranking to glutathione_pathway_verification.tsv.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway level analysis of gene expression used to derive pathway activity levels and compare smoker groups) — http://dulci.biostat.edu/pathways

## Examples

```
import numpy as np; from scipy import stats; Y = (X - X.mean(1)[:,None]) / X.std(1)[:,None]; W = np.linalg.svd(Y)[2]; t, p = stats.ttest_ind(W[0][curr], W[0][never])
```

## Evaluation signals

- Permutation-derived p-value for current vs never smokers falls below 0.0001
- GCLM and GCLC appear as the two genes with the highest absolute first-metagene weights (subunits of glutamate cysteine ligase)
- 10,000 label permutations were run, recording the maximum t-statistic per permutation before computing the p-value
- Pathway-restricted expression matrix was standardized to zero mean and unit variance per gene before SVD Y = WDC
- Output file glutathione_pathway_verification.tsv contains the observed t-statistic, permutation p-value, and gene weight ranking

## Limitations

- KEGG and Biocarta pathway definitions continue to expand and evolve; the versions used for this article are only available at the PLAGE website, with no stated version numbers
- The t-statistic formula is referenced as an equation placeholder and is not rendered in the available section, so exact test details must be reconstructed
- Group sizes are unbalanced and modest (e.g. 23 never smokers), which the permutation scheme partially mitigates but does not eliminate

## Evidence

- [abstract] A number of interesting pathways are identified in comparisons between smokers and non-smokers including ones related to nicotine metabolism, mucus production, and glutathione metabolism.: "A number of interesting pathways are identified in comparisons between smokers and non-smokers including ones related to nicotine metabolism, mucus production, and glutathione metabolism."
- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
- [intro] Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD).: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other (non-diabetic): "we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other (non-diabetic)"
- [discussion] The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website"
