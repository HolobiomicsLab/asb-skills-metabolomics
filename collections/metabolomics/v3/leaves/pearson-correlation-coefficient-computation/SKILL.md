---
name: pearson-correlation-coefficient-computation
description: Use when you have an expression matrix (e.g. Hu-cancer-RNASeq-dataset
  genes x samples) and need Pearson correlations of each feature with a selected feature,
  with or without two-group significance testing.
when_to_use_negative:
- The goal is rank-based or outlier-robust association — the paper uses Spearman correlation
  for those cases (e.g. Met1 co-expression, CDKN2A outlier with Spearman < 0.50),
  not Pearson
- Samples from different batches/conditions are pooled without first checking pairwise
  Pearson correlations among samples, risking uncorrected batch effects (one LUAD
  sample was removed for this reason)
- The input is not expression data or lacks replicate samples per group — the z-test
  requires n1 > 3 and n2 > 3
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: Provides the pairwise Pearson correlation computation and the differential-correlation
    analysis with significance tests on Fisher-transformed or raw coefficients
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MOG
  role: Java workbench used exclusively to generate the statistical co-expression
    and differential-correlation analyses shown in the paper
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Implementation language of MOG in which the pairwise Pearson correlation and
    Fisher z-test logic is embedded
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:bbf85ee2b12e43175676737fe7b4cb96d362c5bffa8969c5a52c76ba87fe0dd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MOG
  - MetaOmGraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pearson-correlation-coefficient-computation

## Summary

Computes pairwise Pearson correlation coefficients of features (or samples) against a user-selected target feature from an expression matrix, replicating MOG's co-expression and differential-correlation analyses, including Fisher-transformed significance testing between two sample groups.

## When to use

Use when you have an expression matrix (e.g. Hu-cancer-RNASeq-dataset genes x samples) and need Pearson correlations of each feature with a selected feature, with or without two-group significance testing.

## When NOT to use

- The goal is rank-based or outlier-robust association — the paper uses Spearman correlation for those cases (e.g. Met1 co-expression, CDKN2A outlier with Spearman < 0.50), not Pearson
- Samples from different batches/conditions are pooled without first checking pairwise Pearson correlations among samples, risking uncorrected batch effects (one LUAD sample was removed for this reason)
- The input is not expression data or lacks replicate samples per group — the z-test requires n1 > 3 and n2 > 3

## Inputs

- Expression matrix (genes/features x samples, e.g. 18,212 genes from the Hu-cancer-RNASeq-dataset)
- Sample-group definition (metadata mapping samples to two groups, e.g. tumor vs non-tumor or early vs late stage)
- User-selected target feature (e.g. a gene like Met1 or CDKN2A)

## Outputs

- Table of Pearson correlation coefficients of every feature with the target feature
- differential_correlation_results.tsv with columns: feature ID, r1, r2, z-statistic, Fisher-transformed P-value, raw-coefficient P-value

## How to apply

Load the expression matrix (rows = features, columns = samples) together with the sample-group metadata that partitions samples into two groups. For each feature, compute the Pearson correlation with the user-selected target feature separately within group 1 and group 2, mirroring MOG's pairwise Pearson computation used to check batch effects (the paper flagged samples with correlations below >0.70 as outliers, e.g. one LUAD sample). To test whether a feature's correlation differs between groups, apply the Fisher z-transformation z = arctanh(r) to both coefficients and run the standard z-test with standard error sqrt(1/(n1-3) + 1/(n2-3)); MOG permits significance testing on either the Fisher-transformed or the raw coefficients, so report both two-tailed P-values. Write per-feature results (feature ID, r1, r2, z-statistic, transformed P-value, raw-coefficient P-value) as a tab-separated table.

## Related tools

- **MetaOmGraph** (Provides the pairwise Pearson correlation computation and the differential-correlation analysis with significance tests on Fisher-transformed or raw coefficients) — https://github.com/urmi-21/MetaOmGraph/
- **MOG** (Java workbench used exclusively to generate the statistical co-expression and differential-correlation analyses shown in the paper) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Implementation language of MOG in which the pairwise Pearson correlation and Fisher z-test logic is embedded) — https://github.com/urmi-21/MetaOmGraph/

## Examples

```
z = (math.atanh(r1) - math.atanh(r2)) / math.sqrt(1/(n1-3) + 1/(n2-3)); p = 2 * (1 - stat.norm.cdf(abs(z)))
```

## Evaluation signals

- Output file differential_correlation_results.tsv exists and contains one row per analyzed feature
- Both correlation coefficients (r1, r2) and both P-value columns (transformed and raw) are present and populated for every feature row
- Pearson r values fall within [-1, 1]; within-group sample correlations should mostly exceed the paper's >0.70 quality threshold if the dataset is clean
- Fisher z-statistic equals (arctanh(r1) - arctanh(r2)) / sqrt(1/(n1-3) + 1/(n2-3)) when recomputed independently
- Two-tailed P-values lie in [0, 1] and features with identical r1 and r2 yield a z-statistic of 0 and P = 1

## Limitations

- The Fisher z-test requires at least 4 samples per group (SE formula uses 1/(n-3)), undefined for smaller groups
- Pearson correlation is sensitive to outliers — the paper removed one LUAD sample whose pairwise Pearson correlation fell below the >0.70 within-organ threshold before analysis
- Testing on raw Pearson coefficients, which MOG also permits, does not follow the Fisher variance-stabilizing framework and may be less reliable

## Evidence

- [abstract] Differential correlation with significance tests: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [results] Pairwise Pearson correlation for batch-effect checks: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [results] Outlier sample removal threshold: [evidence span withheld — anchor status: not_found]
- [readme] MOG is a Java tool for large expression datasets: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets."
