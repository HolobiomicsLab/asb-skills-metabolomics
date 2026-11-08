---
name: time-course-metabolomic-peak-table-formatting
description: Use when a raw time-course metabolomic peak table (e.g., LC-MS plasma
  samples from MetaboLights MTBLS656 or NOREVA_exampledata.zip) must be reformatted,
  before scanning thousands of NOREVA processing workflows.
when_to_use_negative:
- Input is raw instrument data (proprietary vendor formats) rather than a peak table
  — convert first with Proteowizard/nmrML and build the table with XCMS or NMRProc
  Flow
- The dataset is case–control or multiclass rather than time-course — use the normulticlass*
  assessment functions instead of the nortimecourse* path
- The study design lacks time-point labels (or the samples-per-timepoint is <6, making
  downstream criterion Cc inapplicable)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: R package whose PrepareInuputFiles(dataformat, rawdata, label) function standardizes
    the peak table and label file into the unified input for time-course workflow
    scanning
  repo: https://github.com/idrblab/NOREVA
- name: MetaboLights MTBLS656
  role: Source of the example LC-MS quadrupole plasma time-course peak table used
    for the cortisol circadian-pattern validation
  repo: https://www.ebi.ac.uk/metabolights/MTBLS656/files
- name: NOREVA_exampledata
  role: Packaged example data from the NOREVA website used as a ready-made input for
    the unified-format preparation
  repo: https://idrblab.org/noreva/NOREVA_exampledata.zip
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:2f450600b774a211fab46c8dc605626e0e0febacd1fadb73b9a63c2f628934e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - MetaboLights MTBLS656
  - NOREVA
  - NOREVA_exampledata
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# time-course-metabolomic-peak-table-formatting

## Summary

Standardizes a time-course metabolomics peak table (e.g., the MTBLS656 plasma LC-MS dataset with T0–T48 time-point labels) into NOREVA's unified sample-row/feature-column format with a companion label file, so that the subsequent nortimecourse* workflow-scan functions can ingest it.

## When to use

Use when a raw time-course metabolomic peak table (e.g., LC-MS plasma samples from MetaboLights MTBLS656 or NOREVA_exampledata.zip) must be reformatted, before scanning thousands of NOREVA processing workflows.

## When NOT to use

- Input is raw instrument data (proprietary vendor formats) rather than a peak table — convert first with Proteowizard/nmrML and build the table with XCMS or NMRProc Flow
- The dataset is case–control or multiclass rather than time-course — use the normulticlass* assessment functions instead of the nortimecourse* path
- The study design lacks time-point labels (or the samples-per-timepoint is <6, making downstream criterion Cc inapplicable)

## Inputs

- Time-course metabolomic peak table (sample rows, metabolic-feature columns) from MetaboLights MTBLS656 or NOREVA_exampledata.zip
- Label file assigning each sample to a time-point class (e.g., T0–T48)
- Format flag: dataformat=1 (NOREVA-standardized) or dataformat=2 (output of one of 12 supported software tools)

## Outputs

- Prepared NOREVA input object accepted by nortimecourseqcall/nortimecoursenoall
- Downstream: ranked scan of candidate processing workflows (e.g., 1,968 workflows for MTBLS656) with per-workflow multi-criteria scores

## How to apply

Manually standardize the peak table to NOREVA's unified format by preserving rows (samples provided by name) and columns (metabolic features), loading the table from the MetaboLights MTBLS656 files or the NOREVA_exampledata.zip archive. Prepare a separate label file naming the time-point class of each sample (T0–T48 for the cortisol circadian time course). Call PrepareInuputFiles with dataformat=1 for a table already in NOREVA's standardized format, or dataformat=2 for a table exported by one of the 12 supported peak-processing software tools, passing rawdata and the label file. The prepared object then feeds the time-course assessment functions nortimecourseqcall (with QCS, no IS) or nortimecoursenoall (no QCS, no IS), which scan candidate workflows across S1 filtering/imputation, S2 QCS correction, S3 transformation and S4 normalization. Row/column preservation is the critical step because raw data file organizations differ widely, and the study assumption flags (SAalpha, SAbeta, SAgamma) passed to the scan depend on the formatted table satisfying those assumptions.

## Related tools

- **NOREVA** (R package whose PrepareInuputFiles(dataformat, rawdata, label) function standardizes the peak table and label file into the unified input for time-course workflow scanning) — https://github.com/idrblab/NOREVA
- **MetaboLights MTBLS656** (Source of the example LC-MS quadrupole plasma time-course peak table used for the cortisol circadian-pattern validation) — https://www.ebi.ac.uk/metabolights/MTBLS656/files
- **NOREVA_exampledata** (Packaged example data from the NOREVA website used as a ready-made input for the unified-format preparation) — https://idrblab.org/noreva/NOREVA_exampledata.zip

## Examples

```
data <- PrepareInuputFiles(dataformat=1, rawdata="MTBLS656_peaktable.csv", label="MTBLS656_T0_T48_labels.csv"); res <- nortimecourseqall(data, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- The formatted table preserves rows as named samples and columns as features, matching NOREVA's unified format before PrepareInuputFiles is called
- The label file contains one time-point class (T0–T48) per sample row, and PrepareInuputFiles output is accepted without error by nortimecourseqcall or nortimecoursenoall
- The downstream scan completes over the full candidate workflow set (1,968 workflows for MTBLS656) and produces a ranking file
- The ranking file lists MEI+NON+CUT+[QUA-LEV] first and KNN+NON+LOG+[PAR-SUM] last, confirming the formatted input reproduced the published benchmark

## Limitations

- Manual standardization is time-consuming because raw data file organizations are diverse, making row/column preservation challenging
- Successful formatting presupposes upfront experimental design that adopted both QCSs and internal standards — NOREVA does not enforce this during data preparation
- Partial assessment functions can generate output files with names identical to earlier steps, overwriting existing outputs and producing misleading results; keep intermediate files distinct
- Datasets with fewer than 6 samples per timepoint are unsuitable for downstream assessment with criterion Cc

## Evidence

- [discussion] Manually standardizing the peak table to a unified format: [evidence span withheld — anchor status: not_found]
- [discussion] Row/column preservation difficulty: [evidence span withheld — anchor status: not_found]
- [readme] PrepareInuputFiles signature and format flags: [evidence span withheld — anchor status: not_found]
- [readme] Label file role: [evidence span withheld — anchor status: external_doc]
- [readme] Time-course scan entry points: [evidence span withheld — anchor status: not_found]
