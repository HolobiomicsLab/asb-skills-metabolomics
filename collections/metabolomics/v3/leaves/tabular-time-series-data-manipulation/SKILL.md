---
name: tabular-time-series-data-manipulation
description: Use when the input is a table with a time axis and per-trace signal columns
  (e.g. intensities from optical recordings like the bundled Example Data 03), and
  you need to baseline-correct, transform, smooth or quantify traces before plotting
  or export.
when_to_use_negative:
- Input is high-dimensional molecular data (e.g. proteomics/omics matrices) where
  multi-block omics integration or dimensional reduction is needed rather than signal-over-time
  correction.
- Raw image stacks or detector vendor files that are not yet converted into a tabular
  signal-over-time layout — the transformation layer only operates on columns in the
  dataset.
- The task is hypothesis testing across groups (ANOVA, regressions) rather than trace
  manipulation; the statistical tests are toggled by drag & drop from the Analysis
  toolbox, not by column activities.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: Instant Clue
  role: Desktop GUI that loads the time series table, exposes column activities (normalization,
    transformation, imputation, smoothing, rolling window) via the context menu, supports
    interactive AUC determination and charting of signal-over-time traces.
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for the open-source Instant Clue code base; the Python source is freely
    available so the time-series manipulation logic can be inspected or adapted on
    any platform with a working Python version >=3.4.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:63192d5177d70c02f7d95080bffafe6b796a5d21a84b29e774b4dd2ac7964133
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tabular-time-series-data-manipulation

## Summary

Manipulate signal-over-time measurements held in tabular data frames (e.g. optical recordings of POMC neuron activity) by applying column-level transformations such as baseline correction, normalization, smoothing and rolling-window calculations, then interactively determining metrics like area under the curve.

## When to use

Use when the input is a table with a time axis and per-trace signal columns (e.g. intensities from optical recordings like the bundled Example Data 03), and you need to baseline-correct, transform, smooth or quantify traces before plotting or export.

## When NOT to use

- Input is high-dimensional molecular data (e.g. proteomics/omics matrices) where multi-block omics integration or dimensional reduction is needed rather than signal-over-time correction.
- Raw image stacks or detector vendor files that are not yet converted into a tabular signal-over-time layout — the transformation layer only operates on columns in the dataset.
- The task is hypothesis testing across groups (ANOVA, regressions) rather than trace manipulation; the statistical tests are toggled by drag & drop from the Analysis toolbox, not by column activities.

## Inputs

- Tabular time series data frame with a time column and per-trace signal columns (e.g. intensities from optical recordings of POMC neuron activity, as in Example Data 03)
- Excel workbook, tab-delimited .txt, .csv, .xml or compressed .gz/.zip file containing the traces
- Optional categorical condition column for grouping traces

## Outputs

- Baseline-corrected / transformed signal columns added to the data frame (one corrected value per original time point per trace)
- Interactive time-series chart of the traces
- Calculated metrics per trace such as area under the curve (AUC)
- Exported table (e.g. CSV) of corrected traces for downstream use

## How to apply

Load the time series file (Excel, .txt, .csv or compressed formats) into a tabular data frame, then identify the signal columns whose values are measured against the shared time axis. Compute a per-trace baseline (e.g. mean pre-stimulus intensity) and apply it as a subtraction transformation via the context menu's column activities — the same activities layer that covers sorting, string splitting, normalization, transformation, imputation, smoothing and rolling window calculations. Note that if an activity cannot handle missing values, data are automatically pre-filtered before the activity runs, without changing the source table. After correction, trace shapes can be inspected interactively and metrics such as area under the curve (AUC) can be determined, and the corrected traces can be exported for downstream use. All operations operate on columns of the dataset, mirroring the drag & drop/context-menu paradigm of the tool.

## Related tools

- **Instant Clue** (Desktop GUI that loads the time series table, exposes column activities (normalization, transformation, imputation, smoothing, rolling window) via the context menu, supports interactive AUC determination and charting of signal-over-time traces.) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for the open-source Instant Clue code base; the Python source is freely available so the time-series manipulation logic can be inspected or adapted on any platform with a working Python version >=3.4.)

## Examples

```
python3 main.py  # after python3 -m venv env && source env/bin/activate && pip install -r requirements.txt — starts Instant Clue, then load Example Data 03 and apply the baseline-correction column activity via the context menu
```

## Evaluation signals

- After baseline correction, the corrected signal values within each trace's baseline window are centered near zero (offset removed).
- The corrected output has exactly one corrected value per original time point per trace — no rows added or dropped by the transformation itself.
- If the source table contained missing values and the chosen activity cannot handle them, data were automatically pre-filtered before the activity ran while the source data remained unchanged.
- AUC (or other chosen metric) values are determinable interactively from the corrected traces and change reproducibly when the baseline window definition changes.
- Exported file (e.g. baseline_corrected_traces.csv) contains a time column plus one corrected signal column per original trace.

## Limitations

- Only activities exposed via the column context menu are available; more complex per-trace custom operations may require the open-source Python path rather than the GUI.
- The panel of statistical tests and additional data-interpretation activities was still being extended at publication time, so some analyses on corrected traces may not yet be supported in the GUI.
- Activities that cannot handle missing values trigger automatic pre-filtering, which silently reduces the set of rows the activity sees (though the source data are not changed).

## Evidence

- [results] Activities on columns cover normalization/transformation/smoothing/rolling window: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation, imputation of"
- [results] Automatic pre-filtering for activities that cannot handle missing values: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [results] Example Data 03 are optical recordings of POMC neuron activity: "The Example Data 03 (see Methods) are optical recordings of Pro-opiomelanocortin (POMC) neuron activity"
- [intro] Facilitating calculation of metrics like AUC: "(iii) Offering a repertoire of statistical tests like ANOVA, regressions, curve fitting and principal component analysis as well as facilitating calculation of metrics like area under the curve (AUC)"
- [intro] Python source code freely available, requires >=3.4: "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
- [results] Accepted input file types include Excel, txt, csv, xml, gz, zip: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [readme] README: executables available from Releases on GitHub: "Please find the executables for Windows and Mac in the [Releases](https://github.com/hnolCol/instantclue/releases) section."
