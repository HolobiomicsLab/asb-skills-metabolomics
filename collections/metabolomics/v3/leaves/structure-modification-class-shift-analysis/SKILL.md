---
name: structure-modification-class-shift-analysis
description: Use when you need to test if a structure-based classifier's predictions
  are causally driven by specific functional groups, by modifying those groups in
  a known molecule and observing label changes.
when_to_use_negative:
- Input structures already have curated class assignments (e.g., from the Dictionary
  of Natural Products) and the goal is labeling rather than probing feature dependence
  of the model
- The goal is a benchmark comparison of classifier platforms (NPClassifier vs. ClassyFire)
  rather than a structural-perturbation interpretation experiment
- Input is analytical data such as MS/MS spectra rather than molecular structures
  expressible as SMILES/fingerprints
tools:
- name: NPClassifier
  role: Deep neural network whose released models are interrogated with perturbed
    structures to observe class shifts
  repo: github.com/mwang87/NP-Classifier
- name: TensorFlow 2.3.0
  role: Converts the Keras models into HDF5 TF2 models and runs the feed-forward networks
    on the perturbed-structure fingerprints
- name: Python
  role: Language used to encode fingerprints and run the classification networks
- name: docker-compose
  role: Optionally brings up a local dockerized NPClassifier server exposing the /classify
    endpoint for structure classification
  repo: github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:bf37408fc1d17d33a1c0342f355012e4e0b43ef3d1e2134c8572849a4105c2b7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mwang87/NP-Classifier
  tools:
  - NPClassifier
  - Python
  - TensorFlow 2.3.0
  - docker-compose
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# structure-modification-class-shift-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Probes whether a deep-learning structural classifier (NPClassifier) genuinely depends on class-defining structural features by systematically perturbing key moieties (e.g., ester-to-amide in a cyclic depsipeptide, 4-keto to 3-hydroxy-4-keto in a flavanone) and checking whether predicted class labels shift as expected.

## When to use

Use when you need to test if a structure-based classifier's predictions are causally driven by specific functional groups, by modifying those groups in a known molecule and observing label changes.

## When NOT to use

- Input structures already have curated class assignments (e.g., from the Dictionary of Natural Products) and the goal is labeling rather than probing feature dependence of the model
- The goal is a benchmark comparison of classifier platforms (NPClassifier vs. ClassyFire) rather than a structural-perturbation interpretation experiment
- Input is analytical data such as MS/MS spectra rather than molecular structures expressible as SMILES/fingerprints

## Inputs

- SMILES strings of parent natural product structures and their structurally perturbed analogs (e.g., depsipeptide ester-to-amide replacement, flavanone keto-to-hydroxy variants)
- NPClassifier released models (Keras converted to HDF5 TF2, fetched via Classifier/models_folder/models/get_models.sh)
- counted Morgan fingerprints of the perturbed structures (2048- and 4096-bit inputs)

## Outputs

- Table of predicted NPClassifier labels (three hierarchy levels) for each perturbed structure
- Interpretation of whether class-defining structural moieties drive the predicted class (expected shifts: cyclic peptides, dihydroflavonols, flavan-3-ols)

## How to apply

Load the released NPClassifier models from github.com/mwang87/NP-Classifier, converting the Keras models into HDF5 TF2 models with tensorflow 2.3.0. Design minimally perturbed structures that remove or replace a hypothesized class-defining moiety (cyclic depsipeptide with ester bonds replaced by amide bonds; flavanone with 4-keto changed to 3-hydroxy-4-keto; dihydroflavonol with 3-hydroxy-4-keto changed to a simple 3-hydroxy group). Encode each perturbed structure as a counted Morgan fingerprint and pass each fingerprint through the three feed-forward neural networks that classify at the three levels of hierarchy (Pathway, Class, and lower level), which use binary encoding of categories and cosine scoring during training. Assemble a table mapping each perturbed structure to its predicted labels; the interpretation is validated if the shifts match the class-defining logic of the perturbation (cyclic peptides, dihydroflavonols, flavan-3-ols).

## Related tools

- **NPClassifier** (Deep neural network whose released models are interrogated with perturbed structures to observe class shifts) — github.com/mwang87/NP-Classifier
- **TensorFlow 2.3.0** (Converts the Keras models into HDF5 TF2 models and runs the feed-forward networks on the perturbed-structure fingerprints)
- **Python** (Language used to encode fingerprints and run the classification networks)
- **docker-compose** (Optionally brings up a local dockerized NPClassifier server exposing the /classify endpoint for structure classification) — github.com/mwang87/NP-Classifier

## Examples

```
curl 'http://localhost:8000/classify?smiles=<perturbed_structure_SMILES>'
```

## Evaluation signals

- Cyclic depsipeptide with ester bonds replaced by amide bonds is predicted as cyclic peptides
- Flavanone with 4-keto changed to 3-hydroxy-4-keto shifts prediction to dihydroflavonols
- The 3-hydroxy-4-keto variant changed to a simple 3-hydroxy group shifts prediction to flavan-3-ols
- Model serving check passes: input layer names are 'input_2048' and 'input_4096' and output layer name is 'output' via the /model/metadata endpoint
- Top-level predicted labels match the class shifts stated in the Interpretation of Models section of the paper

## Limitations

- Model conversion requires a pinned tensorflow version: 'Make sure you have python installed and tensorflow version 2.3.0 installed to convert the keras models into HDF5 TF2 models.'
- The serving integration is brittle to architecture changes: 'If the model input names change, then we need to change it in the code'
- Interpretation is limited to the specific perturbations tested (one depsipeptide and two flavonoid variants); other class-defining moieties would require analogous perturbation designs

## Evidence

- [discussion] ester-to-amide shifts class: [evidence span withheld — anchor status: not_found]
