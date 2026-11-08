---
name: ann-multilabel-fingerprint-training
description: Use when training a fingerprint predictor from labeled MS/MS spectra
  (positive mode, collision-cell HCD/QqQ/Q-TOF or IT instruments, 100–1010 Da, H+
  or NH4+ adducts) and you need a single network outputting the full fingerprint vector
  rather than per-digit models.
when_to_use_negative:
- Your goal is candidate ranking/annotation of an unknown analyte rather than training
  — use MetFID's prediction and MetaboQuest candidate-retrieval path instead.
- Spectra are negative-mode or acquired on instruments without collision-cell/IT fragmentation,
  which fall outside the paper's selection criteria.
- You need per-digit or per-fingerprint-block classifiers; this skill deliberately
  trains one multi-label network over the whole 528-bit vector.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: MetFID
  role: Reference implementation of the multi-class multi-label ANN fingerprint-prediction
    training pipeline (code repo aohongyu/MetFID or ressomlab/MetFID)
  repo: https://github.com/aohongyu/MetFID
- name: OpenBabel
  role: Computes the MACCS, FP3, and FP4 fingerprints assembled into the 528-bit binary
    training targets
- name: TensorFlow
  role: Backend framework for building and training the Keras ANN (two ReLU hidden
    layers, sigmoid output, Adam optimizer)
- name: PyFingerprint
  role: Dependency used by MetFID to generate fingerprints across RDKit/CDK/OpenBabel
    backends
  repo: https://github.com/hcji/PyFingerprint
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:ef8b9879b52b8c1653024f2acb817ff8f0b1546c91f230ad5ebc618da5a70cec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - Keras
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  - PyFingerprint
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ann-multilabel-fingerprint-training

## Summary

Train a single multi-class multi-label ANN that maps preprocessed MS/MS spectra (binned m/z peaks plus loss features) directly to a 528-bit compound fingerprint (MACCS + FP3 + FP4), as in MetFID. Predicting the whole fingerprint vector at once preserves latent relationships between fingerprint digits that one-model-per-digit designs neglect.

## When to use

Use when training a fingerprint predictor from labeled MS/MS spectra (positive mode, collision-cell HCD/QqQ/Q-TOF or IT instruments, 100–1010 Da, H+ or NH4+ adducts) and you need a single network outputting the full fingerprint vector rather than per-digit models.

## When NOT to use

- Your goal is candidate ranking/annotation of an unknown analyte rather than training — use MetFID's prediction and MetaboQuest candidate-retrieval path instead.
- Spectra are negative-mode or acquired on instruments without collision-cell/IT fragmentation, which fall outside the paper's selection criteria.
- You need per-digit or per-fingerprint-block classifiers; this skill deliberately trains one multi-label network over the whole 528-bit vector.

## Inputs

- MS/MS peak lists (precursor m/z, collision energy, adduct, intensity pairs) from MoNA and NIST 17 libraries
- Compound structures identified by InChIKey for fingerprint computation
- OpenBabel-generated 528-bit fingerprint vectors (MACCS + FP3 + FP4) as training targets

## Outputs

- Trained Keras/TensorFlow ANN model (.h5) mapping binned spectra to 528-bit fingerprints
- Predicted fingerprint vectors for held-out spectra
- Per-model accuracy (macro-averaged) and F1 scores from structure-disjoint five-fold cross validation, e.g. saved to collision_energy_model_comparison.csv

## How to apply

Preprocess spectra by scaling intensities to 0–100, removing spectra with fewer than five peaks above 2% relative intensity, denoising (dropping peaks with intensity below 10), merging peaks with m/z difference ≤ 0.1 per compound (keyed by first InChIKey block), binning m/z into integer bins with loss features computed from precursor-minus-peak m/z differences, and dropping all-zero bins. Compute 528-bit targets with OpenBabel by assembling MACCS, FP3, and FP4 fingerprints into one binary vector. Train one ANN (Keras/TensorFlow) with an input layer, two ReLU hidden layers, and a sigmoid output layer using binary cross-entropy loss and the Adam optimizer for 30 epochs at batch size 100. For variant analyses (e.g., collision-energy ranges ≤30 eV vs ≥30 eV, or IT vs HCD), train separate networks per partition and a combined model, then compare them under structure-disjoint five-fold cross validation where no compound (first InChIKey block) appears in both training and testing folds.

