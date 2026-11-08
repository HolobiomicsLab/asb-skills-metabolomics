---
name: moving-average-parameter-selection
description: Use when raw profile-mode LC-IM-MS TOF data (e.g., Agilent 6560 IM-QTOF
  .d files) shows poorly defined low-abundance peaks and you must choose user-specified
  smoothing point counts for the IM and LC dimensions of a multidimensional moving
  average.
when_to_use_negative:
- The input is already centroided or a peak/feature table — smoothing operates on
  raw profile-mode intensity values, so feature-level data would gain nothing
- The number of smoothing points requested is not smaller than half of the average
  base peak width, which would violate the resolving-power guardrail and degrade narrow
  IM peaks
- The data come from non-TOF IM-MS instrumentation that cannot be read directly —
  per the paper, other IM-MS types require converting to UIMF as an intermediate step
  first
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software housing the multidimensional moving-average smoothing
    implementation; its user-specified point counts per dimension (IM, LC) are the
    parameters this skill selects
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: UIMF-Viewer
  role: PNNL-Comp-Mass-Spec repository associated with the UIMF intermediate format
    used when extending the workflow to other IM-MS instrumentation
  repo: PNNL-Comp-Mass-Spec/UIMF-Viewer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:8547b656794581c374b0dd92b57d4b2006b0da16ddb28eee6210a57b31135aea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  - UIMF-Viewer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# moving-average-parameter-selection

## Summary

Selects the number of moving-average smoothing points per dimension (IM drift bins and LC frames) for the PNNL PreProcessor so that raw LC-IM-MS TOF profile data gains peak-shape quality without degrading resolving power, and writes smoothed output in the original instrument format.

## When to use

Use when raw profile-mode LC-IM-MS TOF data (e.g., Agilent 6560 IM-QTOF .d files) shows poorly defined low-abundance peaks and you must choose user-specified smoothing point counts for the IM and LC dimensions of a multidimensional moving average.

## When NOT to use

- The input is already centroided or a peak/feature table — smoothing operates on raw profile-mode intensity values, so feature-level data would gain nothing
- The number of smoothing points requested is not smaller than half of the average base peak width, which would violate the resolving-power guardrail and degrade narrow IM peaks
- The data come from non-TOF IM-MS instrumentation that cannot be read directly — per the paper, other IM-MS types require converting to UIMF as an intermediate step first

## Inputs

- Raw profile-mode LC-IM-MS TOF MS-file (e.g., Agilent MassHunter .d from an Agilent 6560 IM-QTOF in All Ions mode)
- User-specified number of smoothing points for the IM dimension and the LC dimension
- Average base peak width measurement (for the resolving-power guardrail check)

## Outputs

- Smoothed MS-file in the same instrument format as the input (e.g., Agilent MassHunter .d), written in profile mode without centroiding with original times preserved and an algorithm suffix such as '_MA-d3-c3'
- Preprocessed spectra with improved peak shapes for low-abundance ions, suitable for downstream feature finding (e.g., Lipid Annotator)

## How to apply

Apply the PreProcessor's multidimensional moving average to raw profile-mode intensity values, smoothing first in the IM (drift bin) dimension and then in the LC (frame) dimension, with the number of points specified separately for each dimension. Enforce the resolving-power guardrail: the number of smoothing points must stay smaller than half of the average base peak width, which 3 points satisfies for narrow small-molecule IM peaks. In the paper's lipid demultiplexing workflow, smoothing used 3 points in LC and 7 in IM after demultiplexing with 75% pulses, followed by an intensity threshold of 50 counts and 1 adjacent point spike removal; a 3-point IM x 3-point LC setting is the analogous choice for non-demultiplexed lipid data. Write the smoothed spectra to a new MS-file in the same instrument format as the input (e.g., Agilent MassHunter .d), in profile mode without centroiding, preserving original times, and append an algorithm suffix analogous to '_MA-d3-c3'. The rationale is that multidimensional smoothing improves peak shapes of poorly defined low-abundance signals, which the paper quantified as a 70.7% increase in lipid features consistently detected across 3 replicates.

## Related tools

- **PNNL PreProcessor** (Standalone software housing the multidimensional moving-average smoothing implementation; its user-specified point counts per dimension (IM, LC) are the parameters this skill selects) — https://omics.pnl.gov/software/pnnl-preprocessor
- **UIMF-Viewer** (PNNL-Comp-Mass-Spec repository associated with the UIMF intermediate format used when extending the workflow to other IM-MS instrumentation) — PNNL-Comp-Mass-Spec/UIMF-Viewer

## Evaluation signals

- The smoothed output MS-file exists and is readable in the original .d instrument format, with TOF spectra still in profile mode and original times preserved
- The chosen point counts per dimension satisfy the guardrail: fewer points than half of the average base peak width
- Peak shapes of low-abundance signals visibly improve, benchmarked against the paper's 70.7% increase in LC-IM-MS lipid features consistently detected across 3 replicates
- The output filename carries an algorithm suffix in the paper's convention (e.g., '_MA-d3-c3') documenting the smoothing parameters applied
- Downstream processing time/resources improve, consistent with the paper's report that PreProcessor-generated files processed in 7 min with 3 GB RAM versus 34 min and 6 GB for original files

## Limitations

- The smoothing point count is constrained by the resolving-power guardrail (fewer than half the average base peak width), so wider settings cannot be used on narrow IM peaks
- The smoothing operates on raw LC-IM-MS TOF profile data; applying it to other IM-MS types requires conversion to UIMF as an intermediate step, and no PreProcessor tool version is specified in the paper
- Parameter choice interacts with the rest of the pipeline — the paper's 3-in-LC/7-in-IM setting was tuned for the demultiplexing workflow (75% pulses, 50-count threshold, 1 adjacent point spike removal), not necessarily for other acquisition modes

## Evidence
