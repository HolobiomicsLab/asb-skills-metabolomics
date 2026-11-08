---
name: multi-algorithm-comparison-nta
description: Use when the same LC-HRMS dataset should be processed by several feature-finding
  algorithms (OpenMS, XCMS, enviPick) to compare, combine, or benchmark their feature-group
  outputs.
when_to_use_negative:
- The input is already a processed feature table or consensus feature-group set —
  rerunning feature finding would be redundant.
- The task is downstream annotation (formula/compound assignment, componentization)
  rather than evaluating or combining feature-finding algorithms.
- Only a single feature-finding algorithm is required and no cross-algorithm comparison
  is intended.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2258
tools:
- name: patRoon
  role: R platform providing the harmonized feature extraction and grouping interface
    used to run all three algorithms and compute overlaps
  repo: https://github.com/rickhelmus/patRoon
- name: OpenMS
  role: One of the three feature finding algorithms invoked through patRoon's feature
    extraction step
  repo: http://openms.de/
- name: XCMS
  role: One of the three feature finding algorithms invoked through patRoon's feature
    extraction step
- name: enviPick
  role: One of the three feature finding algorithms invoked through patRoon's feature
    extraction step
- name: patRoonData
  role: Supplies the example LC-HRMS triplicate standard and blank solvent data used
    as comparison input
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:2d42f20bc5fc808ef513837fd5ab5426f7e7860ce880a467915d11a2313bef2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - OpenMS
  - R
  - XCMS
  - enviPick
  - patRoon
  - patRoonData
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multi-algorithm-comparison-nta

## Summary

Runs feature finding independently with multiple algorithms (OpenMS, XCMS, enviPick) through patRoon's harmonized R interface on the same LC-HRMS dataset, groups features per algorithm, and quantifies cross-algorithm overlap to evaluate agreement.

## When to use

Use when the same LC-HRMS dataset should be processed by several feature-finding algorithms (OpenMS, XCMS, enviPick) to compare, combine, or benchmark their feature-group outputs.

## When NOT to use

- The input is already a processed feature table or consensus feature-group set — rerunning feature finding would be redundant.
- The task is downstream annotation (formula/compound assignment, componentization) rather than evaluating or combining feature-finding algorithms.
- Only a single feature-finding algorithm is required and no cross-algorithm comparison is intended.

## Inputs

- patRoonData LC-HRMS raw data: 'standard-X' standard mixture triplicate and 'solvent-X' blank solvent triplicate (mzML, filtered to first 10 min, m/z 75-450)
- patRoon R package with OpenMS, XCMS and enviPick backends installed

## Outputs

- Per-algorithm feature-group sets (OpenMS, XCMS, enviPick) from groupFeatures
- feature_group_comparison.csv: one row per algorithm with numeric feature-group count and pairwise overlap statistics against the other two algorithms
- Consensus feature-group classification (unique-to-one / shared-by-two / fully-overlapped counts)

## How to apply

Load the patRoonData example LC-HRMS data ('standard-X' standard mixture triplicate and 'solvent-X' blank solvent triplicate) into R, then call patRoon's feature extraction interface once per algorithm — OpenMS, XCMS and enviPick — so each receives identical pre-treated mzML input. Group features across the replicate sample analyses for each algorithm result so outputs are directly comparable at the feature-group level. Count feature groups per algorithm and compute pairwise overlap statistics between the three result sets, classifying groups as unique to one algorithm, shared by two, or fully overlapped by all three. Assemble per-algorithm counts and pairwise overlaps into a single table (feature_group_comparison.csv) so algorithm agreement and complementarity can be judged in one view. The rationale is patRoon's core claim: harmonizing tools under one interface makes the use, evaluation and mixing of well-tested algorithms seamless without tedious data conversions.

## Related tools

- **patRoon** (R platform providing the harmonized feature extraction and grouping interface used to run all three algorithms and compute overlaps) — https://github.com/rickhelmus/patRoon
- **OpenMS** (One of the three feature finding algorithms invoked through patRoon's feature extraction step) — http://openms.de/
- **XCMS** (One of the three feature finding algorithms invoked through patRoon's feature extraction step)
- **enviPick** (One of the three feature finding algorithms invoked through patRoon's feature extraction step)
- **patRoonData** (Supplies the example LC-HRMS triplicate standard and blank solvent data used as comparison input) — https://github.com/rickhelmus/patRoonData

## Examples

```
fGroups <- list(OpenMS=findFeatures("openms", analyses), XCMS=findFeatures("xcms", analyses), enviPick=findFeatures("envipick", analyses))
```

## Evaluation signals

- feature_group_comparison.csv contains one row per algorithm (OpenMS, XCMS, enviPick) with a numeric feature-group count and numeric overlap statistics against the other two algorithms.
- Each algorithm produces a valid, non-empty feature-group set after groupFeatures on the same sample analyses.
- Overlap categories (unique to one algorithm, shared by two, fully overlapped) partition the consensus; the reference run gave 1243 consensus groups with 541 (44%) unique, 332 (27%) shared by two, and 370 (30%) fully overlapped.
- Suspect screening sanity check: the same suspects (17 of 18) are detected in each algorithm's feature data; a miss (e.g. aniline) should trace to the intensity threshold of the filter() function, not the comparison logic.
- All three algorithms ran on identical pre-treated input data (same mzML files), so differences reflect the algorithms, not the data.

## Limitations

- The missing suspect (aniline) could only be detected when lowering the intensity threshold of the filter() function, so filter settings can mask cross-algorithm agreement.
- Some integrated workflow algorithms are closed-source, limiting full open reproducibility of the comparison.
- Dependency/algorithm versions and requirements are unspecified and deferred to external installation instructions.
- Demonstration scripts (e.g. Additional file 4) are only available as supplementary archive files, not in the main repository.

## Evidence

- [abstract] the ability to easily use, combine and evaluate different algorithms was demonstrated for three commonly used feature finding algorithms: "the ability to easily use, combine and evaluate different algorithms was demonstrated for three commonly used feature finding algorithms"
- [intro] Example dataset standard-X / solvent-X: "The example dataset contains LC-MS data for a standard mixture with known composition ('standard-X') and a blank solvent ('solvent-X')"
- [readme] Feature extraction algorithms in README: [evidence span withheld — anchor status: external_doc]
- [readme] patRoonData package contents: [evidence span withheld — anchor status: external_doc]
- [readme] Harmonized interface rationale: [evidence span withheld — anchor status: external_doc]
