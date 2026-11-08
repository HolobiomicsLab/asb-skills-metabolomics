---
name: reference-feature-list-matching
description: Use when multiple LC-HRMS data files from different metabolomics experiments
  must be matched to a reference feature list so corresponding ions across batches
  can be compared, e.g. after XCMS feature extraction with observed m/z and retention
  time shifts.
when_to_use_negative:
- Input is already an aligned combined feature matrix — MetMatch is for matching target
  chromatograms against a reference feature list, not for further statistical analysis
  of aligned data
- Only a single chromatogram exists with no separate reference feature list from a
  prior experiment — no reference to match against
- Data are not LC-HRMS chromatograms with m/z and retention time values (e.g. GC-MS
  or direct-infusion spectra without retention time dimension)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMatch
  role: Performs the semi-automated m/z and retention time shift correction and matching
    of target chromatograms against the reference feature list
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to execute MetMatch (JRE version 1.8 or newer)
- name: XCMS
  role: Produces the processed reference feature list (m/z values, retention times,
    peak areas) that MetMatch matches against; its retention time shift function serves
    as a validation comparator
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:31c238e95090ed9f4f3e9f184bcf7589e17152f19f01388c8b128997461c3a13
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE
  - Java JRE 1.8
  - MetMatch
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# reference-feature-list-matching

## Summary

Semi-automated alignment of LC-HRMS target chromatograms against a processed reference feature list (m/z and retention time) using MetMatch, which corrects m/z offsets and non-linear retention time shifts and combines matched chromatographic peak areas, m/z values and retention times into a single data matrix.

## When to use

Use when multiple LC-HRMS data files from different metabolomics experiments must be matched to a reference feature list so corresponding ions across batches can be compared, e.g. after XCMS feature extraction with observed m/z and retention time shifts.

## When NOT to use

- Input is already an aligned combined feature matrix — MetMatch is for matching target chromatograms against a reference feature list, not for further statistical analysis of aligned data
- Only a single chromatogram exists with no separate reference feature list from a prior experiment — no reference to match against
- Data are not LC-HRMS chromatograms with m/z and retention time values (e.g. GC-MS or direct-infusion spectra without retention time dimension)

## Inputs

- Reference feature list of processed LC-HRMS chromatograms with m/z values and retention times (e.g. XCMS-derived)
- Target LC-HRMS chromatogram data files from one or more measurement sequences (e.g. five barley T-2/HT-2 toxin detoxification files)
- Matching settings: allowed m/z deviation (ppm), histogram bin width (0.01 ppm), retention time shift correction

## Outputs

- Single combined data matrix of chromatographic peak areas, m/z values and retention times for all matched features
- Alignment statistics: number of matched metabolite ions, number matched in all target chromatograms, retention time shift function, detected m/z offsets

## How to apply

Load the XCMS-derived reference feature list (m/z values, retention times, peak areas) into MetMatch, which requires Java JRE 1.8 or newer, then load the target LC-HRMS chromatograms (e.g. five barley T-2/HT-2 toxin detoxification files). Run MetMatch's semi-automated alignment, allowing m/z deviations and accounting for differently formed ion species, and apply its non-linear retention time shift correction so target features match over observed shift ranges (e.g. -9 to -24 s). For m/z offset detection, MetMatch estimates the offset using histogram bins of 0.01 ppm. Combine corresponding ions into a single data matrix of peak areas, m/z values and retention times, then validate alignment statistics (e.g. 36 of 36 T-2/HT-2 metabolite ions matched, 32 in all target files, Pearson correlation 0.966 between MetMatch and XCMS retention time shift functions over the 4-22.5 min window).

## Related tools

- **MetMatch** (Performs the semi-automated m/z and retention time shift correction and matching of target chromatograms against the reference feature list) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute MetMatch (JRE version 1.8 or newer))
- **XCMS** (Produces the processed reference feature list (m/z values, retention times, peak areas) that MetMatch matches against; its retention time shift function serves as a validation comparator)

## Evaluation signals

- In the barley T-2/HT-2 toxin test case, all 36 T-2 toxin/HT-2 toxin-derived metabolite ions are matched to the reference and 32 are matched across all five target chromatograms (three peaks fail due to early Rt with too few neighboring peaks; one is missed in two files due to low abundance and poor peak shape)
- Retention time shifts of matched features fall in the observed -9 to -24 s range after non-linear correction
- Pearson correlation between the MetMatch and XCMS retention time shift functions over the 4-22.5 min window is approximately 0.966
- For artificially imposed m/z offsets from -10 to +10 ppm in 1 ppm steps, detected offsets differ from imposed values by within +0.01 to -0.05 ppm, with no correlation between offset magnitude and matching accuracy
- Every matched feature in the output matrix carries peak area, m/z and retention time entries for each target chromatogram

## Limitations

- Peaks eluting early (low Rt) can fail matching because too few neighboring peaks are available to anchor the retention time shift correction
- Low-abundance peaks with poor peak shape may be missed in some target chromatograms (e.g. one ion missed in two of five barley files)
- MetMatch requires Java runtime (JRE) version 1.8 or newer; the tool is distributed under GPLv2 without warranty

## Evidence

- [abstract] Corresponding ions and metabolites are matched and combined into a single data matrix: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [intro] Retains reference-based alignment of measurement sequences: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [other] Non-linear retention time shift correction of target features: [evidence span withheld — anchor status: not_found]
- [abstract] Ion species and alignment of differently formed ion species: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [readme] Java runtime requirement for MetMatch: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
