---
name: lc-ms-feature-detection
description: Use when you have triplicate LC-HRMS data (e.g. standard-X/solvent-X)
  acquired in both positive and negative ionization modes and need a single aligned
  feature table spanning both polarities.
when_to_use_negative:
- Data is single-polarity only — a plain patRoon findFeatures/groupFeatures workflow
  suffices, and makeSet neutralization would be unnecessary
- Input is already a grouped/aligned feature table — feature detection and set combination
  would duplicate prior processing
- Data is GC-MS or LC-IMS-HRMS where the LC-HRMS sets feature-detection assumptions
  (adduct-based neutral masses) do not apply
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: patRoon
  role: R platform providing findFeatures, makeSet, groupFeatures/alignFeatures and
    CSV export for the sets workflow
  repo: https://github.com/rickhelmus/patRoon
- name: OpenMS
  role: Feature detection and grouping/alignment algorithm invoked through patRoon
    for both polarities
  repo: http://openms.de/
- name: patRoonData
  role: Example LC-HRMS data package (standard-X and solvent-X triplicates, positive
    and negative ionization) used as workflow input
  repo: https://github.com/rickhelmus/patRoonData
- name: R
  role: Language in which the whole feature-detection and set-combination workflow
    is scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:6e59192b1d66d43c848690fd5504dea403c908799899a41a0ffd55247e3b771b
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# lc-ms-feature-detection

## Summary

Feature detection and mixed-mode alignment for LC-HRMS non-target analysis using patRoon 2.0's sets workflow: find features per ionization polarity, combine with makeSet, neutralize m/z to masses, and align/group across polarities with OpenMS.

## When to use

Use when you have triplicate LC-HRMS data (e.g. standard-X/solvent-X) acquired in both positive and negative ionization modes and need a single aligned feature table spanning both polarities.

## When NOT to use

- Data is single-polarity only — a plain patRoon findFeatures/groupFeatures workflow suffices, and makeSet neutralization would be unnecessary
- Input is already a grouped/aligned feature table — feature detection and set combination would duplicate prior processing
- Data is GC-MS or LC-IMS-HRMS where the LC-HRMS sets feature-detection assumptions (adduct-based neutral masses) do not apply

## Inputs

- Raw LC-HRMS data (vendor or mzML/mzXML) for a standard mixture and blank solvent, in triplicate, in both positive and negative ionization modes (patRoonData standard-X / solvent-X)
- Adduct specification per polarity ([M+H]+ positive, [M-H]- negative), either manual or from feature adduct annotations
- Blank analysis information for blank subtraction (solvent-X triplicates)

## Outputs

- Per-polarity featureGroups objects from findFeatures (OpenMS)
- A mixed-polarity featureGroupsSet object with m/z values replaced by neutral masses
- An exported CSV feature table (feature_groups_pos_neg.csv) spanning combined neutral-mass feature groups from both polarities

## How to apply

Load the patRoonData example LC-HRMS data (standard-X standard-mixture and solvent-X blank, each in triplicate) for both ionization modes in R. Run findFeatures separately on the positive and negative analyses using the OpenMS algorithm, which extracts features (data points with unique chromatographic/MS information) per polarity. Combine the two featureGroups objects with makeSet, annotating each polarity with its adduct ([M+H]+ for positive, [M-H]- for negative), so patRoon can replace feature m/z values with neutral masses calculated from the adduct information. Then run groupFeatures/alignFeatures with the OpenMS algorithm on the neutralized set to yield a mixed-polarity featureGroupsSet object. Optionally apply blank subtraction using the solvent-X triplicates recorded in the analysis information's blank column before exporting. Export the aligned mixed-polarity feature groups to a CSV feature table (e.g. feature_groups_pos_neg.csv) for downstream formula/compound annotation.

## Related tools

- **patRoon** (R platform providing findFeatures, makeSet, groupFeatures/alignFeatures and CSV export for the sets workflow) — https://github.com/rickhelmus/patRoon
- **OpenMS** (Feature detection and grouping/alignment algorithm invoked through patRoon for both polarities) — http://openms.de/
- **patRoonData** (Example LC-HRMS data package (standard-X and solvent-X triplicates, positive and negative ionization) used as workflow input) — https://github.com/rickhelmus/patRoonData
- **R** (Language in which the whole feature-detection and set-combination workflow is scripted)

## Examples

```
fGroupsPos <- findFeatures(analysesPos, "openms"); fGroupsSet <- makeSet(fGroupsPos, fGroupsNeg, adducts = list("[M+H]+", "[M-H]-")); fGroupsSetAl <- alignFeatures(groupFeatures(fGroupsSet, "openms"))
```

## Evaluation signals

- The exported feature_groups_pos_neg.csv exists and parses as a feature table with feature-group columns from both polarities
- Rows of the exported table span the combined neutral-mass feature groups derived from both positive and negative analyses (not two disjoint polarity-specific tables)
- m/z values were replaced by neutral masses calculated from the assigned adducts before grouping/alignment (set objects carry adduct annotations)
- Feature detection was run independently per polarity with the OpenMS algorithm before makeSet combination, and alignment succeeded on the neutralized set
- Blank subtraction, when applied, used the solvent-X triplicates recorded in the analysis information's blank column

## Limitations

- Mixed-polarity processing was 'a functionality not available in most workflows due to complexity and long processing times' — expect comparatively long processing times on large datasets
- patRoonData example data is filtered to the first ten minutes of the chromatographic run with m/z 75-450, so demonstrated retention-time and mass ranges are limited
- Correct neutral masses depend on accurate adduct annotation per polarity; wrong or missing adducts (manual or feature adduct annotations) would corrupt cross-polarity alignment

## Evidence

- [abstract] patRoon 2.0 is now capable of simultaneously processing, integrating and interpreting mixed mode data: "pat Roon 2.0 is now capable of simultaneously processing, integrating and interpreting mixed mode data - a functionality not available in most workflows due to complexity and long processing times."
- [abstract] Neutral mass replacement in sets workflow: "Then, the feature m/z values are replaced with neutral masses calculated from adduct information (defined manually or via feature adduct annotations), and subsequently aligned and grouped across"
- [intro] Example dataset description: "The example dataset contains LC-MS data for a standard mixture with known composition ('standard-X') and a blank solvent ('solvent-X'), both in triplicate"
- [intro] Feature extraction as first workflow step: "The first step of a LC-MS non-target analysis workflow is typically the extraction of so called 'features'."
- [readme] Combining positive and negative ionization data: [evidence span withheld — anchor status: external_doc]
