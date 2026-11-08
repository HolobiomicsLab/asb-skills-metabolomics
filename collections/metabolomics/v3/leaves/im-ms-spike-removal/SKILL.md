---
name: im-ms-spike-removal
description: Use when raw LC-IM-MS time-of-flight profile data (Agilent .d or UIMF)
  contains random, isolated high-intensity points that are not part of a chromatographic
  or drift-time peak, and the goal is to reduce file size and background before feature
  finding.
when_to_use_negative:
- When the input is already centroided or a feature table — spike removal operates
  on raw profile-mode data points and their IM/MS-dimensional neighbors, so a centroided
  or feature-level input cannot be filtered this way
- When genuine low-abundance, low-signal-to-noise peaks must be preserved — removing
  isolated points may delete legitimate low-abundance signals rather than random noise
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software whose noise filtering module applies the low intensity
    threshold and spike removal to raw LC-IM-MS data points in the IM and MS dimensions
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:4294e0d62c3d4d4835d6b02f71dcdec06d068ae9dc59eedcc5db266410b05f59
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# im-ms-spike-removal

## Summary

Removes isolated 'spike' raw data points from LC-IM-MS MS-files by deleting points that exceed the intensity threshold but lack adjacent neighbors in the IM and MS dimensions, as implemented in the PNNL PreProcessor.

## When to use

Use when raw LC-IM-MS time-of-flight profile data (Agilent .d or UIMF) contains random, isolated high-intensity points that are not part of a chromatographic or drift-time peak, and the goal is to reduce file size and background before feature finding.

## When NOT to use

- When the input is already centroided or a feature table — spike removal operates on raw profile-mode data points and their IM/MS-dimensional neighbors, so a centroided or feature-level input cannot be filtered this way
- When genuine low-abundance, low-signal-to-noise peaks must be preserved — removing isolated points may delete legitimate low-abundance signals rather than random noise

## Inputs

- Smoothed LC-IM-MS MS-file in the instrument's native format (Agilent MassHunter .d or UIMF profile-mode data)
- User-specified intensity threshold value (e.g., 20 counts in the human plasma lipid workflow)
- User-specified adjacent-point criterion for spike removal (e.g., 1 adjacent point)

## Outputs

- Filtered MS-file in profile mode, in the same instrument format as the input (filtered_ms_file.uimf)
- Log file recording the applied algorithms and parameters

## How to apply

Load the smoothed LC-IM-MS MS-file into the PNNL PreProcessor and apply spike removal after the low intensity threshold filter. For each remaining raw data point, compare it against adjacent points in the IM and MS dimensions; delete the point if it is isolated, i.e., lacks 1 adjacent point (the lipid-workflow setting). Spike removal complements the low intensity threshold (e.g., 20 counts in the lipid workflow) by targeting isolated points that do exceed the threshold, and together these filters greatly reduce MS-file size. Run the filters sequentially (threshold first, then spike removal) and write the filtered spectra in profile mode, without centroiding, to a new MS-file in the same instrument format as the input without overwriting the original data. Record the applied algorithms and parameters in a log file so the settings (threshold value and number of adjacent points required) are traceable.

## Related tools

- **PNNL PreProcessor** (Standalone software whose noise filtering module applies the low intensity threshold and spike removal to raw LC-IM-MS data points in the IM and MS dimensions) — https://omics.pnl.gov/software/pnnl-preprocessor

## Examples

```
PNNL PreProcessor --input smoothed_ms_file.uimf --filters "low_intensity_threshold=20;spike_removal=1" --output filtered_ms_file.uimf --log filtering.log
```

## Evaluation signals

- The filtered MS-file exists and parses as a valid instrument-format MS-file in profile mode (not centroided)
- The filtered file is smaller than its input, since background points and spikes are removed
- The original input MS-file is unchanged (not overwritten)
- A log file exists recording the applied algorithms and parameters (threshold value, adjacent-point criterion)

## Limitations

- Spike removal targets isolated points; legitimate low-abundance signal that happens to lack adjacent points above threshold may be removed along with random noise
- Filters are applied to raw profile-mode points; they are not a substitute for centroiding or peak detection downstream

## Evidence
