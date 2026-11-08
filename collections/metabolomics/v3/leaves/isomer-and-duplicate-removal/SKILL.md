---
name: isomer-and-duplicate-removal
description: Use when TP components produced by patRoon 2.0 componentization (parent
  feature linked with corresponding TP features) must be prioritized, and suspected
  isomeric TPs or duplicate TPs arising from multiple pathways through the same parent
  must be filtered before reporting.
when_to_use_negative:
- When the input is raw LC-HRMS data or a feature table that has not yet passed through
  the patRoon TP screening and componentization steps — prioritization filters operate
  on TP components, not raw features
- When only a single filter dimension is needed and the components have already been
  prioritized (e.g., isomers already resolved chromatographically and confirmed)
tools:
- name: patRoon
  role: R package implementing the remove-isomers and duplicates prioritization filters
    within its TP component prioritization functionality
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Example LC-HRMS data package (standard-X and solvent-X) used to demonstrate
    the TP screening and componentization workflow that feeds the filters
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:0b5ca6859019e3724c03d0f49ff94ebc98d62731b9775ac75bd101d435d8df37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - R
  - patRoon
  - patRoon 2.0
  - patRoonData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# isomer-and-duplicate-removal

## Summary

patRoon 2.0 prioritization filters that refine TP components after componentization: the remove-isomers filter drops isomers that MS alone cannot distinguish, and the duplicates filter removes duplicate TPs formed from the same parent through different pathways.

## When to use

Use when TP components produced by patRoon 2.0 componentization (parent feature linked with corresponding TP features) must be prioritized, and suspected isomeric TPs or duplicate TPs arising from multiple pathways through the same parent must be filtered before reporting.

## When NOT to use

- When the input is raw LC-HRMS data or a feature table that has not yet passed through the patRoon TP screening and componentization steps — prioritization filters operate on TP components, not raw features
- When only a single filter dimension is needed and the components have already been prioritized (e.g., isomers already resolved chromatographically and confirmed)

## Inputs

- TP components from patRoon 2.0 componentization (parent features linked with corresponding TP features)
- patRoonData example LC-HRMS data (standard-X standard mixture and solvent-X blank, positive and negative ionization, m/z 75-450, first ten minutes)
- Feature formula annotations (used by related filters such as explained transformation)

## Outputs

- Prioritized TP component table with one row per TP component and per-filter verdicts (including remove-isomers and duplicates verdicts)
- Filtered set of TP components with isomeric and duplicate TPs removed

## How to apply

After the patRoon 2.0 TP screening workflow groups each parent feature with its corresponding TP features into a TP component, run the component prioritization routine with the remove-isomers filter enabled so that isomers — which the article notes 'can be difficult or impossible to distinguish with MS' — are dropped rather than reported as independent candidates. Enable the duplicates filter to remove TPs formed from the same parent through different pathways, collapsing redundant candidates onto one component. Apply these two filters alongside the complementary prioritization filters (RT direction based on chemical polarities, structure similarity for TPs with high MS but low structural similarity, and explained transformation checked against feature formula annotations) to produce a per-component verdict table. In the patRoonData example (standard-X vs solvent-X, positive and negative ionization), the prioritized components can be exported as a CSV with one row per TP component and a verdict column for each filter.

## Related tools

- **patRoon** (R package implementing the remove-isomers and duplicates prioritization filters within its TP component prioritization functionality) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Example LC-HRMS data package (standard-X and solvent-X) used to demonstrate the TP screening and componentization workflow that feeds the filters) — https://github.com/rickhelmus/patRoonData

## Evaluation signals

- Output file (e.g., tp_components_prioritized.csv) exists and contains a row per TP component
- Each row includes a verdict column for the remove-isomers filter and the duplicates filter, in addition to RT direction, structure similarity, and explained transformation verdicts
- No retained TP component contains isomers that MS cannot distinguish or duplicate TPs from the same parent formed through different pathways
- Filter behavior is consistent with the patRoon 2.0 Table 2 definitions of each prioritization filter
- Running the prioritization R script completes without error (script_runs signal)

## Limitations

- Isomers can be difficult or impossible to distinguish with MS, so the remove-isomers filter may discard or merge candidates whose separation would require chromatographic or additional evidence
- The duplicates filter relies on pathway information (same parent, different pathways); errors in TP prediction propagate into duplicate/unique decisions
- These filters are specific to the patRoon 2.0 TP component prioritization stage and do not generalize to other annotation objects

## Evidence

- [abstract] Remove-isomers filter removes isomers difficult or impossible to distinguish with MS: "Remove isomers Removes isomers, which can be difficult or impossible to distinguish with MS."
- [abstract] Duplicates filter removes TPs from same parent via different pathways: "Duplicates Removes duplicate TPs formed from the same parent through different pathways."
- [abstract] TP componentization groups parent features with corresponding TP features: "During TP componentization (Figure 1F), each parent feature is linked with corresponding TP features and grouped in a TP component."
- [abstract] patRoon adds new prioritization strategies: "Further improvements to pat Roon include interactive data curation and new prioritization and identification strategies, described further below."
- [intro] patRoon aims to provide comprehensive NTA workflows: "patRoon aims to provide comprehensive mass spectrometry based non-target analysis (NTA) workflows for environmental analysis."
