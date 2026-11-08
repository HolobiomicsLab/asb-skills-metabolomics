---
name: feature-grouping-across-samples
description: Use when LC-HRMS features must be grouped across replicate sample analyses
  (e.g. patRoonData 'standard-X'/'solvent-X' triplicate injections) or when feature-group
  outputs from OpenMS, XCMS and enviPick must be compared and combined on the same
  dataset.
when_to_use_negative:
- Input is already a grouped feature table — grouping across samples has already been
  performed
- Data are not replicate analyses of related samples (e.g. single-injection runs with
  no cross-sample equivalence to establish)
- Goal is annotation (formula/compound assignment) rather than feature-level comparison
  across algorithms
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: patRoon
  role: Harmonized R interface that runs feature finding and groups features across
    sample analyses for multiple algorithms under a consistent interface
  repo: https://github.com/rickhelmus/patRoon
- name: OpenMS
  role: One of the three feature finding algorithms whose per-algorithm feature groups
    are compared
- name: XCMS
  role: One of the three feature finding algorithms whose per-algorithm feature groups
    are compared
- name: enviPick
  role: One of the three feature finding algorithms whose per-algorithm feature groups
    are compared
- name: patRoonData
  role: Example LC-HRMS data package (standard-X/solvent-X triplicates) used as input
    for the grouping workflow
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:3f7f6c7322d51ac7fe3ce8d7481cd1705d428e878e38dd7acc008b201be2f82c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# feature-grouping-across-samples

## Summary

patRoon's harmonized feature grouping step: features with unique chromatographic and mass spectral properties are extracted per algorithm (OpenMS, XCMS, enviPick) and features considered equivalent across sample analyses (e.g. standard-X/solvent-X triplicates) are grouped, enabling algorithm comparison and consensus building.

## When to use

Use when LC-HRMS features must be grouped across replicate sample analyses (e.g. patRoonData 'standard-X'/'solvent-X' triplicate injections) or when feature-group outputs from OpenMS, XCMS and enviPick must be compared and combined on the same dataset.

## When NOT to use

- Input is already a grouped feature table — grouping across samples has already been performed
- Data are not replicate analyses of related samples (e.g. single-injection runs with no cross-sample equivalence to establish)
- Goal is annotation (formula/compound assignment) rather than feature-level comparison across algorithms

## Inputs

- LC-HRMS raw data converted to mzML (patRoonData 'standard-X' standard mixture and 'solvent-X' blank, triplicate injections each)
- analysisInfo sample/blank metadata table for patRoon
- choice of feature finding algorithm: OpenMS, XCMS or enviPick

## Outputs

- featureGroups objects per algorithm (OpenMS, XCMS, enviPick) with features grouped across sample analyses
- per-algorithm feature-group counts
- pairwise overlap statistics between algorithm results and a consensus feature-group table
- feature_group_comparison.csv comparison table

## How to apply

Load the patRoonData example dataset (triplicate LC-HRMS of a standard mixture 'standard-X' and blank solvent 'solvent-X') and build an analysisInfo table. Run feature finding independently with OpenMS, XCMS and enviPick through patRoon's harmonized interface (findFeatures), then call groupFeatures for each result so that features considered equivalent across the triplicate analyses are grouped. Count feature groups per algorithm and compute pairwise overlaps between the three algorithm outputs; in the patRoon benchmark this yielded a consensus of 1243 feature groups (541 unique to one algorithm, 332 shared by two, 370 fully overlapped). Optionally precede grouping with automatic parameter optimization (IPO), and verify suspect detectability is consistent across algorithms (17 of 18 suspects detected in each dataset; aniline required lowering the intensity threshold of the filter() function). Assemble per-algorithm counts and pairwise overlap statistics into feature_group_comparison.csv.

## Related tools

- **patRoon** (Harmonized R interface that runs feature finding and groups features across sample analyses for multiple algorithms under a consistent interface) — https://github.com/rickhelmus/patRoon
- **OpenMS** (One of the three feature finding algorithms whose per-algorithm feature groups are compared)
- **XCMS** (One of the three feature finding algorithms whose per-algorithm feature groups are compared)
- **enviPick** (One of the three feature finding algorithms whose per-algorithm feature groups are compared)
- **patRoonData** (Example LC-HRMS data package (standard-X/solvent-X triplicates) used as input for the grouping workflow) — https://github.com/rickhelmus/patRoonData

## Examples

```
library(patRoon); fData <- findFeatures(analysisInfo, "openms"); fGroups <- groupFeatures(fData, "openms")
```

## Evaluation signals

- feature_group_comparison.csv contains one row per algorithm (OpenMS, XCMS, enviPick) with a numeric feature-group count
- Pairwise overlap statistics against the other two algorithms are numeric and complete for every algorithm row
- All three algorithms produce valid feature-group sets (paper: 1243 consensus groups; 541 unique to one algorithm, 332 shared by two, 370 fully overlapped)
- Suspect-screening consistency: the same 17 of 18 suspects are detected in each algorithm's grouped dataset (aniline only with lowered intensity threshold)
- Grouping was computed from triplicate analyses (standard-X and solvent-X), not single-injection data

## Limitations

- patRoonData files are pre-filtered to the first ten minutes of the chromatographic run with m/z 75–450, so group counts reflect reduced data
- The missing suspect (aniline) was only detected when lowering the intensity threshold of the filter() function
- Dependency/algorithm versions are unspecified and deferred to external installation instructions; some integrated algorithms are closed-source, limiting full open reproducibility
- No patRoon software version is stated in the article; demo/benchmark scripts are only available as supplementary archive files

## Evidence

- [intro] features considered equivalent across sample analyses are grouped: [evidence span withheld — anchor status: not_found]
- [abstract] three algorithms demonstrated: "the ability to easily use, combine and evaluate different algorithms was demonstrated for three commonly used feature finding algorithms"
- [readme] README functionality table: [evidence span withheld — anchor status: external_doc]
- [readme] patRoonData input data: [evidence span withheld — anchor status: external_doc]
