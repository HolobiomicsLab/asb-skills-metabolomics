---
name: qc-report-html-generation
description: Use when a DIA-MS QC pipeline must deliver per-file, human-readable QC
  output (HTML report plus matrices) summarizing extracted metrics and qualified/unqualified
  LC/MS predictions.
when_to_use_negative:
- The pipeline goal is raw metric extraction or DIA-NN data processing rather than
  reporting — the metrics must already be extracted and predicted before report generation
- A downstream statistical workflow (R/Python) will consume the QC values programmatically
  — in that case the flat matrices alone suffice and the HTML rendering step adds
  no analytical value
- The input is DDA data or an SRM/targeted assay; iDIA-QC's report content is defined
  for DIA files from Orbitrap, TripleTOF, and timsTOF serial instruments
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: iDIA-QC
  role: Python GUI suite whose Output pane renders the HTML QC report and flat metric
    matrices and dispatches email/WeChat notifications
  repo: github.com/guomics-lab/iDIA-QC
- name: DIA-NN
  role: Supplies the peptide/protein quantification results from which most of the
    reported QC metrics are extracted before rendering
- name: msConvert
  role: Provides the mzXML conversions from which metrics F3 and F5 are parsed for
    the report
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:213ec7890cb019a95023ed0fab08e6afc5b1f73aa6fb7acb1c2160e8ca10cab6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - Orbitrap
  - TripleTOF
  - iDIA-QC
  - msConvert
  - timsTOF Pro
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# qc-report-html-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates per-file, self-contained HTML quality-control reports and flat metric matrices from iDIA-QC's extracted DIA-MS QC metrics and machine-learning predictions, so that LC/MS status and fault causes are viewable without R/Python.

## When to use

Use when a DIA-MS QC pipeline must deliver per-file, human-readable QC output (HTML report plus matrices) summarizing extracted metrics and qualified/unqualified LC/MS predictions.

## When NOT to use

- The pipeline goal is raw metric extraction or DIA-NN data processing rather than reporting — the metrics must already be extracted and predicted before report generation
- A downstream statistical workflow (R/Python) will consume the QC values programmatically — in that case the flat matrices alone suffice and the HTML rendering step adds no analytical value
- The input is DDA data or an SRM/targeted assay; iDIA-QC's report content is defined for DIA files from Orbitrap, TripleTOF, and timsTOF serial instruments

## Inputs

- Extracted 17 numerical QC metric values per DIA raw file (from mzXML, DIA-NN outputs, and timsTOF MGF files)
- Machine-learning prediction results (qualified/unqualified for LC and MS performance, with fault-cause guidance)
- User-specified output directory path
- Notification settings (email address or WeChat token ID)

## Outputs

- Self-contained HTML QC report per input raw file, showing all metrics, LC/MS predictions, and longitudinal quality visualization
- Flat metric matrices usable without statistical software such as R/Python
- Optional email or WeChat notification of prediction results to administrators

## How to apply

After converting vendor raw files (.raw/.d/.wiff) to mzXML with msConvert, analyzing them with DIA-NN (v1.7.15), and extracting the 17 numerical QC metrics (F3 and F5 from mzXML, F15 from timsTOF Pro MGF files, the rest from DIA-NN outputs), pass the metric values into the pretrained machine-learning model that assigns a qualified/unqualified evaluation to LC and MS performance with fault-cause guidance. Render the result as a self-contained HTML report covering each raw file, and additionally write the extracted metrics as flat matrices designed to be usable without statistical software such as R/Python. Optionally configure the output notification channel (email or WeChat with token ID) so administrators promptly receive prediction results and visualizations. Confirm completeness against the validation rule that every input raw file yields an HTML report containing all 17 metrics and both LC and MS predictions.

## Related tools

- **iDIA-QC** (Python GUI suite whose Output pane renders the HTML QC report and flat metric matrices and dispatches email/WeChat notifications) — github.com/guomics-lab/iDIA-QC
- **DIA-NN** (Supplies the peptide/protein quantification results from which most of the reported QC metrics are extracted before rendering)
- **msConvert** (Provides the mzXML conversions from which metrics F3 and F5 are parsed for the report)

## Evaluation signals

- Every input raw file (.raw/.d/.wiff) yields one HTML QC report containing all 17 metrics and both LC and MS predictions
- Per-file analysis time stays under 5 minutes
- The flat matrix output opens and is interpretable without statistical software such as R/Python
- Each report's prediction matches the machine-learning model's qualified/unqualified evaluation for LC and MS
- Notification channel (email or WeChat with token ID) delivers the prediction results and visualization when configured

## Limitations

- The instrument-type choice must be correctly assigned per file (Orbitrap, TripleTOF, or timsTOF Pro); the report content assumes the study's instrument families and may not cover newly emerging instruments
- Report metric set covers the 15 practically applicable metrics compiled by the study; metrics from prior schemes (e.g. Rudnick et al. 2010, 46 metrics) are not included
- WeChat notification requires obtaining a token ID via the pushplus public account

## Evidence

- [other] Feed the extracted metric values into the pretrained machine learning model, which assigns a qualified/unqualified evaluation to LC and MS performance and generates fault-cause guidance.: [evidence span withheld — anchor status: not_found]
