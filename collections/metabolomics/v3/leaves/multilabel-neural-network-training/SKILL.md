---
name: multilabel-neural-network-training
description: Use when MS/MS spectra (binned peak vectors) must be mapped to a full
  compound fingerprint vector simultaneously, or when a per-digit/per-block multi-model
  setup is undesirable.
when_to_use_negative:
- The task only requires single-label classification of one fingerprint digit or a
  small fingerprint block, where a softmax/categorical cross-entropy setup is sufficient.
- Spectra cannot be binned or lack the metadata (precursor mass, collision energy,
  InChIKey) needed for the merging, filtering, and loss-feature preprocessing steps.
- Only fingerprints not covered by MACCS, FP3, or FP4 are needed; the article notes
  further investigation is needed to evaluate other fingerprint sets.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: MetFID
  role: ANN/CNN implementation that trains the multi-label fingerprint predictor and
    ships a pre-trained .h5 model
  repo: https://github.com/aohongyu/MetFID
- name: OpenBabel
  role: Computes the MACCS, FP3, and FP4 fingerprints that form the 528-bit training
    labels
- name: TensorFlow
  role: Backend for the Keras model; required dependency for training and inference
- name: PyFingerprint
  role: Alternative Python library providing fingerprint types incl. babel fp2/fp3/fp4,
    listed as a prerequisite in the repo README
  repo: https://github.com/hcji/PyFingerprint
- name: MoNA
  role: Source of MS/MS training spectra (Vaniya/Fiehn, GNPS, RIKEN PlaSMA, MassBank,
    Fiehn HILIC libraries)
  repo: https://mona.fiehnlab.ucdavis.edu/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:a3c44dc63cb764386b973452cf365ad90f6ae384f1e59b2d3ae9f2aedfcedde1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  - PyFingerprint
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# multilabel-neural-network-training

## Summary

Train a single multi-label ANN (Keras/TensorFlow) that maps binned MS/MS peak vectors to an entire 528-bit MACCS+FP3+FP4 fingerprint in one pass, using sigmoid + binary cross-entropy so latent relationships between fingerprint digits are learned jointly rather than through per-digit or per-block models.

## When to use

Use when MS/MS spectra (binned peak vectors) must be mapped to a full compound fingerprint vector simultaneously, or when a per-digit/per-block multi-model setup is undesirable.

## When NOT to use

- The task only requires single-label classification of one fingerprint digit or a small fingerprint block, where a softmax/categorical cross-entropy setup is sufficient.
- Spectra cannot be binned or lack the metadata (precursor mass, collision energy, InChIKey) needed for the merging, filtering, and loss-feature preprocessing steps.
- Only fingerprints not covered by MACCS, FP3, or FP4 are needed; the article notes further investigation is needed to evaluate other fingerprint sets.

## Inputs

- Preprocessed, binned MS/MS peak vectors from MoNA libraries (e.g. 11,784 spectra from 5,667 compounds) and NIST 17 (e.g. 122,481 spectra from 10,730 compounds)
- 528-bit binary fingerprint labels computed via OpenBabel (MACCS + FP3 + FP4) for all training compounds
- Spectrum metadata: precursor m/z, ionization mode (positive/negative), collision energy, and compound identifier (InChIKey)

## Outputs

- A trained .h5 ANN model mapping binned MS/MS vectors to a 528-bit fingerprint vector
- Predicted 528-bit fingerprint for each held-out test spectrum
- Macro-average accuracy and F1 score on a structure-disjoint test set

## How to apply

Assemble the label vector by concatenating MACCS, FP3, and FP4 fingerprints computed with OpenBabel into 528 binary entries, and bin each MS/MS spectrum into prespecified m/z bins after merging same-compound spectra by InChIKey (m/z difference ≤ 0.1), scaling intensities to 0–100, removing bins that are all zeros across the training set, and dropping spectra with fewer than five peaks above 2% relative intensity. In Keras on the TensorFlow backend, build an ANN whose input layer accepts the binned spectrum vectors, add two hidden layers with ReLU activation, and give the 528-node output layer sigmoid activation with binary cross-entropy loss so all digits are predicted simultaneously in one multi-label pass. Compile with the 'Adam' optimizer and train on spectrum–fingerprint pairs for 30 epochs with a batch size of 100, saving the trained model in .h5 format. Optionally explore CNN architectures (the public repo ships a pre-trained CNN model, MetFID_CNN_40088_5618.h5) and consider separate models for low (≤30 eV) vs. high (≥30 eV) collision energies when cluster analysis shows spectra group by collision energy.

## Related tools

- **MetFID** (ANN/CNN implementation that trains the multi-label fingerprint predictor and ships a pre-trained .h5 model) — https://github.com/aohongyu/MetFID
- **OpenBabel** (Computes the MACCS, FP3, and FP4 fingerprints that form the 528-bit training labels)
- **TensorFlow** (Backend for the Keras model; required dependency for training and inference)
- **PyFingerprint** (Alternative Python library providing fingerprint types incl. babel fp2/fp3/fp4, listed as a prerequisite in the repo README) — https://github.com/hcji/PyFingerprint
- **MoNA** (Source of MS/MS training spectra (Vaniya/Fiehn, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC libraries)) — https://mona.fiehnlab.ucdavis.edu/

## Examples

```
python3 main.py  # after placing MetFID_CNN_40088_5618.h5 and _files/testing_compound.txt / _files/inchikey_list.txt in the MetFID root; outputs _files/testing_compound_prediction.txt
```

## Evaluation signals

- Predicted fingerprint vector has exactly 528 binary entries per spectrum.
- Model output layer uses sigmoid activation with binary cross-entropy loss, not softmax/categorical cross-entropy.
- Training curves match the published hyperparameters: 30 epochs, batch size 100, Adam optimizer.
- Held-out evaluation is structure-disjoint (no shared first InChIKey block with training compounds) and reports macro-average accuracy and F1.
- Predicted fingerprints yield Tanimoto-similarity candidate rankings whose top hits can be assessed against inchikey_list.txt candidate blocks.

## Limitations

- Neglects explicit per-digit models, so the article notes it relies on latent hidden relationships between fingerprint digits captured by the joint model.
- Performance depends on collision-energy and instrument clustering: spectra at very different collision energies (e.g. 14 eV vs 38 eV) have very different fragmentation patterns; 30 eV was chosen as the cutoff, and a combined model's accuracy/F1 were slightly lower than energy-separated models.
- Training labels limited to MACCS, FP3, FP4; the authors note further investigation is needed to evaluate different fingerprint sets.
- Fewer than five peaks above 2% relative intensity causes spectra to be discarded from training/evaluation.
- Pre-trained public model is CNN-based (MetFID_CNN_40088_5618.h5) and depends on PubChemPy 1.0.4, Open Babel 3.1.1, tabulate 0.8.9, and TensorFlow; Python 3.5+ only.

## Evidence

- [methods] The molecular fingerprints of all compounds in the training set were determined using OpenBabel: [evidence span withheld — anchor status: not_found]
