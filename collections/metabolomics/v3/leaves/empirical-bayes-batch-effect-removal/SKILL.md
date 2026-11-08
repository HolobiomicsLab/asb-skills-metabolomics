---
name: empirical-bayes-batch-effect-removal
description: Use when a feature table TSV (e.g. an imputed pcpfm table) shows batch-driven
  sub-clustering in QAQC PCA plots and sample metadata contains a batch designation
  field to correct against.
when_to_use_negative:
- The input feature table still contains missing values — the finding notes the pyComBat
  wrapper is sensitive to missing features, so blank masking / rare-feature removal
  / missing value imputation must come first.
- Only one batch exists or no batch metadata field was recorded, leaving nothing for
  empirical-Bayes correction to model.
- The goal is intensity normalization rather than between-batch adjustment — pcpfm
  provides normalization as a separate correction operation.
tools:
- name: pcpfm
  role: Pipeline providing the batch_correct operation that wraps pyComBat and manages
    feature table monikers and the feature_tables directory
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pyComBat
  role: Empirical-Bayes batch-effect correction engine applied to feature intensities
    across designated batches
- name: pandas
  role: Loading the feature table TSV and sample metadata to confirm the batch designation
    field exists before correction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:491bf802e28d6c6b940d033db67fa07229de23868d43dd8505935dad5675c5d8
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# empirical-bayes-batch-effect-removal

## Summary

Remove systematic batch effects from LC-MS metabolomics feature intensities by wrapping pyComBat's empirical-Bayes correction inside the pcpfm pipeline, using a user-specified metadata field to define batches.

## When to use

Use when a feature table TSV (e.g. an imputed pcpfm table) shows batch-driven sub-clustering in QAQC PCA plots and sample metadata contains a batch designation field to correct against.

## When NOT to use

- The input feature table still contains missing values — the finding notes the pyComBat wrapper is sensitive to missing features, so blank masking / rare-feature removal / missing value imputation must come first.
- Only one batch exists or no batch metadata field was recorded, leaving nothing for empirical-Bayes correction to model.
- The goal is intensity normalization rather than between-batch adjustment — pcpfm provides normalization as a separate correction operation.

## Inputs

- Batch-annotated feature table TSV (moniker-addressed, e.g. the imputed table) with rows = features, columns = samples
- Sample metadata CSV (e.g. Sequence.csv-derived) containing a user-specified batch designation field (--by_batch)

## Outputs

- batch_corrected.tsv in the experiment's feature_tables subdirectory, registered under the --new_moniker (batch_corrected), with the same features and samples as the input table but empirically-Bayes-corrected intensities
- QAQC figures (pca.png, tsne.png, etc.) for the new table moniker under QAQC_figs/batch_corrected/

## How to apply

Load the batch-annotated feature table (e.g. the imputed table produced upstream) together with sample metadata in pandas and confirm the --by_batch field exists. Because pyComBat is sensitive to missing features, remove or impute missing values (e.g. via pcpfm's feature interpolation / missing value imputation operation) before running. Execute the pcpfm batch_correct command with --table_moniker <input_moniker>, --new_moniker batch_corrected, and --by_batch <batch_field>, so the pipeline's wrapper applies pyComBat empirical-Bayes batch-effect correction to feature intensities across the designated batches. The corrected intensities are written back under a new moniker and saved as batch_corrected.tsv in the feature_tables subdirectory of the experiment directory.

## Related tools

- **pcpfm** (Pipeline providing the batch_correct operation that wraps pyComBat and manages feature table monikers and the feature_tables directory) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pyComBat** (Empirical-Bayes batch-effect correction engine applied to feature intensities across designated batches)
- **pandas** (Loading the feature table TSV and sample metadata to confirm the batch designation field exists before correction)

## Examples

```
pcpfm batch_correct --table_moniker imputed --new_moniker batch_corrected --by_batch Batch
```

## Evaluation signals

- batch_corrected.tsv exists in TSV format in the feature_tables subdirectory with the same features and samples as the input table.
- QAQC PCA / log TIC plots for the batch_corrected moniker show reduced sub-clustering by batch relative to the pre-correction table (as demonstrated in the paper's 17-batch, 1685-sample analysis).
- The new table is registered under the --new_moniker (batch_corrected) in the experiment.json so downstream operations address the corrected intensities.
- Missing features were removed or imputed upstream, so the pyComBat input contains no values that trigger its missing-feature sensitivity.

## Limitations

- pyComBat batch correction in pcpfm is sensitive to missing features; it must be preceded by removal of uncommon features or missing value imputation.
- Batch designation depends entirely on the user-specified --by_batch metadata field, so mis-annotated or absent batch metadata yields meaningless correction.
- README notes .raw support is limited and sample-name/mzML-name mismatches were only fixed as of 2/28/24, so upstream metadata consistency matters for batch grouping.

## Evidence

- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [results] The pyComBat wrapper requires complete tables: [evidence span withheld — anchor status: not_found]
- [readme] This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM and empirical compounds as a JSON file"
- [readme] you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.) and a "Batch" field if your samples were collected in multiple: "you should include a field for sample type (e.g., "Type") with strings for each type of sample (i.e., standards are marked 'STD', blanks are 'BLANK', etc.) and a "Batch" field if your samples were"
