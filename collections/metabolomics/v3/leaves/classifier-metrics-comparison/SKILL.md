---
name: classifier-metrics-comparison
description: Use when comparing a direct spectral-prediction classifier against a
  two-stage annotate-then-predict pipeline on the same labeled compounds, and per-strategy
  recall, precision, F1, and compound-coverage counts are needed.
when_to_use_negative:
- The chemical structure is already known — then BitterPredict can be run directly
  on the structure without spectrum-to-structure annotation
- Inputs are molecular descriptors/structures rather than mass spectra — the direct
  BitterMasS route requires EI-MS or ESI-MS/MS positive ion mode spectra
- A single classifier's internal test-set performance is the goal (e.g. BitterMasS
  internal precision = 0.83, recall = 0.90) rather than a strategy-level comparison
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: BitterMasS
  role: Random Forest classifier used for the direct spectrum→bitterness predictions
    on each selected EI-MS or ESI-MS/MS spectrum
- name: BitterPredict
  role: Structure-based classifier applied to MassBank-annotated structures in the
    indirect strategy; coverage counted only for predictions passing its applicability
    domain
  repo: Niv-Lab/BitterPredict1
- name: MassBank
  role: Reference library used to annotate each 'unknown' spectrum to a chemical structure
    in the indirect strategy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:665e3ef70d609bf764d4756a77e4696ab033764e45bc0564bfdfe328c35c89b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  - MassBank
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# classifier-metrics-comparison

## Summary

Benchmarks two classifier deployment strategies head-to-head on the same labeled external set: predicting bitterness directly from mass spectra with BitterMasS (Random Forest) versus the indirect spectrum→structure→bitterness route via MassBank annotation plus BitterPredict, using recall/precision/F1 and coverage to decide which route to adopt.

## When to use

Use when comparing a direct spectral-prediction classifier against a two-stage annotate-then-predict pipeline on the same labeled compounds, and per-strategy recall, precision, F1, and compound-coverage counts are needed.

## When NOT to use

- The chemical structure is already known — then BitterPredict can be run directly on the structure without spectrum-to-structure annotation
- Inputs are molecular descriptors/structures rather than mass spectra — the direct BitterMasS route requires EI-MS or ESI-MS/MS positive ion mode spectra
- A single classifier's internal test-set performance is the goal (e.g. BitterMasS internal precision = 0.83, recall = 0.90) rather than a strategy-level comparison

## Inputs

- External validation set of 130 'unknown' mass spectra (EI-MS and ESI-MS/MS positive ion mode) with ground-truth bitter/non-bitter labels
- BitterMasS Random Forest classifier trained on 5414 experimental mass spectra
- BitterPredict structure-based classifier (MATLAB BitterPredict.m accepting CSV or EXCEL descriptor files)
- MassBank reference spectral library for structure annotation

## Outputs

- Per-strategy recall, precision, and F1 metrics (direct vs indirect) for the 130-compound external set
- Compound-coverage counts per strategy (number of spectra annotated and passing BitterPredict's applicability domain)
- strategy_comparison_metrics.csv with metrics and coverage per strategy

## How to apply

Load the merged external set of 130 'unknown' mass spectra (EI-MS and ESI-MS/MS positive ion mode) with ground-truth bitter/non-bitter labels and select one spectrum per compound with a fixed random seed. For the direct strategy, run the BitterMasS Random Forest on each spectrum; for the indirect strategy, annotate each spectrum against the MassBank reference library to assign a structure, then run BitterPredict on the annotated structure, counting a compound as 'covered' only when the prediction passes BitterPredict's applicability domain. Compute recall, precision, and F1 separately per strategy, treating uncovered compounds in the indirect route as missed predictions because structural annotation (the most problematic stage) blocks them. Correct reproduction is judged against reported values: direct strategy recall ~0.93, F1 ~0.77; indirect recall ~0.65, F1 ~0.65; precision ~0.65–0.66 for both — with the direct strategy recommended when the chemical structure is unknown.

## Related tools

- **BitterMasS** (Random Forest classifier used for the direct spectrum→bitterness predictions on each selected EI-MS or ESI-MS/MS spectrum)
- **BitterPredict** (Structure-based classifier applied to MassBank-annotated structures in the indirect strategy; coverage counted only for predictions passing its applicability domain) — Niv-Lab/BitterPredict1
- **MassBank** (Reference library used to annotate each 'unknown' spectrum to a chemical structure in the indirect strategy)

## Examples

```
BitterPredict('external_set_130_descriptors.csv')
```

## Evaluation signals

- Direct-strategy metrics reproduce reported values: recall ~0.93, F1 ~0.77
- Indirect-strategy metrics reproduce reported values: recall ~0.65, F1 ~0.65, with precision ~0.65–0.66 for both strategies
- Coverage accounting: uncovered compounds in the indirect strategy (failed MassBank annotation or failed BitterPredict applicability domain) are counted as missed predictions, so indirect coverage is lower than direct
- Fixed random seed used so the one-spectrum-per-compound selection over the 130 external spectra is reproducible
- strategy_comparison_metrics.csv contains rows for both strategies with recall, precision, F1, and coverage counts

## Limitations

- Structural assignment from the spectrum is the most problematic stage and blocks predictions for many spectra; only a small fraction (5–10%) of spectra are assigned to known structures via reference libraries
- BitterPredict's applicability domain further narrows coverage in the indirect strategy
- Predictions are limited to EI-MS or ESI-MS/MS spectra obtained in positive ion mode; individual EI-MS and ESI-MS/MS sets showed poorer results than the combined set
- No software repository, version, or hosting location is given for BitterMasS, and BitterPredict code was stated as available only 'upon publication'

## Evidence

- [abstract] The spectrum–bitterness prediction strategy was more effective than the spectrum–structure–bitterness prediction strategy and covered more compounds: "The spectrum–bitterness prediction strategy was more effective than the spectrum–structure–bitterness prediction strategy and covered more compounds"
- [abstract] achieving precision = 0.83 and recall = 0.90 for an internal test set: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [discussion] annotating the structure from the spectrum (which is the most problematic stage): "annotating the structure from the spectrum (which is the most problematic stage)"
- [discussion] passing the applicability domain of BitterPredict, and finally the prediction itself: "passing the applicability domain of BitterPredict, and finally the prediction itself"
- [readme] BitterPredict.m gets as input CSV or EXCEL files with required descriptors of molecules, and calucautes a predictions if each molecule is bitter or not.: "BitterPredict.m gets as input CSV or EXCEL files with required descriptors of molecules, and calucautes a predictions if each molecule is bitter or not."
