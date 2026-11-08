---
name: chromatographic-retention-time-error-tracking
description: Use when raw LC-MS or LC-IMS-MS files (Agilent 'd', Thermo '.raw', Bruker
  'd', mzML) need per-ion mass, retention-time, and arrival-time error metrics for
  QC of chromatographic performance.
when_to_use_negative:
- The goal is fragmentation-spectrum QC (expected fragment ions/intensities) — use
  the separate MS2 diagnostic step instead
- The goal is MS/MS spectral library matching or targeted peak quantitation — TandemMatch
  or PeakQuant, not this RT-error tracking step
- Only infusion (no LC front-end) data where retention time is undefined
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Computes the peak metrics (mass, RT, AT/1/K0 errors, spectra counts, TIC)
    for auto-tracked and user-specified ions
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI host that reads the vendor raw formats and runs the PeakQC pipeline from
    raw files through report generation
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:bdfd69ec0cddcdfe4e61eb51748b3a6c8a04f7304785f985c01febd876fc5d56
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# chromatographic-retention-time-error-tracking

## Summary

Track retention-time (and, for IMS runs, arrival-time/1/K0) errors of precursor ions across an MS batch — PeakQC computes these peak metrics for both auto-tracked MS1 ions and user-specified reference ions to flag chromatographic drift.

## When to use

Use when raw LC-MS or LC-IMS-MS files (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) need per-ion mass, retention-time, and arrival-time error metrics for QC of chromatographic performance.

## When NOT to use

- The goal is fragmentation-spectrum QC (expected fragment ions/intensities) — use the separate MS2 diagnostic step instead
- The goal is MS/MS spectral library matching or targeted peak quantitation — TandemMatch or PeakQuant, not this RT-error tracking step
- Only infusion (no LC front-end) data where retention time is undefined

## Inputs

- Raw MS files: Agilent 'd', Thermo '.raw', Bruker 'd', or mzML
- Auto-tracked ion list (topmost abundant MS1 ions per sample group)
- User-specified ions with theoretical/reference mass, RT, and AT/1/K0 values

## Outputs

- CSV peak-metrics reports containing mass, RT, and (where applicable) AT/1/K0 errors, spectral counts, and TIC per ion per run
- JPG/PDF diagnostic figures
- Downstream Isolation Forest error heatmaps per sample group

## How to apply

Load raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) from LC-MS, LC-IMS-MS, DDA/DIA, or direct-infusion acquisitions. Supply an auto-tracked ion list (topmost abundant MS1 ions per sample group) and, optionally, user-specified ions carrying theoretical/reference mass, RT, and AT/1/K0 values. For each tracked ion in each run, locate the detected chromatographic peak and compute the mass error, retention-time error, and arrival-time (or 1/K0) error versus the reference values; also tabulate the number of spectra in which the ion appears and its total ion current (TIC). Aggregate per-ion, per-run metrics into CSV reports and accompany them with JPG/PDF diagnostic figures, then feed them to downstream outlier detection (Isolation Forest error heatmaps per sample group).

## Related tools

- **PeakQC** (Computes the peak metrics (mass, RT, AT/1/K0 errors, spectra counts, TIC) for auto-tracked and user-specified ions) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI host that reads the vendor raw formats and runs the PeakQC pipeline from raw files through report generation) — https://github.com/pnnl/IonToolPack

## Evaluation signals

- peak_metrics.csv exists and contains mass error, RT error, and (where applicable) AT/1/K0 error columns for every auto-tracked and user-specified ion
- Per-ion per-run spectral counts and TIC values are populated, not null/zero
- Report structure matches the example output files in the repo's test_data directory
- Downstream Isolation Forest error heatmap per sample group can be generated from the emitted metrics
- JPG/PDF diagnostic figures accompany each CSV report

## Limitations

- Requires an auto-tracked ion list from the upstream topmost-abundant-ion step or manually supplied reference ions — no fully self-contained ion discovery
- If AT is absent from the input or the MS data lacks IMS separation, mobiligrams/AT-1/K0 errors cannot be computed
- Only supports the listed vendor formats (Agilent 'd', Thermo '.raw', Bruker 'd', mzML); other raw formats require prior conversion
- Standalone pip/conda/docker install instructions are not documented; distribution is via pre-built IonToolPack.exe

## Evidence

- [results] The software finds the topmost abundant ions in the MS1 data which are more frequent per sample group and uses them as 'auto-tracked' ions.: [evidence span withheld — anchor status: not_found]
- [results] mobiligrams (i.e., ion mobility traces) are extracted for user-specified ions if the AT is included in the input and the MS data contains IMS separation: [evidence span withheld — anchor status: not_found]
