---
name: auto-tracked-ion-metric-tracking
description: Use when precursor-ion peak metrics have been extracted for auto-tracked
  and user-specified ions and the operator needs packaged CSV reports and JPG/PDF
  figures to inspect and track ion metrics over a batch of MS runs.
when_to_use_negative:
- Input is raw MS data that has not yet been converted and peak metrics extracted
  — the reporting step operates on extracted metrics tables, not on raw vendor formats
  (e.g. Agilent 'd', Thermo '.raw', Bruker 'd', mzML).
- The goal is identification-based QC (e.g. counting spectral matches in proteomics);
  PeakQC is explicitly positioned as a precursor-ion-metric-based alternative because
  identification-based tools 'do little to pinpoint the specific causes of performance
  deterioration'.
- No batch of MS runs exists to track across — single-run inspection without batch-level
  metric tracking is not the target use case described in the article.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: 'Runs the metric reporting and visualization step: generates CSV reports and
    JPG/PDF figures for auto-tracked and user-specified ion metrics.'
  repo: github.com/pnnl/IonToolPack
- name: MZA
  role: Upstream data-conversion tool in the pipeline (raw data conversion precedes
    metric extraction and reporting).
- name: IonToolPack
  role: Software suite housing PeakQC; provides the no-install GUI entry point where
    raw MS files are imported and 'Process' is clicked to run the pipeline.
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:8fb14a89340391db6e5e65da8d3f5461d049f7bfaecbf8c1d9274982b31dbe89
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IonToolPack
  - MZA
  - PeakQC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# auto-tracked-ion-metric-tracking

## Summary

PeakQC's reporting and visualization step that packages extracted precursor-ion peak metrics (mass, retention-time and arrival-time errors) for auto-tracked and user-specified ions into CSV reports and JPG/PDF figures per ion and per metric, so instrument operators can inspect and track MS data quality across a batch.

## When to use

Use when precursor-ion peak metrics have been extracted for auto-tracked and user-specified ions and the operator needs packaged CSV reports and JPG/PDF figures to inspect and track ion metrics over a batch of MS runs.

## When NOT to use

- Input is raw MS data that has not yet been converted and peak metrics extracted — the reporting step operates on extracted metrics tables, not on raw vendor formats (e.g. Agilent 'd', Thermo '.raw', Bruker 'd', mzML).
- The goal is identification-based QC (e.g. counting spectral matches in proteomics); PeakQC is explicitly positioned as a precursor-ion-metric-based alternative because identification-based tools 'do little to pinpoint the specific causes of performance deterioration'.
- No batch of MS runs exists to track across — single-run inspection without batch-level metric tracking is not the target use case described in the article.

## Inputs

- Extracted precursor-ion peak metrics table (mass error, retention time error, arrival time error) for auto-tracked ions
- Extracted peak metrics for user-specified ions (theoretical or reference values)
- User-specified ion list with expected fragment ions and intensities (for MS2 diagnostic figures)
- MS1/MS2 data supporting mobiligram extraction (arrival time in input, IMS separation in MS data)

## Outputs

- Per-ion and per-metric CSV summary reports
- JPG figures visualizing each ion metric across the batch
- PDF figures for tracking ion metrics
- Diagnostic plots for fragmentation spectra (MS2)
- Mobiligrams (ion mobility traces) for user-specified ions

## How to apply

After the upstream metric-extraction step produces peak metrics (errors in mass, retention time, arrival time) for the topmost abundant MS1 ions per sample group (the 'auto-tracked' ions) and for user-specified ions given by theoretical or reference values, load the extracted-metrics table (e.g. with pandas) and organize it per ion and per metric. Use PeakQC's reporting module to write per-ion, per-metric summary reports in comma-separated value (CSV) format, and use PeakQC's plotting module to render JPG and PDF figures that visualize each ion metric across the batch of MS runs, with diagnostic plots for fragmentation spectra also generated when MS2 signals are extracted. Arrange all CSV reports and JPG/PDF figures into navigable output folders structured per ion and per metric so the operator can inspect each ion's metric trend. Correctness is judged by confirming that every auto-tracked and user-specified ion has both a CSV report and a rendered JPG/PDF figure, and that each figure correctly displays the tracked metric values for that ion across the batch.

