---
name: adduct-aware-ion-species-matching
description: Use when aligning multiple processed LC-HRMS target feature lists against
  a reference feature list where the same metabolite appears as different ion adducts,
  and naive m/z-only matching undercounts matched features.
when_to_use_negative:
- Input is already an aligned combined feature matrix — MetMatch is for aligning unaligned
  per-file feature lists against a reference
- LC-HRMS batches exhibit no m/z or retention time shifts relative to a reference
  chromatogram, so no correction/alignment is needed
- The metabolites of interest do not form multiple ion species, and simple m/z matching
  without adduct handling suffices (though this undercounts adduct-forming metabolites)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetMatch
  role: Semi-automated alignment tool that corrects m/z and retention time shifts
    and recognizes differently formed ion species when matching target features to
    the reference list
  repo: https://github.com/S-T-K/MetMatch
- name: Java
  role: Runtime dependency; MetMatch requires JRE 1.8 or newer
  repo: https://github.com/S-T-K/MetMatch
- name: LC-HRMS
  role: Analytical technique producing the untargeted metabolomics feature lists that
    are aligned (e.g., phenylalanine-derived metabolites in wheat)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:44329fa1df780f8e0d9624bf01f7e4a672941e576e3651d1ed583a86bb06d43d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/S-T-K/MetMatch
  tools:
  - Java
  - LC-HRMS
  - MetMatch
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-aware-ion-species-matching

## Summary

Aligns LC-HRMS feature lists across experiments with MetMatch so that differently formed ion species (adducts) of the same metabolite are recognized as corresponding features, combining matched chromatographic peak areas, m/z values and retention times into a single data matrix. Accounting for adducts substantially increases matched features (92 vs 42 of 151 Phe-derived wheat features).

## When to use

Use when aligning multiple processed LC-HRMS target feature lists against a reference feature list where the same metabolite appears as different ion adducts, and naive m/z-only matching undercounts matched features.

## When NOT to use

- Input is already an aligned combined feature matrix — MetMatch is for aligning unaligned per-file feature lists against a reference
- LC-HRMS batches exhibit no m/z or retention time shifts relative to a reference chromatogram, so no correction/alignment is needed
- The metabolites of interest do not form multiple ion species, and simple m/z matching without adduct handling suffices (though this undercounts adduct-forming metabolites)

## Inputs

- Processed LC-HRMS target feature lists (m/z and retention time per feature) from multiple datafiles, e.g., 22 wheat target datafiles
- A reference feature list serving as alignment anchor, e.g., a merged 1549-feature phenylalanine-derived metabolite reference list
- Java runtime environment (JRE) version 1.8 or newer

## Outputs

- Aligned data matrix combining matched chromatographic peak areas, m/z values and retention times across all datafiles (e.g., metmatch_alignment_results.csv)
- Per-feature match counts restricted to features detected in at least N target files (e.g., 92 of 151 Phe-derived features matched in >=5 files)
- Fitted retention-time shift functions over specified windows (e.g., 3-38 min and 7-38 min) comparable to XCMS Rt shift functions

## How to apply

Load the target feature lists (e.g., 22 wheat target datafiles) and the reference feature list (e.g., a merged 1549-feature phenylalanine-derived list) into MetMatch, which requires Java runtime (JRE) version 1.8 or newer. Enable adduct-aware matching so differently formed ion species of the same metabolite are treated as corresponding features, then run the alignment allowing m/z deviations (the wheat reproduction showed an average -3.4 ppm shift) and non-linear retention-time shifts (7-52 s range) relative to the reference chromatogram. Restrict matching to features detected in a minimum number of target files (e.g., at least 5) and count matches. As a rationale check, repeat the alignment with adduct accounting disabled: in the wheat case this dropped matches from 92/151 to 42/151 Phe-derived features, demonstrating the adduct term is decisive. Validate retention-time shift functions by fitting over defined windows (3-38 min and 7-38 min) and correlating with XCMS Rt shift functions (expected Pearson correlations 0.858 and 0.926). Export the combined matrix of matched chromatographic peak areas, m/z values and retention times.

## Related tools

- **MetMatch** (Semi-automated alignment tool that corrects m/z and retention time shifts and recognizes differently formed ion species when matching target features to the reference list) — https://github.com/S-T-K/MetMatch
- **Java** (Runtime dependency; MetMatch requires JRE 1.8 or newer) — https://github.com/S-T-K/MetMatch
- **LC-HRMS** (Analytical technique producing the untargeted metabolomics feature lists that are aligned (e.g., phenylalanine-derived metabolites in wheat))

## Evaluation signals

- Match count with adduct accounting enabled matches the reported 92 of 151 Phe-derived features matched in at least 5 target datafiles
- Match count with adduct accounting disabled drops to 42, confirming the adduct-aware setting is active and material
- Average m/z deviation reproduces the reported -3.4 ppm and observed Rt shifts fall in the 7-52 s range
- Pearson correlations of MetMatch vs XCMS retention-time shift functions equal 0.858 (3-38 min window) and 0.926 (7-38 min window)
- Exported alignment output is a single combined matrix containing chromatographic peak areas, m/z values and retention times for all matched features

## Limitations

- Matching tolerance must accommodate observed shifts (e.g., -3.4 ppm, 7-52 s Rt shifts); deviations outside configured tolerances will be missed
- Without adduct-aware matching, differently formed ion species of the same metabolite are not combined, drastically reducing matches (92 vs 42 features in the wheat evaluation)
- MetMatch is distributed under GNU GPL without any warranty (NOT EVEN THE IMPLIED WARRANTY OF MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE)
- Discussion-section content of the source article was not available, so caveats beyond those reported could not be extracted

## Evidence

- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [abstract] Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix.: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [intro] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] MetMatch is free software ... distributed WITHOUT ANY WARRANTY (README license terms).: "MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE."
- [readme] MetMatch corrects LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram (README).: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
