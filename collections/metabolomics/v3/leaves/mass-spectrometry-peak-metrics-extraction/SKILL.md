---
name: mass-spectrometry-peak-metrics-extraction
description: Use when batch LC-MS, LC-IMS-MS, DDA/DIA, or direct-infusion raw MS data
  (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) needs precursor-ion QC metrics
  and error reports without omics-specific identification pipelines.
when_to_use_negative:
- When the goal is global batch quality assessment via PCA of time-versus-m/z MS1
  images — that is PeakQC's separate PCA step, not per-ion peak metrics extraction.
- When identification-based QC (spectral-match counts from proteomics/metabolomics
  search pipelines) is sufficient to pinpoint performance issues; this skill is designed
  precisely because such tools 'do little to pinpoint the specific causes of performance
  deterioration'.
- When the raw data cannot be read in a supported format (Agilent 'd', Thermo '.raw',
  Bruker 'd', mzML); convert first, e.g. via MZA.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: 'Executes the peak-metrics extraction pipeline: locates auto-tracked and user-specified
    ion peaks in raw MS runs and computes mass, RT, and AT/1/K0 errors plus spectra
    counts and TIC, emitting CSV reports and JPG/PDF figures.'
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Standalone Windows GUI suite that reads the raw MS files and hosts PeakQC;
    launched by double-clicking IonToolPack.exe after downloading from the Releases
    page.
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Optional data-conversion step in the pipeline used to bring unsupported raw
    vendor formats into a readable form before metrics extraction.
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:a06bb8d1f42916519679111c1dc47858bf0eb5f91ba2a01bd412d61bf519375f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - MZA
  - PeakQC
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mass-spectrometry-peak-metrics-extraction

## Summary

Automated extraction of precursor-ion peak quality metrics (mass error, retention-time error, arrival-time/1/K0 error, number of spectra, and TIC) for both auto-tracked topmost-abundant MS1 ions and user-specified reference ions, as implemented in PeakQC within the IonToolPack suite for omics-agnostic MS quality control.

## When to use

Use when batch LC-MS, LC-IMS-MS, DDA/DIA, or direct-infusion raw MS data (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) needs precursor-ion QC metrics and error reports without omics-specific identification pipelines.

## When NOT to use

- When the goal is global batch quality assessment via PCA of time-versus-m/z MS1 images — that is PeakQC's separate PCA step, not per-ion peak metrics extraction.
- When identification-based QC (spectral-match counts from proteomics/metabolomics search pipelines) is sufficient to pinpoint performance issues; this skill is designed precisely because such tools 'do little to pinpoint the specific causes of performance deterioration'.
- When the raw data cannot be read in a supported format (Agilent 'd', Thermo '.raw', Bruker 'd', mzML); convert first, e.g. via MZA.

## Inputs

- Raw MS data files in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format
- Auto-tracked ion list produced by the preceding topmost-abundant-ion detection step in PeakQC
- User-specified ion list with theoretical or reference m/z, retention time, and arrival time / 1/K0 values
- Optional expected fragment ions and intensities (for user-specified ions needing MS2 and mobiligram checks)

## Outputs

- CSV report of peak metrics (mass error, RT error, AT/1/K0 error where applicable, number of spectra, TIC) per ion and per MS run
- JPG and PDF diagnostic figures accompanying the metric reports
- Per-ion metrics ready for Isolation Forest anomaly detection and error heatmaps

## How to apply

Load the raw MS runs (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) covering infusion, LC-MS, LC-IMS-MS, or DDA/DIA acquisitions into the IonToolPack/PeakQC workflow. Take as ion targets either the auto-tracked ions — the topmost abundant ions in MS1 data that are most frequent per sample group — or user-specified ions carrying theoretical/reference mass, RT, and AT (or 1/K0) values. For each ion in each run, locate the detected chromatographic (and, where applicable, ion-mobility) peak and compute the mass error, retention-time error, and arrival-time/1/K0 error relative to the reference values, along with the number of spectra in which the ion appears and its total ion current (TIC). Aggregate the per-ion, per-run metrics into CSV reports and pair them with JPG/PDF diagnostic figures; these metrics then feed downstream Isolation Forest outlier detection and error heatmaps. Validate that peak_metrics.csv contains all five metric classes for every tracked ion and matches the structure of the example files shipped in the repository's test_data directory.

## Related tools

- **PeakQC** (Executes the peak-metrics extraction pipeline: locates auto-tracked and user-specified ion peaks in raw MS runs and computes mass, RT, and AT/1/K0 errors plus spectra counts and TIC, emitting CSV reports and JPG/PDF figures.) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Standalone Windows GUI suite that reads the raw MS files and hosts PeakQC; launched by double-clicking IonToolPack.exe after downloading from the Releases page.) — https://github.com/pnnl/IonToolPack
- **MZA** (Optional data-conversion step in the pipeline used to bring unsupported raw vendor formats into a readable form before metrics extraction.) — https://github.com/pnnl/IonToolPack

## Examples

```
Download IonToolPack.exe from github.com/pnnl/IonToolPack/releases, run it, import raw MS files (mzML/Thermo .raw), and click "Process"
```

## Evaluation signals

- peak_metrics.csv contains, for every auto-tracked and user-specified ion, error metrics (mass, RT, and where applicable AT/1/K0), number of spectra, and TIC.
- Report structure is consistent with the example input/output files in the repository's test_data directory.
- Errors are computed relative to theoretical/reference values for user-specified ions and relative to the tracked detected peak for auto-tracked ions, not against arbitrary per-run re-estimated values.
- Both CSV reports and JPG/PDF figures are generated for each ion/run combination, matching the documented report/figure pairing.
- Downstream Isolation Forest outlier detection runs on the extracted metrics without schema errors, indicating the metric columns are complete.

## Limitations

- IonToolPack requires no installation but is distributed as a Windows .exe; the README documents double-clicking IonToolPack.exe rather than pip/conda/docker deployment.
- AT/1/K0 error metrics are only meaningful for LC-IMS-MS data or when arrival time is present in the input; for plain LC-MS or infusion runs only mass and RT errors apply.
- The auto-tracked ion set depends on the preceding topmost-abundant-ion detection step; a poor auto-tracking result propagates into the metrics.
- Manual inspection of spectra is time-consuming and subjective — the very motivation for this automated skill — so results should not be hand-curated ad hoc rather than trusting the computed metrics.

## Evidence
