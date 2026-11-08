---
name: output-folder-organization-for-qc-reports
description: Use when per-ion peak metrics for auto-tracked and user-specified ions
  have already been extracted and must be reported as CSV summaries plus JPG/PDF figures
  organized per ion and per metric for operator inspection.
when_to_use_negative:
- The goal is global batch quality assessment rather than per-ion reporting — PeakQC's
  PCA on time-versus-m/z MS1 images covers that and does not require per-ion report
  folders
- Peak metrics have not yet been extracted (raw MS data still needs conversion/processing);
  reporting is a downstream packaging step
- Only fragmentation-spectra diagnostic plots or ion-mobility mobiligrams are needed,
  not per-ion peak-metric reports
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Provides the reporting module (CSV summary reports) and plotting module (JPG/PDF
    figures) used to package extracted ion metrics into per-ion/per-metric output
    folders
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI suite housing PeakQC; imports raw MS files and runs the pipeline whose
    report/figure outputs are organized in the output folders (example inputs/outputs
    in test_data)
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:1595b63a8ecd91545feee822c58e050bb8e9877908062949ab704a172b4ce2cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# output-folder-organization-for-qc-reports

## Summary

Package PeakQC's extracted precursor-ion peak metrics (mass error, retention time error, arrival time error) into per-ion and per-metric CSV summary reports and JPG/PDF figures arranged in navigable output folders so an instrument operator can inspect and track QC across an MS batch.

## When to use

Use when per-ion peak metrics for auto-tracked and user-specified ions have already been extracted and must be reported as CSV summaries plus JPG/PDF figures organized per ion and per metric for operator inspection.

## When NOT to use

- The goal is global batch quality assessment rather than per-ion reporting — PeakQC's PCA on time-versus-m/z MS1 images covers that and does not require per-ion report folders
- Peak metrics have not yet been extracted (raw MS data still needs conversion/processing); reporting is a downstream packaging step
- Only fragmentation-spectra diagnostic plots or ion-mobility mobiligrams are needed, not per-ion peak-metric reports

## Inputs

- Extracted precursor-ion peak metrics table (mass error, retention time error, arrival time error) for auto-tracked and user-specified ions, loadable with pandas
- User-specified ion list with theoretical or reference values (optionally expected fragment ions/intensities and arrival times)
- Sample-group designations for the batch of MS runs (raw files in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format)

## Outputs

- Per-ion and per-metric CSV summary reports
- JPG and PDF figures visualizing each ion metric across the batch of MS runs
- Navigable output folders structured per ion and per metric

## How to apply

Load the extracted peak-metrics table (mass error, retention time error, arrival time error) with pandas and merge the metrics for the topmost-abundant MS1 'auto-tracked' ions with those for user-specified ions given by theoretical or reference values. Call PeakQC's reporting module to write per-ion and per-metric CSV summary reports, and PeakQC's plotting module to render JPG and PDF figures visualizing each ion metric across the batch of MS runs. Organize every CSV report and JPG/PDF figure into navigable output folders structured per ion and per metric so that the instrument operator can browse from an ion to all its tracked metrics. Validation is a qualitative visual QC check that each ion has both a report and a rendered figure and that the figures correctly display the tracked metric values.

## Related tools

- **PeakQC** (Provides the reporting module (CSV summary reports) and plotting module (JPG/PDF figures) used to package extracted ion metrics into per-ion/per-metric output folders) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI suite housing PeakQC; imports raw MS files and runs the pipeline whose report/figure outputs are organized in the output folders (example inputs/outputs in test_data)) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # launch GUI, import raw MS files (Thermo .raw / mzML / Agilent d / Bruker d), click "Process"; see test_data for example outputs
```

## Evaluation signals

- Completeness check: every auto-tracked and user-specified ion in the batch has both a CSV report and a rendered JPG/PDF figure in the output folders
- Qualitative visual QC: each figure correctly displays the tracked metric values (mass error, retention time error, arrival time error) for its ion
- Schema check: CSV reports load with pandas and carry the expected per-metric columns per sample group
- Navigability check: folders are structured per ion and per metric with no orphan or misplaced report/figure files
- Consistency check: generated outputs match the example input/output files in the IonToolPack test_data directory

## Limitations

- Reporting is downstream of metric extraction; missing upstream metrics (e.g., no IMS data means no arrival-time metrics) leave gaps in the per-ion report folders
- Only auto-tracked and user-specified ions are reported — ions outside these sets get no CSV/figure output
- Distribution is a Windows IonToolPack.exe requiring manual GUI interaction; explicit pip/conda/docker installation instructions and a changelog are absent from the repository

## Evidence
