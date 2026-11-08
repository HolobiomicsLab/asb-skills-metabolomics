---
name: minimum-intensity-scalar-imputation
description: Use when a pcpfm/Asari feature table (e.g. preferred_drop_missing TSV)
  contains missing entries after removal of uncommon features and the table must be
  complete for downstream statistical analysis.
when_to_use_negative:
- The input table is already complete with no missing entries, since imputation would
  only scale existing values.
- The analysis requires a globally constant imputation value or model-based imputation;
  this method only implements per-feature minimum-scalar imputation.
- The table is not a feature-by-sample intensity table (e.g. it is an empirical compound
  JSON or annotation table), since the per-feature minimum logic assumes feature columns
  of intensities.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pcpfm
  role: Implements the missing value imputation step as part of its feature-table
    curation workflow (feature interpolation), producing an interpolated feature table
    from the pre-interpolation table.
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python
  role: Used to implement the per-feature minimum-scalar imputation with pandas when
    reproducing the filter outside the pcpfm CLI.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:18fb3f7ab04e81eb94f8478e9debdf934df3b6f53b71cd9e2453dcf624c7ffdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - pcpfm
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# minimum-intensity-scalar-imputation

## Summary

Imputes missing entries in an LC-MS feature table by replacing each missing value with a scalar fraction (default 0.5) of that feature's minimum observed non-zero intensity, as implemented in the pcpfm pipeline. This keeps imputation intensity-scaled per feature rather than using a global constant.

## When to use

Use when a pcpfm/Asari feature table (e.g. preferred_drop_missing TSV) contains missing entries after removal of uncommon features and the table must be complete for downstream statistical analysis.

## When NOT to use

- The input table is already complete with no missing entries, since imputation would only scale existing values.
- The analysis requires a globally constant imputation value or model-based imputation; this method only implements per-feature minimum-scalar imputation.
- The table is not a feature-by-sample intensity table (e.g. it is an empirical compound JSON or annotation table), since the per-feature minimum logic assumes feature columns of intensities.

## Inputs

- feature table TSV with missing values (zero or NaN), e.g. preferred_drop_missing.tsv from pcpfm
- interpolation_ratio parameter (default 0.5)

## Outputs

- complete feature table TSV with no missing values, e.g. preferred_interpolated.tsv, same rows/columns as input

## How to apply

Load the input feature table TSV (e.g. the preferred_drop_missing table produced by dropping infrequent features) with pandas. For each feature (column), compute the minimum non-zero observed intensity across samples, then multiply by the interpolation_ratio (default 0.5) to obtain that feature's imputation value. Replace every missing entry (zero or NaN) with its feature-specific imputation value and write the result, preserving the original sample rows and feature columns, e.g. as preferred_interpolated.tsv. The rationale is that intensity-proportional imputation keeps each feature's imputed values in the same order of magnitude as its real low-abundance signals, which is preferable for log-based statistics downstream.

## Related tools

- **pcpfm** (Implements the missing value imputation step as part of its feature-table curation workflow (feature interpolation), producing an interpolated feature table from the pre-interpolation table.) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python** (Used to implement the per-feature minimum-scalar imputation with pandas when reproducing the filter outside the pcpfm CLI.)

## Examples

```
import pandas as pd; df = pd.read_csv('preferred_drop_missing.tsv', sep='\t'); ratio=0.5; imputed = df.fillna(df.replace(0, pd.NA).min(numeric_only=True)*ratio)
```

## Evaluation signals

- The output file exists and is a valid TSV with the same sample rows and feature columns as the input table.
- No missing values (NaN or zero placeholders) remain in the imputed table.
- For every imputed entry, the value equals interpolation_ratio × that feature's minimum non-zero observed intensity (default ratio 0.5).
- Non-missing entries from the input are unchanged in the output (a diff should show edits only at previously missing positions).

## Limitations

- Imputation value is proportional to the feature's minimum observed intensity, so a feature with a very low minimum will be imputed near zero; the method does not model uncertainty in missingness.
- The default ratio 0.5 is a pipeline default; other interpolation ratios may be needed for specific study designs and must be set explicitly.
- This step follows removal of rare/uncommon features; applying it before that filter would impute features that would otherwise have been dropped.

## Evidence

- [other] Missing value imputation replaces each missing entry with a scalar multiple of that feature's minimum observed intensity in the dataframe; the default imputation ratio is 0.5.: [evidence span withheld — anchor status: not_found]
- [intro] Operations include missing value imputation as a data quality correction.: "Operations to correct common data quality issues are provided including normalization, blank masking, batch correction, removal of uncommon features, and missing value imputation."
- [intro] Feature interpolation is listed among post-processing steps in the pipeline.: "including normalization, feature interpolation, removal of rare features, quality assurance, quality control evaluations, and generates PDF reports"
- [readme] README describes curated feature tables output by PCPFM, including the option of missing-value imputation context.: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM"
