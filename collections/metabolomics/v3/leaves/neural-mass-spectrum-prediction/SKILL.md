---
name: neural-mass-spectrum-prediction
description: Use when you need to predict EI mass spectra from molecular structures
  (SMILES/SDF/InChI) or augment a spectral reference library (e.g., NIST, MassBank,
  Wiley) with predicted spectra for molecules absent from the library.
when_to_use_negative:
- The molecule is already present in the reference library — direct observed-spectrum
  matching is preferable to replacing observed spectra with predictions
- Accurate isotope intensity peaks in ion fragments are required — the article notes
  'NEIMS currently does not have a method to model intensity peaks corresponding to
  isotopes in ion fragments'
- The ionization method is not electron ionization without retraining — the model
  was trained on EI spectra from NIST 2017 and would need retraining for other ionization
  methods
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RDKit Cheminformatics package
  role: Generate additive Extended Circular Fingerprints (length 4096, radius 2) from
    molecular structures as MLP input features
- name: Tensorflow
  role: Construct, train, and run the bidirectional MLP for spectra prediction
- name: Adam optimizer
  role: Stochastic gradient descent optimizer used to train the MLP with the modified
    MSE loss
- name: Vizier
  role: Hyperparameter tuning of the MLP architecture, optimizing recall@10
- name: NEIMS
  role: The pretrained model whose predictions augment the reference library; weights
    downloadable from the project repo
  repo: github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:214ff101b993e62ba127486926b6865132c6c8a9f7d2c5a43cd08fb9bc06303b
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# neural-mass-spectrum-prediction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Predict electron-ionization mass spectra for small molecules with a bidirectional MLP (NEIMS) over additive ECFP fingerprints, then use the predicted spectra to augment mass spectral reference libraries for library matching (~5 ms per molecule, 91.8% recall@10).

## When to use

Use when you need to predict EI mass spectra from molecular structures (SMILES/SDF/InChI) or augment a spectral reference library (e.g., NIST, MassBank, Wiley) with predicted spectra for molecules absent from the library.

## When NOT to use

- The molecule is already present in the reference library — direct observed-spectrum matching is preferable to replacing observed spectra with predictions
- Accurate isotope intensity peaks in ion fragments are required — the article notes 'NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments'
- The ionization method is not electron ionization without retraining — the model was trained on EI spectra from NIST 2017 and would need retraining for other ionization methods

## Inputs

- Molecular structures (SDF, SMILES, or InChIKey format) for candidate molecules
- NIST 2017 Mass Spectral Main Library spectra (240,942 spectra) for training the model
- NIST 2017 replicates library spectra as query set (test/validation halves)
- Observed reference library spectra (NIST main library, MassBank, Wiley Registry) to augment

## Outputs

- Predicted full m/z-intensity vector (annotated spectra) per molecule, ~5 ms per molecule
- Augmented reference library of observed plus predicted spectra
- Recall@10 library-matching accuracy (rank of correct hit per query)

## How to apply

Encode each molecule as an additive Extended Circular Fingerprint (ECFP) of length 4096 with radius 2 using RDKit, then feed the fingerprint vector into the pretrained bidirectional MLP — seven layers of 2000 nodes with residual connections, ReLU activation, and dropout 0.25 — trained in TensorFlow for 100,000 steps with batch size 100 and a modified mean-squared-error loss weighted per eq 1. The model outputs an intensity value for every integral m/z bin; the final bidirectional prediction combines the forward and reverse (spectra-prediction-from-fragment) modes with a coordinate-wise gate. For library matching, build an augmented reference library by replacing spectra of query-set molecules in the NIST main library with model predictions, then match query spectra using weighted cosine similarity (eq 1) restricted to candidates within a 5 Da mass tolerance of the query molecule's mass (reducing the library to a median of 6696 spectra per query). Judge success by recall@10 (rank ≤ 10 of the correct hit); the published acceptance criterion is 91.8% recall-at-10 on the test split, with mass filtering contributing ~6 percentage points.

## Related tools

- **RDKit Cheminformatics package** (Generate additive Extended Circular Fingerprints (length 4096, radius 2) from molecular structures as MLP input features)
- **Tensorflow** (Construct, train, and run the bidirectional MLP for spectra prediction)
- **Adam optimizer** (Stochastic gradient descent optimizer used to train the MLP with the modified MSE loss)
- **Vizier** (Hyperparameter tuning of the MLP architecture, optimizing recall@10)
- **NEIMS** (The pretrained model whose predictions augment the reference library; weights downloadable from the project repo) — github.com/brain-research/deep-molecular-massspec

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Recall@10 (rank ≤ 10 of the correct spectrum) on the held-out test half of the NIST 2017 replicates library should reach ~91.8% with the 5 Da mass filter
- Mass filtering at 5 Da tolerance should reduce the candidate library to a median of 6696 spectra per query molecule
- Per-molecule prediction latency should average ~5 ms
- Spectra prediction output should be a vector with a value at every integral m/z bin (full predicted spectrum, forward+reverse combined)
- Training setup check: 100,000 training steps, batch size 100, dropout 0.25, seven-layer 2000-node residual MLP

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments
- The model was trained for electron ionization; other ionization methods require retraining, though the approach does not rely on EI-specific details
- Library matching accuracy improves only when the molecular mass of the sample can be estimated with some accuracy, enabling mass filtering
- No version numbers are given for NEIMS or its software dependencies; weights require Tensorflow 1.13.2-era tooling per the README

## Evidence

- [readme] README prediction invocation: [evidence span withheld — anchor status: not_found]
