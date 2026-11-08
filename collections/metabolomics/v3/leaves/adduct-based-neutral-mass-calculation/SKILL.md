---
name: adduct-based-neutral-mass-calculation
description: Use when a sets workflow combines separately found positive ([M+H]+)
  and negative ([M-H]-) featureGroups objects with makeSet and the mixed-polarity
  features must be neutralized before cross-polarity grouping.
when_to_use_negative:
- Data contains only a single ionization mode — a plain patRoon workflow (no makeSet/neutralization
  step) is appropriate.
- Feature m/z values are already converted to neutral masses or the features were
  aligned within a single polarity.
- The task is adduct annotation of individual features within one polarity (componentization),
  not cross-polarity combination.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: patRoon 2.0
  role: Provides the sets workflow (makeSet, groupFeatures/alignFeatures) that calculates
    neutral masses from adduct information to combine +/- ionization data
  repo: https://rickhelmus.github.io/patRoon
- name: OpenMS
  role: Feature-detection and grouping/alignment algorithm used on the positive and
    negative analyses before and after neutralization
- name: patRoonData
  role: Example LC-HRMS data (standard-X and solvent-X triplicates, positive and negative
    ionization) used to demonstrate the sets workflow
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:31a2bc1f03cdb583a46e4451387fe42beaa12973e2c6ee497d5ab6e691cece5c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OpenMS
  - R
  - patRoon 2.0
  - patRoonData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Adduct-based neutral mass calculation

## Summary

Replaces feature m/z values with neutral masses calculated from ionization-adduct information so that positive- and negative-ionization HRMS features can be aligned and grouped across polarities in patRoon's sets workflow.

## When to use

Use when a sets workflow combines separately found positive ([M+H]+) and negative ([M-H]-) featureGroups objects with makeSet and the mixed-polarity features must be neutralized before cross-polarity grouping.

## When NOT to use

- Data contains only a single ionization mode — a plain patRoon workflow (no makeSet/neutralization step) is appropriate.
- Feature m/z values are already converted to neutral masses or the features were aligned within a single polarity.
- The task is adduct annotation of individual features within one polarity (componentization), not cross-polarity combination.

## Inputs

- featureGroups object for positive-ionization analyses (e.g. OpenMS features from patRoonData standard-X/solvent-X triplicates)
- featureGroups object for negative-ionization analyses
- adduct definitions per polarity (manually specified or derived from feature adduct annotations)

## Outputs

- featureGroupsSet object with m/z replaced by neutral masses and features aligned/grouped across polarities
- optionally an exported mixed-polarity feature table (e.g. feature_groups_pos_neg.csv)

## How to apply

First run findFeatures separately on the positive and negative analyses (e.g. with the OpenMS algorithm in patRoon) and combine the two featureGroups objects with makeSet, annotating each polarity with its adduct — default de-protonated/protonated adducts [M+H]+ and [M-H]-. Then replace the feature m/z values in the resulting set with neutral masses calculated from the adduct information, which may be defined manually or via feature adduct annotations. Neutralization is essential because a single analyte appears at different m/z in each polarity but at the same neutral mass, so after conversion groupFeatures/alignFeatures can merge the mixed-polarity features into a featureGroupsSet spanning both modes. Rationale: patRoon 2.0 introduced sets workflows to simultaneously process, integrate and interpret mixed-mode HRMS data, something not available in most workflows due to complexity and long processing times.

## Related tools

- **patRoon 2.0** (Provides the sets workflow (makeSet, groupFeatures/alignFeatures) that calculates neutral masses from adduct information to combine +/- ionization data) — https://rickhelmus.github.io/patRoon
- **OpenMS** (Feature-detection and grouping/alignment algorithm used on the positive and negative analyses before and after neutralization)
- **patRoonData** (Example LC-HRMS data (standard-X and solvent-X triplicates, positive and negative ionization) used to demonstrate the sets workflow) — https://github.com/rickhelmus/patRoonData

## Examples

```
fSets <- makeSet(fGroupsPos, fGroupsNeg, adducts = c("[M+H]+", "[M-H]-")); fSetsNeutral <- neutralize(fSets)
```

## Evaluation signals

- The set object carries the expected adducts ([M+H]+ for positive, [M-H]- for negative) after makeSet.
- Feature values in the featureGroupsSet are neutral masses (no longer per-polarity m/z values).
- The aligned result is a mixed-polarity featureGroupsSet containing feature groups derived from both positive and negative analyses.
- The exported feature table (e.g. feature_groups_pos_neg.csv) exists and its columns/rows span the combined neutral-mass feature groups from both polarities.

## Limitations

- Neutral masses must come from correct adduct information — defined manually or via feature adduct annotations — and wrong adduct assignments will mis-align features across polarities.
- patRoonData example data is filtered to the first ten minutes of the chromatographic run with m/z 75-450, so results on the demo set are restricted to that range.

## Evidence

- [abstract] abstract: [evidence span withheld — anchor status: not_found]
- [readme] readme: [evidence span withheld — anchor status: not_found]
