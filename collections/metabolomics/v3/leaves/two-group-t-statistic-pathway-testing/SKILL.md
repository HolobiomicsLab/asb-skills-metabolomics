---
name: two-group-t-statistic-pathway-testing
description: Use when you have PLAGE pathway activity levels per sample and need to
  identify pathways whose mean activity differs significantly between two pre-defined
  groups (e.g., smokers vs non-smokers, diabetic vs non-diabetic).
when_to_use_negative:
- When the goal is differential expression of individual genes rather than predefined
  pathway activity levels — the article notes pathway-based results are complementary,
  not a substitute, for gene-focused analyses.
- When expression data has not yet been reduced to pathway activity levels via SVD
  (e.g., raw microarray intensities); PLAGE decomposition is a prerequisite step.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway activity quantification and
    two-group permutation-based t-statistic testing; used to generate all results
    and statistics in the article
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:e8c71f240426b968d77009d8d41eca0ec4d91fbc29c6574d814327537fceb210
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# two-group-t-statistic-pathway-testing

## Summary

Performs pairwise two-group significance testing of SVD-derived pathway activity levels (PLAGE) by computing a t statistic for each pathway's mean activity difference and evaluating significance against a permutation-based null that tracks the most significant pathway per permutation.

## When to use

Use when you have PLAGE pathway activity levels per sample and need to identify pathways whose mean activity differs significantly between two pre-defined groups (e.g., smokers vs non-smokers, diabetic vs non-diabetic).

## When NOT to use

- When the goal is differential expression of individual genes rather than predefined pathway activity levels — the article notes pathway-based results are complementary, not a substitute, for gene-focused analyses.
- When expression data has not yet been reduced to pathway activity levels via SVD (e.g., raw microarray intensities); PLAGE decomposition is a prerequisite step.

## Inputs

- SVD-derived pathway activity levels (first metagene coefficients cj) per sample for ~400 KEGG/Biocarta pathways
- Sample group labels (e.g., current, former, never smokers; diabetic vs non-diabetic)

## Outputs

- Pathway-level t statistics for each pairwise two-group comparison
- Permutation-adjusted p-values identifying significantly differentially expressed pathways
- Ranked list of top differentially active pathways per comparison

## How to apply

After computing pathway activity levels via singular value decomposition of the pathway-restricted expression matrix, form two-group comparisons (e.g., diabetic vs non-diabetic, or current/former vs never smokers) and calculate a t statistic for each pathway's mean activity level difference. Significance is assessed by 10,000 permutations of sample labels; for each permutation, only the t statistic of the most significant pathway is recorded, yielding a null distribution that accounts for the number of pathways tested. The observed t statistic for a pathway is evaluated against this permutation-derived null to obtain a pathway-level p-value. This approach quantifies the level of activity of each pathway within each sample and uses those activity levels as the basis for making comparisons, which can be complementary to gene-focused analyses.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway activity quantification and two-group permutation-based t-statistic testing; used to generate all results and statistics in the article) — http://dulci.biostat.edu/pathways

## Examples

```
PLAGE web program at http://dulci.biostat.edu/pathways (submit standardized expression matrix + group labels; tool performs SVD-derived activity levels, two-group t statistics, and 10,000-label permutations internally)
```

## Evaluation signals

- The pathway most significantly differentially expressed in both current-vs-never and former-vs-never smoker comparisons is 'gamma-Hexachlorocyclohexane degradation' (KEGG), as reproduced in Table 2.
- The null distribution must record the maximum t statistic across all pathways per permutation, not per-pathway t statistics, to correct for multiple testing over ~400 pathways.
- Exactly 10,000 permutations of sample labels are used for the significance assessment.
- Standardization: each gene's expression level is normalized to zero mean and unit variance over samples before SVD decomposition.
- Results should be consistent with gene-set enrichment analysis (GSEA) applied to the same datasets.

## Limitations

- Pathway activity levels depend on SVD's first metagene; interpretation of the first singular vector's biological meaning may require inspection of gene coefficient magnitudes and gene-set annotation quality.
- The KEGG and Biocarta pathway versions used evolve over time; results depend on the specific pathway versions available at the PLAGE website.
- The t-statistic formula is referenced but not fully rendered in the section text; implementers must reconstruct it from the standard two-group setting.

## Evidence

- [discussion] permutation null for pathway significance: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] pairwise two-group comparisons of mean activity level: "we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other (non-diabetic)"
- [intro] pathway activity quantification via SVD: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [abstract] top pathway finding in smoking study: "A number of interesting pathways are identified in comparisons between smokers and non-smokers including ones related to nicotine metabolism, mucus production, and glutathione metabolism."
