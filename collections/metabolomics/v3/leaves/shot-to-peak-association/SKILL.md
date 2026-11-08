---
name: shot-to-peak-association
description: Use when processing acoustic ejection mass spectrometry (AEMS) data in
  .wiff format where ejection times are not explicitly stored and each 384- or 1536-well
  shot must be matched to its peak in MRM or full-scan acquisitions.
when_to_use_negative:
- Data are conventional LC-MS chromatography runs rather than acoustic ejection (AEMS)
  acquisitions — the sinusoidal shot-timing model assumes an evenly spaced ejection
  interval
- A vendor-specific AEMS software table already provides explicit shot-to-peak alignment,
  so timing inference from the TIC is unnecessary
- Input is already a processed peak table without raw TIC signal — no shot timing
  can be recovered
tools:
- name: rtmsEcho
  role: 'R package implementing the automated shot-to-peak association: TIC-based
    sinusoidal ejection-time inference and high-mass reference-peak frame definition
    for MRM and full-scan AEMS data'
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework that rtmsEcho extends for raw data access and spectral
    extraction
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:363a33e557a4e2dc198e5ed79b3abb488249fbb8be85faee80dcadc2e51e64d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# shot-to-peak-association

## Summary

Automated association of acoustic ejection (shot) events in AEMS .wiff data with the mass-spectral peaks each shot produced, as implemented in rtmsEcho: ejection times are inferred from the total ion chromatogram via sinusoidal optimization, then high-mass reference peaks are used to define quantification time frames.

## When to use

Use when processing acoustic ejection mass spectrometry (AEMS) data in .wiff format where ejection times are not explicitly stored and each 384- or 1536-well shot must be matched to its peak in MRM or full-scan acquisitions.

## When NOT to use

- Data are conventional LC-MS chromatography runs rather than acoustic ejection (AEMS) acquisitions — the sinusoidal shot-timing model assumes an evenly spaced ejection interval
- A vendor-specific AEMS software table already provides explicit shot-to-peak alignment, so timing inference from the TIC is unnecessary
- Input is already a processed peak table without raw TIC signal — no shot timing can be recovered

## Inputs

- AEMS .wiff raw data file (MRM, full-scan precursor ion, or neutral loss acquisitions)
- Start-time estimate and initial estimate of milliseconds between acoustic ejection peaks (nominal ~1000 ms interval)

## Outputs

- Table of inferred, evenly spaced ejection times (e.g. ejection_times.csv) at ~0.01 ms precision
- Peak table keyed by well and m/z with frame boundaries, baseline-corrected integrated areas, and flags for ambiguous frame assignments

## How to apply

First extract the total ion chromatogram (TIC) of total signal intensity versus time from the AEMS .wiff file using rtmsEcho (built on the rtms framework in R). Since ejection times are not stored in the file, construct a sinusoidal reference with a nominal period equal to the ~1000 ms acoustic ejection interval, then optimize the sinusoid's phase/offset against the observed TIC; the fitted, evenly spaced set of ejection times is recovered to within 0.01 ms. For full-scan runs whose persistent high-intensity baseline obscures analyte peaks, detect well-defined high-mass reference peaks in each well's spectrum and use them as anchor points to define time frames bounding the acoustic shot events. Apply those frames to weaker, less well-defined lower-m/z signals, performing baseline correction and area integration within the frames to quantify them, then assemble a peak table keyed by well and m/z.

## Related tools

- **rtmsEcho** (R package implementing the automated shot-to-peak association: TIC-based sinusoidal ejection-time inference and high-mass reference-peak frame definition for MRM and full-scan AEMS data) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework that rtmsEcho extends for raw data access and spectral extraction)

## Evaluation signals

- Fitted ejection times are evenly spaced at the ~1000 ms nominal acoustic ejection interval across the full plate run
- Residuals between the observed TIC and the fitted sinusoid are consistent with the stated ~0.01 ms peak-timing precision
- Frame assignments in the output table are consistent with the detected high-mass reference peaks; ambiguous assignments are flagged for expert review
- Output peak table contains required fields (well, m/z, frame boundaries, integrated area)
- A full 384-well plate processes in under 2 s for MRM runs (<5 transitions) and under 30 s for full-scan runs (up to a 60 Da window)

## Limitations

- Full-scan AEMS runs have persistent high-intensity baseline signals that can obscure analyte peaks; quantification of weak, less well-defined signals depends on high-mass reference peaks being present and well resolved
- The method assumes ejection times form an evenly spaced set at the nominal ~1000 ms interval; runs deviating from this cadence would misalign shots
- No benchmark datasets or accessioned data are named to support the reported 10- to 100-fold efficiency gains, and baseline details of the conventional comparison methods are not provided

## Evidence

- [other] TIC sinusoidal inference of ejection times: [evidence span withheld — anchor status: not_found]
- [other] High-mass reference-peak alignment: [evidence span withheld — anchor status: not_found]
- [abstract] Automated shot-to-peak association in AEMS processing: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes) while automating"
- [discussion] Peak alignment, baseline correction, area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
