---
name: lc-im-ms-multidimensional-smoothing
description: Use when raw LC-IM-MS TOF profile data (e.g. Agilent 6560 IM-QTOF MassHunter
  .d) shows poorly defined low-abundance peaks, and you need a 3-point IM x 3-point
  LC moving-average smoothing before feature finding.
when_to_use_negative:
- The input is already a feature table or peak list — smoothing operates on raw profile-mode
  intensity values of TOF spectra
- The dominant problem is truncated high-abundance ions due to detector saturation
  — that requires the PreProcessor's saturation repair algorithm, not smoothing
- The number of requested smoothing points would equal or exceed half of the average
  base peak width (risk of destroying narrow IM peak resolution)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software housing the multidimensional smoothing algorithm that
    this skill reconstructs; generates new MS-files with enhanced signal quality in
    the same format
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Downstream feature-finding tool that consumes the smoothed .d files directly,
    demonstrating the output format requirement (7 min / 3 GB on smoothed files vs
    34 min / 6 GB on originals)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:6269e9b2a0e120b5fbf2977cb73063b19ac1d616eea332d20326183affcae775
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lc-im-ms-multidimensional-smoothing

## Summary

Reconstructs the PNNL PreProcessor's multidimensional moving-average smoothing for raw LC-IM-MS TOF profile data, smoothing first in the IM (drift bin) dimension then the LC (frame) dimension to improve peak shapes of low-abundance ions.

## When to use

Use when raw LC-IM-MS TOF profile data (e.g. Agilent 6560 IM-QTOF MassHunter .d) shows poorly defined low-abundance peaks, and you need a 3-point IM x 3-point LC moving-average smoothing before feature finding.

## When NOT to use

- The input is already a feature table or peak list — smoothing operates on raw profile-mode intensity values of TOF spectra
- The dominant problem is truncated high-abundance ions due to detector saturation — that requires the PreProcessor's saturation repair algorithm, not smoothing
- The number of requested smoothing points would equal or exceed half of the average base peak width (risk of destroying narrow IM peak resolution)

## Inputs

- Raw profile-mode LC-IM-MS TOF MS-file (Agilent MassHunter .d from Agilent 6560 IM-QTOF, All Ions mode)
- User-specified smoothing points per dimension (e.g. 3 in LC, 3 in IM)
- Average base peak width in the IM dimension (for the guardrail check)

## Outputs

- Smoothed MS-file in the same instrument format as the input (Agilent MassHunter .d), TOF spectra in profile mode without centroiding, original times preserved
- Output filename carrying the algorithm suffix (analogous to '_MA-d3-c3')

## How to apply

Retrieve an original raw MS-file (e.g. Agilent MassHunter .d from an Agilent 6560 IM-QTOF in All Ions mode, such as the lipid extract deposited at ftp://massive.ucsd.edu/MSV000087295) and implement a multidimensional moving-average smoother operating directly on raw profile-mode intensity values. Smooth first along the IM (drift bin) dimension, then along the LC (frame) dimension, using user-specified point counts — 3 points in each dimension matches the paper's lipid workflow and its '_MA-d3-c3' convention. Enforce the guardrail that the number of smoothing points must stay smaller than half of the average base peak width; 3 points satisfies this for small-molecule narrow IM peaks. Write the smoothed spectra to a new MS-file in the same instrument format as the input (.d), keeping each TOF spectrum in profile mode without centroiding, preserving original times, and appending the algorithm suffix (e.g. '_MA-d3-c3') to the filename.

## Related tools

- **PNNL PreProcessor** (Standalone software housing the multidimensional smoothing algorithm that this skill reconstructs; generates new MS-files with enhanced signal quality in the same format) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Downstream feature-finding tool that consumes the smoothed .d files directly, demonstrating the output format requirement (7 min / 3 GB on smoothed files vs 34 min / 6 GB on originals))

## Evaluation signals

- Script runs end-to-end on the deposited original file from ftp://massive.ucsd.edu/MSV000087295 and the smoothed output file exists
- Output file is readable in the original .d instrument format, with TOF spectra still in profile mode (no centroiding) and original drift/retention times preserved
- The smoothing point count (3) is smaller than half of the average base peak width — resolving-power guardrail satisfied
- Output filename carries the algorithm suffix analogous to '_MA-d3-c3'
- Expert review of peak shapes: as reported, smoothing increased LC-IM-MS lipid features consistently detected across 3 replicates by 70.7%

## Limitations

- The smoothing-points parameter is constrained by resolving power: it must stay smaller than half of the average base peak width, limiting how aggressive smoothing can be on narrow IM peaks
- Smoothing targets poorly defined low-abundance ions; truncated high-abundance peaks from detector saturation require the separate saturation repair algorithm
- The PreProcessor's enhancements are specific to IM-MS instrumentation; applying them to other IM-MS types requires vendor support or converting to UIMF as an intermediate step

## Evidence
