---
name: lc-hrms-feature-alignment
description: Use when multiple LC-HRMS measurement sequences or batches need to be
  compared/aligned to a reference feature list, and m/z deviations, retention-time
  shifts (e.g. 7-52 s), or differing ion adducts of the same metabolite complicate
  direct feature matching.
when_to_use_negative:
- Data are unprocessed raw chromatograms without a processed reference feature list
  of m/z and retention-time values
- Peaks are already aligned into a single combined data matrix — re-alignment would
  be redundant
- The comparison involves a single dataset with no reference chromatogram and no expected
  m/z or retention-time shifts between batches
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetMatch
  role: Semi-automated alignment tool that matches corresponding ions and metabolites
    between reference and target feature lists, correcting m/z and retention-time
    shifts and recombining adduct variants
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment required to execute MetMatch (JRE version 1.8 or newer)
- name: XCMS
  role: Upstream processing tool producing the reference feature lists and providing
    retention-time shift functions used as correlation benchmarks
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_001
  - task_004
  - task_007
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:3b17a287f3a9fe7d26584a0e93b0a84f358c3f68b5912398212f6090d8f99229
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Java JRE 1.8
  - LC-HRMS
  - MetMatch
  - XCMS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# lc-hrms-feature-alignment

## Summary

Semi-automated alignment of LC-HRMS feature lists from different metabolomics experiments using MetMatch, which corrects m/z and retention-time shifts against a reference chromatogram and accounts for differently formed ion species, combining matched peak areas, m/z values and retention times into a single data matrix.

## When to use

Use when multiple LC-HRMS measurement sequences or batches need to be compared/aligned to a reference feature list, and m/z deviations, retention-time shifts (e.g. 7-52 s), or differing ion adducts of the same metabolite complicate direct feature matching.

## When NOT to use

- Data are unprocessed raw chromatograms without a processed reference feature list of m/z and retention-time values
- Peaks are already aligned into a single combined data matrix — re-alignment would be redundant
- The comparison involves a single dataset with no reference chromatogram and no expected m/z or retention-time shifts between batches

## Inputs

- Reference feature list (m/z values, retention times, peak areas) from a reference chromatogram, e.g. an XCMS-derived or merged 1549-feature Phe-derived list
- Target LC-HRMS datafiles/feature lists from different experiments or batches (e.g. 22 wheat target files or 5 barley T-2/HT-2 toxin files)
- MetMatch tool running under Java JRE 1.8 or newer

## Outputs

- Aligned data matrix combining chromatographic peak areas, m/z values and retention times of matched features across all target files
- Match counts per feature/file threshold (e.g. 92/151 Phe features matched in ≥5 files; 32/36 barley ions matched in all files)
- Retention-time shift functions (MetMatch vs XCMS) with Pearson correlations, e.g. 0.858, 0.926, 0.966

## How to apply

Load an XCMS-derived reference feature list (m/z values, retention times, peak areas) and the target feature lists into MetMatch (requires Java JRE 1.8+). Run the semi-automated alignment allowing small m/z deviations (the wheat evaluation showed an average -3.4 ppm shift) and non-linear retention-time shifts (observed ranges: 7-52 s for wheat, -9 to -24 s for barley over the 4-22.5 min window). Enable adduct-aware matching so differently formed ion species of the same metabolite are recognized — accounting for adducts raised matched wheat Phe-derived features from 42 to 92 of 151. Restrict matches to features detected in a minimum number of target files (e.g. at least 5 of 22 wheat files) before counting. Validate by computing Pearson correlation of the MetMatch vs XCMS retention-time shift functions (reported 0.858/0.926 for wheat, 0.966 for barley) and export the combined data matrix.

## Related tools

- **MetMatch** (Semi-automated alignment tool that matches corresponding ions and metabolites between reference and target feature lists, correcting m/z and retention-time shifts and recombining adduct variants) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment required to execute MetMatch (JRE version 1.8 or newer))
- **XCMS** (Upstream processing tool producing the reference feature lists and providing retention-time shift functions used as correlation benchmarks)

## Evaluation signals

- Matched-feature counts reproduce published values: 92/151 Phe-derived features matched in ≥5 of 22 wheat files with adduct accounting (42 without), and 36 barley T-2/HT-2 ions with 32 matched in all five files
- Average m/z deviation in the aligned wheat data is about -3.4 ppm and retention-time shifts fall in the reported 7-52 s (wheat) or -9 to -24 s (barley) ranges
- Pearson correlation between MetMatch and XCMS retention-time shift functions matches reported values (0.858 and 0.926 for the 3-38 min and 7-38 min wheat windows; 0.966 for the 4-22.5 min barley window)
- Benchmark matching of the unmodified Phe reference against its own feature list recovers 1446 of 1482 features (F-measure 0.988)
- Exported matrix contains peak areas, m/z values and retention times for every matched feature across all target chromatograms

## Limitations

- Peaks appearing very early in the chromatogram can fail matching due to too few neighboring peaks (three barley peaks were missed for this reason)
- Low-abundance features with poor peak shape may be missed in some target files (one barley ion was missed in two files)
- Without adduct-aware matching, many correspondences are lost: only 42/151 wheat Phe features were matched when differently formed ion species were not accounted for
- MetMatch is distributed under GPLv2+ WITHOUT ANY WARRANTY; the discussion section contains no additional validation guidance

## Evidence

- [abstract] Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference: "Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference"
- [abstract] Adduct accounting abstract quote: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [readme] README Java requirement: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] README purpose: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [abstract] Adduct design abstract: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
