---
name: adduct-mass-range-restriction
description: Use when preparing a CCS training set from the McLean Unified CCS Compendium
  for [M-H]-, [M+H]+ or [M+Na]+ adducts and entries must be restricted to the mass
  ranges supported by DeepCCS, AllCCS or CCSbase.
when_to_use_negative:
- When building training sets for adducts outside the compendium's [M-H]-, [M+H]+
  and [M+Na]+ coverage — CCSP 2.0 training sets do not support dimers such as [2M+H]+
  or [2M-H]-, other less common adducts, or in-source fragments
- When filtering annotation candidates by predicted CCS value — that is a different,
  later step (ΔCCS threshold of 2.8% at 10 ppm mass error), not training-set curation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pandas
  role: Python library used to load the compendium sheets, average duplicate CCS entries,
    and apply the m/z, element-frequency and mass-range exclusion filters
- name: CCSP 2.0
  role: Open-source Jupyter Notebook pipeline whose training-set curation includes
    this mass-range restriction step before linear support vector regression fitting
  repo: https://github.com/facundof2016/CCSP2.0
- name: DeepCCS
  role: Reference ML CCS prediction algorithm whose supported mass ranges define one
    of the exclusion criteria
- name: AllCCS
  role: Reference ML CCS prediction algorithm whose supported mass ranges define one
    of the exclusion criteria
- name: CCSbase
  role: Reference ML CCS prediction method whose supported mass ranges define one
    of the exclusion criteria
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:ef72c41205840b878aca1614456855450b9f084d1659be78e560e00a17aee603
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - AllCCS
  - CCSP 2.0
  - CCSbase
  - DeepCCS
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# adduct-mass-range-restriction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A data-curation step from the CCSP 2.0 pipeline that removes McLean Unified CCS Compendium entries whose m/z falls outside the mass ranges accepted by the ML algorithms DeepCCS, AllCCS and CCSbase, so the training set matches the domain of those models.

## When to use

Use when preparing a CCS training set from the McLean Unified CCS Compendium for [M-H]-, [M+H]+ or [M+Na]+ adducts and entries must be restricted to the mass ranges supported by DeepCCS, AllCCS or CCSbase.

## When NOT to use

- When building training sets for adducts outside the compendium's [M-H]-, [M+H]+ and [M+Na]+ coverage — CCSP 2.0 training sets do not support dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments
- When filtering annotation candidates by predicted CCS value — that is a different, later step (ΔCCS threshold of 2.8% at 10 ppm mass error), not training-set curation

## Inputs

- McLean Unified CCS Compendium spreadsheets for [M-H]-, [M+H]+ and [M+Na]+ adducts (InChI strings, ionic species, theoretical m/z, CCS values)
- Mass-range specifications of the DeepCCS, AllCCS and CCSbase ML algorithms

## Outputs

- Three cleaned CSV tables of CCS training entries (target: 567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions)

## How to apply

Download the complete [M-H]-, [M+H]+ and [M+Na]+ sheets from the McLean Unified CCS Compendium as separate spreadsheets and load them with pandas. First apply the companion scrubbing rules: average duplicate CCS entries, exclude entries whose m/z computed from the provided InChI and ionic species deviates from the listed theoretical m/z by more than 1 mDa, and exclude entries containing any chemical element appearing in fewer than five database entries. Then exclude entries not compatible with the mass ranges set in the ML algorithms used by DeepCCS, AllCCS or CCSbase, since models cannot reliably predict outside the mass domain they were trained on. Write the three cleaned CSVs and confirm the counts match the reported 567 singly deprotonated, 518 singly protonated and 461 singly sodiated ions.

## Related tools

- **Pandas** (Python library used to load the compendium sheets, average duplicate CCS entries, and apply the m/z, element-frequency and mass-range exclusion filters)
- **CCSP 2.0** (Open-source Jupyter Notebook pipeline whose training-set curation includes this mass-range restriction step before linear support vector regression fitting) — https://github.com/facundof2016/CCSP2.0
- **DeepCCS** (Reference ML CCS prediction algorithm whose supported mass ranges define one of the exclusion criteria)
- **AllCCS** (Reference ML CCS prediction algorithm whose supported mass ranges define one of the exclusion criteria)
- **CCSbase** (Reference ML CCS prediction method whose supported mass ranges define one of the exclusion criteria)

## Evaluation signals

- Cleaned table counts match the reported post-scrubbing values of 567 singly deprotonated, 518 singly protonated and 461 singly sodiated ions
- Every retained entry's m/z computed from its InChI and ionic species deviates at most 1 mDa from the listed theoretical m/z
- No retained entry contains a chemical element appearing in fewer than five database entries
- All retained m/z values fall within the mass ranges set in the ML algorithms used by DeepCCS, AllCCS or CCSbase
- Three cleaned CSVs are produced, one per adduct ([M-H]-, [M+H]+, [M+Na]+)

## Limitations

- Mass ranges are tied specifically to DeepCCS, AllCCS and CCSbase; entries outside these domains are dropped even if experimentally valid
- The resulting training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments
- Running CCSP 2.0 requires familiarity with Python; the current version requires Sklearn V1.0.2 or later
- The specific training set must be defined prior to CCS prediction

## Evidence

- [methods] Exclude entries not compatible with the mass ranges set in the ML algorithms used by DeepCCS, AllCCS or CCSbase.: "Exclude entries not compatible with the mass ranges set in the ML algorithms used by DeepCCS, AllCCS or CCSbase."
- [methods] Exclude entries whose m/z computed from the provided InChI and ionic species deviates from the listed theoretical m/z by more than 1 mDa: [evidence span withheld — anchor status: not_found]
- [intro] All drift tube collision cross section measurements utilized for model creation and evaluation were obtained from the McLean Unified CCS Compendium24.: "All drift tube collision cross section measurements utilized for model creation and evaluation were obtained from the McLean Unified CCS Compendium24."
- [abstract] CCSP produces predictions with accuracy equal to or greater than existing machine learning approaches such as CCSbase, DeepCCS and AllCCS: "CCSP produces predictions with accuracy equal to or greater than existing machine learning approaches such as CCSbase, DeepCCS and AllCCS"
- [discussion] not dimers such as [2M+H]+ or [2M-H]-, or other less common adducts or in-source fragments found in ESI-MS experiments: "not dimers such as [2M+H]+ or [2M-H]-, or other less common adducts or in-source fragments found in ESI-MS experiments"
