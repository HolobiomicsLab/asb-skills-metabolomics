---
name: permutation-based-significance-testing
description: Use when comparing mean pathway activity levels between two sample groups
  (e.g., diabetic vs. non-diabetic, current vs. never smokers) and you need a p-value
  that controls for multiple testing across ~400 pathways.
when_to_use_negative:
- When a single pathway was pre-specified before seeing the data and no multiple-testing
  correction is needed — the max-statistic permutation null would be unnecessarily
  conservative.
- When the design has more than two groups or a continuous covariate; the article's
  workflow only defines t-statistic-based two-group pairwise comparisons.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3523
tools:
- name: PLAGE
  role: Web program that implements the SVD-based pathway activity computation and
    the permutation-based max-t significance testing described here
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:0ebd4299d4c748acec9a1e201b81dc8616163ca181fc638ba7d35d7d245b36d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# permutation-based-significance-testing

## Summary

Assess statistical significance of pathway activity differences by randomly permuting sample labels and recording the maximum t-statistic across all pathways per permutation, yielding a family-wise-corrected p-value as used in PLAGE.

## When to use

Use when comparing mean pathway activity levels between two sample groups (e.g., diabetic vs. non-diabetic, current vs. never smokers) and you need a p-value that controls for multiple testing across ~400 pathways.

## When NOT to use

- When a single pathway was pre-specified before seeing the data and no multiple-testing correction is needed — the max-statistic permutation null would be unnecessarily conservative.
- When the design has more than two groups or a continuous covariate; the article's workflow only defines t-statistic-based two-group pairwise comparisons.

## Inputs

- Pathway activity levels (first metagene coefficient vectors) for all pathways within each sample, e.g. derived from PLAGE SVD analysis of KEGG/Biocarta gene-sets
- Group labels (two-group design, e.g. DM2 vs. NGT, current vs. never smokers)

## Outputs

- Observed t-statistic per pathway
- Permutation-derived family-wise-corrected p-value per pathway (fraction of 10,000 permutations with maximum t-statistic exceeding the observed one)

## How to apply

After computing each pathway's activity levels (the first metagene coefficient vector c from the SVD Y = WDC of the standardized, pathway-restricted expression matrix), calculate a t-statistic for each pathway comparing the two groups. Perform 10,000 comparisons by randomly permuting the sample labels and, for each permutation, recording the t-statistic of the most significant pathway (the maximum across all pathways). The p-value for a given pathway is the fraction of permutations whose maximum t-statistic exceeds that pathway's observed t-statistic. This max-statistic approach inherently corrects for the multiplicity of tests across the ~400 KEGG and Biocarta pathway gene-sets.

## Related tools

- **PLAGE** (Web program that implements the SVD-based pathway activity computation and the permutation-based max-t significance testing described here) — http://dulci.biostat.edu/pathways

## Examples

```
via PLAGE web service at http://dulci.biostat.edu/pathways with 10,000 permutations of sample labels; e.g. submit the standardized pathway-restricted matrix Y for the glutathione metabolism gene-set with groups current (n=34) vs. never (n=23) smokers
```

## Evaluation signals

- Verify that the number of permutations matches the article's choice of 10,000 (not fewer, e.g. 1,000 as used in GSEA comparisons).
- Check that the p-value is computed from the distribution of MAXIMUM t-statistics across pathways, not from per-pathway null distributions — i.e. the permutation null is family-wise, not per-test.
- Confirm that the reported p-value for a strongly differential pathway (e.g. glutathione metabolism, current vs. never smokers) falls below 0.0001.
- Sanity-check that permuting sample labels destroys the group structure while retaining the pathway activity matrix dimensions and gene weights unchanged.
- Reproduce the ranking of top pathways: the same permutation procedure applied to the diabetes data should rank Oxidative Phosphorylation comparably to the reported ES=261, p=0.092 result from GSEA.

## Limitations

- Requires a two-group design; the article only performs pairwise comparisons between one group (e.g. diabetic) and the other (non-diabetic).
- The KEGG and Biocarta pathways continue to expand and evolve, so the specific pathway versions used matter and are only archived at the PLAGE website.
- The t-statistic calculation depends on the standardized (zero mean, unit variance) pathway-restricted expression matrix, so results are sensitive to the standardization step.

## Evidence

- [discussion] Permutation count and max-statistic procedure: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] Two-group comparison framing: "we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other (non-diabetic)"
- [discussion] Permutation of sample labels basis: "To accomplish this we calculate a t statistic for each pathway:"
