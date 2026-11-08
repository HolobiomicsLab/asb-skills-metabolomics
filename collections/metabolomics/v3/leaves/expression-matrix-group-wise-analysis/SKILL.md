---
name: expression-matrix-group-wise-analysis
description: Use when an expression matrix has samples partitioned into two groups
  (e.g. leaf vs pollen, early vs late stage, tumour vs non-tumour) and the goal is
  to flag features whose Pearson correlation with a user-selected target feature differs
  significantly between the groups.
when_to_use_negative:
- The goal is differential expression between the two groups, not differential correlation
  — use the group-comparison test MOG applies on log2 transformed data (Mann–Whitney
  U test with BH correction, e.g. P < 10−3) instead.
- 'Either group has fewer than 4 samples: the Fisher-test standard error sqrt(1/(n1-3)+1/(n2-3))
  is undefined when n ≤ 3.'
- The task is sample-level QC — pairwise Pearson correlations among samples from the
  same biological condition to spot outliers/batch effects (as when MOG flagged one
  LUAD sample for removal) — which compares samples, not feature correlations across
  groups.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: 'Reference implementation: MOG performs differential correlation analysis
    with significance tests by applying a Fisher transformation to the two groups''
    Pearson correlation coefficients; the skill reproduces this test outside the GUI'
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: 'Optional: MOG users can send data subsets to R for additional analyses, e.g.
    normal CDF evaluation for the two-tailed P-values'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:8d8322abd837f0086c00e56491e664b6e96f2e13c6d6b55fb73b69b875441f34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# expression-matrix-group-wise-analysis

## Summary

Replicates MetaOmGraph's differential correlation analysis: for every feature, the Pearson correlation with a user-selected target feature is computed separately within two sample groups, then the two coefficients are compared via Fisher z-transformation and a two-tailed z-test, reporting P-values on both the Fisher-transformed and the raw coefficients.

## When to use

Use when an expression matrix has samples partitioned into two groups (e.g. leaf vs pollen, early vs late stage, tumour vs non-tumour) and the goal is to flag features whose Pearson correlation with a user-selected target feature differs significantly between the groups.

## When NOT to use

- The goal is differential expression between the two groups, not differential correlation — use the group-comparison test MOG applies on log2 transformed data (Mann–Whitney U test with BH correction, e.g. P < 10−3) instead.
- Either group has fewer than 4 samples: the Fisher-test standard error sqrt(1/(n1-3)+1/(n2-3)) is undefined when n ≤ 3.
- The task is sample-level QC — pairwise Pearson correlations among samples from the same biological condition to spot outliers/batch effects (as when MOG flagged one LUAD sample for removal) — which compares samples, not feature correlations across groups.

## Inputs

- Expression matrix (rows = features/genes, columns = samples; e.g. the Hu-cancer-RNASeq-dataset or AT-microarray-dataset style tables)
- Sample-group definition mapping each sample to one of two groups (from metadata, e.g. organ, tumour status or stage)
- User-selected target feature ID whose correlations with all other features are tested

## Outputs

- differential_correlation_results.tsv: one row per analyzed feature with columns feature ID, r1, r2, z-statistic, transformed P-value, raw-coefficient P-value

## How to apply

Load the expression matrix together with the sample-group definition splitting samples into group 1 and group 2, and identify the user-selected target feature. For each feature, compute the Pearson correlation with the target within group 1 (r1) and within group 2 (r2), matching MOG's pairwise Pearson computation used to explore sample relatedness. Apply the Fisher z-transformation z = arctanh(r) to both coefficients, then run the standard z-test of equality with standard error sqrt(1/(n1-3) + 1/(n2-3)); the (n−3) correction is why each group needs at least 4 samples. Compute the two-tailed P-value from the z-statistic on the transformed coefficients, and additionally report the P-value on the raw Pearson coefficients since MOG permits testing on either basis. Write feature ID, r1, r2, z-statistic, transformed P-value and raw-coefficient P-value as a tab-separated table to differential_correlation_results.tsv.

## Related tools

- **MetaOmGraph** (Reference implementation: MOG performs differential correlation analysis with significance tests by applying a Fisher transformation to the two groups' Pearson correlation coefficients; the skill reproduces this test outside the GUI) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional: MOG users can send data subsets to R for additional analyses, e.g. normal CDF evaluation for the two-tailed P-values)

## Examples

```
import numpy as np, scipy.stats as st; z=(np.arctanh(r1)-np.arctanh(r2))/np.sqrt(1/(n1-3)+1/(n2-3)); p=2*st.norm.sf(abs(z))
```

## Evaluation signals

- differential_correlation_results.tsv exists and contains exactly one row per analyzed feature.
- Both correlation columns (r1, r2) and both P-value columns (transformed and raw) are present and populated for every row.
- Round-trip check: reported Fisher z values equal arctanh(r) for each coefficient.
- z-statistic equals (arctanh(r1) − arctanh(r2)) / sqrt(1/(n1-3) + 1/(n2-3)) and the transformed P-value matches the two-tailed normal tail probability.
- All P-values lie in [0, 1], and a feature with r1 == r2 yields a z-statistic of 0 and P = 1.

## Limitations

- The Fisher z-test requires n1 > 3 and n2 > 3 per group; correlations from tiny groups cannot be compared.
- Results depend on the test basis: MOG permits significance testing on either the Fisher-transformed or the raw Pearson coefficients, so the two P-value columns may disagree.
- Pearson correlations are sensitive to outliers — the paper's QC removed one lung adenocarcinoma (LUAD) sample whose within-organ correlation fell below the >0.70 threshold, and such outliers can distort r1/r2.

## Evidence

- [abstract] Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests.: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [results] MOG applies a Fisher transformation to the two groups' Pearson correlation coefficients and performs a hypothesis test for equality of the coefficients.: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [readme] MOG is a Java workbench whose group-wise analyses the skill reproduces.: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets."
