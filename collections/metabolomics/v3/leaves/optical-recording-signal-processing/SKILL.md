---
name: optical-recording-signal-processing
description: Use when input is a tabular time series of intensity signals (e.g. Example
  Data 03, optical recordings of POMC neuron activity) and traces need baseline correction,
  smoothing, or area-under-the-curve determination.
when_to_use_negative:
- Input is already baseline-corrected or the goal is multi-trace batch/technical bias
  correction — that requires a ComBat-style tool (pyComBat), not per-trace baseline
  subtraction
- The data are not signal-over-time measurements (e.g. densitometry, qPCR or high-dimensional
  molecular data sets without a time axis), so no baseline window over time can be
  defined
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3361
tools:
- name: Instant Clue
  role: 'Interactive time series module: baseline correction of signal intensities,
    AUC determination, and column activities (transformation, smoothing) via the context
    menu; drag & drop visualization of corrected traces'
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for running Instant Clue from source (requirements.txt install, python3
    main.py) when the bundled executables are not used
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:57bde3dbef04a20fe46e66e6519c0a048d52391e902b8a4b507ce036b9f9765d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# optical-recording-signal-processing

## Summary

Baseline correction and interactive analysis (smoothing, AUC) of continuous signal-over-time measurements, e.g. optical recordings of POMC neuron activity, using Instant Clue's time series module on tabular intensity traces.

## When to use

Use when input is a tabular time series of intensity signals (e.g. Example Data 03, optical recordings of POMC neuron activity) and traces need baseline correction, smoothing, or area-under-the-curve determination.

## When NOT to use

- Input is already baseline-corrected or the goal is multi-trace batch/technical bias correction — that requires a ComBat-style tool (pyComBat), not per-trace baseline subtraction
- The data are not signal-over-time measurements (e.g. densitometry, qPCR or high-dimensional molecular data sets without a time axis), so no baseline window over time can be defined

## Inputs

- Bundled Example Data 03 file: optical recordings of POMC neuron activity (Excel-compatible tabular format)
- Tabular time series with a time axis column and one intensity/signal column per trace

## Outputs

- baseline_corrected_traces.csv: one corrected value per original time point per trace, with a time column plus corrected signal columns
- Interactive AUC values per trace (optional)

## How to apply

Load the time series file (Excel, .txt, .csv, .xml, .gz or .zip are accepted upload formats) into Instant Clue's data-upload module so traces sit in per-trace signal columns against a time axis. For each trace, identify the pre-stimulus baseline window and compute the mean baseline intensity over that window. Apply the baseline-correction transformation — subtract the per-trace baseline from every time point — as a column activity via the context menu, which also covers normalization, transformation, imputation, smoothing and rolling window calculations on columns. Optionally determine the area under the curve interactively in the time series module. Export the corrected traces (time column plus corrected signal columns) e.g. to baseline_corrected_traces.csv; missing values are automatically pre-filtered before activities that cannot handle them, without changing the source data.

## Related tools

- **Instant Clue** (Interactive time series module: baseline correction of signal intensities, AUC determination, and column activities (transformation, smoothing) via the context menu; drag & drop visualization of corrected traces) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for running Instant Clue from source (requirements.txt install, python3 main.py) when the bundled executables are not used) — https://github.com/hnolCol/instantclue

## Examples

```
python3 main.py #starts InstantClue (from instantclue/src/main/python after pip install -r requirements.txt)
```

## Evaluation signals

- Corrected signal values within each trace's baseline window are centered near zero (qualitative check that the baseline offset was removed)
- Output baseline_corrected_traces.csv contains exactly one corrected value per original time point per trace (row and trace count preserved)
- Each corrected column equals the original intensity minus that trace's per-trace baseline mean computed over the pre-stimulus window
- Corrected traces re-plotted in Instant Clue align on a common zero baseline so traces with different raw offsets become comparable

## Limitations

- Activities that cannot handle missing values trigger automatic pre-filtering of the data before submission, so the analyzed subset may be smaller than the source data
- Baseline correction is an interactive GUI activity in Instant Clue; installing from source requires combat before requirements.txt due to specific numpy/pandas versioning requirements (ignore the numpy/pandas version mismatch warning)
- The article lists the time series use case among broad data inputs but does not describe automated batch baseline correction across many recordings

## Evidence

- [results] Example Data 03 are optical recordings of POMC neuron activity: "The Example Data 03 (see Methods) are optical recordings of Pro-opiomelanocortin (POMC) neuron activity"
- [results] Signals can be baseline corrected and AUC determined interactively: [evidence span withheld — anchor status: not_found]
- [results] Column activities via context menu include normalization and transformation: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation"
- [results] Data upload formats supported by Instant Clue: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [readme] Executables and source availability for Instant Clue: "Please find the executables for Windows and Mac in the [Releases](https://github.com/hnolCol/instantclue/releases) section."