## Related tools

- **PeakQC** (Runs the metric reporting and visualization step: generates CSV reports and JPG/PDF figures for auto-tracked and user-specified ion metrics.) — github.com/pnnl/IonToolPack
- **MZA** (Upstream data-conversion tool in the pipeline (raw data conversion precedes metric extraction and reporting).)
- **IonToolPack** (Software suite housing PeakQC; provides the no-install GUI entry point where raw MS files are imported and 'Process' is clicked to run the pipeline.) — https://github.com/pnnl/IonToolPack

## Examples

```
Double click IonToolPack.exe → import raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) → click "Process" — PeakQC generates CSV reports and JPG/PDF figures for auto-tracked and user-specified ion metrics.
```

## Evaluation signals

- Every auto-tracked and user-specified ion has a CSV report present in the output folders.
- Every auto-tracked and user-specified ion has a rendered JPG and PDF figure in the output folders.
- Figures correctly display the tracked metric values (mass error, retention time error, arrival time error) for the corresponding ion across the batch — qualitative visual QC check.
- Reports are formatted as comma-separated values (CSV) per ion and per metric, organized into navigable output folders.
- When expected fragment ions and intensities are provided, diagnostic figures for each molecule and each MS run appear in the outputs.

## Limitations

- The reporting step depends on the upstream extraction of peak metrics; raw data conversion is a required earlier step (total execution time is counted 'from raw data conversion to error heatmap generation').
- Metric reporting is omics-agnostic but ion-based: without sufficient abundant ions in MS1 data, auto-tracking of the 'topmost abundant ions more frequent per sample group' may not surface meaningful metrics.
- MS2 diagnostic figures require the user to supply expected fragment ions and intensities for user-specified ions; if these are not included, fragment-level figures cannot be generated.
- Visual figure-based QC is a qualitative check of whether figures 'correctly display the tracked metric values'.
- The article notes that existing QC tool limitations include tools 'tailored to a single omics type, they can be difficult to deploy, they may lack active maintenance'; PeakQC addresses deployment via a no-install executable (IonToolPack.exe), but the article does not give pip/conda install instructions.

## Evidence

- [results] Peak metrics are then extracted for both auto-tracked and user-specified ions (i.e., theoretical or reference values), and various reports (CSV) and figures (JPG and PDF) are generated.: "Peak metrics are then extracted for both auto-tracked and user-specified ions (i.e., theoretical or reference values), and various reports (CSV) and figures (JPG and PDF) are generated."
- [abstract] Diagnostic plots for fragmentation spectra are also generated.: "Diagnostic plots for fragmentation spectra are also generated."
- [results] ‘mobiligrams’ (i.e., ion mobility traces) are extracted for user-specified ions if the AT is included in the input and the MS data contains IMS separation: "‘mobiligrams’ (i.e., ion mobility traces) are extracted for user-specified ions if the AT is included in the input and the MS data contains IMS separation"
- [results] PeakQC extracts signals from the MS2 data and generates diagnostic figures for each molecule and each MS run for the user-specified ions, if their expected fragment ions and intensities are included: "PeakQC extracts signals from the MS2 data and generates diagnostic figures for each molecule and each MS run for the user-specified ions, if their expected fragment ions and intensities are included"
- [other] PeakQC's reporting step generates reports formatted as comma-separated values (CSV) and figures (JPG, PDF) to visualize and track different ion metrics for the auto-tracked ions found and for the user-specified ions.: [evidence span withheld — anchor status: not_found]
- [readme] Import raw MS files and click “Process”: "Import raw MS files and click “Process”"
- [readme] See example input and output files in test_data: "See example input and output files in test_data"
