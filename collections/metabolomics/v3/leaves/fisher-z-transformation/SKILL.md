---
name: fisher-z-transformation
description: Use when you have an expression matrix split into two sample groups (e.g.
  tumor vs non-tumor in the Hu-cancer-RNASeq-dataset) and want to identify features
  whose Pearson correlation with a user-selected target feature differs significantly
  between groups.
when_to_use_negative:
- The goal is differential expression (group mean differences, e.g. MOG's fold-change
  >= 2 with BH-corrected Mann-Whitney U test P < 10^-3) rather than differences in
  correlation structure between groups.
- Either group has n <= 3 samples, making the standard error sqrt(1/(n1-3) + 1/(n2-3))
  undefined or unstable.
- Only one group of samples exists, so there is no second correlation coefficient
  to compare.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: Java workbench whose differential correlation analysis with significance tests
    this skill replicates; MOG computes the pairwise Pearson correlations within each
    sample group
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional external engine MOG can send data subsets to for additional statistical
    analyses supporting the correlation tests
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:16e3e412ec95fc33372475cd1da2213561663c2130dfd0bd5f4506d3c796995a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# fisher-z-transformation

## Summary

Applies the Fisher z-transformation (z = arctanh(r)) to Pearson correlation coefficients computed in two sample groups, then runs a z-test (SE = sqrt(1/(n1-3) + 1/(n2-3))) per feature to detect significant differential correlation, as implemented in MetaOmGraph's differential correlation analysis.

## When to use

Use when you have an expression matrix split into two sample groups (e.g. tumor vs non-tumor in the Hu-cancer-RNASeq-dataset) and want to identify features whose Pearson correlation with a user-selected target feature differs significantly between groups.

## When NOT to use

- The goal is differential expression (group mean differences, e.g. MOG's fold-change >= 2 with BH-corrected Mann-Whitney U test P < 10^-3) rather than differences in correlation structure between groups.
- Either group has n <= 3 samples, making the standard error sqrt(1/(n1-3) + 1/(n2-3)) undefined or unstable.
- Only one group of samples exists, so there is no second correlation coefficient to compare.

## Inputs

- Expression matrix (e.g. gene x sample RNA-Seq or microarray count table such as the Hu-cancer-RNASeq-dataset with 18212 genes)
- Sample-group definition separating samples into two groups (e.g. via metadata attributes in a MOG project)
- User-selected target feature against which all other features' correlations are computed

## Outputs

- differential_correlation_results.tsv: tab-separated table with one row per analyzed feature containing feature ID, r1, r2, z-statistic, Fisher-transformed P-value, and raw-coefficient P-value

## How to apply

Load the expression matrix with the sample-group definition, and for each feature compute the Pearson correlation coefficient with the user-selected target feature separately within group 1 and group 2, replicating MOG's pairwise Pearson correlation computation. Apply the Fisher z-transformation z = arctanh(r) to both coefficients of each feature, then test equality with the standard z-test using standard error sqrt(1/(n1-3) + 1/(n2-3)). Compute a two-tailed P-value from the z-statistic on the transformed coefficients, and additionally report the P-value computed on the raw Pearson coefficients, since MOG permits testing significance on either basis. Write feature ID, r1, r2, z-statistic, transformed P-value, and raw-coefficient P-value as a tab-separated table to differential_correlation_results.tsv.

## Related tools

- **MetaOmGraph** (Java workbench whose differential correlation analysis with significance tests this skill replicates; MOG computes the pairwise Pearson correlations within each sample group) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional external engine MOG can send data subsets to for additional statistical analyses supporting the correlation tests)

## Examples

```
from scipy.stats import norm; import math; z = (math.atanh(r1) - math.atanh(r2)) / math.sqrt(1/(n1-3) + 1/(n2-3)); p = 2 * norm.sf(abs(z))
```

## Evaluation signals

- differential_correlation_results.tsv exists and contains one row per analyzed feature with both correlation coefficients (r1, r2) and both P-value columns present.
- Significance-test columns (z-statistic, transformed P-value, raw-coefficient P-value) are populated for every feature, matching the paper's stated capability of differential correlation analysis with significance tests.
- Per feature, z-statistic equals (arctanh(r1) - arctanh(r2)) / sqrt(1/(n1-3) + 1/(n2-3)).
- Transformed P-values are two-tailed values derived from the z-statistic on Fisher-transformed coefficients, and differ from the raw-coefficient P-values reported alongside them.
- Spot-check a few features by recomputing Pearson correlations within each group and re-deriving the P-values independently.

## Limitations

- The standard error formula sqrt(1/(n1-3) + 1/(n2-3)) requires more than 3 samples in each group.
- MOG permits testing significance on either the Fisher-transformed or the raw coefficients, so the choice of basis changes the reported P-values.
- Features with |r| = 1 in either group yield infinite transformed values and undefined z-statistics.

## Evidence

- [full_text] MOG applies a Fisher transformation to the two groups' Pearson correlation coefficients and performs a hypothesis test for equality of the coefficients, letting the user test significance on either the Fisher-transformed or the raw coefficients.: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract statement of MOG's statistical capabilities.: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [full_text] Task workflow step specifying the transformation formula.: [evidence span withheld — anchor status: not_found]
- [readme] README description of MOG's analysis scope.: [evidence span withheld — anchor status: external_doc]
