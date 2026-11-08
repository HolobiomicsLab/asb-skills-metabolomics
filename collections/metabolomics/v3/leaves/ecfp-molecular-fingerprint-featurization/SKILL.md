---
name: ecfp-molecular-fingerprint-featurization
description: Use when a neural-network or regression model needs a fixed-length molecular
  representation computed from SMILES/SDF structures, e.g. to predict electron-ionization
  mass spectra in the NEIMS pipeline.
when_to_use_negative:
- The task requires modeling isotope intensity peaks in ion fragments — the article
  states NEIMS lacks a method for this
- Binary (presence/absence) ECFP bits suffice and substructure occurrence counts are
  not needed — the additive fingerprint is specifically chosen to preserve counts
- The ionization method is not electron ionization and no retraining data exists for
  that method — the featurization is coupled to an EI-trained MLP head
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3314
tools:
- name: RDKit Cheminformatics package
  role: Generates the additive Extended Circular Fingerprints (length 4096, radius
    2) that featurize each molecule for the MLP
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: Tensorflow
  role: Constructs and trains the multilayer-perceptron that consumes the ECFP features
    and outputs per-m/z-bin intensities
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: Adam optimizer
  role: Optimizes the MLP parameters over 100,000 training steps with batch size 100
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:7cd20acef9dd6aa288ce1f7a89469db6f21633499e8fdfb6bbc894db43afad03
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/brain-research/deep-molecular-massspec
  tools:
  - Adam optimizer
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ecfp-molecular-fingerprint-featurization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Converts small-molecule structures (SDF/SMILES) into additive Extended Circular Fingerprints (ECFPs) of length 4096 with radius 2 via RDKit, providing the fixed-size input vector that the NEIMS MLP consumes to regress mass-spectral intensity at every integral m/z bin.

## When to use

Use when a neural-network or regression model needs a fixed-length molecular representation computed from SMILES/SDF structures, e.g. to predict electron-ionization mass spectra in the NEIMS pipeline.

## When NOT to use

- The task requires modeling isotope intensity peaks in ion fragments — the article states NEIMS lacks a method for this
- Binary (presence/absence) ECFP bits suffice and substructure occurrence counts are not needed — the additive fingerprint is specifically chosen to preserve counts
- The ionization method is not electron ionization and no retraining data exists for that method — the featurization is coupled to an EI-trained MLP head

## Inputs

- Molecular structures in SDF format (e.g. examples/pentachlorobenzene.sdf) or SMILES/InChIKey from the NIST training_splits
- NIST 2017 Mass Spectral Main Library spectra with associated molecular structures (240,942 training spectra)

## Outputs

- 4096-length additive ECFP count vectors (radius 2) per molecule
- Predicted 1000-bin EI mass-spectral intensity vectors after MLP regression (forward_mlp_predictions.csv)

## How to apply

Map each molecule to an additive Extended Circular Fingerprint with the RDKit Cheminformatics package, using a fingerprint length of 4096 and radius 2 so subgraph occurrence counts (rather than binary presence bits) are hashed into the vector. Feed these ECFP vectors into the multilayer perceptron whose hidden layers learn fragmentation-pattern features, with a per-bin affine output layer regressing intensity at each integral m/z index up to 1000 bins. Train end-to-end with a modified mean-squared-error loss (weighted like the cosine similarity of eq 1) using Adam for 100,000 steps at batch size 100 on the 240,942-spectra NIST 2017 Mass Spectral Main Library training set. After prediction, zero out intensities at m/z values above the molecule's molecular mass and export the 1000-bin vector (e.g. to forward_mlp_predictions.csv). The additive (count-based) formulation preserves substructure multiplicity, which matters for molecules with repeated fragments that dominate EI fragmentation.

## Related tools

- **RDKit Cheminformatics package** (Generates the additive Extended Circular Fingerprints (length 4096, radius 2) that featurize each molecule for the MLP) — https://github.com/brain-research/deep-molecular-massspec
- **Tensorflow** (Constructs and trains the multilayer-perceptron that consumes the ECFP features and outputs per-m/z-bin intensities) — https://github.com/brain-research/deep-molecular-massspec
- **Adam optimizer** (Optimizes the MLP parameters over 100,000 training steps with batch size 100)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Fingerprint dimensionality is exactly 4096 for every molecule — verify vector length before batching into the MLP
- Predicted spectrum is a well-formed 1000-bin vector with no nonzero intensities at m/z values exceeding the molecule's molecular mass
- Training loss (modified MSE weighted per eq 1) decreases monotonically enough over 100,000 Adam steps at batch size 100 to indicate convergence
- Full-model library-matching recall@10 approaches the reported 91.8% when the ECFP-featurized forward head is combined with bidirectional (forward+reverse) prediction
- Molecules with repeated identical fragments should show correspondingly higher fingerprint counts than their binary-ECFP counterparts, confirming the additive (count) mode is active

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments
- Featurization is tied to the EI-trained MLP; retraining is needed to predict spectra for other ionization methods
- Fingerprint radius of 2 and length of 4096 were tuned via Vizier for this task; other molecular-representation tasks may need different settings
- Graph-convolutional or bond-centered molecular representations would likely improve predictive accuracy over fixed ECFPs

## Evidence

- [readme] README quickstart invocation of prediction script: [evidence span withheld — anchor status: not_found]
- [readme] Required packages in README: [evidence span withheld — anchor status: external_doc]
