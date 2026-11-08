---
name: two-group-correlation-comparison
description: Use when an expression matrix is split into two sample groups and the
  goal is to detect features whose Pearson correlation with a target feature changes
  significantly between the groups, as in MOG's differential correlation analysis.
when_to_use_negative:
- The goal is differential expression (mean-level change between groups), which MOG
  handles via fold-change >= 2 and BH-corrected Mann–Whitney U test P < 10^-3 rather
  than correlation testing
- The analysis compares overall co-expression across all samples (e.g. Spearman >=
  0.60 across all samples) rather than testing whether correlations differ between
  two groups
- More than two sample groups are involved; the Fisher z-test for equality of two
  coefficients is defined only for a two-group contrast
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3307
tools:
- name: MetaOmGraph
  role: Standalone Java workbench whose differential correlation analysis with significance
    tests this skill replicates; provides the pairwise Pearson correlation computation
    and the option to test significance on Fisher-transformed or raw coefficients
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Implementation language of MOG; Java 8+ runtime is required to run MOG's correlation
    and differential correlation routines
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:777b60522972a3c0811ced5ac0ae620af1e4f7a347f8e3b6c51c8c74c08e1663
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
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Two-group correlation comparison

## Summary

Statistical skill for identifying features whose Pearson correlation with a user-selected feature differs significantly between two groups of samples, as implemented in MetaOmGraph (MOG) via a Fisher transformation followed by a z-test for equality of correlation coefficients.

## When to use

Use when an expression matrix is split into two sample groups and the goal is to detect features whose Pearson correlation with a target feature changes significantly between the groups, as in MOG's differential correlation analysis.

## When NOT to use

- The goal is differential expression (mean-level change between groups), which MOG handles via fold-change >= 2 and BH-corrected Mann–Whitney U test P < 10^-3 rather than correlation testing
- The analysis compares overall co-expression across all samples (e.g. Spearman >= 0.60 across all samples) rather than testing whether correlations differ between two groups
- More than two sample groups are involved; the Fisher z-test for equality of two coefficients is defined only for a two-group contrast

## Inputs

- Expression matrix (e.g. tab-delimited gene-by-sample table such as the 18,212-gene Hu-cancer-RNASeq-dataset)
- Sample-group definition separating samples into two groups (e.g. tumor vs non-tumor, or early stage I/II vs late stage III+)
- User-selected target feature (e.g. a gene like CDKN2A) to correlate against all other features

## Outputs

- Tab-separated table differential_correlation_results.tsv with one row per analyzed feature: feature ID, r1, r2, z-statistic, Fisher-transformed P-value, and raw-coefficient P-value

## How to apply

Load the expression matrix together with a sample-group definition that partitions samples into two groups (e.g. tumor vs non-tumor in the Hu-cancer-RNASeq-dataset). For each feature, compute the Pearson correlation with the user-selected target feature separately within group 1 and group 2, replicating MOG's pairwise Pearson correlation computation. Apply the Fisher z-transformation (z = arctanh(r)) to both coefficients, then test equality with the standard z-test whose standard error is sqrt(1/(n1-3) + 1/(n2-3)). Compute a two-tailed P-value from the z-statistic on the transformed coefficients, and additionally report the P-value computed on the raw Pearson coefficients, since MOG permits testing significance on either basis. Write all per-feature results (feature ID, r1, r2, z-statistic, transformed P-value, raw-coefficient P-value) to a tab-separated file such as differential_correlation_results.tsv.

## Related tools

- **MetaOmGraph** (Standalone Java workbench whose differential correlation analysis with significance tests this skill replicates; provides the pairwise Pearson correlation computation and the option to test significance on Fisher-transformed or raw coefficients) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Implementation language of MOG; Java 8+ runtime is required to run MOG's correlation and differential correlation routines) — https://github.com/urmi-21/MetaOmGraph/

## Examples

```
z <- (atanh(r1) - atanh(r2)) / sqrt(1/(n1-3) + 1/(n2-3)); p_fisher <- 2*pnorm(-abs(z)); p_raw <- p_raw_coefficient_test
```

## Evaluation signals

- differential_correlation_results.tsv exists and contains exactly one row per analyzed feature
- Each row includes both group correlation coefficients (r1, r2) and both P-value columns (Fisher-transformed and raw-coefficient) fully populated
- The z-statistic is reproducible from r1, r2, n1, and n2 via z = (arctanh(r1) - arctanh(r2)) / sqrt(1/(n1-3) + 1/(n2-3))
- P-values fall in [0, 1] and the transformed P-value matches a two-tailed normal test on the z-statistic
- Significance-test columns are populated, matching the paper's stated capability of differential correlation analysis with significance tests

## Limitations

- Requires at least 4 samples per group since the standard error formula sqrt(1/(n1-3) + 1/(n2-3)) is undefined for group sizes <= 3
- Replicates pairwise Pearson correlation computation group-by-group; sample-quality screening matters, as in the Hu-cancer-RNASeq-dataset where one LUAD sample with Pearson correlation <= 0.70 was removed before analysis
- MOG permits testing on raw or Fisher-transformed coefficients; P-values from the two bases can differ, so the choice of basis must be reported

## Evidence

- [abstract] MOG statistical analyses include differential correlation with significance tests: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [results] MOG computes pairwise Pearson correlations among samples: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [readme] MOG is a Java tool for exploratory analysis of large datasets: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets."
