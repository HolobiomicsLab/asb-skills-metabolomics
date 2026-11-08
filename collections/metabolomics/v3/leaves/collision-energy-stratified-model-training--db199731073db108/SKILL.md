---
name: collision-energy-stratified-model-training
description: Decision support for choosing whether to train separate MS/MS fingerprint-prediction
  models per collision-energy range (low <=30 eV, high >=30 eV) instead of one model
  spanning all energies.
when_to_use_negative:
- Evaluating whether combining IT and HCD spectra affects performance — the paper
  found combining IT and HCD does not affect prediction performance, so stratification
  by that axis is unnecessary
- Input spectra lack collision energy metadata, making the 30 eV low/high partition
  impossible to apply
- The task is candidate ranking/annotation with the released pretrained MetFID_CNN
  model rather than benchmarking training strategies
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: MetFID
  role: ANN framework whose models are retrained per collision-energy stratum and
    combined
  repo: https://github.com/aohongyu/MetFID
- name: MoNA
  role: Source of MS/MS spectra with varied collision energies for training/testing
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Spectral library used for cluster analysis deriving the 30 eV cutoff and for
    training data
- name: OpenBabel
  role: Computes MACCS/FP3/FP4 528-bit fingerprints used as regression targets
- name: TensorFlow
  role: Backend for training the Keras ANNs (Adam, 30 epochs, batch size 100)
  repo: https://www.tensorflow.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:f06798d31da593b25772f612cd492d4f6b81f23ce2b3e61d25c61fb23862690c
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
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# collision-energy-stratified-model-training

## Summary

Trains separate MetFID ANNs on HCD spectra partitioned by collision energy range around the empirically derived 30 eV cutoff, and compares them against a combined all-energies model to quantify whether stratification improves fingerprint-prediction accuracy and F1.

## When to use

Decision support for choosing whether to train separate MS/MS fingerprint-prediction models per collision-energy range (low <=30 eV, high >=30 eV) instead of one model spanning all energies.

## When NOT to use

- Evaluating whether combining IT and HCD spectra affects performance — the paper found combining IT and HCD does not affect prediction performance, so stratification by that axis is unnecessary
- Input spectra lack collision energy metadata, making the 30 eV low/high partition impossible to apply
- The task is candidate ranking/annotation with the released pretrained MetFID_CNN model rather than benchmarking training strategies

## Inputs

- MoNA and NIST 17 MS/MS spectra (positive mode, HCD/QqQ/Q-TOF or IT collision-cell instruments, 100–1010 Da, H+ or NH4+ adducts)
- Per-spectrum collision energy metadata
- Compound structures for fingerprint generation (InChIKey identifiers)
- OpenBabel-computed 528-bit MACCS+FP3+FP4 fingerprint vectors as ANN training targets

## Outputs

- Trained low-energy, high-energy, and combined ANN models (TensorFlow/Keras)
- Accuracy and F1 comparison table (collision_energy_model_comparison.csv) for the three models
- Validation verdict on whether the combined model underperforms energy-specific models

## How to apply

First verify that collision energy materially changes fragmentation in your data — the paper's cluster analysis of 916 NIST 17 compounds showed spectra from similar collision energies cluster together and that 30 eV is a reasonable low/high cutoff. Preprocess spectra (positive mode, HCD/QqQ/Q-TOF or IT collision cell, 100–1010 Da, H+ or NH4+ adducts) by scaling intensities to 0–100, dropping spectra with fewer than five peaks above 2% relative intensity, denoising, and binning m/z into integer bins with all-zero bins removed. Compute 528-bit fingerprints (MACCS + FP3 + FP4 via OpenBabel) as ANN targets. Train three Keras/TensorFlow ANNs (input layer, two ReLU hidden layers, sigmoid output with binary cross-entropy, Adam optimizer, 30 epochs, batch size 100): one on low-energy (<=30 eV), one on high-energy (>=30 eV), and one on all combined spectra. Evaluate all three with structure-disjoint five-fold cross-validation on the 6677 HCD compounds, ensuring no compound (first InChIKey block) appears in both train and test folds, then compute macro-averaged accuracy and F1 per model.

## Related tools

- **MetFID** (ANN framework whose models are retrained per collision-energy stratum and combined) — https://github.com/aohongyu/MetFID
- **MoNA** (Source of MS/MS spectra with varied collision energies for training/testing) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Spectral library used for cluster analysis deriving the 30 eV cutoff and for training data)
- **OpenBabel** (Computes MACCS/FP3/FP4 528-bit fingerprints used as regression targets)
- **TensorFlow** (Backend for training the Keras ANNs (Adam, 30 epochs, batch size 100)) — https://www.tensorflow.org/

## Examples

```
python main.py  # after partitioning spectra into low (<=30 eV), high (>=30 eV), and combined sets, train each ANN in Keras/TensorFlow with Adam, 30 epochs, batch size 100
```

## Evaluation signals

- Combined model's accuracy must be slightly lower than the low-energy model's accuracy when tested on low-energy spectra
- Combined model's F1 scores must be lower than both the low- and high-energy models' F1 scores
- Cross-validation folds are structure-disjoint: no compound (first InChIKey block) appears in both training and testing folds
- All three models use identical architecture/hyperparameters (two ReLU hidden layers, sigmoid output, Adam, 30 epochs, batch size 100) so the only variable is energy stratification
- Cluster analysis of NIST 17 spectra (916 compounds, >30 spectra/compound) should support 30 eV as the low/high cutoff before partitioning

## Limitations

- No released pre-trained model weights for MetFID; the pretrained CNN (MetFID_CNN_40088_5618.h5) on GitHub corresponds to the later CNN variant, not this benchmark
- Findings are established on 6677 HCD compounds; energy stratification conclusions may not transfer to other collision-cell instrument types
- Cutoff of 30 eV was derived empirically by cluster analysis of NIST 17 spectra and may need re-derivation for other datasets

## Evidence
