---
name: missing-value-imputation-and-normalization
description: Use when a preprocessed metabolomics peak-intensity/concentration matrix
  (CSV with a SampleID/group sample table) still contains missing values or batch/intensity
  artifacts and needs cleaning before statistical metabolite selection.
when_to_use_negative:
- Raw spectral files are the input — iMAP deliberately does not implement raw data
  preprocessing or peak picking/annotation modules.
- The data matrix is already cleaned and scaled and the goal is metabolite selection
  (univariate/PLS-DA VIP) or data-interpretation (MSEA/MetPA), which are downstream
  iMAP steps.
- The analysis goal is predictive modeling or network construction — those use iMAP's
  newly added modules, not the data-processing filters.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: iMAP
  role: Web server hosting the data-processing module (missing-value handling, normalization,
    batch correction, transformation, scaling) executed through its backend R scripts
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Desktop predecessor implementing the same one-stop data-processing workflow
    that iMAP's web modules extend
  repo: https://github.com/IP4M
- name: R-scripts-for-iMAP
  role: Backend R scripts executing the missing-value filtering, imputation, and normalization
    steps
  repo: https://github.com/IP4M/R-scripts-for-iMAP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:f37972bffcd44b21c1faedb2db8a58719a1072ab8a8789ff194a47f579be148c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hmisc
  - IP4M
  - MetaboAnalystR
  - R
  - R-scripts-for-iMAP
  - iMAP
  - ropls
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Missing-value imputation and normalization

## Summary

The core of iMAP's Step 1 data-processing layer: highlighting and filtering samples/variables with excessive missing values, filling the remaining missing values, then normalizing, batch-correcting, and scaling peak-intensity/concentration matrices before metabolites-selection.

## When to use

Use when a preprocessed metabolomics peak-intensity/concentration matrix (CSV with a SampleID/group sample table) still contains missing values or batch/intensity artifacts and needs cleaning before statistical metabolite selection.

## When NOT to use

- Raw spectral files are the input — iMAP deliberately does not implement raw data preprocessing or peak picking/annotation modules.
- The data matrix is already cleaned and scaled and the goal is metabolite selection (univariate/PLS-DA VIP) or data-interpretation (MSEA/MetPA), which are downstream iMAP steps.
- The analysis goal is predictive modeling or network construction — those use iMAP's newly added modules, not the data-processing filters.

## Inputs

- Preprocessed peak-intensity/concentration data matrix (CSV)
- Sample information table with SampleID and group (CSV)
- Optional variable information table (CSV)

## Outputs

- Processed data table (CSV) with missing values filled, normalized, batch-corrected, and scaled
- Basic statistics summary and PCA plot from the data-processing module

## How to apply

Load the preprocessed data matrix plus the sample information table (SampleID, group) and optional variable table as CSV into iMAP's data-processing module. First highlight missing values, then remove samples or variables exceeding the missing-value thresholds (the article's demonstration removed samples with >90% zeros and variables with >40% zeros, normalizing each sample to counts per million), and fill remaining missing values. Normalize the data, correct batch effects, and remove variables unstable across quality-control samples. Finish by selecting variables by absolute value, applying transformation and scaling, and emitting a statistics summary and PCA; export the processed table as a CSV artifact that feeds the metabolites-selection layer.

## Related tools

- **iMAP** (Web server hosting the data-processing module (missing-value handling, normalization, batch correction, transformation, scaling) executed through its backend R scripts) — https://imap.metaboprofile.cloud/
- **IP4M** (Desktop predecessor implementing the same one-stop data-processing workflow that iMAP's web modules extend) — https://github.com/IP4M
- **R-scripts-for-iMAP** (Backend R scripts executing the missing-value filtering, imputation, and normalization steps) — https://github.com/IP4M/R-scripts-for-iMAP

## Examples

```
Rscript process_data.R --input peak_table.csv --sample_info samples.csv --remove_samples_gt90pct_zeros --remove_vars_gt40pct_zeros --normalize cpm --fill_missing
```

## Evaluation signals

- Samples/variables exceeding the missing-value thresholds (e.g. >90% zeros in samples, >40% zeros in variables in the demonstration) are removed before imputation.
- The processed matrix contains no remaining missing values and each sample is normalized (demonstration: counts per million).
- QC-unstable variables are flagged and removed after batch-effect correction.
- The data-processing layer terminates with a statistics summary and PCA, and exports its output as a CSV artifact in iMAP's native format.
- The processed CSV satisfies downstream metabolites-selection thresholds, confirming the artifact is valid input for Step 2.

## Limitations

- iMAP does not implement raw data preprocessing or peak picking/annotation — inputs must already be preprocessed peak tables.
- Default thresholds (e.g. >90%/40% zeros) are demonstration values; users must set thresholds appropriate to their cohort (e.g. the 234-sample colorectal cancer dataset or the 308-sample IBD stool metabolomics dataset).
- No version number is stated for iMAP or IP4M, and the associated repository has no changelog, so behavior across versions is unverifiable.

## Evidence

- [intro] Data filtering and normalization: "Step 1: data-processing including data filtering and normalization"
- [methods] Data-processing module chain: "In the workflow, the input data will first go through a data-processing module including highlighting missing value, removing samples/variables which have more missing values than the thresholds,"
- [methods] Zero-filter thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Transformation and scaling: "selecting variables by their absolute value, transformation, and scaling"
- [intro] Raw preprocessing out of scope: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [readme] README platform description: "a desktop platform IP4M (integrated Platform for Metabolomics Data Analysis) which allows users to perform a nearly complete metabolomics data analysis in one-stop"
