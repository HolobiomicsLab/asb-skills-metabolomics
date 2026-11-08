---
name: batch-effect-qc-evaluation
description: Use when a study's samples were acquired in multiple batches (a 'Batch'
  metadata field exists) and PCA or log TIC QAQC plots of the feature table show sub-clustering
  by batch that must be corrected before downstream statistics.
when_to_use_negative:
- The feature table still contains many missing features — pyComBat batch correction
  is sensitive to missingness, so rare-feature removal and missing value imputation
  must precede it.
- The study was acquired in a single batch with no batch designation metadata field,
  so there is no --by_batch variable to correct on.
- The input is raw data (.raw or .mzML) that has not yet been preprocessed into a
  feature table by Asari — batch correction operates on feature intensities, not raw
  spectra.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: pcpfm
  role: Pipeline CLI whose batch_correct operation runs the batch correction on a
    named feature-table moniker and writes the corrected table
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pyComBat
  role: Wrapped empirical-Bayes batch-effect correction engine applied to feature
    intensities across designated batches
- name: pandas
  role: Loads the feature table TSV and sample metadata to confirm the batch field
    exists before correction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:356ad210893abdcdff65fd4cfd1f823024d67a721ac085ed6e35d3667dcf1807
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# batch-effect-qc-evaluation

## Summary

Apply empirical-Bayes batch correction to a feature table using the pcpfm batch_correct command, which wraps pyComBat, and verify the correction by comparing pre/post QAQC PCA and log TIC plots. This skill covers both running the correction and judging that it reduced batch-driven sub-clustering.

## When to use

Use when a study's samples were acquired in multiple batches (a 'Batch' metadata field exists) and PCA or log TIC QAQC plots of the feature table show sub-clustering by batch that must be corrected before downstream statistics.

## When NOT to use

- The feature table still contains many missing features — pyComBat batch correction is sensitive to missingness, so rare-feature removal and missing value imputation must precede it.
- The study was acquired in a single batch with no batch designation metadata field, so there is no --by_batch variable to correct on.
- The input is raw data (.raw or .mzML) that has not yet been preprocessed into a feature table by Asari — batch correction operates on feature intensities, not raw spectra.

## Inputs

- Batch-annotated feature table TSV (e.g., imputed table) from the experiment's feature_tables subdirectory
- Sample metadata CSV containing a user-specified batch designation field (e.g., 'Batch')

## Outputs

- batch_corrected.tsv in the feature_tables subdirectory under the 'batch_corrected' moniker
- QAQC figures (pca.png, tsne.png, log TIC plots) for the batch_corrected table in QAQC_figs/

## How to apply

Load the batch-annotated feature table TSV (e.g., the imputed table produced upstream in the pcpfm feature_tables directory) and its sample metadata with pandas, confirming the user-specified --by_batch field exists. Because the pcpfm wrapper around pyComBat is sensitive to missing features, first remove rare features or impute missing values (via the pipeline's feature interpolation / removal of uncommon features operations). Then run the pcpfm batch_correct command on the experiment with --table_moniker <input_moniker>, --new_moniker batch_corrected, and --by_batch <batch_field>, which applies pyComBat's empirical-Bayes batch-effect correction to feature intensities across the designated batches. The corrected table is written as batch_corrected.tsv in the feature_tables subdirectory. Validate by confirming the output TSV retains the same features and samples as the input, then generate QAQC figures (PCA, t-SNE, log TIC) as the paper demonstrated on its 17-batch, 1685-sample study, checking that sub-clustering by batch is reduced relative to the pre-correction table.

## Related tools

- **pcpfm** (Pipeline CLI whose batch_correct operation runs the batch correction on a named feature-table moniker and writes the corrected table) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pyComBat** (Wrapped empirical-Bayes batch-effect correction engine applied to feature intensities across designated batches)
- **pandas** (Loads the feature table TSV and sample metadata to confirm the batch field exists before correction)

## Examples

```
pcpfm batch_correct --table_moniker imputed --new_moniker batch_corrected --by_batch Batch
```

## Evaluation signals

- batch_corrected.tsv exists in the feature_tables subdirectory in TSV format under the 'batch_corrected' moniker.
- The corrected table contains the same features and samples as the input table (no rows or columns dropped by the correction step).
- QAQC PCA plot of the batch_corrected table shows reduced sub-clustering by batch relative to the pre-correction table, per the paper's 17-batch, 1685-sample demonstration.
- Log TIC QAQC plots confirm no global intensity distortion was introduced by the correction.
- The --by_batch metadata field was verified to exist in the sample metadata before running the command.

## Limitations

- pyComBat-based batch correction is sensitive to missing features; features must be removed or imputed beforehand or the step can fail or misbehave.
- Correction quality depends entirely on a correct user-specified batch designation field in the metadata; misassigned batches will distort intensities rather than fix them.
- Verification of success is visual/qualitative via QAQC PCA and log TIC plots rather than a numeric pass/fail criterion.

## Evidence

- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
- [readme] and a "Batch" field if your samples were collected in multiple: "and a "Batch" field if your samples were collected in multiple"
- [readme] data normalization and batch correction: "- data normalization and batch correction"
