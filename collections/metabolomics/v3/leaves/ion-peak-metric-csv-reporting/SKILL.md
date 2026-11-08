---
name: ion-peak-metric-csv-reporting
description: Use when extracted precursor-ion peak metrics (mass error, RT error,
  arrival time error) for auto-tracked and user-specified ions need to be packaged
  into CSV reports and JPG/PDF figures for operator inspection and batch tracking.
when_to_use_negative:
- Input is already a finished QC report or feature list comparison — use Comparador
  in IonToolPack instead of re-running PeakQC reporting.
- The goal is targeted abundance quantitation rather than QC metric reporting — PeakQuant
  handles targeted MS1 peak abundance extraction.
- Data are lipid identification spectra needing library matching rather than QC metric
  extraction — TandemMatch or LIQUID is the appropriate path.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: PeakQC
  role: 'Core QC pipeline: performs PCA on MS1 time-versus-m/z images, auto-tracks
    ions, extracts peak metrics, and generates CSV reports and JPG/PDF figures'
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI wrapper that loads raw MS files in multiple vendor formats and runs PeakQC
    (and companion tools Mirador, TandemMatch, PeakQuant, Comparador) without installation
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:e679640417ded986819ebf26e4f14e729d55ae8afdc921b5dd120cdc3b8f1513
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# ion-peak-metric-csv-reporting

## Summary

Generates per-ion CSV summary reports and JPG/PDF diagnostic figures from precursor-ion peak metrics (mass error, retention time error, arrival time error) for both auto-tracked MS1 ions and user-specified ions, using PeakQC's reporting and plotting modules within the IonToolPack.

## When to use

Use when extracted precursor-ion peak metrics (mass error, RT error, arrival time error) for auto-tracked and user-specified ions need to be packaged into CSV reports and JPG/PDF figures for operator inspection and batch tracking.

## When NOT to use

- Input is already a finished QC report or feature list comparison — use Comparador in IonToolPack instead of re-running PeakQC reporting.
- The goal is targeted abundance quantitation rather than QC metric reporting — PeakQuant handles targeted MS1 peak abundance extraction.
- Data are lipid identification spectra needing library matching rather than QC metric extraction — TandemMatch or LIQUID is the appropriate path.

## Inputs

- Raw MS files: Agilent '.d', Thermo '.raw', Bruker '.d', or mzML (LC-MS, LC-IMS-MS, direct infusion; DDA or DIA)
- Extracted precursor-ion peak metrics for auto-tracked ions (topmost abundant MS1 ions most frequent per sample group)
- Extracted peak metrics for user-specified ions given as theoretical or reference values with expected fragment ions/intensities for MS2 figures
- Input m/z, RT, and arrival time (AT) values for user-specified targets

## Outputs

- Per-ion and per-metric CSV summary reports
- JPG and PDF figures visualizing each ion metric across the batch of MS runs
- Diagnostic figures for fragmentation spectra (MS2 mirror plots per molecule per MS run)
- Navigable output folders structured per ion and metric
- Error heatmaps from Isolation Forest outlier detection on individual peak metrics per sample group

## How to apply

Load the extracted peak-metric table (errors in mass, retention time, arrival time) from upstream metric extraction and merge metrics for the auto-tracked ions with those for the user-specified ions given by theoretical or reference values. Run PeakQC's reporting module to write per-ion and per-metric CSV summary reports, and its plotting module to render JPG and PDF figures visualizing each ion metric across the batch of MS runs (infusion, LC-MS, or LC-IMS-MS, DDA or DIA). Organize the CSV reports and JPG/PDF figures into navigable output folders structured per ion and metric. Run the full pipeline from raw data conversion through error heatmap generation in one batch so per-run execution times are also captured. The rationale is to give the instrument operator an omics-agnostic, file-based QC record that complements the PCA-on-MS1 global assessment and Isolation Forest outlier detection, so declining performance can be traced to specific metrics like mass calibration or chromatography.

## Related tools

- **PeakQC** (Core QC pipeline: performs PCA on MS1 time-versus-m/z images, auto-tracks ions, extracts peak metrics, and generates CSV reports and JPG/PDF figures) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI wrapper that loads raw MS files in multiple vendor formats and runs PeakQC (and companion tools Mirador, TandemMatch, PeakQuant, Comparador) without installation) — https://github.com/pnnl/IonToolPack

## Examples

```
Download the latest IonToolPack.exe from github.com/pnnl/IonToolPack/releases, double click it, import raw MS files, and click “Process” to run PeakQC and generate CSV reports and JPG/PDF figures.
```

## Evaluation signals

- Every auto-tracked and user-specified ion has both a CSV report and a rendered JPG/PDF figure in the output folders.
- Figures correctly display the tracked metric values (mass error, RT error, AT error) — verified qualitatively against the extracted metrics table.
- Output folders are navigable and structured per ion and per metric, with no orphaned figures or reports.
- Isolation Forest error heatmaps flag anomalous peak metrics per sample group, consistent with the underlying CSV metric values.
- The report content matches the auto-tracked ion set derived from the most frequent topmost abundant MS1 ions per sample group.

## Limitations

- Early version was GUI-driven within IonToolPack; unattended command-line functionality for large-scale automated QC comparisons across batches over months and years was planned but not yet released.
- Traditional manual QC inspection of spectra is time-consuming and subjective; this skill replaces that but depends on upstream raw data conversion via MZA.
- PeakQC is designed for QC of MS data; it does not perform lipid/protein identification (use TandemMatch, LIQUID) or feature-list comparison (use Comparador).

## Evidence
