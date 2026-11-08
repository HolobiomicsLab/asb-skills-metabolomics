---
name: permutation-based-pathway-significance-testing
description: Use when a gene-by-sample matrix has been converted to pathway activity
  levels for ~400 KEGG/Biocarta gene-sets and you need empirical p-values for phenotype
  associations without parametric distributional assumptions.
when_to_use_negative:
- You have no predefined pathway gene-set definitions (KEGG/Biocarta) for your genes
  — there is nothing to build pathway activity levels or their null distribution on.
- The analysis goal is to yield gene rankings (e.g., signal-to-noise ordered gene
  lists) rather than pathway-level p-values — use gene-set enrichment analysis (GSEA)
  instead, as in the article's companion comparison.
- The phenotype association is at the individual-gene level with no interest in group-defined
  pathway features — the paper argues individual-gene analyses are complementary,
  not substitutes.
edam_operation: http://edamontology.org/operation_3653
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway activity computation whose
    t statistics and correlations are then assessed by this permutation-testing skill
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:469855c1eee76da4540399ec9be721a1d69a163253aa3442591ee6f124969efa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_3653
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# permutation-based-pathway-significance-testing

## Summary

Empirical significance assessment for pathway activity levels (PLAGE-style SVD first-row coefficients) by randomly permuting sample labels or continuous phenotypes 10,000 times and comparing each pathway's observed statistic against the extremal (maximum t-statistic or minimum/maximum correlation) statistic from permuted data.

## When to use

Use when a gene-by-sample matrix has been converted to pathway activity levels for ~400 KEGG/Biocarta gene-sets and you need empirical p-values for phenotype associations without parametric distributional assumptions.

## When NOT to use

- You have no predefined pathway gene-set definitions (KEGG/Biocarta) for your genes — there is nothing to build pathway activity levels or their null distribution on.
- The analysis goal is to yield gene rankings (e.g., signal-to-noise ordered gene lists) rather than pathway-level p-values — use gene-set enrichment analysis (GSEA) instead, as in the article's companion comparison.
- The phenotype association is at the individual-gene level with no interest in group-defined pathway features — the paper argues individual-gene analyses are complementary, not substitutes.

## Inputs

- Pathway activity level matrices (PLAGE first-row SVD coefficients) for ~400 KEGG/Biocarta pathways and complexes
- Phenotype labels per sample (e.g., DM2 vs NGT in the 43-sample type 2 diabetes dataset, or smoker/non-smoker groups in the 75-sample airway epithelia dataset)
- Continuous phenotype values per sample (e.g., blood glucose concentration 1 hour after OGTT)
- A large number of permutations (10,000 used in this article; related GSEA comparison uses 1000 permutations of sample labels)

## Outputs

- Per-pathway permutation null distribution (maximum t-statistic per permutation for group comparisons; extremal correlations for continuous phenotypes)
- Empirical p-value per pathway-phenotype association (e.g., OxPhos vs 1-hour OGTT glucose: r = -0.52, p = 0.03)
- Table of computed vs reported correlation coefficients and p-values for candidate pathways (e.g., OxPhos, Biocarta PKA activation, KEGG ATP synthesis)

## How to apply

First compute each pathway's activity level per sample (SVD Y = WDC of the standardized genes-by-samples matrix; the first-row coefficients cj of C are the activity levels). For group comparisons, calculate a t statistic for each pathway's mean activity difference (e.g., diabetic vs non-diabetic), then run 10,000 permutations of sample labels, recording for each permutation the t statistic of the most significant (maximum) pathway to build the null distribution. For continuous phenotypes (e.g., 1-hour OGTT blood glucose in the 43-sample type 2 diabetes dataset), compute the Pearson r between each pathway's activity levels and the phenotype, permute the phenotype values 10,000 times, and take the fraction of extremal (minimum/maximum) permuted correlations stronger than the observed value in the corresponding direction. Decision point: use the max-across-pathways null for multiple-testing control across the ~400 gene-sets rather than per-pathway permutation alone.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway activity computation whose t statistics and correlations are then assessed by this permutation-testing skill) — http://dulci.biostat.edu/pathways

## Examples

```
from numpy import corrcoef, random_perm; r_obs = corrcoef(activity, glucose)[0,1]; p = sum(min(r_perm)/ (max(r_perm)) for _ in range(10000))
```

## Evaluation signals

- Computed correlation coefficients for pathway-phenotype associations match reported values (OxPhos r = -0.52, PKA activation r = +0.47, ATP synthesis r = -0.43) within ±0.05 with the same sign.
- Significance ordering matches reports: OxPhos is the most significant, with p ≈ 0.03, followed by PKA activation (p = 0.06) and ATP synthesis (p = 0.17).
- Each permutation records the extremal statistic (max t-statistic for group tests; min and max correlation for OGTT-glucose tests) across pathways, not per-pathway nulls alone.
- Empirical p-values fall within sampling uncertainty of reported values using 10,000 permutations.
- Group-comparison p-values (e.g., the GSEA-based comparison with 1000 permutations: ribosome ES 278 p = 0.073 in DM2 vs NGT; ES 304 p = 0.047 in former vs never smokers) reproduce within permutation-sampling variability.

## Limitations

- The null is built from the most significant pathway per permutation (max t-statistic / extremal correlation), which makes p-values depend on the full ~400-gene-set panel used and its composition.
- KEGG and Biocarta pathways continue to expand and evolve; the article notes specific versions matter and were preserved on the PLAGE website, so reproducibility requires pinning gene-set versions.
- The t-statistic formula referenced in the article is an equation placeholder in the source, so exact statistic definition must be filled in from the full text or code.

## Evidence

- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other (non-diabetic): "we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other"
- [abstract] A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided.: "A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided."
