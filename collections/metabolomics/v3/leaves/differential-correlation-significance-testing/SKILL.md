---
name: differential-correlation-significance-testing
description: Use when comparing whether a feature's Pearson correlation with a user-selected
  target feature differs significantly between two groups of samples (e.g. tumor vs
  non-tumor), and a per-feature significance test on the correlation difference is
  required.
when_to_use_negative:
- The goal is differential expression between two groups — MOG's workflow for that
  uses a Mann–Whitney U test on log2 transformed data (e.g. BH-corrected P < 10^-3),
  not correlation testing.
- There is no two-group sample definition in the metadata and the user wants a single
  correlation/co-expression value per feature rather than a test of difference between
  groups.
- Either group has n ≤ 3 samples — the standard error sqrt(1/(n1-3) + 1/(n2-3)) is
  undefined or degenerate in that regime.
tools:
- name: MOG
  role: Reference implementation whose pairwise Pearson correlation computation and
    differential correlation analysis with significance tests this skill replicates
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MetaOmGraph
  role: Java workbench providing differential correlation analysis with significance
    tests; its behavior (testing on Fisher-transformed or raw coefficients) defines
    the expected output semantics
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:217554d5bb9413cdfe9b319415fdb3975833e2dc6fa6f7af8bb7a191dc510f6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# differential-correlation-significance-testing

## Summary

Replicates MetaOmGraph's differential correlation analysis: for each feature, compute Pearson correlations with a user-selected target feature in two sample groups, then test the equality of the two coefficients via Fisher z-transformation, reporting P-values on both transformed and raw coefficients.

## When to use

Use when comparing whether a feature's Pearson correlation with a user-selected target feature differs significantly between two groups of samples (e.g. tumor vs non-tumor), and a per-feature significance test on the correlation difference is required.

## When NOT to use

- The goal is differential expression between two groups — MOG's workflow for that uses a Mann–Whitney U test on log2 transformed data (e.g. BH-corrected P < 10^-3), not correlation testing.
- There is no two-group sample definition in the metadata and the user wants a single correlation/co-expression value per feature rather than a test of difference between groups.
- Either group has n ≤ 3 samples — the standard error sqrt(1/(n1-3) + 1/(n2-3)) is undefined or degenerate in that regime.

## Inputs

- Expression matrix (features x samples, as used in MOG projects)
- Sample-group definition (metadata assigning each sample to one of two groups)
- User-selected target feature to correlate against

## Outputs

- differential_correlation_results.tsv — tab-separated table with one row per feature: feature ID, r1, r2, z-statistic, Fisher-transformed P-value, raw-coefficient P-value

## How to apply

Load the expression matrix together with the sample-group definition that separates samples into two groups. For each feature, compute the Pearson correlation coefficient with the user-selected target feature separately within group 1 (r1) and group 2 (r2), replicating MOG's pairwise Pearson computation. Apply the Fisher z-transformation (z = arctanh(r)) to both coefficients and test equality with the standard z-test using standard error sqrt(1/(n1-3) + 1/(n2-3)). Compute a two-tailed P-value from the z-statistic on the transformed coefficients, and additionally report the P-value computed on the raw Pearson coefficients, since MOG permits testing significance on either basis. Write all per-feature results as a tab-separated table to differential_correlation_results.tsv.

## Related tools

- **MOG** (Reference implementation whose pairwise Pearson correlation computation and differential correlation analysis with significance tests this skill replicates) — https://github.com/urmi-21/MetaOmGraph/
- **MetaOmGraph** (Java workbench providing differential correlation analysis with significance tests; its behavior (testing on Fisher-transformed or raw coefficients) defines the expected output semantics) — https://github.com/urmi-21/MetaOmGraph/

## Examples

```
z1, z2 = math.atanh(r1), math.atanh(r2); z_stat = (z1 - z2) / math.sqrt(1/(n1-3) + 1/(n2-3))
```

## Evaluation signals

- differential_correlation_results.tsv exists and contains one row per analyzed feature.
- Both P-value columns (Fisher-transformed and raw-coefficient) are present and populated for every row.
- Per-feature z-statistic recomputes as (arctanh(r1) - arctanh(r2)) / sqrt(1/(n1-3) + 1/(n2-3)).
- All r1 and r2 values fall within [-1, 1] and all P-values within [0, 1].
- r1 equals the within-group-1 Pearson correlation of the feature with the target feature, matching MOG's pairwise Pearson computation.

## Limitations

- The significance test can be run on either the Fisher-transformed or the raw Pearson coefficients; the two bases yield different P-values and the choice must be reported.
- The z-test standard error sqrt(1/(n1-3) + 1/(n2-3)) requires more than 3 samples per group.
- The analysis is meaningful only when sample metadata cleanly partitions samples into the two groups being compared.

## Evidence

- [abstract] MOG statistical analyses include differential correlation with significance tests: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [readme] MOG is a Java tool for interactive exploration of large datasets: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets. MOG overcomes the challenges posed by big size and complexity of big datasets by efficient handling of data"
- [intro] MOG is presented as a Java software for large expression datasets: "we present MetaOmGraph (MOG), a Java software, to interactively explore and visualize large expression datasets."
