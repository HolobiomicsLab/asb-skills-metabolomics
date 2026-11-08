---
name: feature-alignment-across-polarities
description: Use when LC-HRMS data were acquired in both positive and negative ionization
  modes and a single combined, aligned feature table spanning both polarities is needed
  (e.g., patRoon standard-X/solvent-X triplicates).
when_to_use_negative:
- Data were acquired in a single ionization mode only — a standard patRoon findFeatures/groupFeatures/alignFeatures
  workflow suffices without makeSet
- Input is already a combined neutral-mass feature table or featureGroupsSet from
  a prior sets run
- The goal is parent/TP componentization or suspect screening prioritization rather
  than cross-polarity feature grouping
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: patRoon
  role: 'Provides the sets workflow: makeSet to combine +/- featureGroups, neutral-mass
    replacement from adducts, and cross-polarity groupFeatures/alignFeatures (Native
    sets functionality)'
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Supplies the example positive and negative LC-HRMS triplicate analyses (standard-X,
    solvent-X) used to demonstrate the sets workflow
  repo: https://github.com/rickhelmus/patRoonData
- name: OpenMS
  role: Feature-detection and feature-grouping/alignment algorithm invoked via patRoon's
    findFeatures and groupFeatures/alignFeatures
  repo: http://openms.de/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:aac8731915765d3e7b842452842a332de048f0d9af595eaab3077ee74b79edac
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# feature-alignment-across-polarities

## Summary

A patRoon 2.0 'sets workflow' that separately processes positive and negative ionization LC-HRMS data, replaces feature m/z values with neutral masses derived from adducts ([M+H]+, [M-H]-), and aligns/groups features across polarities into one mixed-polarity featureGroupsSet.

## When to use

Use when LC-HRMS data were acquired in both positive and negative ionization modes and a single combined, aligned feature table spanning both polarities is needed (e.g., patRoon standard-X/solvent-X triplicates).

## When NOT to use

- Data were acquired in a single ionization mode only — a standard patRoon findFeatures/groupFeatures/alignFeatures workflow suffices without makeSet
- Input is already a combined neutral-mass feature table or featureGroupsSet from a prior sets run
- The goal is parent/TP componentization or suspect screening prioritization rather than cross-polarity feature grouping

## Inputs

- Positive-ionization LC-HRMS raw analyses (patRoonData standard-X/solvent-X triplicates, mzML)
- Negative-ionization LC-HRMS raw analyses (same samples, mzML)
- Per-polarity featureGroups objects produced by findFeatures
- Adduct annotations per polarity ([M+H]+, [M-H]-)

## Outputs

- A mixed-polarity featureGroupsSet object with neutral-mass feature groups
- Exported CSV feature table (e.g., feature_groups_pos_neg.csv) spanning both polarities

## How to apply

In R with patRoon 2.0, run findFeatures separately on the positive and negative analyses (e.g., with the OpenMS algorithm) on the patRoonData example data (standard-X and solvent-X triplicates). Combine the two featureGroups objects with makeSet, annotating each polarity with its default adduct — [M+H]+ for positive, [M-H]- for negative — either manually or via feature adduct annotations. Neutralize the feature m/z values in the set by replacing them with neutral masses calculated from the adduct information. Then apply groupFeatures/alignFeatures on the neutralized set to align and group features across polarities, yielding a mixed-polarity featureGroupsSet object. Finally export the aligned set to a CSV feature table (e.g., feature_groups_pos_neg.csv) for downstream inspection.

## Related tools

- **patRoon** (Provides the sets workflow: makeSet to combine +/- featureGroups, neutral-mass replacement from adducts, and cross-polarity groupFeatures/alignFeatures (Native sets functionality)) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Supplies the example positive and negative LC-HRMS triplicate analyses (standard-X, solvent-X) used to demonstrate the sets workflow) — https://github.com/rickhelmus/patRoonData
- **OpenMS** (Feature-detection and feature-grouping/alignment algorithm invoked via patRoon's findFeatures and groupFeatures/alignFeatures) — http://openms.de/

## Examples

```
fGroupsSet <- makeSet(fGroupsPos, fGroupsNeg, adducts = list("[M+H]+", "[M-H]-")); fGroupsSetNeutral <- neutralize(fGroupsSet); fGroupsSetAligned <- alignFeatures(groupFeatures(fGroupsSetNeutral, algorithm = "openms"))
```

## Evaluation signals

- The exported feature table (feature_groups_pos_neg.csv) exists on disk
- The CSV contains feature-group columns derived from both positive and negative polarities
- Rows span combined neutral-mass feature groups, i.e., features from both analyses appear in one aligned set
- Each polarity's features are annotated with the expected default adduct ([M+H]+ positive, [M-H]- negative) and the working values are neutral masses rather than raw m/z
- Data fall within the patRoonData scope: first ten minutes of the chromatographic run with m/z 75-450

## Limitations

- Cross-polarity combination was not available in most prior workflows due to complexity and long processing times, so processing costs remain a consideration
- Adducts must be defined manually or via feature adduct annotations; incorrect adduct assignment propagates into wrong neutral masses and misalignment
- The patRoonData example data are filtered to the first ten minutes with m/z 75-450, so demonstrated behavior may not generalize to full-range acquisitions

## Evidence

- [abstract] patRoon 2.0 is now capable of simultaneously processing, integrating and interpreting mixed mode data - a functionality not available in most workflows due to complexity and long processing times.: "pat Roon 2.0 is now capable of simultaneously processing, integrating and interpreting mixed mode data - a functionality not available in most workflows due to complexity and long processing times."
- [abstract] Feature m/z values are replaced with neutral masses calculated from adduct information before alignment across polarities.: "Then, the feature m/z values are replaced with neutral masses calculated from adduct information (defined manually or via feature adduct annotations), and subsequently aligned and grouped across"
- [readme] patRoon is vendor agnostic and includes novel functionality, such as combining positive and negative ionization data, estimating identification levels, transformation product screening.: [evidence span withheld — anchor status: not_found]
- [intro] Grouping and aligning features between analyses is an explicit step of the patRoon workflow.: "# Group and align features between analyses"
