---
name: lc-ms-feature-table-batch-correction
description: Use when a PCPFM-derived feature table (TSV) exhibits batch-driven sub-clustering
  in QAQC PCA plots and sample metadata contains a batch designation field, and missing/uncommon
  features have already been removed or imputed.
when_to_use_negative:
- The feature table still contains missing values and has not been imputed or filtered
  (pyComBat batch correction is sensitive to missing features)
- The goal is to remove blanks, normalize, or impute rather than correct batch effects
  — pcpfm provides separate blank masking, normalization, and missing value imputation
  operations
- The study is single-batch with no batch designation field in the sample metadata
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Python-centric pipeline whose batch_correct operation wraps pyComBat and writes
    the corrected feature table
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pyComBat
  role: Empirical-Bayes batch-effect correction engine applied to feature intensities
    across designated batches
- name: pandas
  role: Used to load and confirm the feature table and batch metadata field before
    correction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:8e83d7ac09ae08bac9f4f38bfa66aa6ffcaf4d8b520038fdaf04c8b824ca845b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - Python 3
  - pandas
  - pcpfm
  - pyComBat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lc-ms-feature-table-batch-correction

## Summary

Apply empirical-Bayes batch-effect correction (pyComBat, wrapped by the pcpfm batch_correct operation) to an LC-MS feature table from the Python-Centric Pipeline for Metabolomics (PCPFM), using a user-specified batch metadata field to reduce batch-driven sub-clustering prior to downstream statistical analysis.

## When to use

Use when a PCPFM-derived feature table (TSV) exhibits batch-driven sub-clustering in QAQC PCA plots and sample metadata contains a batch designation field, and missing/uncommon features have already been removed or imputed.

## When NOT to use

- The feature table still contains missing values and has not been imputed or filtered (pyComBat batch correction is sensitive to missing features)
- The goal is to remove blanks, normalize, or impute rather than correct batch effects — pcpfm provides separate blank masking, normalization, and missing value imputation operations
- The study is single-batch with no batch designation field in the sample metadata

## Inputs

- Batch-annotated feature table TSV (e.g. an imputed table from the feature_tables subdirectory of the experiment directory)
- Sample metadata CSV/TSV containing a batch designation field (e.g. a 'Batch' field for multi-batch studies)
- pcpfm experiment directory (experiment.json)

## Outputs

- batch_corrected.tsv in the feature_tables subdirectory, with the same features and samples as the input table but batch-corrected intensities
- New feature table moniker 'batch_corrected' registered in the experiment

## How to apply

Load the batch-annotated feature table TSV (e.g. the imputed table produced upstream) and its sample metadata with pandas, confirming the batch designation field exists. Run the pcpfm batch_correct command on the experiment, passing --table_moniker <input_moniker>, --new_moniker batch_corrected, and --by_batch <batch_field> so that batches are determined by the user-specified metadata field (README guidance recommends a 'Batch' field for multi-batch samples). This invokes the pipeline's wrapper around pyComBat, which applies empirical-Bayes batch-effect correction to feature intensities across the designated batches. The corrected intensities are written to a new feature table moniker and saved as batch_corrected.tsv in the feature_tables subdirectory. Because pyComBat is sensitive to missing values, ensure missing features are removed or imputed beforehand (pcpfm provides missing value imputation and removal of uncommon features as separate operations).

## Related tools

- **pcpfm** (Python-centric pipeline whose batch_correct operation wraps pyComBat and writes the corrected feature table) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pyComBat** (Empirical-Bayes batch-effect correction engine applied to feature intensities across designated batches)
- **pandas** (Used to load and confirm the feature table and batch metadata field before correction)

## Examples

```
pcpfm batch_correct --table_moniker <input_moniker> --new_moniker batch_corrected --by_batch <batch_field>
```

## Evaluation signals

- batch_corrected.tsv exists in TSV format in the feature_tables subdirectory with the same features and samples as the input table
- QAQC PCA plot of the corrected table shows reduced sub-clustering by batch relative to the pre-correction table
- Log TIC QAQC plots show no introduced sample-level artifacts after correction
- The input table was already imputed / had missing features removed, matching the paper's demonstration on a 17-batch, 1685-sample dataset

## Limitations

- pyComBat batch correction is sensitive to missing features; they should be removed or imputed beforehand via upstream pcpfm operations
- Batch designation must be supplied via a user-specified --by_batch metadata field; correction quality depends on accurate batch annotation in the sequence/metadata CSV
- Only systematic batch effects in feature intensity are addressed — other data quality issues (blank signal, outliers such as samples with |number-of-features Z-score| > 2.5) require separate pcpfm operations

## Evidence

- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM and empirical compounds as a JSON file representing putative"
- [readme] a "Batch" field if your samples were collected in multiple: "you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.) and a "Batch" field if your samples were"
