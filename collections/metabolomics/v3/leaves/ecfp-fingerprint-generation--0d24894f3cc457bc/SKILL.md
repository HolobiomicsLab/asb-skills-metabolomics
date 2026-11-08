---
name: ecfp-fingerprint-generation
description: Use when molecules must be featurized as fixed-length count/circular
  fingerprints to feed a spectral-prediction or molecular ML model, e.g., encoding
  NIST 2017 library molecules for NEIMS inference.
when_to_use_negative:
- The downstream model expects a different molecular representation (e.g., graph-convolutional
  features — the paper notes bond-centered graph representations would likely improve
  accuracy)
- 'Isotopologue-resolved spectra are the target: NEIMS cannot model isotope intensity
  peaks regardless of fingerprinting'
- The task requires interpreting individual substructural contributions rather than
  fixed-length featurization for regression
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3420
tools:
- name: RDKit Cheminformatics package
  role: Generates the additive ECFPs of length 4096 with radius 2 from input molecular
    structures
  repo: https://www.rdkit.org/docs/Install.html
- name: Tensorflow
  role: Runs the downstream NEIMS MLP that consumes the ECFP fingerprint vectors to
    predict mass spectra
  repo: https://www.tensorflow.org/install
- name: NEIMS
  role: Pretrained model (massspec_weights) that takes the 4096-dim ECFP input and
    outputs annotated spectra predictions
  repo: github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:f8e3bb21407b353452fe64e51f93400be955cffd44e1b0f4396faa87f2896ab9
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# ecfp-fingerprint-generation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Converts small molecules (SMILES/SDF/InChI) into additive Extended Circular Fingerprints (ECFPs) of length 4096 with radius 2 using RDKit, the input representation required by the NEIMS neural network for mass spectra prediction.

## When to use

Use when molecules must be featurized as fixed-length count/circular fingerprints to feed a spectral-prediction or molecular ML model, e.g., encoding NIST 2017 library molecules for NEIMS inference.

## When NOT to use

- The downstream model expects a different molecular representation (e.g., graph-convolutional features — the paper notes bond-centered graph representations would likely improve accuracy)
- Isotopologue-resolved spectra are the target: NEIMS cannot model isotope intensity peaks regardless of fingerprinting
- The task requires interpreting individual substructural contributions rather than fixed-length featurization for regression

## Inputs

- Molecular structures in SMILES format (from training_splits directory)
- Molecular structures as InChIKeys (from training_splits directory)
- SDF files of candidate molecules (e.g., examples/pentachlorobenzene.sdf)
- Molecules from the NIST 2017 Mass Spectral Main Library (240,942 spectra)

## Outputs

- Additive ECFP vectors of length 4096 (count-based, radius 2) per molecule, ready for NEIMS MLP input
- Featurized spectra-prediction inputs for the augmented reference library

## How to apply

Take each molecule's structure representation (SMILES or SDF, as used in the training_splits and examples of the deep-molecular-massspec repo) and generate additive ECFPs with the RDKit Cheminformatics package using fingerprint length 4096 and radius 2. Unlike binary ECFPs, the additive (count-based) variant retains multiplicity of substructural fragments, which lets the downstream multilayer perceptron learn fragment-intensity relationships for electron-ionization fragmentation. Feed the resulting 4096-dimensional vector directly into the pretrained NEIMS model (seven layers of 2000 nodes, bidirectional forward/reverse prediction) as its only input feature. The same featurization must be applied identically to training, validation, and query molecules so that fingerprint semantics match between library and query spectra.

## Related tools

- **RDKit Cheminformatics package** (Generates the additive ECFPs of length 4096 with radius 2 from input molecular structures) — https://www.rdkit.org/docs/Install.html
- **Tensorflow** (Runs the downstream NEIMS MLP that consumes the ECFP fingerprint vectors to predict mass spectra) — https://www.tensorflow.org/install
- **NEIMS** (Pretrained model (massspec_weights) that takes the 4096-dim ECFP input and outputs annotated spectra predictions) — github.com/brain-research/deep-molecular-massspec

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Every fingerprint vector has exactly 4096 dimensions, matching the model's expected input (fingerprint length 4096)
- Fingerprints are additive/count-based: repeated substructural fragments increase counts rather than setting a single binary bit
- The same featurization is applied to query molecules and NIST main library molecules — a mismatch would corrupt library matching recall@10
- Downstream check: NEIMS predictions from these fingerprints achieve ~91.8% recall-at-10 on the NIST 2017 replicates test half with a 5 Da mass filter
- Runtime sanity check: per-molecule prediction from fingerprint through the MLP averages on the order of 5 ms

## Limitations

- Radius-2, length-4096 ECFPs were found optimal for this architecture; hyperparameters were tuned with Vizier, so other fingerprint settings are unvalidated for NEIMS
- Fingerprint-based input cannot encode 3D conformer or collision-energy information; NEIMS also cannot model isotope intensity peaks
- The paper suggests graph-convolutional (bond-centered) representations would likely improve predictive accuracy over ECFPs

## Evidence

- [readme] README prediction entry point: [evidence span withheld — anchor status: not_found]
- [readme] Molecule representations provided for splits: [evidence span withheld — anchor status: not_found]
