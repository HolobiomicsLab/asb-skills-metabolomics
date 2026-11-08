---
name: ontology-output-size-verification
description: Use when you have downloaded released .h5 model files for an ontology-based
  classifier and need to confirm each network's output size matches the number of
  categories at its classification level before reuse or retraining.
when_to_use_negative:
- Your input is raw chemical structures or SMILES strings to be classified via the
  /classify?smiles=<> API rather than model files whose architecture must be audited.
- The target is a genuine multitask model with three simultaneous output heads on
  a single network — NPClassifier instead uses three separate single-task networks,
  so one-output-head-per-file checks would misfire.
- No reference ontology category counts are available, so there is no ground truth
  to compare output layer sizes against.
tools:
- name: TensorFlow 2.3.0
  role: Used to load the HDF5 Keras models and enumerate input/hidden/output layer
    names and shapes for size verification
- name: Python
  role: Programming language in which the models are inspected and the architecture
    summaries are compiled
- name: NPClassifier
  role: Source of the released Pathway, Superclass, and Class model files being verified
  repo: github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:8d8afe84b0cba24ae8cfc0330fe764a8ffddfa77da7d39406495dcee5964aa62
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ontology-output-size-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies that each released deep learning classifier's output layer dimension exactly matches its ontology level's category count (for NPClassifier: 7 Pathways, 70 Superclasses, 672 Classes), by enumerating layers of the downloaded HDF5 models with the TensorFlow 2.3.0/Keras API.

## When to use

Use when you have downloaded released .h5 model files for an ontology-based classifier and need to confirm each network's output size matches the number of categories at its classification level before reuse or retraining.

## When NOT to use

- Your input is raw chemical structures or SMILES strings to be classified via the /classify?smiles=<> API rather than model files whose architecture must be audited.
- The target is a genuine multitask model with three simultaneous output heads on a single network — NPClassifier instead uses three separate single-task networks, so one-output-head-per-file checks would misfire.
- No reference ontology category counts are available, so there is no ground truth to compare output layer sizes against.

## Inputs

- Released NPClassifier HDF5 model files (.h5) for Pathway, Superclass, and Class levels obtained via get_models.sh
- Ontology category counts per classification level (7 Pathways, 70 Superclasses, 672 Classes)
- TensorFlow 2.3.0 / Keras environment

## Outputs

- Per-network layer inventory (input shape, hidden layer count/activations, dropout placement, output size)
- Structured model_architecture_summary.json containing one architecture summary per classification level
- Pass/fail flag per network indicating whether output layer size matches the ontology dimension

## How to apply

Download the released NPClassifier models (Pathway, Superclass, Class) as HDF5 by running the repository's get_models.sh script under Classifier/models_folder/models, which requires TensorFlow 2.3.0 installed for converting the Keras models into HDF5 TF2 models. Load each .h5 file with the TensorFlow 2.3.0/Keras API and enumerate its layers, recording the input shape (input names should be input_2048 and input_4096 for the counted Morgan fingerprints), the three hidden layers with ReLU activation and batch normalization, dropout placement on fully connected layers, and the output layer size (the output layer's name should be 'output'). Check that each network is a single-task model with one dedicated output head, since NPClassifier uses three separate feed-forward networks rather than a multitask model with three simultaneous output heads. Confirm the recorded output sizes equal the ontology dimensions — 7 Pathways, 70 Superclasses, 672 Classes — and save one structured architecture summary per network (e.g., as model_architecture_summary.json). Any mismatch signals a misconverted model file, a wrong ontology version, or a corrupted download.

## Related tools

- **TensorFlow 2.3.0** (Used to load the HDF5 Keras models and enumerate input/hidden/output layer names and shapes for size verification)
- **Python** (Programming language in which the models are inspected and the architecture summaries are compiled)
- **NPClassifier** (Source of the released Pathway, Superclass, and Class model files being verified) — github.com/mwang87/NP-Classifier

## Examples

```
cd Classifier/models_folder/models && sh ./get_models.sh
```

## Evaluation signals

- Output layer sizes of the three loaded networks equal 7 (Pathway), 70 (Superclass), and 672 (Class).
- Input layer names are 'input_2048' and 'input_4096' and the output layer name is 'output', as specified in the repository README.
- Exactly three separate HDF5 files are found and loaded (one per ontology level), each with a single output head, not a multitask model with three heads.
- Each network's hidden layers use ReLU activation with batch normalization and dropout on the fully connected layers.
- The three compiled summaries in model_architecture_summary.json each contain a non-empty layer inventory with an explicitly recorded output size.

## Limitations

- Model conversion requires a specific environment: tensorflow version 2.3.0 must be installed to convert the Keras models into HDF5 TF2 models, so version drift can break loading.
- If the model input names change, code that references them must be updated accordingly, per the README's metadata-checking note.
- Verification only confirms layer sizes/names match the ontology; it does not validate prediction quality, which requires the stratified test split (20%) used in the article.

## Evidence


Output layer's name should be "output""
