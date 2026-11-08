---
name: im-ms-peak-quality-assessment
description: Use when raw LC-IM-MS TOF profile data (e.g., Agilent 6560 .d files,
  All Ions mode) shows poor peak definition of low-abundance ions or saturation-truncated
  high-abundance peaks, and you must judge whether preprocessing improved peak quality.
when_to_use_negative:
- Input is already a centroided or feature table output (e.g., from MS-DIAL, Skyline,
  OpenMS, or Lipid Annotator) rather than raw profile-mode LC-IM-MS data
- Data are not IM-MS TOF data or cannot be handled in the original instrument format
  or via UIMF conversion, since the enhancements target LC-IM-MS raw data
- The goal is detector-saturation repair of truncated high-abundance peaks rather
  than smoothing/peak-shape assessment of low-abundance ions
edam_operation: http://edamontology.org/operation_0364
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone tool whose multidimensional smoothing (3 points in LC x 3 points
    in IM) generates the enhanced MS-files whose peak-quality improvements this skill
    assesses
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Downstream feature-finding/annotation tool used to quantify quality gains
    (feature counts, annotations, processing time) on smoothed vs original MS-files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:6d27f4882a2aa1dc8378b85509bd7b998be2bf093f46fc0a90bc64088c6d2c83
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_0364
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# im-ms-peak-quality-assessment

## Summary

Assessing whether preprocessing of raw LC-IM-MS TOF data (e.g., multidimensional smoothing with the PNNL PreProcessor) improves peak shapes of poorly defined low-abundance signals and extends the dynamic range, using replicate feature-detection and downstream annotation metrics as evidence.

## When to use

Use when raw LC-IM-MS TOF profile data (e.g., Agilent 6560 .d files, All Ions mode) shows poor peak definition of low-abundance ions or saturation-truncated high-abundance peaks, and you must judge whether preprocessing improved peak quality.

## When NOT to use

- Input is already a centroided or feature table output (e.g., from MS-DIAL, Skyline, OpenMS, or Lipid Annotator) rather than raw profile-mode LC-IM-MS data
- Data are not IM-MS TOF data or cannot be handled in the original instrument format or via UIMF conversion, since the enhancements target LC-IM-MS raw data
- The goal is detector-saturation repair of truncated high-abundance peaks rather than smoothing/peak-shape assessment of low-abundance ions

## Inputs

- Original raw profile-mode LC-IM-MS TOF MS-file (e.g., Agilent MassHunter .d from an Agilent 6560 IM-QTOF in All Ions mode)
- Replicate raw MS-files (e.g., triplicate LC-IM-MS runs of a human plasma lipid extract) for consistency assessment

## Outputs

- Smoothed MS-file in the same instrument format as the input (e.g., Agilent .d), TOF spectra in profile mode with original times preserved, named with a suffix like '_MA-d3-c3'
- Peak-quality metrics: count of lipid features consistently detected across replicates, total lipid annotations, and downstream processing time/RAM

## How to apply

Retrieve the original raw profile-mode MS-file (e.g., an Agilent MassHunter .d acquired on an Agilent 6560 IM-QTOF) and run the PNNL PreProcessor's multidimensional moving average smoothing, operating on raw intensity values first in the IM (drift bin) dimension and then in the LC (frame) dimension; for the lipid workflow use 3 points in each dimension, respecting the guardrail that the number of smoothing points stays smaller than half of the average base peak width. Write the smoothed spectra to a new MS-file in the same instrument format as the input, in profile mode without centroiding and with original times preserved, using a suffix analogous to the paper's '_MA-d3-c3' convention. Judge quality by processing replicate files and downstream feature finding: in the paper, smoothing increased the number of LC-IM-MS lipid features consistently detected across 3 replicates by 70.7%, and PreProcessor output gave a 19.4% increase in lipid annotations found in at least 2 replicates versus processing original files. Efficiency metrics (7 min / 3 GB RAM for PreProcessor-generated files vs 34 min / 6 GB RAM for the 3 original MS-files in Lipid Annotator) serve as secondary quality signals.

## Related tools

- **PNNL PreProcessor** (Standalone tool whose multidimensional smoothing (3 points in LC x 3 points in IM) generates the enhanced MS-files whose peak-quality improvements this skill assesses) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Downstream feature-finding/annotation tool used to quantify quality gains (feature counts, annotations, processing time) on smoothed vs original MS-files)

## Evaluation signals

- The smoothed output file exists and is readable in the original .d instrument format, with TOF spectra in profile mode and original times preserved
- The user-specified number of smoothing points stays smaller than half of the average base peak width (3 points satisfies this for small-molecule narrow IM peaks)
- Replicate consistency improves, benchmarked against the paper's 70.7% increase in LC-IM-MS lipid features consistently detected across 3 replicates
- Downstream annotation gain is in the expected direction, e.g., the paper's 19.4% increase in lipid annotations for features found in at least 2 replicates
- Downstream processing efficiency improves, e.g., 7 min / 3 GB RAM for PreProcessor-generated files vs 34 min / 6 GB RAM for original files

## Limitations

- The resolving-power guardrail constrains smoothing: the number of points must stay smaller than half of the average base peak width, so aggressive smoothing would degrade narrow IM peaks
- Data enhancements are applicable to other IM-MS instrumentation only by respective vendors or through converting to UIMF as an intermediate step
- No tool version is specified for the PreProcessor in the article, complicating exact reproduction

## Evidence
