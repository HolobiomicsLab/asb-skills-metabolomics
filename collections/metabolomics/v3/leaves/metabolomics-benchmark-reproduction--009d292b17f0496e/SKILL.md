---
name: metabolomics-benchmark-reproduction
description: Use when a task requires validating or reproducing published feature-alignment
  results for LC-HRMS metabolomics data, especially simulated ion-species swap tests
  comparing a reference feature list against a target ion list.
when_to_use_negative:
- The data are raw, unprocessed LC-HRMS chromatograms rather than a processed feature
  list (m/z values and retention times) that can serve as an alignment reference.
- The goal is de novo peak detection or statistical metabolite identification rather
  than comparison and alignment of feature lists from different measurement sequences.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetMatch
  role: Semi-automated tool that aligns the modified reference and target feature
    lists, correcting m/z and retention time shifts and accounting for differently
    formed ion species of detected metabolites
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment required to execute MetMatch (JRE version 1.8 or newer)
- name: LC-HRMS
  role: Analytical technique generating the chromatographic data whose feature lists
    are aligned and compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:4e58ae5fa59fefb467398e5d879e49c4425e8d43381362ad664cc49c9e889c01
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - LC-HRMS
  - MetMatch
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# metabolomics-benchmark-reproduction

## Summary

Reproduce a published LC-HRMS feature-alignment benchmark by re-running the MetMatch alignment workflow (m/z and retention time correction, ion-species transformation) and verifying that match counts equal the reported reference values, e.g. 1446 matched peaks against Table 4 of the benchmark evaluation.

## When to use

Use when a task requires validating or reproducing published feature-alignment results for LC-HRMS metabolomics data, especially simulated ion-species swap tests comparing a reference feature list against a target ion list.

## When NOT to use

- The data are raw, unprocessed LC-HRMS chromatograms rather than a processed feature list (m/z values and retention times) that can serve as an alignment reference.
- The goal is de novo peak detection or statistical metabolite identification rather than comparison and alignment of feature lists from different measurement sequences.

## Inputs

- Benchmark reference feature list with m/z values and retention times
- Target ion list containing A1 and A2 ion species of every metabolite
- Published benchmark evaluation results (Section 3.10.1, Table 4)
- MetMatch installation with Java runtime (JRE) version 1.8 or newer

## Outputs

- MetMatch alignment output pairing reference and target features across m/z and retention time shifts
- table4_matched_peaks.csv with the confirmed count of 1446 matched peaks

## How to apply

Load the benchmark reference feature list (m/z values and retention times) that serves as the alignment reference. Recalculate all reference feature m/z values so each metabolite is represented as a different ion species in reference vs. target (A1 in reference, A2 in target), applying the ion-species mass transformation supported by MetMatch. Assemble a target ion list containing both the A1 and A2 ion species of every metabolite. Run MetMatch — which requires Java runtime (JRE) version 1.8 or newer — to align the modified reference and target lists, correcting for m/z and retention time shifts relative to a reference chromatogram. Compare the MetMatch alignment output against the Section 3.10.1 benchmark (Table 4); success is confirmed when the number of matched peaks is identical to the benchmark evaluation, which here was 1446 chromatographic peaks.

## Related tools

- **MetMatch** (Semi-automated tool that aligns the modified reference and target feature lists, correcting m/z and retention time shifts and accounting for differently formed ion species of detected metabolites) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment required to execute MetMatch (JRE version 1.8 or newer))
- **LC-HRMS** (Analytical technique generating the chromatographic data whose feature lists are aligned and compared)

## Evaluation signals

- The number of matched peaks from the simulated ion-species swap alignment equals the Section 3.10.1 benchmark value in Table 4 (1446 matched peaks).
- MetMatch output is identical to the benchmark evaluation despite all reference features being simulated as a different ion species (A1 in reference vs. A2 in target).
- The alignment correctly recognizes both m/z and retention time shifts between the reference and target data files.
- The confirmed result is written to table4_matched_peaks.csv and matches the published Table 4 count.

## Limitations

- MetMatch requires Java runtime (JRE) version 1.8 or newer; older environments cannot run the alignment.
- MetMatch is distributed under GPL without warranty, as stated in the repository README.
- The tool performs semi-automated comparison based on a processed reference feature list; it depends on that list serving as the alignment reference.

## Evidence

- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [abstract] the tool recognizes both m/z and retention time shifts: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [intro] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [intro] It provides methods to correct LC-HRMS data: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch is a software tool for the semi-automated comparison: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
