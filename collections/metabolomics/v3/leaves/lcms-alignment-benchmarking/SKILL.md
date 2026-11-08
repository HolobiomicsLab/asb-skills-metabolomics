---
name: lcms-alignment-benchmarking
description: Use when benchmarking the m/z offset detection accuracy of an LC-HRMS
  alignment tool by injecting known m/z shifts into a reference feature list and comparing
  imposed versus detected offsets.
when_to_use_negative:
- The goal is to align experimental (naturally shifted) batches rather than to benchmark
  offset-detection accuracy with synthetically imposed shifts
- The input is already an aligned, combined data matrix of chromatographic peak areas,
  m/z values and retention times
- Retention time (rather than m/z) offset accuracy is the target, since this benchmark
  protocol as sourced covers the m/z offset test of Figure 8
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetMatch
  role: Semi-automated tool used to detect the artificially imposed m/z offsets when
    matching the target chromatogram to the modified reference feature list
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to execute MetMatch
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:b508a019fd36c422de433c6b54a35027a74809b7f416a1972c888781eff32b35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE
  - Java JRE 1.8
  - MetMatch
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lcms-alignment-benchmarking

## Summary

A benchmarking protocol for quantifying how accurately MetMatch detects artificially imposed m/z offsets (−10 to +10 ppm in 1 ppm increments) when aligning LC-HRMS feature lists to a reference chromatogram, validating software alignment accuracy at 0.01 ppm resolution.

## When to use

Use when benchmarking the m/z offset detection accuracy of an LC-HRMS alignment tool by injecting known m/z shifts into a reference feature list and comparing imposed versus detected offsets.

## When NOT to use

- The goal is to align experimental (naturally shifted) batches rather than to benchmark offset-detection accuracy with synthetically imposed shifts
- The input is already an aligned, combined data matrix of chromatographic peak areas, m/z values and retention times
- Retention time (rather than m/z) offset accuracy is the target, since this benchmark protocol as sourced covers the m/z offset test of Figure 8

## Inputs

- Processed reference LC-HRMS feature list (m/z values and retention times)
- 21 modified copies of the reference feature list with imposed m/z offsets of −10 to +10 ppm in 1 ppm steps

## Outputs

- Table of imposed versus MetMatch-detected m/z offsets for all 21 simulated cases (Figure 8 table)
- Per-case discrepancies (detected minus imposed) in ppm, each within +0.01 to −0.05 ppm

## How to apply

Load the processed reference feature list (m/z values and retention times from LC-HRMS chromatograms) into MetMatch, then generate 21 modified copies of the feature list, each shifted by a single imposed m/z offset from −10 to +10 ppm in 1 ppm increments. Run MetMatch's m/z offset detection on each modified feature list against the reference using histogram bins of 0.01 ppm to estimate the offset. Tabulate imposed versus detected offsets for all 21 cases and compute the discrepancies (detected minus imposed), as presented in the article's Figure 8. Validate the result by confirming that every detected offset lies within the reported discrepancy range of +0.01 to −0.05 ppm of the imposed value and that no correlation exists between offset magnitude and matching accuracy. The rationale is that semi-automated alignment must be demonstrated to correct m/z shifts relative to a reference chromatogram before it can be trusted for combining features into a single data matrix.

## Related tools

- **MetMatch** (Semi-automated tool used to detect the artificially imposed m/z offsets when matching the target chromatogram to the modified reference feature list) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute MetMatch)

## Evaluation signals

- Every detected offset for the 21 simulated m/z offsets differs from the imposed offset by within +0.01 to −0.05 ppm
- No correlation is observed between offset magnitude and matching accuracy across the −10 to +10 ppm range
- Histogram bins of 0.01 ppm resolve the detected offset at sub-0.1 ppm precision
- All 21 modified feature lists (−10 to +10 ppm in 1 ppm increments) are processed without missing or unmatched offset estimates

## Limitations

- The validation is limited to artificially imposed m/z offsets (Section 3.10.2, Figure 8); it does not by itself establish accuracy on naturally occurring shifts in real batches
- MetMatch requires Java runtime (JRE) version 1.8 or newer, so the benchmark cannot be run on older Java environments
- The Discussion section content is not available in the indexed document, so caveats discussed there could not be traced

## Evidence

- [results] For all 21 simulated m/z offsets, the difference between the artificially imposed offset and the offset detected by MetMatch was within the range of +0.01 to −0.05 ppm: [evidence span withheld — anchor status: not_found]
- [abstract] Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference: "Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference"
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
