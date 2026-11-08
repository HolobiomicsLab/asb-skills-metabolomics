---
name: mz-offset-histogram-detection
description: Use when LC-HRMS feature lists (m/z + retention time) from different
  measurement sequences or batches need correction for systematic m/z shifts relative
  to a reference before alignment.
when_to_use_negative:
- Input is raw, unprocessed chromatographic data rather than a feature list with m/z
  values and retention times
- The m/z differences between the target and reference are not systematic but randomly
  distributed, so a single offset correction would not apply
- Only retention time shifts (not m/z offsets) need to be corrected
- Input is already an aligned, combined data matrix
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MetMatch
  role: Semi-automated tool that performs histogram-based m/z offset detection and
    correction of LC-HRMS feature lists against a reference chromatogram
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
  source_content_hash: sha256:e4baea2c7841aabc48e3c37438284fbc9c3defac99295aa208df3624744ddece
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE
  - Java JRE 1.8
  - MetMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# mz-offset-histogram-detection

## Summary

Detects systematic m/z offsets of LC-HRMS feature lists relative to a reference chromatogram using MetMatch's histogram-based offset estimation, as validated across artificially imposed offsets of −10 to +10 ppm.

## When to use

Use when LC-HRMS feature lists (m/z + retention time) from different measurement sequences or batches need correction for systematic m/z shifts relative to a reference before alignment.

## When NOT to use

- Input is raw, unprocessed chromatographic data rather than a feature list with m/z values and retention times
- The m/z differences between the target and reference are not systematic but randomly distributed, so a single offset correction would not apply
- Only retention time shifts (not m/z offsets) need to be corrected
- Input is already an aligned, combined data matrix

## Inputs

- Reference feature list of processed LC-HRMS chromatograms (m/z values and retention times)
- Target feature list(s) or modified feature list copies with suspected m/z offsets
- Histogram bin width setting (e.g., 0.01 ppm)

## Outputs

- Estimated m/z offset per datafile/batch (in ppm)
- Corrected m/z values for the target feature list
- Imposed-vs-detected offset comparison table for validation

## How to apply

Load a processed reference feature list (m/z values and retention times) into MetMatch, which requires Java JRE 1.8 or newer. Run MetMatch's m/z offset detection on each shifted datafile against the reference, using fine histogram bins (e.g., 0.01 ppm) to estimate the systematic offset from the distribution of m/z differences. The estimated offset is then used to correct m/z values prior to matching and alignment into a single data matrix. Validate by tabulating imposed vs. detected offsets: in the article's artificial-data evaluation, 21 simulated offsets (−10 to +10 ppm in 1 ppm steps) were all detected within +0.01 to −0.05 ppm of the imposed value, with no correlation between offset magnitude and matching accuracy.

## Related tools

- **MetMatch** (Semi-automated tool that performs histogram-based m/z offset detection and correction of LC-HRMS feature lists against a reference chromatogram) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute MetMatch)

## Evaluation signals

- For a known imposed offset (e.g., 21 offsets from −10 to +10 ppm in 1 ppm steps), the detected offset minus the imposed offset falls within +0.01 to −0.05 ppm
- Detected offset magnitude shows no correlation with matching accuracy across the tested offset range
- Histogram of m/z differences between target and reference shows a dominant systematic shift at the estimated offset when using 0.01 ppm bins
- After correction, target m/z values align to the reference feature list within the matching tolerance
- MetMatch runs under JRE 1.8 or newer without errors

## Limitations

- MetMatch is distributed under GNU GPL WITHOUT ANY WARRANTY; not for safety-critical use
- Requires a processed reference feature list; raw chromatograms must be feature-processed beforehand
- Requires Java JRE 1.8 or newer
- Detected offsets may systematically deviate by a small constant amount (up to −0.05 ppm) from the true offset
- The discussion section of the source document was unavailable, limiting caveats on failure modes

## Evidence

- [abstract] the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [abstract] Based on a feature list ... that serves as a reference: "Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference"
- [intro] It provides methods to correct LC-HRMS data ...: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch requires Java runtime (JRE) ...: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
