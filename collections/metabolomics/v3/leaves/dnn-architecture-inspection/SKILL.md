---
name: dnn-architecture-inspection
description: Use when released model files (e.g., HDF5/Keras checkpoints) exist for
  a paper's DNN and you must reconstruct per-network architecture details — layer
  shapes, activations, dropout, output sizes — to answer what network design was adopted.
when_to_use_negative:
- The paper only reports architecture in prose and no model files or checkpoints were
  released — report from text instead of inspecting binaries.
- The task is to classify new SMILES structures via the /classify?smiles=<> API rather
  than to audit the model architecture itself.
- You need hyperparameters (hidden layer units, learning rates, dropout rate chosen
  by Keras Tuner Hyperband) but only the trained weights are available without training
  logs — inspection reveals structure, not the tuning search.
edam_operation: http://edamontology.org/operation_3939
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow 2.3.0
  role: Framework used to load the HDF5 keras models and enumerate layers; README
    pins tensorflow 2.3.0 for converting keras models into HDF5 TF2 models
- name: NPClassifier
  role: Subject model whose three feed-forward networks (Pathway, Superclass, Class)
    are inspected
  repo: github.com/mwang87/NP-Classifier
- name: docker-compose
  role: 'Optional: bring up the dockerized TensorFlow serving stack to check model
    metadata via the /model/metadata endpoint and confirm input/output layer names'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:8b1314061960525775730b77254051bfa92d89c55bcdf27d790f8c1b632a8a42
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
  edam_operation: http://edamontology.org/operation_3939
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# dnn-architecture-inspection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the layer-by-layer architecture of a released deep neural network (here, NPClassifier's three feed-forward networks for Pathway, Superclass, and Class) by downloading the model files, loading them with Keras/TensorFlow, and enumerating layers to verify design details such as hidden-layer activations, dropout, and output dimensions.

## When to use

Use when released model files (e.g., HDF5/Keras checkpoints) exist for a paper's DNN and you must reconstruct per-network architecture details — layer shapes, activations, dropout, output sizes — to answer what network design was adopted.

## When NOT to use

- The paper only reports architecture in prose and no model files or checkpoints were released — report from text instead of inspecting binaries.
- The task is to classify new SMILES structures via the /classify?smiles=<> API rather than to audit the model architecture itself.
- You need hyperparameters (hidden layer units, learning rates, dropout rate chosen by Keras Tuner Hyperband) but only the trained weights are available without training logs — inspection reveals structure, not the tuning search.

## Inputs

- Released model checkpoint files (HDF5, TF2-compatible Keras models)
- Repository download script (e.g., get_models.sh) and README metadata on layer naming
- Paper-reported ontology dimensions (number of output classes per classification level)

## Outputs

- Per-network layer enumeration (input shape, hidden layers, activations, dropout placement, output size)
- model_architecture_summary.json compiling one summary per network
- Verified architecture claim (e.g., three single-task networks vs. multitask design)

## How to apply

First fetch the released model files using the repository's download script (e.g., run `sh ./get_models.sh` in Classifier/models_folder/models), confirming the environment has tensorflow 2.3.0 installed since the README requires it to convert keras models into HDF5 TF2 models. Load each HDF5 file with the TensorFlow/Keras API and enumerate its layers, recording input layer shapes (input layer names should be 'input_2048' and 'input_4096' per the README, with output layer name 'output'). For NPClassifier specifically, document the three separate single-task feed-forward networks — Pathway, Superclass, and Class — noting hidden layers with ReLU activation, batch normalization, and dropout applied to fully connected layers. Cross-check each network's output layer size against the ontology dimensions (7 pathways, 70 superclasses, 672 classes); a mismatch flags an incorrect or stale model file. Compile one structured summary per network (e.g., model_architecture_summary.json) and note whether the multitask alternative (three output heads predicting all levels simultaneously) was rejected in favor of single-task networks.

## Related tools

- **TensorFlow 2.3.0** (Framework used to load the HDF5 keras models and enumerate layers; README pins tensorflow 2.3.0 for converting keras models into HDF5 TF2 models)
- **NPClassifier** (Subject model whose three feed-forward networks (Pathway, Superclass, Class) are inspected) — github.com/mwang87/NP-Classifier
- **docker-compose** (Optional: bring up the dockerized TensorFlow serving stack to check model metadata via the /model/metadata endpoint and confirm input/output layer names)

## Examples

```
cd Classifier/models_folder/models && sh ./get_models.sh
```

## Evaluation signals

- Enumerated output layer sizes equal 7, 70, and 672 for Pathway, Superclass, and Class networks respectively
- Input layer names match the README expectation: 'input_2048' and 'input_4096'; output layer name is 'output'
- Three distinct model files load independently as single-task networks, consistent with the finding that NPClassifier uses three separate networks rather than one multitask model
- Each hidden fully connected layer shows dropout applied, matching the paper's reported regularization strategy
- /model/metadata endpoint (via tensorflow serving) returns metadata consistent with the locally inspected HDF5 layer names

## Limitations

- Architecture inspection requires the exact TensorFlow version (2.3.0) the authors used for HDF5 conversion; other versions may fail to load the keras models.
- Inspection reveals layer structure but not training provenance details such as hyperparameter search results from Keras Tuner Hyperband or the merged train+validation final-training step.
- Layer names in the model may have changed, in which case tensorflow-serving pass-through code must be updated (per README).

## Evidence

- [discussion] three separate single-task feed-forward networks documented: "The DNN for the NPClassifier was composed of three different networks that classified the molecular structure at the three levels of hierarchy with feed-forward neural network architecture"
- [discussion] dropout regularization placement verified: "Dropout was applied to the fully connected layers to improve generalization"
- [readme] environment pinning required to load HDF5 models: "Make sure you have python installed and tensorflow version 2.3.0 installed to convert the keras models into HDF5 TF2 models."
- [readme] layer naming invariants for verification: "Input layers' names should be "input_2048" and "input_4096""
- [readme] model metadata check endpoint for cross-validation: "We pass through tensorflow serving at this url: ```/model/metadata```"
