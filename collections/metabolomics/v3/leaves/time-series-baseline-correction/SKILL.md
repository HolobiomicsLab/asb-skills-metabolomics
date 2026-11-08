---
name: time-series-baseline-correction
description: Use when the input is a table of signal intensity columns measured along
  a shared time axis (e.g. intensity traces of neuron activity) and you need to remove
  each trace's baseline offset before computing or exporting area-under-the-curve
  or corrected-signal values.
when_to_use_negative:
- Data are high-dimensional static omics measurements (transcriptomics/proteomics
  expression matrices) with no time axis - use Instant Clue's clustering, PCA or multi-block
  integration instead.
- Signal traces already have zero-offset baseline correction applied or contain only
  categorical/densitometric measurements (immunoblot, qPCR) with no continuous time
  dimension.
- You need batch-effect correction across microarray expression data - Instant Clue's
  combat integration (pyComBat) targets that, not per-trace signal baseline removal.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3070
- http://edamontology.org/topic_3292
tools:
- name: Instant Clue
  role: Desktop GUI whose time series module baseline-corrects signals over a time
    axis and lets the area under the curve be determined interactively; the same context-menu
    column activities (normalization, transformation, smoothing, rolling window calculations)
    provide the baseline subtraction mechanism.
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for running Instant Clue from source when the packaged executables
    are unavailable; the time series baseline-correction activities are exercised
    inside the Python-based application.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:d9759ba0d96a8aa3f59d0d964247bc002b6b79464e246787aea69466dff65331
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3070
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# time-series-baseline-correction

## Summary

Baseline-correct continuous signal-over-time traces (e.g. optical intensity recordings of POMC neuron activity) by subtracting a per-trace pre-stimulus baseline mean, then interactively determine the area under the corrected curve using Instant Clue's time series module.

## When to use

Use when the input is a table of signal intensity columns measured along a shared time axis (e.g. intensity traces of neuron activity) and you need to remove each trace's baseline offset before computing or exporting area-under-the-curve or corrected-signal values.

## When NOT to use

- Data are high-dimensional static omics measurements (transcriptomics/proteomics expression matrices) with no time axis - use Instant Clue's clustering, PCA or multi-block integration instead.
- Signal traces already have zero-offset baseline correction applied or contain only categorical/densitometric measurements (immunoblot, qPCR) with no continuous time dimension.
- You need batch-effect correction across microarray expression data - Instant Clue's combat integration (pyComBat) targets that, not per-trace signal baseline removal.

## Inputs

- Tabular time series file (Excel, .txt, .csv, .xml, .gz, .zip) with one column per signal trace plus a time column
- Example Data 03: optical recordings of Pro-opiomelanocortin (POMC) neuron activity bundled with Instant Clue

## Outputs

- Baseline-corrected signal columns (per-trace offset removed)
- Interactively determined area under the curve values per trace
- Exported corrected traces (time column plus corrected signal columns) as e.g. CSV

## How to apply

Load the time series file (Instant Clue accepts Excel, tab-delimited .txt, .csv, .xml and compressed .gz/.zip inputs) so each trace occupies a column and the time axis is a separate column. Locate the per-trace pre-stimulus baseline window in the recording, compute the mean signal over that window, and apply the subtract-baseline transformation as a column activity via the context menu (Instant Clue's activities include normalization and transformation on columns). Corrected traces should be centered near zero within the baseline window. Instant Clue's time series module then allows the area under the curve to be determined interactively on the corrected signals. Note that if an activity cannot handle missing values the data are automatically filtered before submission, without changing the source data, so gaps in a trace will silently reduce the rows used for baseline correction.

## Related tools

- **Instant Clue** (Desktop GUI whose time series module baseline-corrects signals over a time axis and lets the area under the curve be determined interactively; the same context-menu column activities (normalization, transformation, smoothing, rolling window calculations) provide the baseline subtraction mechanism.) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for running Instant Clue from source when the packaged executables are unavailable; the time series baseline-correction activities are exercised inside the Python-based application.)

## Examples

```
python3 main.py  # start Instant Clue, load POMC Example Data 03 (.xlsx), context-menu: Transform > subtract baseline mean per trace, then AUC on corrected signals
```

## Evaluation signals

- Within each trace's baseline window the corrected signal values are centered near zero (mean offset removed).
- The corrected trace has exactly one value per original time point per trace - no rows dropped or duplicated by the transformation.
- Exported corrected-traces file contains a time column plus one corrected signal column per original trace.
- Area-under-the-curve read-outs on corrected traces are consistent with the sign/magnitude of the deviation of the signal from the subtracted baseline.
- Source data are unchanged - Instant Clue automatically filters missing values before submission to activities 'without changing the source data'.

## Limitations

- Activities that cannot handle missing values cause automatic pre-filtering of data, so traces with gaps silently contribute fewer rows to the baseline mean and downstream calculations.
- The pre-stimulus baseline window must be chosen by the user; Instant Clue's drag-and-drop GUI does not describe an automatic baseline-window detection method in the article.
- When running from source, the combat package is not in requirements.txt because it has very specific pandas/numpy versioning requirements, and install-time numpy/pandas version-mismatch warnings must be ignored.

## Evidence

- [results] The Example Data 03 (see Methods) are optical recordings of Pro-opiomelanocortin (POMC) neuron activity: "The Example Data 03 (see Methods) are optical recordings of Pro-opiomelanocortin (POMC) neuron activity"
- [results] Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation, imputation of: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation, imputation of"
- [results] if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip): "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [readme] Please be aware that the package [combat](https://github.com/epigenelabs/pyComBat/tree/master/combat) is not in the requirements table because it has very specific versioning requirements of pandas and numpy.: "Please be aware that the package [combat](https://github.com/epigenelabs/pyComBat/tree/master/combat) is not in the requirements table because it has very specific versioning requirements of pandas"
