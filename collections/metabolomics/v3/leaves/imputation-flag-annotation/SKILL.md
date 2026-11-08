---
name: imputation-flag-annotation
description: Use when a wide-format feature table (samples as columns, features as
  rows) contains missing entries that must be filled before downstream statistics,
  and each imputed feature needs a binary indicator flag for provenance.
when_to_use_negative:
- The input is not a feature table (e.g. raw peak-picking output); the standard workflow
  places imputation after data preprocessing, following feature identification.
- A single imputed value per missing entry must be traceable across reruns and the
  method must be stochastic — the KNN route described here is deterministic, returning
  the same result for a given dataset.
- The missingness is so pervasive that imputation would fabricate the majority of
  the table rather than fill sparse gaps; use flagging/filtering tools (e.g. Threshold
  Based Flags) to drop such features first.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SECIMTools
  role: Provides the Imputation tool (group mean / group median / KNN) run in standalone
    Python mode, plus the flag-file convention for the imputation indicator column
  repo: https://github.com/SECIMTools
- name: Python
  role: Execution environment in which the SECIMTools package is installed and the
    Imputation tool is invoked in standalone mode
- name: Galaxy
  role: Alternative wrapper in which SECIMTools tools, including imputation, can be
    run as part of integrated workflows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:d123f7fb8b651fdb12ccd0e90b4809775e05c06d47fa0a4ca20e5bff8ada027d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# imputation-flag-annotation

## Summary

Fills missing values in metabolomics feature tables using SECIMTools' Imputation tool — group mean, group median, or K-nearest-neighbor (KNN) — and emits a flag column marking imputed features per the SECIMTools flag-file convention.

## When to use

Use when a wide-format feature table (samples as columns, features as rows) contains missing entries that must be filled before downstream statistics, and each imputed feature needs a binary indicator flag for provenance.

## When NOT to use

- The input is not a feature table (e.g. raw peak-picking output); the standard workflow places imputation after data preprocessing, following feature identification.
- A single imputed value per missing entry must be traceable across reruns and the method must be stochastic — the KNN route described here is deterministic, returning the same result for a given dataset.
- The missingness is so pervasive that imputation would fabricate the majority of the table rather than fill sparse gaps; use flagging/filtering tools (e.g. Threshold Based Flags) to drop such features first.

## Inputs

- wide-format metabolomics feature table (CSV; samples as columns, features as rows) with missing values
- choice of imputation method: group mean, group median, or KNN

## Outputs

- imputed feature table in CSV format (schema matches input dimensions)
- per-feature imputation flag column (binary flag/count) appended to the output table

## How to apply

Load the wide-format feature table into the Python execution environment and verify the sample-as-columns, feature-as-rows layout. Invoke the SECIMTools Imputation tool in standalone mode, choosing among group mean, group median, or KNN; KNN is deterministic for a given dataset, filling each missing entry from its nearest neighbors among available feature data. Emit the completed table as CSV and append a per-feature imputation indicator column (binary flag/count) consistent with the SECIMTools flag-file convention. The output schema must match the input dimensions plus the flag field.

## Related tools

- **SECIMTools** (Provides the Imputation tool (group mean / group median / KNN) run in standalone Python mode, plus the flag-file convention for the imputation indicator column) — https://github.com/SECIMTools
- **Python** (Execution environment in which the SECIMTools package is installed and the Imputation tool is invoked in standalone mode)
- **Galaxy** (Alternative wrapper in which SECIMTools tools, including imputation, can be run as part of integrated workflows)

## Evaluation signals

- The script runs without error and the output file (e.g. imputed_knn_features.csv) exists in CSV format.
- The output schema matches the input dimensions exactly, with the imputation flag field present.
- No missing (NaN/empty) entries remain in the imputed table.
- KNN reruns on the same dataset return identical results, confirming deterministic behavior.
- The appended flag column correctly marks exactly the features that had missing values imputed (binary flag/count per SECIMTools flag-file convention).

## Limitations

- The standalone methods text documents only group mean, group median, and KNN; other imputation strategies are not provided by this tool.
- Deterministic KNN means repeated runs give the same result — it cannot supply stochastic variability in imputed values.
- Imputation should follow data preprocessing/filtering steps (e.g. blank feature filtering, threshold-based flags) in the standard workflow, not replace them.

## Evidence

- [methods] Imputation tool includes the use of the group mean or group median in place of any missing values as well as K-nearest neighbor (KNN): "The Imputation tool includes the use of the group mean or group median in place of any missing values as well as K-nearest neighbor (KNN)"
- [abstract] SECIMTools ... is a set of Python applications that are available both as standalone tools and wrapped for use in Galaxy.: "SECIMTools (SouthEast Center for Integrated Metabolomics) is a set of Python applications that are available both as standalone tools and wrapped for use in Galaxy."
- [intro] advanced imputation methods: [evidence span withheld — anchor status: not_found]
- [readme] The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy.: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
