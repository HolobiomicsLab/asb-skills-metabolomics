---
name: qc-metrics-report-generation
description: Use when raw vendor MS files (Agilent 'd', Thermo '.raw', Bruker 'd',
  or mzML) from LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion runs need per-ion quality
  metrics and outlier-flagged QC reports before downstream omics analysis.
when_to_use_negative:
- Input is already a processed feature table or identification result rather than
  raw instrument files — PeakQC's metrics require raw spectra to locate peaks and
  compute errors
- Only identity-based QC (total spectral match counts) is needed to detect general
  performance drift without pinpointing causes like chromatography, sensitivity, or
  mass calibration
- The data is not MS acquisition data (e.g. genomics or imaging) — PeakQC is omics-agnostic
  across MS omics types but operates on mass spectrometry acquisitions only
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: 'Executes the QC pipeline: extracts precursor-ion peak metrics, runs Isolation
    Forest outlier detection, and generates CSV reports and JPG/PDF figures'
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Suite that houses PeakQC and reads multiple instrument formats via a GUI;
    provides the auto-tracked ion lists PeakQC consumes
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:3340242b0aef8ffc078cb13f84b944590824a608a09009a115bc295f395a2ea1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# qc-metrics-report-generation

## Summary

A PeakQC/IonToolPack skill that extracts per-ion QC metrics (mass error, retention-time error, arrival-time or 1/K0 error, spectral counts, and TIC) for both auto-tracked and user-specified precursor ions, then aggregates them into CSV reports with JPG/PDF diagnostic figures for omics-agnostic MS quality control.

## When to use

Use when raw vendor MS files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) from LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion runs need per-ion quality metrics and outlier-flagged QC reports before downstream omics analysis.

## When NOT to use

- Input is already a processed feature table or identification result rather than raw instrument files — PeakQC's metrics require raw spectra to locate peaks and compute errors
- Only identity-based QC (total spectral match counts) is needed to detect general performance drift without pinpointing causes like chromatography, sensitivity, or mass calibration
- The data is not MS acquisition data (e.g. genomics or imaging) — PeakQC is omics-agnostic across MS omics types but operates on mass spectrometry acquisitions only

## Inputs

- Raw MS files in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format (LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion)
- Auto-tracked ion list from PeakQC's topmost-abundant-ion detection step
- User-specified ion table with theoretical or reference mass, retention time, and arrival time (or 1/K0) values
- Expected fragment ions and intensities for user-specified ions (for MS2 diagnostic figures)
- Sample-group metadata for per-group outlier detection

## Outputs

- CSV QC reports (e.g. peak_metrics.csv) with per-ion mass error, RT error, AT/1/K0 error, number of spectra, and TIC
- JPG/PDF diagnostic figures including error heatmaps
- MS2 fragmentation-spectrum diagnostic figures for user-specified ions
- Mobiligrams (ion mobility traces) for user-specified ions when AT is provided and IMS separation exists

## How to apply

Import the raw MS runs accepted by IonToolPack (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) alongside an auto-tracked ion list from the topmost-abundant-ion detection step and any user-specified ions carrying theoretical or reference mass, RT, and AT/1/K0 values. For each tracked ion in each run, locate the detected chromatographic or ion-mobility peak and compute the error in mass, retention time, and, where IMS data is present, arrival time (or 1/K0) relative to the reference values; simultaneously tabulate the number of spectra in which the ion appears and its total ion current (TIC). Feed the individual peak metrics into an Isolation Forest outlier detector run per sample group, then aggregate everything into CSV reports (e.g. peak_metrics.csv) and pair them with JPG/PDF diagnostic figures such as error heatmaps. Validation hinges on structural consistency with the test_data example outputs: every auto-tracked and user-specified ion must appear with its error metrics, spectral counts, and TIC, and fragmentation-spectrum diagnostic plots must be present for user-specified ions whose expected fragment ions and intensities were provided.

## Related tools

- **PeakQC** (Executes the QC pipeline: extracts precursor-ion peak metrics, runs Isolation Forest outlier detection, and generates CSV reports and JPG/PDF figures) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Suite that houses PeakQC and reads multiple instrument formats via a GUI; provides the auto-tracked ion lists PeakQC consumes) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # import raw MS files, load auto-tracked + user-specified ion lists, click "Process"
```

## Evaluation signals

- peak_metrics.csv contains error metrics (mass, RT, and AT/1/K0 where applicable), spectral counts, and TIC for every auto-tracked and user-specified ion
- Report structure matches the example output files in the IonToolPack test_data directory
- Isolation Forest outliers are surfaced per sample group and rendered as error heatmaps
- MS2 diagnostic figures appear only for user-specified ions that had expected fragment ions and intensities supplied
- Mobiligrams are emitted only when arrival time is in the input and the MS data contains IMS separation

## Limitations

- No standalone pip/conda/docker installation — IonToolPack is distributed as an .exe download with no install step
- Existing QC approaches 'do little to pinpoint the specific causes of performance deterioration'; PeakQC addresses this but still depends on auto-tracked ion lists from a prior step
- MS2 diagnostic figures require the user to supply expected fragment ions and intensities; mobiligrams require AT in the input and IMS separation in the data
- Large-scale comparisons across batches over months/years are described as planned command-line functionality, not yet available in the GUI workflow

## Evidence
