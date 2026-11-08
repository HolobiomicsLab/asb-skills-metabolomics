---
name: applicability-domain-filtering
description: Use when predicting bitterness (or any property) with a structure-based
  classifier like BitterPredict whose validity is restricted to a chemical-property
  envelope, and you must decide which compounds are 'covered' before scoring precision/recall.
when_to_use_negative:
- When the structure is unknown and you plan to predict bitterness directly from the
  mass spectrum with BitterMasS — no applicability-domain filter on A log P/MW applies
  to the spectrum-based model
- When the goal is maximum coverage over a 'dark' metabolome set and excluding out-of-domain
  compounds would drop most of the dataset
tools:
- name: BitterPredict
  role: Structure-based classifier whose applicability domain (A log P, MW thresholds)
    defines which compounds are eligible for prediction
  repo: https://github.com/Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:52d00790840439d4d1c60517e8817cb3a9313c619a4ddbe5d78e07bfa710fe63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# applicability-domain-filtering

## Summary

Filters compounds through a machine-learning classifier's applicability domain before prediction — here, BitterPredict's chemical-structure criteria (A log P between -3 and 7, MW ≤ 700) — so that only in-domain compounds receive predictions and out-of-domain compounds are counted as uncovered rather than force-predicted.

## When to use

Use when predicting bitterness (or any property) with a structure-based classifier like BitterPredict whose validity is restricted to a chemical-property envelope, and you must decide which compounds are 'covered' before scoring precision/recall.

## When NOT to use

- When the structure is unknown and you plan to predict bitterness directly from the mass spectrum with BitterMasS — no applicability-domain filter on A log P/MW applies to the spectrum-based model
- When the goal is maximum coverage over a 'dark' metabolome set and excluding out-of-domain compounds would drop most of the dataset

## Inputs

- Compounds with known chemical structures from an external test set (e.g., 106 literature-extracted EI-MS / ESI-MS/MS positive-mode spectra)
- Computed A log P values per compound
- Computed molecular weights per compound

## Outputs

- In-domain compound subset passing -3 ≤ A log P ≤ 7 and MW ≤ 700
- Coverage counts of compounds passing vs. excluded by the applicability domain
- Prediction-eligibility flags used to treat uncovered compounds as missed predictions in recall/F1 computation

## How to apply

Before running BitterPredict, compute the applicability-domain descriptors for each compound: A log P and molecular weight. Retain only compounds satisfying -3 ≤ A log P ≤ 7 and MW ≤ 700; treat excluded compounds as uncovered/missed predictions rather than assigning them a class label. This filtering is a distinct stage in the spectrum-structure-bitterness strategy (after annotating the structure from the spectrum and before the prediction itself), and it directly reduces prediction coverage — which is why the direct BitterMasS spectrum-bitterness strategy covered more compounds. In evaluation workflows, compute metrics such as recall, precision, and F1 only over in-domain compounds and report coverage counts alongside them.

## Related tools

- **BitterPredict** (Structure-based classifier whose applicability domain (A log P, MW thresholds) defines which compounds are eligible for prediction) — https://github.com/Niv-Lab/BitterPredict1

## Evaluation signals

- Every retained compound satisfies -3 ≤ A log P ≤ 7 and MW ≤ 700; any compound violating either bound is excluded
- Coverage counts (in-domain vs. out-of-domain) are reported alongside recall/precision/F1 rather than silently dropped
- Uncovered compounds in the indirect spectrum-structure-bitterness pipeline are explicitly counted as missed predictions
- Filtered metrics for BitterPredict on external set 1 align with reported values (precision ≈ 0.81, recall ≈ 0.81), and coverage is narrower than BitterMasS's on the same compounds

## Limitations

- The applicability domain narrows coverage: compounds failing the A log P/MW bounds cannot receive BitterPredict predictions at all
- In the spectrum-structure-bitterness strategy, filtering compounds out compounds compounds the prior loss from structural annotation, so coverage drops further
- BitterPredict's applicability domain was defined for its training space; applicability to compound classes outside bitter/nonbitter training chemistry is untested in this article

## Evidence

- [results] BitterPredict applicability domain thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Applicability domain as a pipeline stage: "passing the applicability domain of BitterPredict, and finally the prediction itself"
- [abstract] Coverage consequence of domain filtering: "The spectrum–bitterness prediction strategy was more effective than the spectrum–structure–bitterness prediction strategy and covered more compounds"
- [readme] BitterPredict usage from README: "BitterPredict.m gets as input CSV or EXCEL files with required descriptors of molecules, and calucautes a predictions if each molecule is bitter or not."