## Related tools

- **MetFID** (Reference implementation of the multi-class multi-label ANN fingerprint-prediction training pipeline (code repo aohongyu/MetFID or ressomlab/MetFID)) — https://github.com/aohongyu/MetFID
- **OpenBabel** (Computes the MACCS, FP3, and FP4 fingerprints assembled into the 528-bit binary training targets)
- **TensorFlow** (Backend framework for building and training the Keras ANN (two ReLU hidden layers, sigmoid output, Adam optimizer))
- **PyFingerprint** (Dependency used by MetFID to generate fingerprints across RDKit/CDK/OpenBabel backends) — https://github.com/hcji/PyFingerprint

## Examples

```
python3 main.py  # run inside the MetFID folder after installing PyFingerprint, PubChemPy 1.0.4, Open Babel 3.1.1, tabulate 0.8.9 and Tensorflow
```

## Evaluation signals

- Each input spectrum yields a 528-dimensional binary sigmoid output matching the MACCS+FP3+FP4 target vector length.
- Structure-disjoint five-fold CV: no first-InChIKey-block compound appears in both training and testing folds.
- Held-out macro-averaged accuracy and F1 are computed for every trained variant and tabulated (e.g., collision_energy_model_comparison.csv).
- Replication check: combined collision-energy model shows accuracy slightly lower than the low-energy (≤30 eV) model on low-energy test spectra and F1 lower than both energy-specific models.
- Binning removes all-zero bins across the training set, so input dimensionality equals the count of non-empty integer m/z bins plus loss features.

## Limitations

- Results are specific to positive-mode, 100–1010 Da spectra with H+ or NH4+ adducts from collision-cell (HCD/QqQ/Q-TOF) or IT instruments; other regimes are untested here.
- Training-set fingerprints come from OpenBabel MACCS+FP3+FP4 only; the paper notes further investigation is needed to evaluate different fingerprint sets.
- Only ANN variants (e.g., collision-energy partitions) were explored; no pre-trained weights are released for arbitrary reuse and other architectures were left as future work.
- Collision-energy partitioning relies on the 30 eV cutoff identified by cluster analysis of NIST 17 spectra; it may not transfer to other instrument platforms.

## Evidence

- [intro] we recently proposed a multi-class and multi-label task formulation using an artificial neural network (ANN) to be able to predict a vector representing the whole compound fingerprint at one time: "we recently proposed a multi-class and multi-label task formulation using an artificial neural network (ANN) to be able to predict a vector representing the whole compound fingerprint at one time"
- [methods] Specifically, MACCS, FP3, and FP4 fingerprints were determined and assembled into a vector comprising of 528 binary entries: "Specifically, MACCS, FP3, and FP4 fingerprints were determined and assembled into a vector comprising of 528 binary entries"
- [methods] We scaled the peak intensities such that all intensity values are between 0 and 100. Spectra that consisted of fewer than five peaks with relative intensity above 2% were removed: "We scaled the peak intensities such that all intensity values are between 0 and 100. Spectra that consisted of fewer than five peaks with relative intensity above 2% were removed"
- [results] we found that 30 eV was a reasonable cutoff for high and low collision energies: "we found that 30 eV was a reasonable cutoff for high and low collision energies"
- [readme] please download the latest CNN model -- [MetFID_CNN_40088_5618.h5][5] and put it in the root directory of MetFID.: "please download the latest CNN model -- [MetFID_CNN_40088_5618.h5][5] and put it in the root directory of MetFID."
