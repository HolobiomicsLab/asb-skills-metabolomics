---
name: toxin-detoxification-metabolomics-data-processing
description: Use when comparing untargeted LC-HRMS feature lists across multiple measurement
  sequences/batches (e.g. five barley target files vs. a reference experiment) where
  retention time shifts (here -9 to -24 s) and m/z deviations must be corrected before
  metabolite matching.
when_to_use_negative:
- The input is raw unprocessed LC-HRMS data with no XCMS-derived feature list — MetMatch
  aligns processed feature lists, not raw chromatograms
- Only a single measurement sequence/batch exists with no reference experiment to
  align against
- The goal is de novo peak detection or identification rather than comparison and
  alignment of already-detected features across experiments
edam_operation: http://edamontology.org/operation_2945
tools:
- name: MetMatch
  role: 'Semi-automated alignment tool: matches target ions against the reference
    feature list, corrects non-linear retention time shifts, and combines matched
    ions into a single data matrix'
  repo: https://github.com/S-T-K/MetMatch
- name: XCMS
  role: Produces the reference feature list (m/z values, retention times, peak areas)
    that MetMatch uses as the alignment reference
- name: Java JRE 1.8
  role: Runtime environment required to execute MetMatch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:a8829a38c48ff62095ce9dfbd50f220d93391c6dd01a3f54c97cea8c5789bbb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/S-T-K/MetMatch
  tools:
  - Java JRE 1.8
  - MetMatch
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# toxin-detoxification-metabolomics-data-processing

## Summary

Semi-automated alignment and comparison of LC-HRMS metabolomics datasets using MetMatch, exemplified on T-2 toxin/HT-2 toxin detoxification products in barley, correcting m/z and retention time shifts against an XCMS-derived reference feature list and combining matched ions into a single data matrix.

## When to use

Use when comparing untargeted LC-HRMS feature lists across multiple measurement sequences/batches (e.g. five barley target files vs. a reference experiment) where retention time shifts (here -9 to -24 s) and m/z deviations must be corrected before metabolite matching.

## When NOT to use

- The input is raw unprocessed LC-HRMS data with no XCMS-derived feature list — MetMatch aligns processed feature lists, not raw chromatograms
- Only a single measurement sequence/batch exists with no reference experiment to align against
- The goal is de novo peak detection or identification rather than comparison and alignment of already-detected features across experiments

## Inputs

- XCMS-derived reference feature list with m/z values, retention times, and peak areas
- Multiple LC-HRMS target chromatograms from a metabolomics experiment (e.g. five barley T-2/HT-2 toxin detoxification data files)
- Java runtime environment (JRE 1.8 or newer)

## Outputs

- Single aligned data matrix of matched features containing chromatographic peak areas, m/z values, and retention times
- Alignment statistics (e.g. 36 matched T-2/HT-2 toxin metabolite ions, 32 matched in all target chromatograms)
- Retention time shift function with shift range (e.g. -9 to -24 s) for comparison with XCMS

## How to apply

Load an XCMS-processed reference feature list (m/z values, retention times, peak areas) into MetMatch, which requires Java JRE 1.8 or newer, then load the target LC-HRMS chromatograms (e.g. five barley files from the T-2/HT-2 toxin detoxification experiment). Run the semi-automated alignment allowing m/z deviations and accounting for differently formed ion species, then apply MetMatch's non-linear retention time shift correction so features are matched over the observed shift range (here -9 to -24 s). Combine corresponding ions into a single data matrix of chromatographic peak areas, m/z values, and retention times for all matched features. Validate by counting matched metabolite ions (expect 36 T-2/HT-2-derived ions, 32 matched in all target chromatograms) and by computing the Pearson correlation between the MetMatch and XCMS retention time shift functions over the 4-22.5 min window (expect 0.966).

## Related tools

- **MetMatch** (Semi-automated alignment tool: matches target ions against the reference feature list, corrects non-linear retention time shifts, and combines matched ions into a single data matrix) — https://github.com/S-T-K/MetMatch
- **XCMS** (Produces the reference feature list (m/z values, retention times, peak areas) that MetMatch uses as the alignment reference)
- **Java JRE 1.8** (Runtime environment required to execute MetMatch)

## Evaluation signals

- Count of matched metabolite ions equals the expected reference quantity (36 T-2/HT-2 toxin-derived ions in the barley test case)
- Number of features matched in all target chromatograms equals 32 of 36; investigate failures (3 peaks failed due to early Rt with too few neighboring peaks, 1 missed in two files due to low abundance and poor peak shape)
- Retention time shifts of matched features fall within the observed range of -9 to -24 s
- Pearson correlation between the MetMatch and XCMS retention time shift functions over the 4-22.5 min window is approximately 0.966
- Output matrix contains peak areas, m/z values, and retention times for every matched feature across all target chromatograms

## Limitations

- MetMatch requires Java runtime (JRE) version 1.8 or newer
- Peak matching can fail for peaks with early retention times that have too few neighboring peaks, and for low-abundance peaks with poor peak shape
- The software is distributed WITHOUT ANY WARRANTY under the GNU General Public License (version 2 or later)

## Evidence

- [abstract] abstract: "Here, a novel software tool for the semi-automated alignment of different measurement sequences is presented."
- [intro] intro: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] readme: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
