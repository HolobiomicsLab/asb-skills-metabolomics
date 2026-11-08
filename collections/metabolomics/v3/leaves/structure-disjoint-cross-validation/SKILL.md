---
name: structure-disjoint-cross-validation
description: Use when evaluating MS/MS-to-fingerprint prediction models (MetFID-style
  ANNs) where multiple spectra of the same compound exist across collision energies
  or instruments, and naive random splits would leak structural identity.
when_to_use_negative:
- Each compound contributes only one spectrum, so random folds already guarantee no
  compound overlaps between train and test — plain k-fold CV suffices.
- Evaluating final deployment performance on a fully independent external dataset
  rather than comparing model variants by cross-validation.
- The task is candidate ranking from a retrieved list (MetFID candidate retrieval),
  not fingerprint-prediction model training/evaluation.
tools:
- name: MetFID
  role: The multi-class multi-label ANN fingerprint predictor whose model variants
    are compared under this structure-disjoint five-fold CV scheme
  repo: https://github.com/aohongyu/MetFID
- name: TensorFlow
  role: Keras/TensorFlow backend used to train each ANN variant per fold (ReLU hidden
    layers, sigmoid output, binary cross-entropy, Adam)
  repo: https://www.tensorflow.org/
- name: OpenBabel
  role: Computes the 528-bit MACCS+FP3+FP4 fingerprint targets for each compound before
    fold assignment
  repo: https://openbabel.org/wiki/Python
- name: InChIKey
  role: Compound identifier used to merge spectra per compound and enforce the structure-disjoint
    first-block constraint on folds
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:f848341dcf27c903b51826449358b3cf269a3d04be91f404d5ec3a855686dc16
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - InChIKey
  - Keras
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  - TensorFlow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# structure-disjoint-cross-validation

## Summary

A five-fold cross-validation scheme for fingerprint-prediction ANNs in which folds are split by compound identity (first InChIKey block) so that no compound appears in both training and testing folds, preventing structural leakage when evaluating models on MS/MS spectra (e.g., the 6677-compound HCD set with low/high/combined collision-energy models).

## When to use

Use when evaluating MS/MS-to-fingerprint prediction models (MetFID-style ANNs) where multiple spectra of the same compound exist across collision energies or instruments, and naive random splits would leak structural identity.

## When NOT to use

- Each compound contributes only one spectrum, so random folds already guarantee no compound overlaps between train and test — plain k-fold CV suffices.
- Evaluating final deployment performance on a fully independent external dataset rather than comparing model variants by cross-validation.
- The task is candidate ranking from a retrieved list (MetFID candidate retrieval), not fingerprint-prediction model training/evaluation.

## Inputs

- Preprocessed MS/MS spectra (positive mode, collision-cell HCD/QqQ/Q-TOF or IT, 100–1010 Da, H+ or NH4+ adducts) scaled to 0–100, denoised, with spectra of fewer than five peaks above 2% relative intensity removed
- Integer-binned m/z spectra with all-zero bins dropped
- 528-bit molecular fingerprints (MACCS + FP3 + FP4 via OpenBabel) keyed by InChIKey per compound
- Collision-energy metadata per spectrum for partitioning at the 30 eV cutoff

## Outputs

- Five-fold structure-disjoint train/test compound assignments (fold membership by first InChIKey block)
- Per-fold macro-averaged accuracy and F1 scores for each model variant
- Aggregated model-comparison table (e.g., collision_energy_model_comparison.csv)

## How to apply

Merge all spectra of a compound using its InChIKey as the compound identifier, then assign whole compounds — not spectra — to the five folds, checking the first InChIKey block so no compound is present in both training and testing partitions. Train the ANN variants being compared (here: low-energy ≤30 eV, high-energy ≥30 eV, and combined models; two ReLU hidden layers, sigmoid output, binary cross-entropy, Adam, 30 epochs, batch size 100 in TensorFlow/Keras) using only training-fold compounds, then predict fingerprints for held-out compounds and compute macro-averaged accuracy and F1 scores per fold. Repeat across all five folds and aggregate so every model variant is evaluated on identical structure-disjoint test sets; this is the design MetFID used on 6677 HCD compounds from MoNA and NIST 17 to show the combined-energy model scores slightly lower accuracy than the low-energy model and lower F1 than both energy-specific models.

## Related tools

- **MetFID** (The multi-class multi-label ANN fingerprint predictor whose model variants are compared under this structure-disjoint five-fold CV scheme) — https://github.com/aohongyu/MetFID
- **TensorFlow** (Keras/TensorFlow backend used to train each ANN variant per fold (ReLU hidden layers, sigmoid output, binary cross-entropy, Adam)) — https://www.tensorflow.org/
- **OpenBabel** (Computes the 528-bit MACCS+FP3+FP4 fingerprint targets for each compound before fold assignment) — https://openbabel.org/wiki/Python
- **InChIKey** (Compound identifier used to merge spectra per compound and enforce the structure-disjoint first-block constraint on folds)

## Evaluation signals

- No first-InChIKey-block value appears in both the training and testing compound sets of any fold
- Combined-energy model shows slightly lower accuracy than the low-energy model on low-energy test spectra
- Combined-energy model F1 scores are lower than both the low- and high-energy model F1 scores
- All folds use the same compound universe (6677 HCD compounds) and identical preprocessing/scaling so model variants are directly comparable
- Reported metrics are macro-averaged accuracy and F1 computed per model on its corresponding testing spectra

## Limitations

- Validation set restricted to HCD spectra from 6677 compounds with specific adducts (H+, NH4+) and mass range (100–1010 Da); the 30 eV cutoff was calibrated on NIST 17 cluster analysis and may not generalize to other instrument families.
- Cluster analysis showed IT spectra cluster with collision-cell spectra despite different fragmentation mechanisms, so energy-based partitioning is an approximation of spectral similarity.
- No comparison against non-ML reference tools is provided in the discussion, and no pre-trained per-energy model weights are released, limiting independent reproduction.

## Evidence

- [methods] Fold-disjointness constraint by first InChIKey block: [evidence span withheld — anchor status: not_found]
