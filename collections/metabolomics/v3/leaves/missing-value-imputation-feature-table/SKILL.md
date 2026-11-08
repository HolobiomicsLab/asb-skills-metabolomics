---
name: missing-value-imputation-feature-table
description: Use when a LC-MS feature table (e.g. preferred_drop_missing TSV) still
  contains missing values after removal of rare features and a complete table is required
  for downstream statistics or annotation.
when_to_use_negative:
- Input is already a complete table with no missing values — imputation would alter
  observed intensities.
- The goal is removal of rare/uncommon features rather than filling gaps; use the
  drop_missing filtering step instead.
- Data are not a feature table (e.g. raw spectra or empCpd JSON) — imputation operates
  on tabular feature intensities.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: pipeline providing the missing value imputation (feature interpolation) step
    over pcpfm feature tables
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python
  role: implementation language for the pandas-based imputation over the feature table
    dataframe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:27d4cd253777cacce80c7d2922a575e6f43c3a9828e0415fe116a8c6ee8adc32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - pcpfm
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# missing-value-imputation-feature-table

## Summary

Impute missing entries in a metabolomics feature table by replacing each missing value with a fraction of that feature's minimum observed intensity, as implemented in the pcpfm pipeline's 'feature interpolation' step.

## When to use

Use when a LC-MS feature table (e.g. preferred_drop_missing TSV) still contains missing values after removal of rare features and a complete table is required for downstream statistics or annotation.

## When NOT to use

- Input is already a complete table with no missing values — imputation would alter observed intensities.
- The goal is removal of rare/uncommon features rather than filling gaps; use the drop_missing filtering step instead.
- Data are not a feature table (e.g. raw spectra or empCpd JSON) — imputation operates on tabular feature intensities.

## Inputs

- feature table TSV with missing values (e.g. preferred_drop_missing.tsv from pcpfm)

## Outputs

- complete feature table TSV with no missing values (e.g. preferred_interpolated.tsv)

## How to apply

Load the feature table TSV (e.g. preferred_drop_missing.tsv, produced after removing uncommon features) with pandas. For each feature column, compute the minimum observed intensity across samples; the imputation value is interpolation_ratio × that minimum, with the default interpolation_ratio of 0.5. Replace every missing (zero or NaN) entry with its feature's imputation value, then write the result as preferred_interpolated.tsv preserving sample rows and feature columns. The rationale is to keep rare-but-real signals representable without inflating them above the feature's lowest observed signal.

## Related tools

- **pcpfm** (pipeline providing the missing value imputation (feature interpolation) step over pcpfm feature tables) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python** (implementation language for the pandas-based imputation over the feature table dataframe)

## Examples

```
import pandas as pd; df = pd.read_csv('preferred_drop_missing.tsv', sep='\t'); ratio = 0.5; df = df.fillna(df.min() * ratio); df.to_csv('preferred_interpolated.tsv', sep='\t')
```

## Evaluation signals

- Output preferred_interpolated.tsv exists and parses as a TSV.
- No missing/NaN or zero entries remain in the output table.
- Every imputed entry equals interpolation_ratio (default 0.5) × the corresponding feature's minimum observed non-zero intensity.
- Sample rows and feature columns are unchanged relative to the input table (no rows/columns added or dropped).
- Non-missing entries in the output equal their original values in the input table.

## Limitations

- Imputed values are pseudo-values derived from each feature's minimum, not real measurements, and may bias statistics on very sparse features.
- The default 0.5 ratio is pipeline-specific; changing it changes all imputed entries.
- Asari may leave some features missing entirely because they do not pass the quality (signal-to-noise) threshold — such features remain absent rather than being imputed in feature tables.

## Evidence

- [other] Missing value imputation replaces each missing entry with a scalar multiple of that feature's minimum observed intensity in the dataframe; the default imputation ratio is 0.5: [evidence span withheld — anchor status: not_found]
- [intro] feature interpolation (missing value imputation): "including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports"
- [intro] Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [readme] Output table produced after dropping infrequent features: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
