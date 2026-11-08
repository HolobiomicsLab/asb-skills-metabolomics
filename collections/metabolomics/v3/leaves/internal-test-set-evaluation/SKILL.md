---
name: internal-test-set-evaluation
description: Use when retraining or evaluating the BitterMasS Random Forest classifier
  on subsets of combined MassBank spectra and you need held-out precision/recall comparable
  to the paper's internal test set (precision = 0.83, recall = 0.90).
when_to_use_negative:
- Evaluating on the external validation sets (106 literature-extracted spectra or
  the 26 laboratory-measured compounds) — these require the separate external-set
  protocol, not the internal held-out split
- The goal is spectrum-to-structure annotation followed by structure-based bitterness
  prediction (BitterPredict route) rather than direct spectrum-bitterness classification
  with BitterMasS
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: BitterMasS
  role: Random Forest classifier retrained on progressively larger training subsets
    and scored on the held-out internal test set
  repo: https://github.com/Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:434c04a41005cb6cf59adb2db026e9fdb6374d8148213453e71d8ee36ae4f620
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# internal-test-set-evaluation

## Summary

Holds out a fixed internal test set from the combined EI-MS/ESI-MS/MS spectral training data so a retrained BitterMasS Random Forest classifier can be scored with precision and recall, supporting learning-curve analyses of training-set-size effects.

## When to use

Use when retraining or evaluating the BitterMasS Random Forest classifier on subsets of combined MassBank spectra and you need held-out precision/recall comparable to the paper's internal test set (precision = 0.83, recall = 0.90).

## When NOT to use

- Evaluating on the external validation sets (106 literature-extracted spectra or the 26 laboratory-measured compounds) — these require the separate external-set protocol, not the internal held-out split
- The goal is spectrum-to-structure annotation followed by structure-based bitterness prediction (BitterPredict route) rather than direct spectrum-bitterness classification with BitterMasS

## Inputs

- Combined bitter/nonbitter mass spectral training dataset (5414 experimental EI-MS and ESI-MS/MS positive-mode spectra from MassBank)
- Series of progressively larger training-subset sizes sampled from the combined spectral dataset
- Fixed internal test set held out from the combined spectral dataset

## Outputs

- Precision and recall values on the internal test set for each training-set size
- Learning-curve figure (learning_curve.png) plotting prediction performance metrics against training-set size

## How to apply

Load the combined training set of 5414 experimental EI-MS and ESI-MS/MS positive-mode spectra extracted from MassBank (bitter and nonbitter labels). Carve out a fixed internal test set before any subset sampling so that, for each training-set size in a learning-curve sweep, the same held-out spectra are used for scoring. For each size, retrain the Random Forest classifier in the BitterMasS configuration on the sampled subset and compute precision and recall on the internal test set. Plot the metrics against training-set size (as in the reported Figure S6) to confirm the pattern that performance improves dramatically with initial increases in training-set size and then only moderately. Judge success by reproducing the expected learning-curve shape and confirming baseline precision/recall near the reported 0.83/0.90 at full training-set size.

## Related tools

- **BitterMasS** (Random Forest classifier retrained on progressively larger training subsets and scored on the held-out internal test set) — https://github.com/Niv-Lab/BitterPredict1

## Evaluation signals

- Reproduced learning curve exhibits dramatic improvement with initial training-set increases followed by only moderate gains, consistent with Figure S6
- At full training-set size, internal-test-set precision ≈ 0.83 and recall ≈ 0.90 as reported in the abstract
- The same fixed internal test set is used across all training-subset sizes (no leakage between sampled training subsets and held-out spectra)
- Metrics are computed per subset size and the plot is saved as learning_curve.png

## Limitations

- Prediction on individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set, so internal-test-set scores are only meaningful for the combined training configuration
- Mass spectral data were available for only ~30% of the initial bitter/nonbitter training compounds, limiting training-data coverage
- No software/code availability or repository is provided for BitterMasS itself, and no model retraining details or hyperparameters are given in the paper, so reproduction depends on inferred configuration

## Evidence

- [abstract] internal test set performance: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [abstract] training dataset composition: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [discussion] combined vs individual sets: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] spectra source: "Experimental EI-MS and ESI-MS/MS positive ion mode mass spectra for these compounds were extracted from the MassBank database"
