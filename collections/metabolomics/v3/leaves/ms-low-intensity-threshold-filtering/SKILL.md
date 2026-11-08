---
name: ms-low-intensity-threshold-filtering
description: Use when raw LC-IM-MS or LC-DT IM-MS TOF profile-mode data (Agilent .d
  or UIMF) contains a large background of low-count points that inflate file size
  and degrade downstream feature finding or lipid annotation.
when_to_use_negative:
- Input is already centroided or a peak/feature list rather than raw profile-mode
  IM-MS data points — the threshold operates on raw counts of individual points
- The downstream tool requires the original unfiltered file, e.g., when saturation
  repair or smoothing must be applied to the full dynamic range before thresholding
- MS-files where the threshold would remove true low-abundance analyte signals the
  user intends to retain, since the filter deletes all points not exceeding the specified
  count value
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PNNL PreProcessor
  role: Houses the low intensity threshold algorithm that removes raw LC-IM-MS data
    points below the user-specified count value and writes the filtered profile-mode
    MS-file
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:2838cd67dc30bd6d075eaf938d1c6a1dce016c364be0cc08707dbe408d275441
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ms-low-intensity-threshold-filtering

## Summary

The PNNL PreProcessor's low intensity threshold filter removes raw LC-IM-MS data points that do not exceed a user-specified count value (e.g., 20 counts in the lipid workflow), reducing file size and cleaning background before spike removal and downstream IM-MS processing.

## When to use

Use when raw LC-IM-MS or LC-DT IM-MS TOF profile-mode data (Agilent .d or UIMF) contains a large background of low-count points that inflate file size and degrade downstream feature finding or lipid annotation.

## When NOT to use

- Input is already centroided or a peak/feature list rather than raw profile-mode IM-MS data points — the threshold operates on raw counts of individual points
- The downstream tool requires the original unfiltered file, e.g., when saturation repair or smoothing must be applied to the full dynamic range before thresholding
- MS-files where the threshold would remove true low-abundance analyte signals the user intends to retain, since the filter deletes all points not exceeding the specified count value

## Inputs

- Raw LC-IM-MS MS-file in instrument-native format (Agilent MassHunter .d or UIMF profile-mode TOF data)
- User-specified low intensity threshold value in counts (e.g., 20 counts for the lipid workflow, 50 counts for the demultiplexing workflow)

## Outputs

- Filtered MS-file in the same instrument format as the input, still in profile mode (no centroiding), containing only points above the intensity threshold
- Log file recording the applied algorithms and parameters

## How to apply

Load the raw LC-IM-MS MS-file in the instrument's native format (Agilent MassHunter .d or UIMF profile-mode TOF data). Apply the low intensity threshold filter by removing all raw data points whose counts do not exceed the user-specified value — for the human plasma lipid LC-DT IM-MS workflow the threshold is 20 counts, while the HeLa demultiplexing workflow uses an intensity threshold of 50 counts. This step precedes spike removal, which deletes isolated points lacking adjacent neighbors in the IM and MS dimensions; both filters are applied to the smoothed profile data. Write the filtered spectra back in profile mode (no centroiding) to a new MS-file in the same instrument format as the input, without overwriting the original, and log the applied algorithms and parameters. The rationale is that these filters extend the dynamic range of the detection system while greatly reducing MS-file size for faster, memory-efficient downstream processing.

## Related tools

- **PNNL PreProcessor** (Houses the low intensity threshold algorithm that removes raw LC-IM-MS data points below the user-specified count value and writes the filtered profile-mode MS-file) — https://omics.pnl.gov/software/pnnl-preprocessor

## Evaluation signals

- The filtered MS-file parses as a valid instrument-format file (e.g., UIMF) and is in profile mode with no centroiding applied
- The filtered file is smaller than its input MS-file, consistent with background point removal
- No raw data point in the output has a count value at or below the configured threshold (e.g., all points exceed 20 counts in the lipid workflow)
- The original input MS-file is unchanged (no overwrite), and a log file exists recording the threshold algorithm and parameter value used
- Downstream acceptance: feature finding or Lipid Annotator runs on the filtered file and retains the expected low-abundance signals (smoothing + thresholding increased consistently detected LC-IM-MS lipid features by 70.7%)

## Limitations

- The threshold deletes all points not exceeding the specified value, so an overly high value can remove genuine low-abundance analyte signals along with background
- Thresholding alone does not address detector-saturated high-abundance peaks; saturation repair is a separate algorithm in the PreProcessor
- Parameter values are workflow- and instrument-dependent (20 counts for lipids vs 50 counts for the HeLa demultiplexing run), so the threshold must be re-tuned per dataset

## Evidence
