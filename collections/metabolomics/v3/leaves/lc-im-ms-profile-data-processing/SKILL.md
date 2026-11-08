---
name: lc-im-ms-profile-data-processing
description: Use when you have raw LC-DT IM-MS profile data (e.g. Agilent .d files)
  with low-abundance ions of poor peak definition or detector-saturated high-abundance
  isotopic peaks, and need preprocessed MS-files for feature finding or annotation.
when_to_use_negative:
- The input is already a peak-picked or feature-level table rather than raw/profile
  LC-IM-MS data
- Data were acquired on instrumentation without drift-time multiplexing, so demultiplexing
  and the IM interpolation steps are not applicable
- You need centroided, database-search-ready spectra rather than profile data for
  downstream processing
tools:
- name: PNNL PreProcessor
  role: Standalone executable implementing LC compression, IM interpolation, demultiplexing,
    smoothing, spike removal, and saturation repair on raw profile data
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Downstream tool that consumes the preprocessed .d files for feature finding
    and lipid annotation to demonstrate the accuracy gains
- name: UIMF-Viewer
  role: Repository referenced for handling the intermediate UIMF format used when
    converting data as an intermediate step
  repo: PNNL-Comp-Mass-Spec/UIMF-Viewer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:0883891cef50d84f66a193797f14308695eaeb2090f69f8f9647e044250c4774
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ChemCalc
  - Lipid Annotator
  - PNNL PreProcessor
  - R
  - UIMF-Viewer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# lc-im-ms-profile-data-processing

## Summary

A skill covering the LC–DT IM–MS profile-data processing chain implemented in the PNNL PreProcessor: LC compression, IM interpolation, demultiplexing, smoothing, spike removal, and saturation repair applied at the raw/profile level to extend dynamic range while shrinking file size.

## When to use

Use when you have raw LC-DT IM-MS profile data (e.g. Agilent .d files) with low-abundance ions of poor peak definition or detector-saturated high-abundance isotopic peaks, and need preprocessed MS-files for feature finding or annotation.

## When NOT to use

- The input is already a peak-picked or feature-level table rather than raw/profile LC-IM-MS data
- Data were acquired on instrumentation without drift-time multiplexing, so demultiplexing and the IM interpolation steps are not applicable
- You need centroided, database-search-ready spectra rather than profile data for downstream processing

## Inputs

- Raw LC-DT IM-MS profile MS-file from an instrument producing .d format (e.g. a human plasma lipid extract analyzed in triplicate)

## Outputs

- Preprocessed MS-file in the same instrument format, profile mode, typically with a suffix appended to the file name
- Extended dynamic range signal suitable for downstream feature finding and annotation

## How to apply

Run the PNNL PreProcessor on the raw instrument file in profile mode. First compress the LC dimension (4 frames become 1) then interpolate the IM dimension (every drift bin becomes 3) to reshape the profile grid. Next demultiplex using 75% pulses for artifact removal, then smooth multidimensionally with points: 3 in LC and 7 in IM, set an intensity threshold of 50 counts, and remove 1 adjacent point spike. Optionally enable saturation repair with a threshold of 40% of the abundance limit so saturated isotopic peaks are reconstructed from the isotopic ratios measured at later LC or IM arrival times where the ion is unsaturated; write the output in the same instrument format as the input.

## Related tools

- **PNNL PreProcessor** (Standalone executable implementing LC compression, IM interpolation, demultiplexing, smoothing, spike removal, and saturation repair on raw profile data) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Downstream tool that consumes the preprocessed .d files for feature finding and lipid annotation to demonstrate the accuracy gains)
- **UIMF-Viewer** (Repository referenced for handling the intermediate UIMF format used when converting data as an intermediate step) — PNNL-Comp-Mass-Spec/UIMF-Viewer

## Examples

```
Run PNNL PreProcessor on the raw .d file with LC compression 4:1, IM interpolation 1:3, demultiplexing (75% pulses), smoothing (3 LC / 7 IM), intensity threshold 50 counts, spike removal 1, and saturation repair threshold 40% of the abundance limit; output in same format, profile mode, suffixed _SR.
```

## Evaluation signals

- Output file exists and is in the same instrument format as the input (e.g. Agilent .d) and in profile mode
- Number of LC-IM-MS lipid features consistently detected across 3 replicates increases (reported +70.7% after smoothing)
- After saturation repair the monoisotopic peak height of a saturated ion roughly triples and mass error drops from 10.6 to 1.9 pm
- Downstream annotation count rises (reported 19.4% more lipid annotations for features found in ≥2 replicates) and processing time/RAM drop versus raw files
- Output structure matches the deposited preprocessed reference file in MSV000087295

## Limitations

- Saturation repair works on raw/profile-level spectra and needs an unsaturated reference isotopic distribution observed at later LC or IM time; it cannot recover peaks with no unsaturated counterpart
- The reported gains (19.4% more annotations, two-times faster processing) are specific to LC-DT IM-MS data-independent acquisition of a complex human plasma lipid extract and may not transfer directly
- Excessive smoothing or compression parameters can alter peak shapes; the tuned settings (points 3 LC / 7 IM, 4:1 LC compression, 1:3 IM interpolation) are data-dependent
- Applicability to other IM-MS instrumentation requires vendor support or conversion to UIMF as an intermediate step
- No tool version is specified, so exact behavior may vary between software releases

## Evidence

- [results] Saturation repair tripled the monoisotopic peak height of the MS-data and reduced the mass error from 10.6 to 1.9 pm.: [evidence span withheld — anchor status: not_found]
- [results] Smoothing increased the number of LC-IM-MS lipid features consistently detected across 3 replicates by 70.7%.: [evidence span withheld — anchor status: not_found]
