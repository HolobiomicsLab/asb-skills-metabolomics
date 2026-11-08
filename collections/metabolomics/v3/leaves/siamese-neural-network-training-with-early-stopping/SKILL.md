---
name: siamese-neural-network-training-with-early-stopping
description: Use when you have >100,000 annotated MS/MS spectra with InChIKey/SMILES
  annotations and need a learned spectral similarity score (e.g. Tanimoto prediction)
  instead of cosine-type spectral metrics.
when_to_use_negative:
- Your spectral dataset is much smaller than ~100,000 spectra — the README states
  training your own model 'should contain a substantial amount of spectra to learn
  relevant features, say > 100,000 spectra', and pair sampling can be suboptimal for
  smaller sets; use a pre-trained model instead.
- You need deterministic, reproducible similarity scores across retrainings — the
  article notes 'a score itself is not deterministic but might change when a new model
  is trained'.
- You need an intuitive explanation for a specific similarity score — the article
  flags 'the lack of an intuitive explanation of why a certain score is given' as
  a deep-learning explainability gap.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore
  role: Siamese neural network library whose training loop implements the Adam/MSE/early-stopping
    procedure on binned spectrum pairs
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Spectrum import, handling, filtering, and integration of MS2DeepScore score
    computations
  repo: https://github.com/matchms/ms2deepscore
- name: RDKit
  role: Computes Daylight (and Morgan) 2048-bit molecular fingerprints used as Tanimoto
    training labels
- name: GNPS
  role: Source of the annotated MS/MS spectra (210,407 spectra retrieved 25/01/2021)
    used for training/validation/test data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:b0eb229dab47cddf061f10c7bf2cc7038b1ff5b1ff2925b26646979c71bb0ec4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - GNPS
  - MS2DeepScore
  - RDKit
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# siamese-neural-network-training-with-early-stopping

## Summary

Train a Siamese neural network (MS2DeepScore) on pairs of GNPS-derived MS/MS spectra to predict Tanimoto structural similarity, using Adam with MSE loss, early stopping on validation loss, and molecule-level train/validation/test splits.

## When to use

Use when you have >100,000 annotated MS/MS spectra with InChIKey/SMILES annotations and need a learned spectral similarity score (e.g. Tanimoto prediction) instead of cosine-type spectral metrics.

## When NOT to use

- Your spectral dataset is much smaller than ~100,000 spectra — the README states training your own model 'should contain a substantial amount of spectra to learn relevant features, say > 100,000 spectra', and pair sampling can be suboptimal for smaller sets; use a pre-trained model instead.
- You need deterministic, reproducible similarity scores across retrainings — the article notes 'a score itself is not deterministic but might change when a new model is trained'.
- You need an intuitive explanation for a specific similarity score — the article flags 'the lack of an intuitive explanation of why a certain score is given' as a deep-learning explainability gap.

## Inputs

- Annotated GNPS MS/MS spectra (cleaned dataset of 210,407 spectra from Zenodo record 4699300; training used 109,734 spectra)
- Molecular structures as InChIKeys with SMILES/InChI for fingerprint computation
- RDKit Daylight 2048-bit fingerprints (pairwise Tanimoto similarity labels)

## Outputs

- Trained Siamese model weights (e.g. ms2deepscore_model.hdf5 / ms2deepscore_model.pt)
- 200-dimensional spectrum embeddings per input spectrum
- Predicted Tanimoto similarity scores for spectrum pairs
- Evaluation metrics: overall and per-Tanimoto-bin RMSE (expected 0.13–0.2 on 3601 test spectra)

## How to apply

Split the 15,062 unique InChIKeys into training (14,062), validation (500), and test (500) sets — split by molecule, not by spectrum, so test spectra come from unseen compounds. Preprocess spectra by removing peaks <0.1% of maximum intensity, keeping at most the 1000 highest-intensity peaks, square-root transforming intensities, and binning peaks into 10,000 equal-width bins from 10 to 1000 m/z. Compute Tanimoto labels for spectrum pairs from RDKit Daylight 2048-bit fingerprints, then train the Siamese network with the Adam optimizer (MSE loss, batch size 32, learning rate 0.001), applying L1/L2 weight penalties, dropout, three data augmentations (low-intensity peak removal, intensity jitter, new peak addition), and early stopping after 5 epochs without validation-loss decrease. Evaluate RMSE between predicted and true Tanimoto scores on the 3601-spectrum test set; expect RMSE between 0.13 and 0.2, with accuracy lowest for extreme (highest/lowest) similarity bins.

## Related tools

- **MS2DeepScore** (Siamese neural network library whose training loop implements the Adam/MSE/early-stopping procedure on binned spectrum pairs) — https://github.com/matchms/ms2deepscore
- **matchms** (Spectrum import, handling, filtering, and integration of MS2DeepScore score computations) — https://github.com/matchms/ms2deepscore
- **RDKit** (Computes Daylight (and Morgan) 2048-bit molecular fingerprints used as Tanimoto training labels)
- **GNPS** (Source of the annotated MS/MS spectra (210,407 spectra retrieved 25/01/2021) used for training/validation/test data)

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; from ms2deepscore.wrapper_functions.training_wrapper_functions import train_ms2deepscore_wrapper; train_ms2deepscore_wrapper(SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf'), SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Test-set RMSE between predicted and true Tanimoto scores falls in the 0.13–0.2 range (paper reports ~0.15 on 3601 spectra of 500 unseen compounds).
- Per-bin RMSE is lowest for mid-range similarity bins and degrades toward extreme (highest/lowest) Tanimoto bins.
- Validation loss stops improving after ≤5 epochs (early-stopping criterion triggered); training used batch size 32 and learning rate 0.001 with Adam/MSE.
- No InChIKey overlap between training (14,062), validation (500), and test (500) molecule sets — leakage check on molecule-level split.
- Predicted Tanimoto scores concentrate between 0.1 and 0.9 for test-set compound pairs.

## Limitations

- Accuracy is lowest for extreme (highest/lowest) Tanimoto similarity bins, and it is difficult to discriminate high Tanimoto scores (0.8–0.9) from a near-complete chemical match.
- Predictions are not deterministic across retrainings and change with training data or hyperparameters.
- Spectral libraries (GNPS-derived training data) are biased toward certain compound classes, contain mixed instrumentation types of varying quality, and are unbalanced with respect to compound classes; deeper/wider networks and less restrictive binning did not significantly reduce the Tanimoto prediction error.

## Evidence

- [discussion] Models were trained with the Adam optimizer [44, 45] that optimized the mean squared error (MSE) loss. We used a batch size of 32, and a learning rate of 0.001.: "Models were trained with the Adam optimizer [44, 45] that optimized the mean squared error (MSE) loss. We used a batch size of 32, and a learning rate of 0.001."
- [discussion] The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500).: "The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500)."
- [discussion] removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks.: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [results] MS2DeepScore generally performs very well and can predict Tanimoto scores between 0.1 and 0.9 with a RMSE between 0.13 and 0.2: "MS2DeepScore generally performs very well and can predict Tanimoto scores between 0.1 and 0.9 with a RMSE between 0.13 and 0.2"
- [readme] ms2deepscore` provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra.: "`ms2deepscore` provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra."
