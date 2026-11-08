---
name: ppm-mass-deviation-evaluation
description: Use when aligning processed LC-HRMS feature lists (m/z, retention time)
  across batches and a systematic ppm-level mass shift (e.g. -3.4 ppm) plus non-linear
  retention-time shifts (e.g. 7-52 s) must be corrected against a reference chromatogram
  before feature matching.
when_to_use_negative:
- Input is raw LC-HRMS vendor data that has not yet been processed into a feature
  list with m/z values and retention times; MetMatch aligns pre-processed feature
  lists
- The metabolite experiment does not exhibit m/z or retention-time shifts relative
  to a reference chromatogram, so correction and ppm-deviation evaluation are unnecessary
- Differently formed ion adducts are not expected or the reference list already collapses
  adducts, in which case adduct-aware matching changes little (though note the 42
  vs 92 match difference observed in the wheat case)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetMatch
  role: Semi-automated alignment tool used to correct ppm-level m/z deviations and
    retention-time shifts relative to a reference feature list and to match features
    with adduct awareness
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Required runtime environment for executing the MetMatch application (Java
    JRE version 1.8 or newer)
- name: LC-HRMS
  role: Analytical technique generating the high-resolution feature data (m/z, retention
    time) on which the ppm mass-deviation evaluation is performed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:b57773b3676629ad7d0bff6c50fc7a16b30d1348c3ca5157af923fac1adb715e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Java JRE 1.8
  - LC-HRMS
  - MetMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ppm-mass-deviation-evaluation

## Summary

A skill for evaluating and correcting ppm-scale systematic m/z deviations (e.g. the -3.4 ppm average shift observed in the wheat phenylalanine-derived metabolite experiment) when aligning LC-HRMS feature lists across measurement batches using MetMatch. Accurate m/z-deviation handling, combined with adduct-aware matching, directly determines how many metabolite features can be matched across experiments.

## When to use

Use when aligning processed LC-HRMS feature lists (m/z, retention time) across batches and a systematic ppm-level mass shift (e.g. -3.4 ppm) plus non-linear retention-time shifts (e.g. 7-52 s) must be corrected against a reference chromatogram before feature matching.

## When NOT to use

- Input is raw LC-HRMS vendor data that has not yet been processed into a feature list with m/z values and retention times; MetMatch aligns pre-processed feature lists
- The metabolite experiment does not exhibit m/z or retention-time shifts relative to a reference chromatogram, so correction and ppm-deviation evaluation are unnecessary
- Differently formed ion adducts are not expected or the reference list already collapses adducts, in which case adduct-aware matching changes little (though note the 42 vs 92 match difference observed in the wheat case)

## Inputs

- Processed target LC-HRMS feature lists (m/z values and retention times) from untargeted metabolomics measurements, e.g. 22 wheat target datafiles
- A reference feature list serving as the alignment basis, e.g. a merged 1549-feature phenylalanine-derived metabolite list
- Estimated tolerances: m/z deviation in ppm (average -3.4 ppm) and retention-time shift range (7-52 s)

## Outputs

- Aligned/combined data matrix of matched chromatographic peak areas, m/z values and retention times (e.g. metmatch_alignment_results.csv)
- Match counts per feature and minimum-file threshold, e.g. 92 of 151 Phe-derived features matched in at least five target datafiles
- Fitted retention-time shift functions over the 3-38 min and 7-38 min windows for comparison against XCMS

## How to apply

Load processed target feature lists (e.g. 22 wheat target datafiles) and a reference feature list (the merged 1549-feature phenylalanine-derived list) into MetMatch, which requires Java runtime (JRE) version 1.8 or newer. Configure the alignment to allow m/z deviations sized to the observed systematic shift — in the wheat case an average of -3.4 ppm — and non-linear retention-time shifts in the 7-52 s range relative to the reference chromatogram. Enable adduct-aware matching so that differently formed ion species of the same metabolite are recognized as corresponding features; this is critical, since with adduct accounting 92 of 151 Phe-derived features were matched in at least five target files versus only 42 without it. Validate the correction by fitting MetMatch retention-time shift functions over the 3-38 min and 7-38 min windows and computing Pearson correlations against XCMS Rt shift functions (expected 0.858 and 0.926). Export chromatographic peak areas, m/z values and retention times as a single combined data matrix.

## Related tools

- **MetMatch** (Semi-automated alignment tool used to correct ppm-level m/z deviations and retention-time shifts relative to a reference feature list and to match features with adduct awareness) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Required runtime environment for executing the MetMatch application (Java JRE version 1.8 or newer))
- **LC-HRMS** (Analytical technique generating the high-resolution feature data (m/z, retention time) on which the ppm mass-deviation evaluation is performed)

## Evaluation signals

- Reproduce 92 of 151 Phe-derived metabolite features matched in at least five of the 22 wheat target datafiles when adduct-aware matching is enabled
- Verify that disabling accounting for differently formed ion adducts drops matches to 42 of the 151 Phe-derived features
- Confirm the measured average m/z deviation matches -3.4 ppm and observed retention-time shifts fall in the 7-52 s range
- Check Pearson correlations of MetMatch retention-time shift functions against XCMS Rt shift functions reach 0.858 (3-38 min window) and 0.926 (7-38 min window)
- Validate that the exported combined data matrix (metmatch_alignment_results.csv) contains matched chromatographic peak areas, m/z values and retention times for the aligned features

## Limitations

- MetMatch requires Java runtime (JRE) version 1.8 or newer
- The software is distributed without any warranty ('WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE')
- Failing to account for differently formed ion species of detected metabolites substantially reduces matched features (42 vs 92 of 151 in the wheat evaluation), so results depend critically on adduct-aware configuration

## Evidence

- [abstract] the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [intro] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
