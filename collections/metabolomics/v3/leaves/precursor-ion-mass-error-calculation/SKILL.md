---
name: precursor-ion-mass-error-calculation
description: Use when performing omics-agnostic MS quality control on raw MS runs
  (LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion) and you need per-ion error metrics
  (mass, RT, AT/1/K0) plus spectral counts and TIC to detect instrument drift or outliers.
when_to_use_negative:
- When the goal is proteomics identification-based QC (e.g., spectral-match counts
  from a search engine) — PeakQC is omics-agnostic and its metrics do not rely on
  identifications, but this skill specifically targets precursor-ion error metrics
  rather than identification-based QC.
- When input is already a processed feature table without raw MS signal — the metric
  extraction requires the raw chromatographic/mobility peaks to locate and measure.
- When only global, sample-level QC is needed (e.g., PCA on time-versus-m/z MS1 images)
  without per-ion error breakdown — the broader PeakQC PCA step covers that, not this
  metric-extraction skill.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Performs the automated extraction and inspection of precursor-ion peak metrics
    (mass error, retention time, arrival time) for auto-tracked and user-specified
    ions, emitting CSV reports and JPG/PDF figures.
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI suite that hosts PeakQC, imports the supported raw MS formats (Agilent
    'd', Thermo '.raw', Bruker 'd', mzML), and runs the QC workflow including metric
    extraction.
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:f4475da25a1eb27d623ddc582e246e515ff77199b49fab1dd8d64c1ae696fdf0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# precursor-ion-mass-error-calculation

## Summary

Automated extraction of precursor-ion peak metrics — mass error, retention-time error, and arrival-time (AT/1/K0) error — for both auto-tracked MS1 ions and user-specified ion targets, as implemented in the PeakQC QC pipeline of IonToolPack.

## When to use

Use when performing omics-agnostic MS quality control on raw MS runs (LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion) and you need per-ion error metrics (mass, RT, AT/1/K0) plus spectral counts and TIC to detect instrument drift or outliers.

## When NOT to use

- When the goal is proteomics identification-based QC (e.g., spectral-match counts from a search engine) — PeakQC is omics-agnostic and its metrics do not rely on identifications, but this skill specifically targets precursor-ion error metrics rather than identification-based QC.
- When input is already a processed feature table without raw MS signal — the metric extraction requires the raw chromatographic/mobility peaks to locate and measure.
- When only global, sample-level QC is needed (e.g., PCA on time-versus-m/z MS1 images) without per-ion error breakdown — the broader PeakQC PCA step covers that, not this metric-extraction skill.

## Inputs

- Raw MS files in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format (LC-MS, LC-IMS-MS, DDA or DIA, with/without fragmentation, or direct infusion)
- Auto-tracked ion list (topmost abundant ions per sample group produced by the prior PeakQC ion-detection step)
- User-specified ion targets carrying theoretical or reference m/z, retention time, and arrival time (or 1/K0) values

## Outputs

- CSV report of peak metrics per ion per run: mass error, retention-time error, arrival-time (or 1/K0) error where applicable, number of spectra, and TIC
- JPG and PDF diagnostic figures for the extracted ion peaks
- Mobiligrams (ion mobility traces) for user-specified ions when AT is supplied and the MS data contain IMS separation

## How to apply

Load raw MS files in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format into IonToolPack and run the PeakQC pipeline. For each auto-tracked ion (the topmost abundant MS1 ions identified per sample group) or each user-specified ion with theoretical/reference m/z, RT, and arrival-time (or 1/K0) values, locate the detected chromatographic or mobility peak in each run and compute the error in mass, retention time, and arrival time relative to the reference. Also tabulate the number of spectra in which each ion appears and its total ion current (TIC). Aggregate the per-ion, per-run metrics into CSV reports and accompany them with JPG/PDF diagnostic figures; when the input supplies AT and the data contain IMS separation, extract mobiligrams for the specified ions. The rationale is that these precursor-ion metrics expose specific causes of performance deterioration (mass calibration, chromatography, sensitivity) that identification-count-based QC cannot pinpoint.

## Related tools

- **PeakQC** (Performs the automated extraction and inspection of precursor-ion peak metrics (mass error, retention time, arrival time) for auto-tracked and user-specified ions, emitting CSV reports and JPG/PDF figures.) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI suite that hosts PeakQC, imports the supported raw MS formats (Agilent 'd', Thermo '.raw', Bruker 'd', mzML), and runs the QC workflow including metric extraction.) — https://github.com/pnnl/IonToolPack

## Examples

```
Double click IonToolPack.exe → import raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) → click “Process” to run PeakQC and generate the peak-metrics CSV and JPG/PDF figures
```

## Evaluation signals

- Check that the CSV report contains columns for mass error, RT error, and (where AT was supplied and IMS data are present) AT/1/K0 error for every tracked ion and run.
- Confirm spectral count and TIC values are present and finite for each ion-peak row.
- Verify the report is structurally consistent with the example output files in the IonToolPack test_data directory.
- Compare extracted RT and mass errors against the user-supplied reference values to confirm the sign and magnitude are sensible for the instrument's calibration state.
- For IMS data, confirm mobiligrams are produced only for user-specified ions whose AT was included in the input.

## Limitations

- PeakQC supports multiple acquisition schemes, but existing QC tools were noted to often be tailored to a single omics type or lack support for hybrid platforms like ion mobility spectrometry; this skill inherits those constraints through the tool's own supported format and acquisition list.
- Diagnostic figures and metrics are generated only for ions that are either auto-tracked or explicitly specified; ions outside those sets are not evaluated.
- The paper's GitHub README provides no standalone pip/conda installation instructions — IonToolPack is distributed as a Windows executable and requires no installation, which may limit deployment on non-Windows environments.

## Evidence
