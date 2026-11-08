---
name: metabolomics-missing-value-imputation
description: Use when a metabolomics feature table (samples as columns, features as
  rows) contains missing values that must be filled before downstream statistics such
  as ANOVA, PLS-DA, or random forest classification.
when_to_use_negative:
- The table has no missing values — running imputation would alter complete data unnecessarily
- Features have not yet passed upstream QC (e.g. Blank Feature Filtering, Threshold
  Based Flags, CV Flags) — the SECIMTools workflow places filtering and diagnostics
  before imputation
- The goal is peak picking or feature identification rather than post-preprocessing
  gap filling — that is handled by Galaxy-M or Workflow4Metabolomics upstream
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: SECIMTools
  role: Provides the Imputation tool (group mean, group median, KNN) runnable standalone
    in Python or wrapped for Galaxy; installed via the secimtools PyPI package
  repo: https://github.com/SECIMTools
- name: Python
  role: Execution environment for running SECIMTools Imputation in standalone mode
    and writing the imputed CSV with flag columns
  repo: https://pypi.python.org/pypi/secimtools
- name: Galaxy
  role: Alternative wrapped interface for running SECIMTools imputation within integrated
    workflows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:76eb3c2bda76444a68991d37fcdbe46e6c37d84231834348c5e2f80a684733f5
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# metabolomics-missing-value-imputation

## Summary

Fills missing entries in wide-format metabolomics feature tables using the SECIMTools Imputation tool, which substitutes the group mean, group median, or K-nearest-neighbor (KNN) estimates, and emits a flag column marking imputed features per the SECIMTools flag-file convention.

## When to use

Use when a metabolomics feature table (samples as columns, features as rows) contains missing values that must be filled before downstream statistics such as ANOVA, PLS-DA, or random forest classification.

## When NOT to use

- The table has no missing values — running imputation would alter complete data unnecessarily
- Features have not yet passed upstream QC (e.g. Blank Feature Filtering, Threshold Based Flags, CV Flags) — the SECIMTools workflow places filtering and diagnostics before imputation
- The goal is peak picking or feature identification rather than post-preprocessing gap filling — that is handled by Galaxy-M or Workflow4Metabolomics upstream

## Inputs

- Wide-format metabolomics feature table (samples as columns, features as rows) with missing values, e.g. the SECIMTools bundled example dataset
- Group/phenotype labels for samples (required for group mean / group median methods)

## Outputs

- CSV file of the completed feature table (e.g. imputed_knn_features.csv) with output schema matching input dimensions
- Per-feature imputation indicator column (binary flag/count) marking features with imputed values, per the SECIMTools flag-file convention

## How to apply

Load the wide-format feature table (sample-as-columns, feature-as-rows) into Python and verify its dimensions. Invoke the SECIMTools Imputation tool in standalone (Python) mode, choosing among group mean, group median, or KNN imputation; KNN fills each missing value from the values of its nearest neighbors among the available feature data and is deterministic, returning the same result for a given dataset. Emit the completed table as a CSV and append a per-feature imputation indicator column (binary flag/count) marking which features had missing values imputed, consistent with the SECIMTools flag-file convention. Validate that the output schema matches the input dimensions with the imputation flag field present.

## Related tools

- **SECIMTools** (Provides the Imputation tool (group mean, group median, KNN) runnable standalone in Python or wrapped for Galaxy; installed via the secimtools PyPI package) — https://github.com/SECIMTools
- **Python** (Execution environment for running SECIMTools Imputation in standalone mode and writing the imputed CSV with flag columns) — https://pypi.python.org/pypi/secimtools
- **Galaxy** (Alternative wrapped interface for running SECIMTools imputation within integrated workflows)

## Evaluation signals

- Script runs without error and imputed_knn_features.csv exists in CSV format
- Output schema matches the input dimensions (sample-as-columns, feature-as-rows layout preserved) with the imputation flag field present
- Re-running KNN imputation on the same dataset yields identical results (determinism check)
- No missing entries remain in the emitted feature table
- Flag column correctly marks exactly the features that originally contained missing values

## Limitations

- Group mean / group median substitution collapses within-group variability for imputed entries; KNN is the advanced option but depends on the quality of neighbor structure in the feature data
- No version number for SECIMTools is given in the discussion of the source article, so reproducibility requires pinning the installed package version yourself
- Article does not specify KNN parameters (e.g. k), so defaults must be documented per run

## Evidence

- [methods] Imputation tool includes group mean/median/KNN: "The Imputation tool includes the use of the group mean or group median in place of any missing values as well as K-nearest neighbor (KNN)"
- [abstract] Standard formats link tools in workflows: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
- [readme] SECIMTools runs standalone or via Galaxy: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [intro] Imputation is a workflow step in SECIMTools: "New functionality includes: blank feature filtering"
- [methods] Package availability on PyPI: "we packaged all tools as a python package available from https://pypi.python.org/pypi/secimtools"
