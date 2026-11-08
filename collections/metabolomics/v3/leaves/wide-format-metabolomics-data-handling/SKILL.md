---
name: wide-format-metabolomics-data-handling
description: Use when a metabolomics feature table (post peak-picking, samples-as-columns/features-as-rows)
  contains missing values that must be filled before downstream statistical analysis
  or tool linkage in SECIMTools/Galaxy.
when_to_use_negative:
- Input is raw instrument output (pre peak-picking / feature identification) rather
  than a feature table — the article assigns peak picking to upstream platforms such
  as Galaxy-M and Workflow4Metabolomics, not SECIMTools
- The table has no missing values and only needs re-scaling — use the Data Normalization
  and Re-Scaling tool (sample mean, median, or sum of all features as scaling factors)
  instead
- Features should be removed rather than imputed — flag-and-filter tools (Blank Feature
  Filtering, Threshold Based Flags) are the correct choice for low-quality or below-detection
  features
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: Provides the Imputation tool (group mean / group median / KNN) and the standard
    wide-format data model plus flag-file convention used to chain tools
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Alternative interface — SECIMTools tools are wrapped for use in Galaxy, and
    public-repository files can be imported into Galaxy and used in SECIMTools
- name: Python
  role: Runtime for running the Imputation tool in standalone mode
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:6939f45c089a8542d7c79fdfc50cb24f32bfa7c886936beaee4aefb573cc9e42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# wide-format-metabolomics-data-handling

## Summary

Handles metabolomics feature tables in SECIMTools' standard wide format — samples as columns, features as rows — and fills missing values via group mean, group median, or K-nearest-neighbor (KNN) imputation, emitting a completed table with a per-feature imputation flag column. This enforces the standard data formats that allow arbitrary linkages between SECIMTools tools for novel workflow designs.

## When to use

Use when a metabolomics feature table (post peak-picking, samples-as-columns/features-as-rows) contains missing values that must be filled before downstream statistical analysis or tool linkage in SECIMTools/Galaxy.

## When NOT to use

- Input is raw instrument output (pre peak-picking / feature identification) rather than a feature table — the article assigns peak picking to upstream platforms such as Galaxy-M and Workflow4Metabolomics, not SECIMTools
- The table has no missing values and only needs re-scaling — use the Data Normalization and Re-Scaling tool (sample mean, median, or sum of all features as scaling factors) instead
- Features should be removed rather than imputed — flag-and-filter tools (Blank Feature Filtering, Threshold Based Flags) are the correct choice for low-quality or below-detection features

## Inputs

- Wide-format metabolomics feature table (CSV; rows = features, columns = samples) with missing values, e.g. SECIMTools bundled example wide-format dataset
- Group/annotation information defining sample groups for group mean/median substitution

## Outputs

- Completed feature table as CSV (imputed_knn_features.csv style), with output schema matching input dimensions
- Per-feature imputation indicator column (binary flag/count) marking features that had missing values imputed

## How to apply

Load the wide-format dataset (e.g. a SECIMTools bundled example feature table) and verify the sample-as-columns, feature-as-rows layout expected by the suite. Invoke the SECIMTools Imputation tool in standalone Python mode (or via Galaxy) and choose a method: substitute the group mean or group median in place of missing values, or use K-nearest-neighbor (KNN) imputation, which fills each missing value from its nearest neighbors among the available feature data and is deterministic (the same dataset returns the same result). Append a per-feature imputation indicator column (binary flag/count) marking which features had missing values imputed, consistent with the SECIMTools flag-file convention. The rationale is that SECIMTools relies on standard data formats so tools can be arbitrarily chained, so the imputed table must preserve input dimensions and flag conventions to feed subsequent steps (e.g. Data Normalization and Re-Scaling, CV Flags).

## Related tools

- **SECIMTools** (Provides the Imputation tool (group mean / group median / KNN) and the standard wide-format data model plus flag-file convention used to chain tools) — https://github.com/SECIMTools
- **Galaxy** (Alternative interface — SECIMTools tools are wrapped for use in Galaxy, and public-repository files can be imported into Galaxy and used in SECIMTools)
- **Python** (Runtime for running the Imputation tool in standalone mode)

## Examples

```
pip install secimtools  # SECIMTools Python package from https://pypi.python.org/pypi/secimtools; then run the Imputation tool standalone (e.g. KNN) on the wide-format CSV
```

## Evaluation signals

- Script/tool run completes without error and the imputed output CSV exists
- Output schema matches the input dimensions (same feature rows and sample columns) with the imputation flag field present
- KNN imputation is deterministic: re-running on the same dataset reproduces the identical imputed table
- Each originally missing entry is filled — no residual empty cells remain in the output feature table
- The per-feature imputation indicator column correctly marks exactly those features that contained missing values, per the SECIMTools flag-file convention

## Limitations

- Method choice matters: group mean/median substitution and KNN imputation are the options offered by the Imputation tool; KNN is deterministic for a given dataset, but no imputation method recovers truly unmeasured signal
- Imputation is a preprocessing-stage step — it cannot be applied before peak picking / feature identification has produced the feature table
- No version number for SECIMTools or Galaxy is given in the discussion; the GitHub reference [27] is not resolved to a repository URL in the text, so tool-version provenance must be recorded separately

## Evidence

- [abstract] Standard data formats and utilities allow arbitrary linkages between tools for novel workflow designs: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
- [methods] Imputation methods: "The Imputation tool includes the use of the group mean or group median in place of any missing values as well as K-nearest neighbor (KNN)"
- [abstract] Standalone or Galaxy execution: "SECIMTools (SouthEast Center for Integrated Metabolomics) is a set of Python applications that are available both as standalone tools and wrapped for use in Galaxy."
- [readme] Packaging and installation: "SECIMTools are available as a secimtools pypi package. Project has also been packaged for bioconda and Galaxy Genomics Framework."
