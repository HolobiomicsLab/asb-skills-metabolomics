---
name: ion-mobility-arrival-time-metrics
description: Use when raw MS data from LC-IMS-MS acquisitions (or any run with IMS
  separation) must be quality-controlled by computing arrival-time or 1/K0 errors
  for auto-tracked or user-specified precursor ions.
when_to_use_negative:
- Input is direct-infusion or plain LC-MS data with no ion mobility separation and
  no AT/1/K0 reference values — PeakQC still runs but the AT-error branch of this
  skill yields nothing.
- Analysis goal is identification-based QC (e.g. counting spectral matches) rather
  than pinpointing specific causes of performance deterioration such as mass calibration
  or IMS performance.
- Data are vendor formats other than Agilent 'd', Thermo '.raw', Bruker 'd', or mzML.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Executes the omics-agnostic QC pipeline that extracts arrival-time (AT or
    1/K0) metrics and mobiligrams for precursor ions from LC-IMS-MS data.
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Packaging/GUI suite housing PeakQC; reads raw files in the supported vendor
    formats and exposes the peak-metrics workflow to the practitioner.
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:1dd4e4bf2301c14f7dcb02ea1e59a67c8e88441ec88bd5835322b91222690a9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ion-mobility-arrival-time-metrics

## Summary

Extracts ion-mobility arrival-time (AT or 1/K0) quality metrics for precursor ions alongside mass and retention-time errors using PeakQC, so that instrument performance in LC-IMS-MS runs can be monitored without manual inspection.

## When to use

Use when raw MS data from LC-IMS-MS acquisitions (or any run with IMS separation) must be quality-controlled by computing arrival-time or 1/K0 errors for auto-tracked or user-specified precursor ions.

## When NOT to use

- Input is direct-infusion or plain LC-MS data with no ion mobility separation and no AT/1/K0 reference values — PeakQC still runs but the AT-error branch of this skill yields nothing.
- Analysis goal is identification-based QC (e.g. counting spectral matches) rather than pinpointing specific causes of performance deterioration such as mass calibration or IMS performance.
- Data are vendor formats other than Agilent 'd', Thermo '.raw', Bruker 'd', or mzML.

## Inputs

- Raw MS files from LC-IMS-MS acquisitions in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format
- Auto-tracked ion list (topmost abundant MS1 ions per sample group) from a prior PeakQC/IonToolPack detection step
- User-specified ions with theoretical or reference mass, retention time, and arrival time (or 1/K0) values

## Outputs

- CSV report of per-ion, per-run peak metrics including arrival-time (or 1/K0) error, mass error, RT error, number of spectra, and total ion current (TIC)
- JPG/PDF diagnostic figures including mobiligrams (ion mobility traces) for user-specified ions
- Isolation Forest error heatmaps flagging outlier arrival-time metrics per sample group

## How to apply

1. Load raw MS files accepted by IonToolPack/PeakQC (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) covering LC-IMS-MS acquisitions with or without DDA/DIA fragmentation. 2. Supply the auto-tracked ion list (topmost abundant MS1 ions per sample group) and/or user-specified ions carrying theoretical or reference mass, RT, and AT/1/K0 values. 3. For each ion in each MS run, locate the detected chromatographic/mobility peak and compute the arrival-time (or 1/K0) error relative to the reference value, together with mass error, RT error, number of spectra, and TIC. 4. When AT is present in the input and the data contain IMS separation, extract 'mobiligrams' (ion mobility traces) for the user-specified ions as diagnostic figures. 5. Aggregate per-ion, per-run metrics into CSV reports and JPG/PDF figures; the underlying workflow also performs outlier detection per sample group using the Isolation Forest algorithm and generates error heatmaps. Rationale: the article motivates this by noting that existing QC tools are mostly proteomics-dedicated, hard to deploy, or do not support hybrid platforms like ion mobility spectrometry.

## Related tools

- **PeakQC** (Executes the omics-agnostic QC pipeline that extracts arrival-time (AT or 1/K0) metrics and mobiligrams for precursor ions from LC-IMS-MS data.) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Packaging/GUI suite housing PeakQC; reads raw files in the supported vendor formats and exposes the peak-metrics workflow to the practitioner.) — https://github.com/pnnl/IonToolPack

## Examples

```
Double-click IonToolPack.exe → PeakQC tab → import raw MS files (Agilent 'd' / Thermo '.raw' / Bruker 'd' / mzML) → supply auto-tracked ion list and user-specified ions with theoretical mass, RT, AT/1/K0 → click “Process”.
```

## Evaluation signals

- peak_metrics.csv (or equivalent CSV report) contains, for every auto-tracked and user-specified ion, an arrival-time (or 1/K0) error column populated only when IMS separation is present and AT reference values were supplied.
- Mass-error and RT-error columns appear alongside the AT metric, and spectral counts plus TIC values are present for the same ion peaks, matching the article's stated metric set.
- Mobiligram figures (JPG/PDF) are generated for user-specified ions whenever AT is in the input and the MS data contain IMS separation.
- Error heatmaps from the Isolation Forest step are structurally consistent with the example outputs in the package's test_data directory.
- The report structure mirrors the reference outputs shipped in https://github.com/pnnl/IonToolPack/tree/master/test_data.

## Limitations

- The article does not provide explicit standalone installation instructions (pip/conda/docker) for PeakQC; it is distributed as part of the IonToolPack.exe release.
- AT/1/K0 error extraction is only meaningful for user-specified ions that carry theoretical or reference AT values; auto-tracked ions without such reference values yield no AT error.
- PeakQC supports multiple instrument formats and acquisition types, but the specific handling of edge cases in direct infusion or DIA-only IMS data is not detailed in the article.

## Evidence

- [results] workflow for user-specified ions: [evidence span withheld — anchor status: not_found]
- [readme] GUI workflow: [evidence span withheld — anchor status: not_found]
