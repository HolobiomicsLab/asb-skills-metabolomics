---
name: mixed-ionization-mode-feature-set-combination
description: Use when HRMS data were acquired in both positive and negative ionization
  modes and the goal is a single aligned feature table combining both polarities,
  rather than processing each mode independently.
when_to_use_negative:
- Only one ionization mode was acquired — a classical single-polarity patRoon workflow
  suffices
- Features have already been aligned into a combined feature table — re-running makeSet
  on an aligned table is unnecessary
- The analysis goal is transformation product screening on a single polarity rather
  than cross-polarity feature alignment
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: patRoon
  role: R package providing findFeatures, makeSet, groupFeatures/alignFeatures for
    the sets workflow combining positive/negative ionization data
  repo: https://github.com/rickhelmus/patRoon
- name: OpenMS
  role: Feature-detection and feature-grouping/alignment algorithm used within patRoon
    for each polarity and for the combined set
- name: patRoonData
  role: Example raw LC-HRMS data package (standard-X and solvent-X triplicates, positive
    and negative modes) used to demonstrate the workflow
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:bed8a7d18b4f7f937869e6e4ba3b157f8993cf2f319b9570e47612acc7b69f82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - OpenMS
  - R
  - patRoon
  - patRoon 2.0
  - patRoonData
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mixed-ionization-mode-feature-set-combination

## Summary

A patRoon 2.0 sets workflow that separately detects features in positive and negative ionization LC-HRMS analyses, combines them into a single set via makeSet with default adducts ([M+H]+ and [M-H]-), converts m/z to neutral masses, and aligns/groups features across polarities into one mixed-polarity featureGroupsSet.

## When to use

Use when HRMS data were acquired in both positive and negative ionization modes and the goal is a single aligned feature table combining both polarities, rather than processing each mode independently.

## When NOT to use

- Only one ionization mode was acquired — a classical single-polarity patRoon workflow suffices
- Features have already been aligned into a combined feature table — re-running makeSet on an aligned table is unnecessary
- The analysis goal is transformation product screening on a single polarity rather than cross-polarity feature alignment

## Inputs

- patRoonData example LC-HRMS data (standard-X and solvent-X triplicates) in both positive and negative ionization modes
- Positive-mode featureGroups object from findFeatures (OpenMS algorithm)
- Negative-mode featureGroups object from findFeatures (OpenMS algorithm)

## Outputs

- Mixed-polarity featureGroupsSet object aligned and grouped across polarities
- Exported CSV feature table of aligned mixed-polarity feature groups (e.g. feature_groups_pos_neg.csv)

## How to apply

First run findFeatures separately on the positive and negative analyses using the OpenMS feature-detection algorithm in patRoon. Combine the two featureGroups objects with makeSet, annotating each polarity with its adduct ([M+H]+ for positive, [M-H]- for negative). Then replace the feature m/z values in the set with neutral masses calculated from the adduct information (defined manually or via feature adduct annotations). Finally, run groupFeatures/alignFeatures with the OpenMS algorithm on the neutralized set to yield a mixed-polarity featureGroupsSet spanning both ionization modes. This exploits patRoon 2.0's ability to simultaneously process, integrate and interpret mixed-mode HRMS data, a functionality unavailable in most workflows due to complexity and long processing times.

## Related tools

- **patRoon** (R package providing findFeatures, makeSet, groupFeatures/alignFeatures for the sets workflow combining positive/negative ionization data) — https://github.com/rickhelmus/patRoon
- **OpenMS** (Feature-detection and feature-grouping/alignment algorithm used within patRoon for each polarity and for the combined set)
- **patRoonData** (Example raw LC-HRMS data package (standard-X and solvent-X triplicates, positive and negative modes) used to demonstrate the workflow) — https://github.com/rickhelmus/patRoonData

## Examples

```
fSets <- makeSet(fGroupsPos, fGroupsNeg, adducts = list(positive = "[M+H]+", negative = "[M-H]-")); fSets <- neutralizeSmallMolMasses(fSets); fGroupsSet <- alignFeatures(groupFeatures(fSets, algorithm = "openms"))
```

## Evaluation signals

- The exported feature table (feature_groups_pos_neg.csv) exists and contains feature-group columns from both polarities
- Rows in the exported feature table span combined neutral-mass feature groups from the positive and negative analyses
- Each polarity in the set is annotated with its correct default adduct: [M+H]+ for positive, [M-H]- for negative
- Feature m/z values in the set have been replaced with neutral masses calculated from the adduct information before alignment
- The result is a featureGroupsSet object, not two independent featureGroups objects

## Limitations

- Adduct information must be defined manually or via feature adduct annotations; misassigned adducts will produce incorrect neutral masses
- Mixed-mode combined processing was previously impractical in most workflows due to complexity and long processing times
- patRoonData example data are filtered to the first ten minutes of the chromatographic run and m/z 75-450, so demonstrated results are limited to that window

## Evidence

- [abstract] patRoon 2.0 is now capable of simultaneously processing, integrating and interpreting mixed mode data: "pat Roon 2.0 is now capable of simultaneously processing, integrating and interpreting mixed mode data - a functionality not available in most workflows"
- [abstract] sets workflow neutralization and alignment: "Then, the feature m/z values are replaced with neutral masses calculated from adduct information (defined manually or via feature adduct annotations), and subsequently aligned and grouped across"
- [readme] README sets workflow description: [evidence span withheld — anchor status: external_doc]
- [readme] patRoonData example data scope: [evidence span withheld — anchor status: external_doc]
