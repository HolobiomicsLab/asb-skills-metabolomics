---
name: ion-species-mz-transformation
description: Use when reference and target LC-HRMS feature lists contain the same
  metabolites detected as different ion species/adducts, and peak matching fails or
  must be tested against an adduct swap.
when_to_use_negative:
- Reference and target features are already recorded as the same ion species, so no
  adduct transformation is needed before alignment
- The input is already a post-alignment combined data matrix rather than an unaligned
  reference/target feature list
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: MetMatch
  role: Semi-automated alignment of the modified reference and target feature lists,
    correcting m/z and retention time shifts and matching peaks across different ion
    species
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime dependency for executing MetMatch (JRE version 1.8 or newer)
- name: LC-HRMS
  role: Analytical technique generating the chromatographic feature lists that are
    transformed and aligned
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:052f2399f9ed504545adce1fa5fe5e1082f1995e70af0533ed4159fddda3b2d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - LC-HRMS
  - MetMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ion-species-mz-transformation

## Summary

Transform LC-HRMS feature m/z values between ion species (e.g., simulated A1-to-A2 adduct swap) so that MetMatch can align reference and target feature lists whose metabolites are detected as differently formed ion species.

## When to use

Use when reference and target LC-HRMS feature lists contain the same metabolites detected as different ion species/adducts, and peak matching fails or must be tested against an adduct swap.

## When NOT to use

- Reference and target features are already recorded as the same ion species, so no adduct transformation is needed before alignment
- The input is already a post-alignment combined data matrix rather than an unaligned reference/target feature list

## Inputs

- Reference feature list of LC-HRMS chromatographic features with m/z values and retention times
- Target ion list containing the A1 and A2 ion species of each metabolite
- Ion-species (adduct) mass transformation mapping between A1 and A2

## Outputs

- Transformed reference feature list with recalculated A2 m/z values
- MetMatch alignment output (matched peaks with combined peak areas, m/z values, and retention times)
- table4_matched_peaks.csv recording the confirmed match count (1446 matched peaks)

## How to apply

Load the benchmark reference feature list of m/z values and retention times that serves as the alignment reference. Recalculate every reference feature's m/z so each metabolite is represented as a different ion species in reference vs. target (A1 in reference, A2 in target), using the ion-species mass transformation supported by MetMatch. Assemble a target ion list containing both the A1 and A2 ion species of every metabolite. Run MetMatch (requires Java JRE 1.8 or newer) to align the modified reference and target lists, correcting for m/z and retention time shifts, then compare the alignment output against the Section 3.10.1 benchmark (Table 4).

## Related tools

- **MetMatch** (Semi-automated alignment of the modified reference and target feature lists, correcting m/z and retention time shifts and matching peaks across different ion species) — github.com/S-T-K/MetMatch
- **Java** (Runtime dependency for executing MetMatch (JRE version 1.8 or newer))
- **LC-HRMS** (Analytical technique generating the chromatographic feature lists that are transformed and aligned)

## Evaluation signals

- MetMatch matches 1446 chromatographic peaks to features despite the simulated A1-to-A2 ion-species change
- Alignment results are identical to the Section 3.10.1 benchmark evaluation (Table 4)
- The confirmed count of 1446 matched peaks is written to table4_matched_peaks.csv and agrees with the benchmark
- The alignment output combines peak areas, m/z values, and retention times into a single data matrix without ion-species-dependent mismatches

## Limitations

- MetMatch requires Java runtime (JRE) version 1.8 or newer
- The tool is distributed WITHOUT ANY WARRANTY; issues and bugs must be reported to the GitHub repository
- Validation of the ion-species swap relies on a specific benchmark (1446 peaks, Table 4); other adduct pairs or datasets are not covered by the provided evidence

## Evidence

- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [abstract] the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
