---
name: raw-profile-mode-spectra-processing
description: Use when raw profile-mode LC-IM-MS TOF spectra (e.g., Agilent MassHunter
  .d from an Agilent 6560 in All Ions mode) show poorly defined low-abundance peaks
  and smoothing must precede feature finding or lipid annotation.
when_to_use_negative:
- The input MS-file is already centroided or already feature-level (e.g., a feature
  table) — this skill operates on raw profile-mode intensity values only
- The data are not LC-IM-MS TOF and cannot be handled by the vendor format or converted
  to UIMF as an intermediate step — the article notes enhancements for other IM-MS
  instrumentation require respective vendors or UIMF conversion
- High-abundance peaks are truncated due to detector saturation — that failure mode
  calls for the saturation repair algorithm instead of smoothing
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: PNNL PreProcessor
  role: Standalone software housing the multidimensional moving-average smoothing
    algorithm; reads the raw LC-IM-MS TOF MS-file and writes the smoothed MS-file
    in the same instrument format
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Downstream feature finding and annotation tool that directly processes the
    PreProcessor-generated Agilent .d MS-files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:d9b34e95afe1cf5303dae3eb971abdb8029d734c39a5670fcf839890772b6d48
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# raw-profile-mode-spectra-processing

## Summary

Applies the PNNL PreProcessor's multidimensional moving-average smoothing to raw profile-mode LC-IM-MS TOF data (e.g., Agilent 6560 IM-QTOF MassHunter .d files), smoothing intensities first in the IM dimension and then in the LC dimension to improve peak shapes of poorly defined low-abundance signals before downstream feature finding.

## When to use

Use when raw profile-mode LC-IM-MS TOF spectra (e.g., Agilent MassHunter .d from an Agilent 6560 in All Ions mode) show poorly defined low-abundance peaks and smoothing must precede feature finding or lipid annotation.

## When NOT to use

- The input MS-file is already centroided or already feature-level (e.g., a feature table) — this skill operates on raw profile-mode intensity values only
- The data are not LC-IM-MS TOF and cannot be handled by the vendor format or converted to UIMF as an intermediate step — the article notes enhancements for other IM-MS instrumentation require respective vendors or UIMF conversion
- High-abundance peaks are truncated due to detector saturation — that failure mode calls for the saturation repair algorithm instead of smoothing

## Inputs

- Raw profile-mode LC-IM-MS TOF MS-file (e.g., Agilent MassHunter .d acquired on an Agilent 6560 IM-QTOF in All Ions mode)
- User-specified number of smoothing points per dimension (3 in IM and 3 in LC for the lipid workflow)

## Outputs

- Smoothed MS-file in the same instrument format as the input (e.g., Agilent MassHunter .d) with TOF spectra in profile mode, no centroiding, and original times preserved
- Output filename carrying the algorithm suffix (analogous to the paper's '_MA-d3-c3' convention)

## How to apply

Retrieve an original raw MS-file (e.g., a lipid extract .d acquired on an Agilent 6560 IM-QTOF) and load it into the PNNL PreProcessor. Implement or configure a multidimensional moving-average smoother operating on raw profile-mode intensity values, smoothing first in the IM (drift bin) dimension and then in the LC (frame) dimension; for the paper's lipid workflow on narrow small-molecule IM peaks, use 3 points in each dimension (the '_MA-d3-c3' convention). Enforce the guardrail that the number of smoothing points stays smaller than half of the average base peak width, which 3 points satisfies for narrow IM peaks. Write the smoothed spectra to a new MS-file in the same instrument format as the input, with each TOF spectrum in profile mode without centroiding and original times preserved, appending the algorithm suffix. Run end-to-end and confirm the smoothed output exists and is readable in the original instrument format before downstream processing (e.g., Lipid Annotator feature finding on .d files).

## Related tools

- **PNNL PreProcessor** (Standalone software housing the multidimensional moving-average smoothing algorithm; reads the raw LC-IM-MS TOF MS-file and writes the smoothed MS-file in the same instrument format) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Downstream feature finding and annotation tool that directly processes the PreProcessor-generated Agilent .d MS-files)

## Evaluation signals

- The smoothed output file exists and is readable in the original instrument format (e.g., Agilent MassHunter .d)
- Each output TOF spectrum remains in profile mode without centroiding and preserves original times
- The number of smoothing points is smaller than half of the average base peak width (3 points satisfies this for narrow small-molecule IM peaks)
- Peak shapes of low-abundance signals visibly improve; the paper reports a 70.7% increase in LC-IM-MS lipid features consistently detected across 3 replicates after smoothing
- Output filename carries the algorithm suffix consistent with the '_MA-d3-c3' convention

## Limitations

- The number of smoothing points must stay smaller than half of the average base peak width, constraining aggressive smoothing of narrow IM peaks
- For other types of IM-MS instrumentation, the data enhancements apply only through respective vendors or by converting to UIMF as an intermediate step
- No tool version for the PreProcessor is specified in the discussion, and no quantitative performance metrics are cited there

## Evidence
