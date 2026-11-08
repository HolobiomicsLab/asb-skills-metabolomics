---
name: metabolomics-feature-table-wrangling
description: Use when a pcpfm/Asari feature table (preferred_drop_missing.tsv or full_feature_table.tsv)
  still contains missing values, infrequent features, or failed-injection samples,
  and you need a curated, ready-for-statistics table.
when_to_use_negative:
- The input is raw mzML or .raw data that has not yet been preprocessed by Asari into
  a feature table
- The table is already complete (no missing values, no infrequent features, no outlier
  samples) so re-running imputation would alter valid zeros or low signals
- You need de novo MS2 annotation or empirical-compound (khipu) grouping rather than
  table curation
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: pcpfm
  role: pipeline whose data-quality operations implement blank masking, normalization,
    batch correction, removal of uncommon features and missing value imputation on
    feature tables
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: produces the initial preferred/full feature tables (TSV) that this wrangling
    skill operates on
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python
  role: pandas-based scripting environment for executing the per-feature minimum ×
    interpolation_ratio imputation logic
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:3c9574b1c3ca7fbbee345ead20a03d0038808d80536d262981b66e1067f1e7e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - Python
  - pcpfm
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# metabolomics-feature-table-wrangling

## Summary

A skill for curating Asari/pcpfm feature tables (TSV) through operations like normalization, blank masking, batch correction, removal of uncommon features, outlier sample dropping (|Z|>2.5 on feature-count Z-score), and missing value imputation via per-feature minimum × interpolation_ratio (default 0.5).

## When to use

Use when a pcpfm/Asari feature table (preferred_drop_missing.tsv or full_feature_table.tsv) still contains missing values, infrequent features, or failed-injection samples, and you need a curated, ready-for-statistics table.

## When NOT to use

- The input is raw mzML or .raw data that has not yet been preprocessed by Asari into a feature table
- The table is already complete (no missing values, no infrequent features, no outlier samples) so re-running imputation would alter valid zeros or low signals
- You need de novo MS2 annotation or empirical-compound (khipu) grouping rather than table curation

## Inputs

- pandas DataFrame or TSV feature table from pcpfm/Asari (e.g. preferred_drop_missing.tsv with sample rows and feature columns)
- pcpfm experiment.json or sample metadata CSV identifying sample types (BLANK, QC, STD, Unknown) and Batch field

## Outputs

- Curated feature table TSV (e.g. preferred_interpolated.tsv) with no remaining missing values
- Per-feature imputation values computed as interpolation_ratio × feature minimum intensity

## How to apply

Load the input feature table TSV (e.g. preferred_drop_missing.tsv) with pandas, preserving sample rows and feature columns. First drop outlier samples whose number-of-features Z-score has absolute value greater than the default 2.5 threshold. Then remove rare/uncommon features that are detected in too few samples (the 'preferred_drop_missing' table is produced by dropping infrequent features). Finally impute remaining missing entries: for each feature (column) compute the minimum non-zero observed intensity, multiply by the interpolation_ratio (default 0.5), and replace every missing entry (zero or NaN) with that per-feature imputation value. Write the result to preferred_interpolated.tsv so that downstream normalization, blank masking, batch correction, or annotation steps operate on a complete matrix.

## Related tools

- **pcpfm** (pipeline whose data-quality operations implement blank masking, normalization, batch correction, removal of uncommon features and missing value imputation on feature tables) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (produces the initial preferred/full feature tables (TSV) that this wrangling skill operates on) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python** (pandas-based scripting environment for executing the per-feature minimum × interpolation_ratio imputation logic)

## Examples

```
import pandas as pd; df = pd.read_csv('preferred_drop_missing.tsv', sep='\t'); df = df.fillna(df.min() * 0.5); df.to_csv('preferred_interpolated.tsv', sep='\t', index=False)
```

## Evaluation signals

- Output file exists and is a valid TSV with the same sample rows and feature columns as the input table
- No remaining missing values (NaN or zero) in the output table
- Every imputed entry equals 0.5 × that feature's minimum observed non-zero intensity (interpolation_ratio × per-feature minimum)
- Samples whose number-of-features Z-score has absolute value > 2.5 are absent from the output
- Feature count in the output is ≤ the input, reflecting removal of infrequent features

## Limitations

- Imputation replaces missing entries with a fraction of the feature's minimum intensity, which introduces biased low values and can distort downstream statistics if treated as measured signal
- Samples failing quality (absolute Z-score of feature count > 2.5) are dropped by default, which can bias sample composition if failures are not randomly distributed
- The skill does not itself perform MS1/MS2 annotation, khipu empirical-compound grouping, or blank masking — those are separate pcpfm operations that may need to follow curation
- Feature tables must already exist from Asari preprocessing; .raw and mzML conversion and peak picking are outside this skill's scope

## Evidence

- [results] Missing value imputation replaces each missing entry with a scalar multiple of that feature's minimum observed intensity in the dataframe; the default imputation ratio is 0.5, so missing values are filled with 0.5 times the feature's minimum value.: [evidence span withheld — anchor status: not_found]
- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [intro] by default, samples that have an absolute value for their number of features Z-score greater than 2.5 are dropped.: "by default, samples that have an absolute value for their number of features Z-score greater than 2.5 are dropped."
- [readme] feature_tables user_created_table_1.tsv user_created_table_2.tsv ...: "feature_tables
          user_created_table_1.tsv
          user_created_table_2.tsv"
