---
name: metabolite-ion-species-matching
description: Use when comparing untargeted LC-HRMS metabolomics experiments (e.g.
  T-2/HT-2 toxin detoxification products in barley) where target files show m/z and
  retention time shifts relative to a reference feature list.
when_to_use_negative:
- Input is already an aligned feature matrix — MetMatch is for aligning/comparing
  different unprocessed measurement sequences, not re-processing combined matrices
- Data are not LC-HRMS chromatograms with m/z and retention time information (the
  tool corrects LC-HRMS data exhibiting m/z and retention time shifts)
- No reference feature list is available, since alignment is performed against a reference
  feature list
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MetMatch
  role: Semi-automated tool that aligns target LC-HRMS chromatograms to a reference
    feature list, correcting m/z and retention time shifts and matching differently
    formed ion species
  repo: https://github.com/S-T-K/MetMatch
- name: XCMS
  role: Produces the reference feature list (processed chromatograms with m/z values
    and retention times) used as the alignment reference
- name: Java JRE 1.8
  role: Runtime environment required to execute MetMatch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:659ab39ab4b1df0385b97b7286bddfa6fcaf486271d059fc19db6182c4b5dbef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/S-T-K/MetMatch
  tools:
  - Java JRE 1.8
  - MetMatch
  - XCMS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-ion-species-matching

## Summary

Semi-automated alignment of LC-HRMS target chromatograms against an XCMS-derived reference feature list using MetMatch, correcting m/z and non-linear retention time shifts (e.g. -9 to -24 s) and accounting for differently formed ion species of metabolites.

## When to use

Use when comparing untargeted LC-HRMS metabolomics experiments (e.g. T-2/HT-2 toxin detoxification products in barley) where target files show m/z and retention time shifts relative to a reference feature list.

## When NOT to use

- Input is already an aligned feature matrix — MetMatch is for aligning/comparing different unprocessed measurement sequences, not re-processing combined matrices
- Data are not LC-HRMS chromatograms with m/z and retention time information (the tool corrects LC-HRMS data exhibiting m/z and retention time shifts)
- No reference feature list is available, since alignment is performed against a reference feature list

## Inputs

- XCMS-derived reference feature list with m/z values, retention times and peak areas
- Target LC-HRMS chromatogram data files from a different measurement sequence/batch (e.g. five barley files from a T-2/HT-2 toxin detoxification experiment)

## Outputs

- Matched feature set linking target ions to reference list entries
- Single combined data matrix of chromatographic peak areas, m/z values and retention times for all matched features
- Retention time shift function and alignment statistics (number of matched ions, count matched in all target chromatograms)

## How to apply

Load the XCMS-derived reference feature list (m/z values, retention times, peak areas) into MetMatch and load the target LC-HRMS chromatograms (e.g. five barley detoxification-experiment files). Run MetMatch's semi-automated alignment, allowing m/z deviations and accounting for differently formed ion species, then apply the non-linear retention time shift correction over the observed shift range (-9 to -24 s in the barley case). Corresponding ions are matched and combined into a single data matrix of chromatographic peak areas, m/z values and retention times. MetMatch requires Java runtime (JRE) version 1.8 or newer.

## Related tools

- **MetMatch** (Semi-automated tool that aligns target LC-HRMS chromatograms to a reference feature list, correcting m/z and retention time shifts and matching differently formed ion species) — https://github.com/S-T-K/MetMatch
- **XCMS** (Produces the reference feature list (processed chromatograms with m/z values and retention times) used as the alignment reference)
- **Java JRE 1.8** (Runtime environment required to execute MetMatch)

## Evaluation signals

- Count of metabolite ions matched against the reference list matches expectation (e.g. 36 T-2/HT-2 toxin-derived metabolite ions in the barley test case)
- Count of ions successfully matched in all target chromatograms matches expectation (e.g. 32 of 36 in the barley case)
- Retention time shift falls within the observed range (e.g. -9 to -24 s)
- Pearson correlation between the MetMatch and XCMS retention time shift functions over the 4–22.5 min window is high (e.g. 0.966)
- Output data matrix contains chromatographic peak areas, m/z values and retention times for all matched features

## Limitations

- Peaks eluting early with too few neighboring peaks fail retention time shift correction (three barley peaks failed for this reason)
- Low-abundance ions with poor peak shape may be missed in some target files (one barley ion was missed in two files)
- MetMatch requires Java runtime (JRE) version 1.8 or newer
- MetMatch is distributed WITHOUT ANY WARRANTY under the GNU GPL

## Evidence

- [intro] MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments.: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [abstract] Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix.: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
