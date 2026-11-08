---
name: mass-spectra-bitterness-prediction
description: Use when you have positive-ion-mode EI-MS or ESI-MS/MS spectra (e.g.
  from MassBank or DrugBank) for a compound whose chemical structure is unknown, and
  need a bitter/non-bitter classification.
when_to_use_negative:
- The chemical structure of the compound is known and it passes the BitterPredict
  applicability domain (-3 <= A log P <= 7, MW <= 700) — BitterPredict achieves higher
  balanced accuracy and F1 in that setting
- Spectra were acquired in negative ion mode — the current BitterMasS implementation
  only accepts EI-MS or ESI-MS/MS positive ion mode spectra
- Predictions are made on individual EI-MS or individual ESI-MS/MS sets separately
  — the article reports poor results on individual sets compared with the combined
  set
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0038
- http://edamontology.org/topic_3474
tools:
- name: BitterMasS
  role: Random Forest classifier trained on 5414 experimental mass spectra; predicts
    bitterness directly from EI-MS/ESI-MS/MS spectra without requiring compound structure
- name: BitterPredict
  role: Structure-based ML classifier used as the benchmark when structures are known;
    applies an applicability-domain filter (-3 <= A log P <= 7, MW <= 700) before
    prediction
  repo: Niv-Lab/BitterPredict1
- name: MassBank
  role: Source database of experimental EI-MS and ESI-MS/MS positive ion mode spectra
    for the test compounds
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:653e42095542a372892c661197a3b81f15d9410d73567659288cfb937b0e9c9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  - MassBank
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0038
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectra-bitterness-prediction

## Summary

Predict bitter vs. non-bitter taste directly from a compound's experimental mass spectrum (EI-MS or ESI-MS/MS, positive ion mode) using BitterMasS, a Random Forest classifier trained on 5414 experimental spectra, bypassing the need for a known chemical structure.

## When to use

Use when you have positive-ion-mode EI-MS or ESI-MS/MS spectra (e.g. from MassBank or DrugBank) for a compound whose chemical structure is unknown, and need a bitter/non-bitter classification.

## When NOT to use

- The chemical structure of the compound is known and it passes the BitterPredict applicability domain (-3 <= A log P <= 7, MW <= 700) — BitterPredict achieves higher balanced accuracy and F1 in that setting
- Spectra were acquired in negative ion mode — the current BitterMasS implementation only accepts EI-MS or ESI-MS/MS positive ion mode spectra
- Predictions are made on individual EI-MS or individual ESI-MS/MS sets separately — the article reports poor results on individual sets compared with the combined set

## Inputs

- Experimental mass spectra in positive ion mode (EI-MS and/or ESI-MS/MS), e.g. retrieved from MassBank or DrugBank
- Multiple spectra per compound for majority-vote aggregation (e.g. 759 spectra for 106 compounds in external test set 1)
- Optional: sensory (taste-panel) bitter/non-bitter labels for validation

## Outputs

- Per-spectrum bitter/non-bitter predictions from the BitterMasS Random Forest classifier
- Per-compound bitter/non-bitter labels after majority voting across spectra
- Precision and recall (and balanced accuracy / F1 when benchmarked against BitterPredict) computed against sensory labels

## How to apply

Run the trained BitterMasS Random Forest classifier directly on each experimental mass spectrum (positive-ion-mode EI-MS or ESI-MS/MS) to obtain per-spectrum bitter/non-bitter predictions. When multiple spectra exist per compound (e.g. 759 MassBank/DrugBank spectra covering 106 literature-extracted compounds in external test set 1), aggregate per-spectrum predictions by majority voting to assign one label per compound. Compare the resulting per-compound labels against sensory (taste-panel) labels using precision and recall — reported external test set 1 targets are precision = 0.67 and recall = 0.93. When structures ARE known, benchmark against BitterPredict (applicability domain: -3 <= A log P <= 7, MW <= 700), which achieves higher balanced accuracy and F1 (precision = 0.81, recall = 0.81 on external set 1). Prefer direct spectrum-based prediction when structure is unavailable, because the spectrum-to-structure annotation step succeeds for only ~5-10% of spectra ('dark metabolome') and BitterPredict's applicability domain excludes some compounds.

## Related tools

- **BitterMasS** (Random Forest classifier trained on 5414 experimental mass spectra; predicts bitterness directly from EI-MS/ESI-MS/MS spectra without requiring compound structure)
- **BitterPredict** (Structure-based ML classifier used as the benchmark when structures are known; applies an applicability-domain filter (-3 <= A log P <= 7, MW <= 700) before prediction) — Niv-Lab/BitterPredict1
- **MassBank** (Source database of experimental EI-MS and ESI-MS/MS positive ion mode spectra for the test compounds)

## Examples

```
BitterPredict('descriptors.csv')
```

## Evaluation signals

- On external test set 1 (759 MassBank/DrugBank spectra, 106 literature-extracted compounds, majority-vote aggregation), precision = 0.67 and recall = 0.93 should be reproduced
- On the internal test set, expected performance is precision = 0.83 and recall = 0.90
- When benchmarked against BitterPredict on external set 1, BitterPredict should reach precision = 0.81 and recall = 0.81, with higher BA and F1 than BitterMasS
- Every spectrum submitted must be positive-ion-mode EI-MS or ESI-MS/MS; negative-mode spectra are out of scope
- Per-compound labels must come from majority voting over all spectra of that compound, not from single-spectrum calls

## Limitations

- Only ~30% of compounds in the initial bitter/non-bitter training sets had at least one identified mass spectrum, limiting training data
- Nonbitter compound spectral representation in the training set is insufficient; improvement is expected from expanding it
- Predictions on individual EI-MS or individual ESI-MS/MS sets showed poor results compared with the combined set
- Known structures represent only 5-10% of the metabolome ('dark' metabolome), which motivates but also limits validation of spectrum-based prediction
- No software version number or BitterMasS code hosting location is provided; BitterPredict1 README states 'full code will be available upon publication'

## Evidence

- [abstract] Random Forest trained on 5414 experimental spectra: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [abstract] Internal test set performance: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [abstract] External test set 1 performance: "For these external test cases, BitterMasS exhibited 67% precision and 93% recall for the first"
- [intro] Research gap motivating direct spectrum-based prediction: "However, to the best of our knowledge, there have been no attempts to predict the taste of a compound from its mass spectrum."
- [discussion] Recommendation for unknown structures: "the direct prediction of bitterness from the mass spectrum is recommended when the chemical structure of a compound is not yet unknown"
- [discussion] Applicable ion modes: "In the current implementation, the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
- [discussion] Individual spectrum sets underperform: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] Training data limitation: "only 30% of the compounds in the initial bitter and nonbitter sets in the training set had at least one mass spectrum identified"
- [readme] BitterPredict input usage from README: "BitterPredict.m gets as input CSV or EXCEL files with required descriptors of molecules, and calucautes a predictions if each molecule is bitter or not."
