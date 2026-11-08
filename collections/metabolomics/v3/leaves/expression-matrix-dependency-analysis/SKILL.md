---
name: expression-matrix-dependency-analysis
description: Use when you have a numeric expression matrix (feature rows x sample
  columns) and need pairwise, non-linear dependency scores between features that may
  not be captured by Pearson or Spearman correlation.
when_to_use_negative:
- The analysis goal is strictly linear monotonic association, where MOG's Spearman
  correlation (e.g. threshold >= 0.60 for co-expressed gene pairs) is already appropriate
- The input is not a numeric feature x sample matrix (e.g. raw reads, metadata tables,
  or already-computed distance matrices)
- The matrix is too large for in-memory pairwise estimation; MOG's big-data workflows
  rely on indexing/buffering rather than exhaustive MI computation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: Java workbench whose Mutual Information module implements the B-spline (Daub
    et al.) MI estimator for pairwise feature associations
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Runtime environment for the MOG MI module and custom B-spline estimator implementation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:6f11216c1998b90fb63063c2ca330a1b7e756e4b3848f724c8f5e96f65320414
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MOG
  - MetaOmGraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# expression-matrix-dependency-analysis

## Summary

Estimates pairwise statistical associations between features of an expression matrix using MetaOmGraph's B-spline mutual information estimator (Daub et al.), a non-linear dependency measure that yields exactly zero for independent features. It complements linear measures such as the Pearson/Spearman correlations MOG applies in its co-expression analyses.

## When to use

Use when you have a numeric expression matrix (feature rows x sample columns) and need pairwise, non-linear dependency scores between features that may not be captured by Pearson or Spearman correlation.

## When NOT to use

- The analysis goal is strictly linear monotonic association, where MOG's Spearman correlation (e.g. threshold >= 0.60 for co-expressed gene pairs) is already appropriate
- The input is not a numeric feature x sample matrix (e.g. raw reads, metadata tables, or already-computed distance matrices)
- The matrix is too large for in-memory pairwise estimation; MOG's big-data workflows rely on indexing/buffering rather than exhaustive MI computation

## Inputs

- Numeric expression matrix (feature rows x sample columns, e.g. CSV/TSV expression values)
- Optional set of selected feature pairs for which to estimate pairwise MI
- Independent reference MI implementation outputs for validation

## Outputs

- Pairwise MI matrix written as mi_matrix.csv (feature-pair mutual information values)
- Validation report comparing B-spline MI estimates against reference implementation

## How to apply

Load a small numeric expression matrix (feature rows x sample columns) and implement the Daub et al. B-spline MI estimator: estimate each feature's marginal density with B-splines and evaluate I(X;Y) for every feature pair, exploiting the property that I(X;Y)=0 exactly when X and Y are statistically independent. Compute MI between the same pairs with an independent reference MI implementation as ground truth, then verify sanity properties: MI values must be non-negative and pairs of simulated random/independent features must yield MI near zero. Quantify agreement between the B-spline estimator and the reference implementation across all pairs (value-level agreement within tolerance). This is the MI module of MetaOmGraph (MOG), a Java software; MOG's statistical analyses are the same engine behind its co-expression, differential expression, and differential correlation workflows.

## Related tools

- **MetaOmGraph** (Java workbench whose Mutual Information module implements the B-spline (Daub et al.) MI estimator for pairwise feature associations) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Runtime environment for the MOG MI module and custom B-spline estimator implementation)

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- All pairwise MI values are non-negative
- Pairs of simulated random/independent features yield MI values near zero (I(X;Y)=0 for independent variables)
- B-spline MI estimates agree with an independent reference MI implementation within a pre-set tolerance across all feature pairs
- Output mi_matrix.csv is a complete symmetric matrix over all selected feature pairs with no missing values
- Validation report records the per-pair and aggregate agreement summary

## Limitations

- The B-spline MI estimator requires tuning of spline binning; estimates depend on the Daub et al. density-estimation parameters
- Exhaustive pairwise MI is computationally heavier than the Pearson/Spearman correlation MOG uses for its large Hu-cancer-RNASeq and AT-microarray analyses, so it suits small feature sets rather than all 18,212 genes
- MOG's big-data handling (indexing and buffering) is designed for exploration; users must supply R 3.4+ (optional) or Java 8+ for full functionality

## Evidence

- [readme] Daub et al. B-spline MI is the method behind MOG's Mutual Information module: [evidence span withheld — anchor status: external_doc]
- [results] MOG's statistical analyses power its association workflows: "The statistical analyses and visualizations shown were generated exclusively using MOG."
- [readme] MOG handles large expression datasets with statistical analyses: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets."
- [methods] Repository for obtaining the MOG source implementing the MI module: "git clone https://github.com/urmi-21/MetaOmGraph.git"
