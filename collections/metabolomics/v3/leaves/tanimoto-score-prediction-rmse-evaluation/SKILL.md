---
name: tanimoto-score-prediction-rmse-evaluation
description: Use when a trained MS2DeepScore (or similar Siamese spectral similarity)
  model must be validated for Tanimoto prediction accuracy on held-out spectra from
  compounds absent from training, and you need an RMSE plus per-similarity-bin error
  profile.
when_to_use_negative:
- The goal is exact compound identification — MS2DeepScore cannot reliably discriminate
  high Tanimoto scores (0.8–0.9) from a near-complete chemical match
- The query spectra are strongly outside the training library's biased compound classes
  or instrumentation types, where prediction error is unquantified
- You already have molecular fingerprints/structures and only need a classical chemical
  similarity — compute Tanimoto directly with RDKit instead of predicting it from
  spectra
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: MS2DeepScore
  role: Siamese neural network whose predicted Tanimoto scores are evaluated; used
    via the ms2deepscore Python library (v0.2.0 for the reported results)
  repo: https://github.com/matchms/ms2deepscore
- name: RDKit
  role: Computes Daylight 2048-bit fingerprints and the ground-truth Tanimoto similarity
    labels for spectrum pairs
- name: matchms
  role: Python library for mass spectra import, handling and comparisons; integrates
    MS2DeepScore
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:afd192ffabeb13f599d4c111c70cf14ff27c4213f06999f4b1fd0d559db4f6f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tanimoto-score-prediction-rmse-evaluation

## Summary

Benchmark a trained MS2DeepScore Siamese model by computing RMSE between predicted and true Tanimoto scores on a molecule-disjoint held-out test set (3601 GNPS-derived spectra of 500 unseen compounds), expecting overall RMSE ~0.15 (range 0.13–0.2) with per-bin error breakdown.

## When to use

Use when a trained MS2DeepScore (or similar Siamese spectral similarity) model must be validated for Tanimoto prediction accuracy on held-out spectra from compounds absent from training, and you need an RMSE plus per-similarity-bin error profile.

## When NOT to use

- The goal is exact compound identification — MS2DeepScore cannot reliably discriminate high Tanimoto scores (0.8–0.9) from a near-complete chemical match
- The query spectra are strongly outside the training library's biased compound classes or instrumentation types, where prediction error is unquantified
- You already have molecular fingerprints/structures and only need a classical chemical similarity — compute Tanimoto directly with RDKit instead of predicting it from spectra

## Inputs

- Trained MS2DeepScore model weights (e.g. ms2deepscore_model.hdf5)
- Held-out test set: 3601 preprocessed MS/MS spectra of 500 unique compounds (GNPS-derived, Zenodo record 4699300)
- Ground-truth Tanimoto similarity labels from RDKit Daylight 2048-bit fingerprints
- Molecule-level InChIKey split definition (training/validation/test = 14,062/500/500)

## Outputs

- Overall RMSE between predicted and true Tanimoto scores on the test set (expected ~0.15, range 0.13–0.2)
- RMSE per Tanimoto-score bin (stored in training_rmse.json)
- Predicted-vs-true Tanimoto scatter/bin diagnostic for the 3601-spectrum test set

## How to apply

First split the 15,062 unique InChIKeys by molecule into training (14,062), validation (500), and test (500) sets so no compound leaks between sets; the test set comprises 3601 spectra. Compute ground-truth labels for all spectrum pairs with RDKit Daylight 2048-bit fingerprints (Tanimoto similarity). Run the trained model over the test spectra to predict Tanimoto scores, then compute overall RMSE between predicted and true scores, and additionally RMSE per Tanimoto-score bin to expose accuracy loss at extremes (best between 0.1 and 0.9, worst for highest/lowest bins). Optionally aggregate predictions by taking the median over spectral pairs of the same compound pair, which improves reliability over individual predictions. Expect overall RMSE between 0.13 and 0.2 (~0.15) as the reproduction target; save metrics to a JSON file (e.g. training_rmse.json) alongside the model weights.

## Related tools

- **MS2DeepScore** (Siamese neural network whose predicted Tanimoto scores are evaluated; used via the ms2deepscore Python library (v0.2.0 for the reported results)) — https://github.com/matchms/ms2deepscore
- **RDKit** (Computes Daylight 2048-bit fingerprints and the ground-truth Tanimoto similarity labels for spectrum pairs)
- **matchms** (Python library for mass spectra import, handling and comparisons; integrates MS2DeepScore) — https://github.com/matchms/ms2deepscore

## Examples

```
rmse = np.sqrt(np.mean((predicted_tanimoto - true_tanimoto)**2))  # on 3601 test spectra of 500 unseen compounds, expect ~0.15
```

## Evaluation signals

- Overall test-set RMSE falls in the reported 0.13–0.2 window (~0.15) on the 3601-spectrum test set
- Molecule-level split purity: 15,062 InChIKeys partitioned 14,062/500/500 with zero compound overlap between training and test
- Per-bin check: RMSE best between Tanimoto 0.1 and 0.9, lowest accuracy for extreme (highest/lowest) similarity bins
- Median aggregation over the ~100 spectral-pair predictions per compound pair improves accuracy versus individual spectral-pair predictions
- Predicted Tanimoto scores bounded between 0.1 and 0.9 for the test set

## Limitations

- Accuracy is lowest for extreme (highest/lowest) Tanimoto bins, and 0.8–0.9 scores cannot be discriminated from a near-complete chemical match
- Predicted scores are not deterministic and can change when a model is retrained with different parameters or training data
- The GNPS training library is biased toward certain compound classes and mixes instrumentation types of varying quality, capping achievable precision (deeper/wider networks and less restrictive binning did not significantly reduce the error)

## Evidence

- [results] Abstract finding on test RMSE: "MS2DeepScore generally performs very well and can predict Tanimoto scores between 0.1 and 0.9 with a RMSE between 0.13 and 0.2"
- [discussion] Molecule-level split definition: "The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500)."
- [readme] Library purpose and task framing: "`ms2deepscore` provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra."
- [results] Median aggregation improvement: "structural similarity score predictions can be improved by taking the median of the scores calculated for different pairs of spectra corresponding to the same compound pairs"
