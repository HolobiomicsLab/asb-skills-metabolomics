---
name: feedforward-layer-enumeration
description: Use when you need to reconstruct, verify, or document the layer-by-layer
  architecture of released HDF5 Keras/TF2 model files (e.g., NPClassifier's three
  single-task networks) before reimplementation or reuse.
when_to_use_negative:
- You only need to classify a SMILES string — use the deployed /classify?smiles=<>
  API instead of loading and inspecting model files
- The models are multitask architectures with shared output heads — NPClassifier was
  built as three separate single-task feed-forward networks, so a single shared-layer
  enumeration does not apply
- The model files are not HDF5 TF2 exports (e.g., PyTorch checkpoints or TF1 frozen
  graphs) — the Keras model.layers enumeration path requires TF 2.3.0-compatible HDF5
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow 2.3.0
  role: Framework used to load the HDF5 Keras models and enumerate their layers for
    the architecture reconstruction
  repo: github.com/mwang87/NP-Classifier
- name: Python
  role: Programming language in which the models are converted and loaded (get_models.sh
    requires python with tensorflow 2.3.0)
- name: NPClassifier
  role: Source of the three released feed-forward network model files (Pathway, Superclass,
    Class) being enumerated
  repo: github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:4eae1701b61f8115da79624134d35935aa509898367c35e636d1a3ea8ed03636
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
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# feedforward-layer-enumeration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the internal architecture of released Keras/TensorFlow DNN model files by loading each model, enumerating its layers, and recording input shapes, hidden-layer activations, dropout placement, and output sizes. In NPClassifier this establishes that Pathway, Superclass, and Class prediction are three separate single-task feed-forward networks rather than one multitask model.

## When to use

Use when you need to reconstruct, verify, or document the layer-by-layer architecture of released HDF5 Keras/TF2 model files (e.g., NPClassifier's three single-task networks) before reimplementation or reuse.

## When NOT to use

- You only need to classify a SMILES string — use the deployed /classify?smiles=<> API instead of loading and inspecting model files
- The models are multitask architectures with shared output heads — NPClassifier was built as three separate single-task feed-forward networks, so a single shared-layer enumeration does not apply
- The model files are not HDF5 TF2 exports (e.g., PyTorch checkpoints or TF1 frozen graphs) — the Keras model.layers enumeration path requires TF 2.3.0-compatible HDF5

## Inputs

- Three HDF5 NPClassifier model files (Pathway, Superclass, Class) downloaded via get_models.sh from github.com/mwang87/NP-Classifier
- Python environment with tensorflow version 2.3.0 installed

## Outputs

- model_architecture_summary.json containing one structured layer inventory per network (input shape, hidden layers with ReLU/batch normalization, dropout placement, output layer size)
- Verified output-dimension mapping: 7 pathways, 70 superclasses, 672 classes

## How to apply

Download the released model files by running the repository's get_models.sh script (Classifier/models_folder/models), ensuring Python with TensorFlow 2.3.0 is available to convert/load the Keras models as HDF5 TF2 models. Load each of the three HDF5 files (Pathway, Superclass, Class) through the TensorFlow 2.3.0/Keras API and enumerate its layers, recording the input layer shape (input names 'input_2048' and 'input_4096' reflect the counted Morgan fingerprint dimensions), the three hidden feed-forward layers with ReLU activation and batch normalization, dropout on the fully connected layers, and the output layer size. Use the output layer sizes as a verification gate against the ontology dimensions: 7 pathways, 70 superclasses, and 672 classes. Compile one structured summary per network and save all three as model_architecture_summary.json so the three-single-task-networks design (versus a multitask three-output-head alternative) is explicitly documented.

## Related tools

- **TensorFlow 2.3.0** (Framework used to load the HDF5 Keras models and enumerate their layers for the architecture reconstruction) — github.com/mwang87/NP-Classifier
- **Python** (Programming language in which the models are converted and loaded (get_models.sh requires python with tensorflow 2.3.0))
- **NPClassifier** (Source of the three released feed-forward network model files (Pathway, Superclass, Class) being enumerated) — github.com/mwang87/NP-Classifier

## Examples

```
cd Classifier/models_folder/models && sh ./get_models.sh && python -c "import tensorflow as tf; m=tf.keras.models.load_model('pathway.hdf5'); [print(l.name, l.output_shape) for l in m.layers]"
```

## Evaluation signals

- Each enumerated network has exactly three hidden layers with ReLU activation and batch normalization, with dropout applied to the fully connected layers
- Output layer sizes match the ontology dimensions: 7 for Pathway, 70 for Superclass, 672 for Class
- Input layer names are 'input_2048' and 'input_4096' (counted Morgan fingerprint input dimensions) and the output layer name is 'output'
- Three separate summaries are produced — one per single-task network — and all three are persisted in model_architecture_summary.json
- The summarized architecture matches the paper's description of three distinct networks classifying the three hierarchy levels, not one multitask model

## Limitations

- If the model input names change, code that references them (e.g., tensorflow serving metadata at /model/metadata) must be updated, so enumeration results are version-sensitive
- Converting Keras models to HDF5 requires exactly tensorflow version 2.3.0; other versions may fail to load the files
- Layer enumeration alone does not capture hyperparameters (hidden layer units, learning rates, regularization, dropout rate) that were optimized with Keras Tuner's Hyperband algorithm

## Evidence
