---
name: lc-ms-time-series-feature-extraction
description: Use when a time-course LC-MS peak table (sample rows, feature columns,
  time-point labels such as T0–T48) has been scanned by NOREVA's workflow ranking
  and you must extract a known rhythmic metabolite's trace from top- and bottom-ranked
  workflow outputs to visually validate processing quality.
when_to_use_negative:
- The input is raw LC-MS instrument files rather than a peak table — raw data must
  first be converted/peak-picked with tools such as Proteowizard or XCMS before NOREVA's
  unified peak-table input
- The dataset is case–control or multi-class rather than time-course — use NOREVA's
  multiclass functions (normulticlassqcall/normulticlassnoall/normulticlassisall)
  instead
- Fewer than 6 samples exist in one time point, since criterion Cc is unsuitable for
  such small-sample time-course datasets
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: Runs the time-course workflow scan (nortimecourseqcall) over 1,968 S1–S4 method
    combinations, ranks all workflows, and emits the processed peak-table outputs
    from which the cortisol time series is extracted
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Execution environment for the NOREVA R-based protocol and for plotting the
    first-ranked vs last-ranked cortisol traces into the comparison figure
  repo: https://www.r-project.org/
- name: NOREVA_exampledata
  role: Source of the MTBLS656 plasma time-course peak table used as the input for
    the cortisol circadian-pattern validation
  repo: https://idrblab.org/noreva/NOREVA_exampledata.zip
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:1f36aa24f48baf171cf35db4a11b0b2608620c928ee5c47bb2ced57b0548f9a7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - MetaboLights MTBLS656
  - NOREVA
  - NOREVA_exampledata
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-ms-time-series-feature-extraction

## Summary

Extracts a biologically rhythmic metabolite's intensity time series (e.g. the cortisol circadian trace across T0–T48 of the MTBLS656 plasma time-course peak table) from the processed outputs of NOREVA-ranked LC-MS time-course workflows, and plots the first-ranked versus last-ranked workflow traces side-by-side to validate that workflow choice preserves or distorts the reported circadian pattern.

## When to use

Use when a time-course LC-MS peak table (sample rows, feature columns, time-point labels such as T0–T48) has been scanned by NOREVA's workflow ranking and you must extract a known rhythmic metabolite's trace from top- and bottom-ranked workflow outputs to visually validate processing quality.

## When NOT to use

- The input is raw LC-MS instrument files rather than a peak table — raw data must first be converted/peak-picked with tools such as Proteowizard or XCMS before NOREVA's unified peak-table input
- The dataset is case–control or multi-class rather than time-course — use NOREVA's multiclass functions (normulticlassqcall/normulticlassnoall/normulticlassisall) instead
- Fewer than 6 samples exist in one time point, since criterion Cc is unsuitable for such small-sample time-course datasets

## Inputs

- MTBLS656 plasma LC-MS time-course peak table (sample rows, feature columns, time-point labels T0–T48) from NOREVA_exampledata.zip or MetaboLights
- Time-point label file for the time-course dataset (NOREVA 'label' file)
- NOREVA workflow ranking output listing all 1,968 scanned workflows
- Processed peak-table outputs of the first-ranked (MEI+NON+CUT+[QUA-LEV]) and last-ranked (KNN+NON+LOG+[PAR-SUM]) workflows

## Outputs

- figure5_cortisol_pattern.png: single comparison panel of the cortisol intensity time series (T0–T48) under the first-ranked and last-ranked workflows
- CSV table of the underlying processed cortisol intensity values per time point and per workflow
- NOREVA ranking file of all 1,968 workflows ordered by multi-criteria performance scores

## How to apply

Format the MTBLS656 plasma time-course peak table (sample rows, feature columns, T0–T48 labels) into NOREVA's unified peak-table format and supply the time-point label file, then run NOREVA's time-course scan (nortimecourseqcall) over the 1,968 candidate workflows combining S1 filtering/imputation (MEI, KNN, MIN, CUT), S2 QCS correction (NON), S3 transformation (LOG, NON) and S4 normalization (QUA-LEV, PAR-SUM), with study assumptions alpha/beta/gamma specified. Verify the ranking output lists MEI+NON+CUT+[QUA-LEV] first and KNN+NON+LOG+[PAR-SUM] last. Extract the cortisol feature's intensity values across all time points T0–T48 from the processed output tables of both the first-ranked and last-ranked workflows. Assemble the two traces into a single Figure 5-style comparison panel showing that the first-ranked workflow preserves the reported circadian rise/fall of cortisol while the last-ranked workflow distorts it, and export the underlying processed values to CSV. Rationale: workflow performance is dataset-dependent, so the biological time-series check provides a study-specific sanity test of the multi-criteria ranking.

## Related tools

- **NOREVA** (Runs the time-course workflow scan (nortimecourseqcall) over 1,968 S1–S4 method combinations, ranks all workflows, and emits the processed peak-table outputs from which the cortisol time series is extracted) — https://github.com/idrblab/NOREVA
- **R** (Execution environment for the NOREVA R-based protocol and for plotting the first-ranked vs last-ranked cortisol traces into the comparison figure) — https://www.r-project.org/
- **NOREVA_exampledata** (Source of the MTBLS656 plasma time-course peak table used as the input for the cortisol circadian-pattern validation) — https://idrblab.org/noreva/NOREVA_exampledata.zip

## Examples

```
nortimecourseqcall(fileName, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- The NOREVA ranking file lists MEI+NON+CUT+[QUA-LEV] as the first-ranked and KNN+NON+LOG+[PAR-SUM] as the last-ranked of the 1,968 workflows
- The first-ranked workflow's cortisol trace qualitatively matches the reported circadian rise/fall pattern across T0–T48
- The last-ranked workflow's cortisol trace visibly fails to reproduce the circadian pattern
- The comparison panel and the accompanying CSV contain the same underlying processed intensity values for both workflows
- All 1,968 workflows receive multi-criteria performance scores and are fully ranked (no unscanned combinations remain)

## Limitations

- Scanning thousands of workflows is time-consuming: hundreds to thousands of minutes are required even with parallel computing and memory management
- Criterion Cc is unsuitable for datasets with fewer than 6 samples in one time point or class
- Manually standardizing the peak table to NOREVA's unified format is time-consuming due to diverse raw-data file organizations, and successful application depends on upfront experimental design adopting both QCSs and ISs
- Some partial assessment functions generate output files with names identical to earlier steps, which can overwrite existing outputs and produce misleading results

## Evidence

- [discussion] Data normalization adjusts values and statistical distributions of metabolomic data to remove undesired variation while preserving existing variability: "Data normalization adjusts values and statistical distributions of metabolomic data to remove undesired variation while preserving existing variability"
- [discussion] Manually standardize a peak table file of the unified format by preserving the rows (samples provided by name) and columns: [evidence span withheld — anchor status: not_found]
- [discussion] The time cost of the scan: "the time spent applying the NOREVA protocol varies from hundreds to thousands of minutes, which demands a significant acceleration"
- [discussion] Small-sample time-point limitation for criterion Cc: "the datasets of small sample size (<6) in one timepoint or class are unsuitable to be assessed using criterion Cc"
- [supplementary] Example data availability: "All data used in this publication have been made available on the NOREVA website (https://idrblab.org/noreva/NOREVA_exampledata.zip)"
- [readme] Peak-table preparation function: [evidence span withheld — anchor status: external_doc]
- [readme] Parallel computing in NOREVA 2.1.1: [evidence span withheld — anchor status: external_doc]
