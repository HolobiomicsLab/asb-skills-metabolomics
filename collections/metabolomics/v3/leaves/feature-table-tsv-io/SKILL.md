---
name: feature-table-tsv-io
description: Use when a pcpfm experiment's curated feature intensities must be loaded,
  filtered, or persisted — every pipeline operation (blank masking, batch correction,
  imputation, normalization) consumes and emits a TSV feature table under a new moniker.
when_to_use_negative:
- Input is raw LC-MS data (.raw or .mzML) — Asari preprocessing, not feature-table
  TSV I/O, is the correct entry point
- The target object is an empirical-compound JSON (empCpd.json) rather than a feature-intensity
  table
- The table needs MS1/MS2 annotation added — use the annotation commands rather than
  plain TSV round-tripping
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the TSV feature table and sample metadata and confirms required fields
    (e.g. batch designation) before operating
- name: pcpfm
  role: CLI whose operations (batch_correct, blank masking) write curated feature
    tables under new monikers as TSVs in the feature_tables directory
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Produces the upstream preferred_Feature_table.tsv / full_feature_table.tsv
    that TSV I/O operations consume
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:9dfc44c4aeb5ac107404db051e152bee59c04f065da12e955954870aedf9d904
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - Python
  - Python 3
  - numpy
  - pandas
  - pcpfm
  - pyComBat
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Feature table TSV I/O

## Summary

Reading and writing pcpfm/Asari feature tables as moniker-named TSV files (e.g. preferred_Feature_table.tsv, preferred_blank_masked.tsv, batch_corrected.tsv) within an experiment's feature_tables directory, typically with pandas.

## When to use

Use when a pcpfm experiment's curated feature intensities must be loaded, filtered, or persisted — every pipeline operation (blank masking, batch correction, imputation, normalization) consumes and emits a TSV feature table under a new moniker.

## When NOT to use

- Input is raw LC-MS data (.raw or .mzML) — Asari preprocessing, not feature-table TSV I/O, is the correct entry point
- The target object is an empirical-compound JSON (empCpd.json) rather than a feature-intensity table
- The table needs MS1/MS2 annotation added — use the annotation commands rather than plain TSV round-tripping

## Inputs

- Asari preferred feature table (preferred_Feature_table.tsv) or full feature table (full_feature_table.tsv) as TSV
- Prior operation output TSV in the feature_tables directory (e.g. preferred_blank_masked.tsv, batch_corrected.tsv)
- Experiment sample metadata CSV (sample names, sample_type/Type field, optional Batch field)
- experiment.json tracking table monikers

## Outputs

- New moniker-named TSV feature table in the experiment's feature_tables subdirectory (e.g. preferred_blank_masked.tsv, batch_corrected.tsv)
- Updated experiment state referencing the new table moniker

## How to apply

Load the source feature table (TSV, e.g. asari's preferred_Feature_table.tsv or a prior operation's output) together with the experiment sample-metadata CSV using pandas, confirming that any field the operation needs (sample_type, batch designation) exists. Apply the intended operation — e.g. blank masking with blank_intensity_ratio=3 (zeros excluded) or pyCombat batch correction keyed on the user-specified --by_batch field — then write the result under a NEW moniker as a TSV in the experiment's feature_tables subdirectory (e.g. preferred_blank_masked.tsv, batch_corrected.tsv). Batch correction is sensitive to missing features, so imputation or rare-feature removal should precede it. Each write keeps the same feature rows/columns as its input minus dropped features, so chained operations form a traceable moniker lineage inside experiment.json.

## Related tools

- **pandas** (Loads the TSV feature table and sample metadata and confirms required fields (e.g. batch designation) before operating)
- **pcpfm** (CLI whose operations (batch_correct, blank masking) write curated feature tables under new monikers as TSVs in the feature_tables directory) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Produces the upstream preferred_Feature_table.tsv / full_feature_table.tsv that TSV I/O operations consume)

## Examples

```
pcpfm batch_correct --table_moniker preferred --new_moniker batch_corrected --by_batch Batch
```

## Evaluation signals

- Output TSV (e.g. batch_corrected.tsv or preferred_blank_masked.tsv) exists in the feature_tables subdirectory and parses as valid TSV
- Row/column (feature/sample) dimensions match the input table minus intentionally dropped features
- Blank-masked output: every retained feature satisfies the 3-fold sample-vs-blank intensity criterion with zeros excluded
- Batch-corrected output: same feature and sample set as input, with QAQC PCA / log TIC plots showing reduced batch sub-clustering relative to the pre-correction table
- Moniker lineage is traceable: each table name maps to the operation and input moniker that produced it

## Limitations

- pyCombat batch correction on the feature table is sensitive to missing features; they must be removed or imputed beforehand
- Blank masking defaults to blank_intensity_ratio=3 and uses a user-specified query_field; different blank types (process vs solvent) may require re-running
- Sample names that do not match their mzML file names were a known pipeline issue fixed as of 2/28/24

## Evidence

- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] README outputs description: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
- [readme] README feature-table layout: "asari_results/
          preferred_Feature_table.tsv
          export/
              full_feature_table.tsv"
- [other] task_006 workflow write step: [evidence span withheld — anchor status: not_found]
- [other] task_003 workflow write step: [evidence span withheld — anchor status: not_found]
