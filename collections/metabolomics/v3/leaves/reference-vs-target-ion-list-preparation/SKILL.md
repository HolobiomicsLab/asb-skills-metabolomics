---
name: reference-vs-target-ion-list-preparation
description: Use when aligning untargeted LC-HRMS metabolomics batches where the same
  metabolites may be detected as different ion species (e.g., A1 vs. A2 adducts) and
  features must be matched across m/z and retention time shifts.
when_to_use_negative:
- The reference and target data were measured under identical ion-species conditions
  and no adduct transformation is needed — use plain benchmark alignment instead
- Input is already an aligned combined data matrix rather than raw per-batch feature
  lists
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMatch
  role: Semi-automated alignment of reference and target feature lists, correcting
    m/z and retention time shifts and accounting for differently formed ion species
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to execute MetMatch
- name: LC-HRMS
  role: Analytical technique generating the chromatographic data from which the reference
    and target feature lists are derived
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:7e2eb550617fc06428bc7ba39a113f634291cee660a044899f7734256280ea82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Java JRE 1.8
  - LC-HRMS
  - MetMatch
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# reference-vs-target-ion-list-preparation

## Summary

Prepare the reference and target feature lists (m/z values and retention times) for MetMatch alignment of LC-HRMS data, including recalculating reference m/z values so metabolites appear as different ion species (e.g., A1 in reference vs. A2 in target) when comparing experiments with differing ion formation.

## When to use

Use when aligning untargeted LC-HRMS metabolomics batches where the same metabolites may be detected as different ion species (e.g., A1 vs. A2 adducts) and features must be matched across m/z and retention time shifts.

## When NOT to use

- The reference and target data were measured under identical ion-species conditions and no adduct transformation is needed — use plain benchmark alignment instead
- Input is already an aligned combined data matrix rather than raw per-batch feature lists

## Inputs

- Reference feature list (processed LC-HRMS chromatograms with m/z values and retention times)
- Target ion list containing the A1 and A2 ion species of every metabolite
- Ion-species mass transformation parameters for converting reference metabolites from A1 to A2

## Outputs

- Modified reference feature list with recalculated A2 m/z values
- MetMatch alignment output (matched chromatographic peak areas, m/z values and retention times combined into a single data matrix)
- table4_matched_peaks.csv recording the confirmed 1446 matched peaks

## How to apply

Load a benchmark reference feature list of m/z values and retention times that will serve as the alignment reference. Recalculate all reference feature m/z values so each metabolite is represented as a different ion species in reference vs. target (A1 in reference, A2 in target), using the ion-species mass transformation supported by MetMatch. Assemble the target ion list containing both the A1 and A2 ion species of every metabolite. Run MetMatch (requires Java JRE 1.8 or newer) to align the modified reference and target feature lists, correcting for m/z and retention time shifts; MetMatch is designed to account for differently formed ion species of detected metabolites. Judge success by comparing the alignment output against a benchmark evaluation (e.g., the Section 3.10.1 benchmark, Table 4), where correct ion-species handling yields 1446 matched chromatographic peaks identical to the benchmark result.

## Related tools

- **MetMatch** (Semi-automated alignment of reference and target feature lists, correcting m/z and retention time shifts and accounting for differently formed ion species) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute MetMatch)
- **LC-HRMS** (Analytical technique generating the chromatographic data from which the reference and target feature lists are derived)

## Evaluation signals

- Alignment output matches the Section 3.10.1 benchmark (Table 4) exactly
- 1446 chromatographic peaks are matched to reference features despite the simulated A1-to-A2 ion-species change
- Results are identical to the benchmark evaluation where no ion-species swap was simulated
- MetMatch reports no runtime errors under JRE 1.8 or newer and processes both A1 and A2 target ions

## Limitations

- Results rely on MetMatch correctly accounting for differently formed ion species; mis-specified adduct transformations could reduce matches below the 1446-peak benchmark
- MetMatch requires Java runtime (JRE) version 1.8 or newer
- MetMatch is distributed WITHOUT ANY WARRANTY, per the GNU GPL under which it is licensed
- Discussion-section content of the source article was not available for further caveats

## Evidence

- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [intro] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
