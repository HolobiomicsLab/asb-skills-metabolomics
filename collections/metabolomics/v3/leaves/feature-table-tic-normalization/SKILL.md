---
name: feature-table-tic-normalization
description: Use when an Asari-derived feature table (e.g. preferred_Feature_table.tsv)
  needs per-sample intensity scaling before statistics, especially when per-sample
  TIC correlates with feature counts or when samples were acquired in multiple batches.
when_to_use_negative:
- Data are still raw acquisitions (.raw/.mzML) and have not been preprocessed into
  a feature table by Asari.
- The table is already normalized or the goal is a model-based batch correction rather
  than TIC-based per-sample scaling.
- The feature table contains only rare/sporadic features, so no features pass the
  TIC_normalization_percentile threshold and factors would be unreliable.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Provides the `normalize` command implementing one-pass and --by_batch two-pass
    TIC normalization
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pandas
  role: Loads the feature table TSV and computes per-sample TIC and median/mean factors
    over retained features
- name: Python 3
  role: Runtime for the pcpfm pipeline and the normalization workflow
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:28949e1c70400fc3f7b5bab488765f4b86730429acf7f209ba9a5aef66cdafab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - Python 3
  - numpy
  - pandas
  - pcpfm
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Feature-table TIC normalization (one-pass and batch-aware two-pass)

## Summary

Per-sample total ion current (TIC) normalization of LC-MS feature tables as implemented in the pcpfm `normalize` operation, including a two-pass batch-aware variant that scales batches to one another. It corrects run-to-run intensity variation so downstream statistical analysis is not driven by sample-level feature counts.

## When to use

Use when an Asari-derived feature table (e.g. preferred_Feature_table.tsv) needs per-sample intensity scaling before statistics, especially when per-sample TIC correlates with feature counts or when samples were acquired in multiple batches.

## When NOT to use

- Data are still raw acquisitions (.raw/.mzML) and have not been preprocessed into a feature table by Asari.
- The table is already normalized or the goal is a model-based batch correction rather than TIC-based per-sample scaling.
- The feature table contains only rare/sporadic features, so no features pass the TIC_normalization_percentile threshold and factors would be unreliable.

## Inputs

- Feature table TSV (e.g. asari_results/preferred_Feature_table.tsv or a table in the experiment's feature_tables directory)
- Sample metadata/sequence CSV with a Batch field for batch-aware mode
- Parameters: TIC_normalization_percentile (e.g. 0.90), aggregation choice (median default vs mean), --by_batch flag, new moniker

## Outputs

- Normalized feature table TSV named with a new moniker (e.g. preferred_normalized.tsv) in feature_tables/
- QA/QC figures directory entry (pca.png, tsne.png, ...) for the new table moniker

## How to apply

Load the input feature table TSV (e.g. the Asari preferred feature table) with pandas. Restrict normalization-factor calculations to features present in at least TIC_normalization_percentile of samples (0.90 in the paper's example) so only common features contribute. Compute each sample's TIC over the retained features and derive the normalization factor using the median (default) or mean, then divide all of that sample's feature intensities by its factor. With --by_batch, first normalize each batch independently, then scale batches to one another using median TIC values computed from conserved features (two-pass). Write the result as a new moniker TSV (e.g. preferred_normalized.tsv) in the experiment's feature_tables directory.

## Related tools

- **pcpfm** (Provides the `normalize` command implementing one-pass and --by_batch two-pass TIC normalization) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pandas** (Loads the feature table TSV and computes per-sample TIC and median/mean factors over retained features)
- **Python 3** (Runtime for the pcpfm pipeline and the normalization workflow) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics

## Examples

```
pcpfm normalize --input_feature_table preferred_Feature_table.tsv --new_moniker preferred_normalized --by_batch
```

## Evaluation signals

- The output TSV exists in feature_tables/ under the new moniker (e.g. preferred_normalized.tsv).
- Output has the same samples and features as the input table (only intensities changed).
- Post-normalization per-sample TIC values over conserved features are consistent across samples rather than driven by per-sample feature counts.
- In batch-aware mode, medians of per-batch TIC (on conserved features) converge after the second scaling pass.

## Limitations

- Factors are derived only from features present in at least TIC_normalization_percentile of samples, so tables dominated by sporadic features give unstable factors.
- The default uses the median; switching to the mean changes factor robustness to outliers.
- Sample-name/mzML filename mismatches were a known pipeline issue fixed as of 2/28/24, which can affect table alignment.

## Evidence

- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [intro] including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations: "including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports"
- [readme] README: data normalization and batch correction: "- perform quality control
- data normalization and batch correction"
- [readme] README: feature tables directory organization: "feature_tables
          user_created_table_1.tsv
          user_created_table_2.tsv"
