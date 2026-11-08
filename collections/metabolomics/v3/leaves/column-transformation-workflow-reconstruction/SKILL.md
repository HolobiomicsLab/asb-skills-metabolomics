---
name: column-transformation-workflow-reconstruction
description: Use when a tabular dataset (Excel/txt/csv/xml/gz/zip) needs per-column
  transformations such as baseline correction of time-series signal traces, AUC calculation,
  or normalization expressed as a repeatable column-activity sequence.
when_to_use_negative:
- The data are high-dimensional omics matrices needing batch-effect correction via
  pyComBat or statistical tests (ANOVA, PCA) rather than column-level signal transformations
- The input is not a tabular signal-over-time frame (e.g. image files or figure assets
  destined for subplot assembly)
- The transformation must handle missing values itself — activities that cannot handle
  missing values auto-filter the data first, so row loss is expected
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Instant Clue
  role: Desktop GUI (PyQt6-based, Python >=3.4) in which column activities (transformation,
    normalization, smoothing, AUC) are applied via the context menu and exported;
    source code and executables from the Releases section
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: 'Runtime for running Instant Clue from source: virtualenv setup, requirements.txt
    install, and launching main.py'
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:a7fb15b328af9b5024169398df0692cae5cba26a375050e7219545a6298ba2bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# column-transformation-workflow-reconstruction

## Summary

Reconstructs per-column computational workflows (sorting, string splitting, normalization, transformation, imputation, smoothing, rolling windows) as reproducible transformation pipelines in Instant Clue, e.g. baseline-correcting POMC neuron optical recording traces from bundled Example Data 03.

## When to use

Use when a tabular dataset (Excel/txt/csv/xml/gz/zip) needs per-column transformations such as baseline correction of time-series signal traces, AUC calculation, or normalization expressed as a repeatable column-activity sequence.

## When NOT to use

- The data are high-dimensional omics matrices needing batch-effect correction via pyComBat or statistical tests (ANOVA, PCA) rather than column-level signal transformations
- The input is not a tabular signal-over-time frame (e.g. image files or figure assets destined for subplot assembly)
- The transformation must handle missing values itself — activities that cannot handle missing values auto-filter the data first, so row loss is expected

## Inputs

- Tabular data frame with a time axis column and per-trace signal/intensity columns (e.g. Example Data 03 optical recordings of POMC neuron activity)
- Excel (.xlsx), tab-delimited (.txt), .csv, .xml, .gz or .zip source files loadable via Instant Clue's data-upload module
- Defined pre-stimulus baseline window per trace (column indices or time range)

## Outputs

- Baseline-corrected trace table (e.g. baseline_corrected_traces.csv) with one corrected value per original time point per trace
- Optionally interactive AUC values determined per corrected trace
- Recorded sequence of column activities (sorting, splitting, normalization, transformation, smoothing) applied to the dataset

## How to apply

Load the source table through Instant Clue's data-upload module, which accepts Excel, .txt, .csv, .xml and compressed .gz/.zip files — for the time-series use case this is the bundled Example Data 03 of POMC neuron optical activity. Identify the signal columns (per-trace intensities) and the pre-stimulus baseline window per trace, then compute the per-trace mean baseline intensity. Apply the transformation (subtracting the per-trace baseline from every time point) as a column activity via the context menu, the same mechanism the paper describes for sorting, string splitting/replacement, normalization, transformation, imputation, smoothing and rolling-window calculations; note that activities that cannot handle missing values trigger automatic pre-filtering without altering the source data. Chain additional column activities as needed (e.g. interactive area-under-the-curve determination on corrected traces), then export the corrected frame (time column plus corrected signal columns) for validation.

## Related tools

- **Instant Clue** (Desktop GUI (PyQt6-based, Python >=3.4) in which column activities (transformation, normalization, smoothing, AUC) are applied via the context menu and exported; source code and executables from the Releases section) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for running Instant Clue from source: virtualenv setup, requirements.txt install, and launching main.py) — https://github.com/hnolCol/instantclue

## Examples

```
python3 -m venv env; source env/bin/activate; pip install -r requirements.txt; python3 main.py
```

## Evaluation signals

- Corrected signal values within each trace's baseline window are centered near zero, confirming the per-trace baseline offset was removed
- Output file baseline_corrected_traces.csv contains exactly one corrected value per original time point per trace (no row count change)
- The time column passes through unchanged so each corrected trace remains aligned to its original time axis
- Missing-value pre-filtering did not silently alter the source dataset — only the activity submission view was filtered
- Every transformation step is expressible as a named column activity (context-menu action) so the workflow can be replayed

## Limitations

- Activities that cannot handle missing values automatically pre-filter the data before submission, which can silently drop rows from the analysis view
- Transformation and normalization operate per column; multi-column or cross-trace operations must be encoded as individual column activities
- As of the article, the panel of statistical tests and additional interpretation activities was still planned, so complex downstream statistics may need external tooling
- Running from source has known versioning friction: pyComBat is excluded from requirements.txt because of specific pandas/numpy version requirements

## Evidence

- [results] Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation, imputation of: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation, imputation of"
- [results] signals can be baseline corrected and AUC determined: [evidence span withheld — anchor status: not_found]
- [results] Example Data 03 are POMC neuron optical recordings: "The Example Data 03 (see Methods) are optical recordings of Pro-opiomelanocortin (POMC) neuron activity"
- [results] Missing-value auto-filtering before activities: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [readme] README: executables and tutorial location: "Please find the executables for Windows and Mac in the [Releases](https://github.com/hnolCol/instantclue/releases) section."
