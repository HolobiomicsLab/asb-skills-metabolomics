---
name: mass-spectral-bitterness-classification
description: Use when the input is an unlabeled EI-MS or ESI-MS/MS positive-mode mass
  spectrum (structure unknown or 'dark' metabolome) and the goal is a bitter/non-bitter
  class prediction directly from spectral features.
when_to_use_negative:
- When the chemical structure is already known and descriptors can be computed — BitterPredict
  predicts bitterness from structure and is the appropriate tool instead
- When spectra are in negative ion mode — the model makes predictions based on EI-MS
  or ESI-MS/MS spectra obtained in positive ion mode only
- When training only on the EI-MS-only or ESI-MS/MS-only subset — prediction on individual
  sets showed poor results compared with the combined set
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: BitterMasS
  role: Random Forest classifier applied to the spectrum features to output the bitter/non-bitter
    prediction directly
- name: BitterPredict
  role: Baseline for the indirect spectrum-structure-bitterness strategy; applied
    to structures assigned from spectra, counting only predictions passing its applicability
    domain
  repo: Niv-Lab/BitterPredict1
- name: MassBank
  role: Source of experimental EI-MS and ESI-MS/MS positive ion mode training spectra
    and reference library for structure annotation in the indirect strategy
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:23f5eb33fd0929ce6f368c57ad1772c046fd3ce018fce22b3008a0b01ac1471c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  - MassBank
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mass-spectral-bitterness-classification

## Summary

BitterMasS-style direct prediction of bitter vs non-bitter compounds from EI-MS or ESI-MS/MS (positive ion mode) mass spectra, using a Random Forest classifier trained on 5414 experimental MassBank spectra, avoiding the lossy spectrum-to-structure-to-bitterness detour.

## When to use

Use when the input is an unlabeled EI-MS or ESI-MS/MS positive-mode mass spectrum (structure unknown or 'dark' metabolome) and the goal is a bitter/non-bitter class prediction directly from spectral features.

## When NOT to use

- When the chemical structure is already known and descriptors can be computed — BitterPredict predicts bitterness from structure and is the appropriate tool instead
- When spectra are in negative ion mode — the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode only
- When training only on the EI-MS-only or ESI-MS/MS-only subset — prediction on individual sets showed poor results compared with the combined set

## Inputs

- EI-MS mass spectra in positive ion mode (per-compound peak lists/spectral vectors from MassBank or literature)
- ESI-MS/MS positive ion mode mass spectra
- Bitter/non-bitter training labels for spectra (derived from BitterDB/BitterPredict bitter and nonbitter compound sets)
- Optional: held-out internal test set and external validation spectra (106 literature-extracted spectra; 26 laboratory-measured compounds)

## Outputs

- Per-spectrum binary bitter/non-bitter prediction
- Precision, recall, and F1 metrics on internal test and external sets
- Optional learning-curve plot of precision/recall vs training-set size (learning_curve.png)
- Optional strategy-comparison table (direct spectrum-bitterness vs spectrum-structure-bitterness metrics, e.g. strategy_comparison_metrics.csv)

## How to apply

Load experimental EI-MS and ESI-MS/MS positive ion mode spectra (e.g. from MassBank), assemble the combined bitter/nonbitter training set (5414 spectra in the reference implementation), and train the Random Forest classifier in the BitterMasS configuration on spectral features. Hold out an internal test set to estimate precision/recall (reported: precision 0.83, recall 0.90). For each query spectrum, run the classifier directly rather than annotating a structure via a reference library (e.g. MassBank) and applying BitterPredict, since structure annotation covers only ~5-10% of spectra and the applicability-domain filter further narrows coverage. If retraining, plot precision and recall against training-set size to check that the learning curve improves dramatically at first and only moderately thereafter (cf. Figure S6). Treat predictions on individual EI-MS-only or ESI-MS/MS-only sets as unreliable; the combined set performed best.

## Related tools

- **BitterMasS** (Random Forest classifier applied to the spectrum features to output the bitter/non-bitter prediction directly)
- **BitterPredict** (Baseline for the indirect spectrum-structure-bitterness strategy; applied to structures assigned from spectra, counting only predictions passing its applicability domain) — Niv-Lab/BitterPredict1
- **MassBank** (Source of experimental EI-MS and ESI-MS/MS positive ion mode training spectra and reference library for structure annotation in the indirect strategy)

## Evaluation signals

- Internal test set precision and recall should approach the reported values (precision = 0.83, recall = 0.90)
- External validation on 106 literature-extracted spectra should show ~67% precision and ~93% recall; on 26 laboratory-measured compounds ~58% accuracy and ~99% recall
- Direct spectrum-bitterness strategy metrics should exceed the indirect strategy on the 130-spectrum external set (direct recall ~0.93, F1 ~0.77 vs indirect recall ~0.65, F1 ~0.65; precision ~0.65-0.66 for both)
- Retraining learning curve should show dramatic early improvement followed by only moderate gains (Figure S6 pattern)
- Combined EI-MS + ESI-MS/MS training set should outperform predictions trained on either spectral mode individually

## Limitations

- Only EI-MS and ESI-MS/MS positive ion mode spectra are supported in the current implementation; no version number or hosted model files are provided
- Only ~30% of the initial bitter and nonbitter training compounds had at least one mass spectrum, and nonbitter spectral representation is insufficient, limiting model quality
- Structure assignment from spectra (the most problematic stage) covers only ~5-10% of the 'dark' metabolome if the indirect route is taken; most publications reporting spectra do not include the actual spectra
- The external validation set from 18 publications with sensory studies is not deposited with a hosting location or accession, hindering exact reproduction

## Evidence

- [abstract] BitterMasS achieved precision = 0.83 and recall = 0.90 on an internal test set: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [abstract] Direct strategy outperforms indirect: "The spectrum–bitterness prediction strategy was more effective than the spectrum–structure–bitterness prediction strategy and covered more compounds"
- [abstract] Training data source and size: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [discussion] Combined set outperforms individual sets: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] Positive ion mode scope: "In the current implementation, the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
- [discussion] Spectra extracted from MassBank: "Experimental EI-MS and ESI-MS/MS positive ion mode mass spectra for these compounds were extracted from the MassBank database"
