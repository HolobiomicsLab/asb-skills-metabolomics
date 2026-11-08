---
name: hdf5-model-loading
description: Use when a task requires inspecting, reconstructing, or serving NPClassifier's
  pretrained Pathway/Superclass/Class networks and the models must first be fetched
  and loaded from HDF5 files.
when_to_use_negative:
- When the goal is to train a new classifier from SMILES data rather than load the
  pretrained released models
- When a multitask model with three simultaneous output heads is required — NPClassifier
  releases three separate single-task networks instead
- When the available TensorFlow version differs from 2.3.0 and HDF5 keras model conversion
  compatibility cannot be guaranteed
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3360
tools:
- name: TensorFlow 2.3.0
  role: Required version used to convert the keras models into HDF5 TF2 models and
    to load each HDF5 file for layer inspection
  repo: github.com/mwang87/NP-Classifier
- name: NPClassifier
  role: Source of the three pretrained Pathway/Superclass/Class DNN models being downloaded
    and loaded
  repo: github.com/mwang87/NP-Classifier
- name: Python
  role: Runtime for running get_models.sh prerequisites and the Keras loading/inspection
    code
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:affa1ea0e49c24660907cf85a689d73fb2c1eb6b7cd27a22c6fbce754364ec3d
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# hdf5-model-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Download and load NPClassifier's three released single-task DNN models (Pathway, Superclass, Class) as HDF5 files converted for TensorFlow 2.3.0/Keras, so their layer structures can be inspected or served. This is the entry point for any architecture reconstruction or model deployment work on the NP-Classifier repository.

## When to use

Use when a task requires inspecting, reconstructing, or serving NPClassifier's pretrained Pathway/Superclass/Class networks and the models must first be fetched and loaded from HDF5 files.

## When NOT to use

- When the goal is to train a new classifier from SMILES data rather than load the pretrained released models
- When a multitask model with three simultaneous output heads is required — NPClassifier releases three separate single-task networks instead
- When the available TensorFlow version differs from 2.3.0 and HDF5 keras model conversion compatibility cannot be guaranteed

## Inputs

- get_models.sh download script from Classifier/models_folder/models in mwang87/NP-Classifier
- Three released NPClassifier keras model files (Pathway, Superclass, Class) in HDF5 TF2 format
- Python environment with tensorflow 2.3.0 installed

## Outputs

- Three loaded Keras model objects, one per ontology level (Pathway, Superclass, Class)
- Per-network layer inventory: input layer shape and names (input_2048/input_4096), hidden layers, dropout placement, and output layer sizes

## How to apply

Run the repository's get_models.sh script inside Classifier/models_folder/models to fetch the three released NPClassifier model files (one for each of the three classification levels: Pathway, Superclass, and Class). Ensure python is installed with tensorflow version 2.3.0, which is required to convert the keras models into HDF5 TF2 models. Load each HDF5 file with the TensorFlow 2.3.0/Keras API and enumerate its layers, recording the input layer shape (input layers should be named 'input_2048' and 'input_4096' for the two fingerprint widths), the hidden fully connected layers with ReLU activation, batch normalization, and dropout, and the output layer (named 'output'). NPClassifier ships three separate single-task networks rather than one multitask model, so each file must be loaded and validated independently against the ontology dimensions (7 pathways, 70 superclasses, 672 classes).

## Related tools

- **TensorFlow 2.3.0** (Required version used to convert the keras models into HDF5 TF2 models and to load each HDF5 file for layer inspection) — github.com/mwang87/NP-Classifier
- **NPClassifier** (Source of the three pretrained Pathway/Superclass/Class DNN models being downloaded and loaded) — github.com/mwang87/NP-Classifier
- **Python** (Runtime for running get_models.sh prerequisites and the Keras loading/inspection code)

## Examples

```
cd Classifier/models_folder/models && sh ./get_models.sh
```

## Evaluation signals

- get_models.sh completes and produces three HDF5 model files, one each for Pathway, Superclass, and Class
- Each loaded Keras model exposes input layers named 'input_2048' and/or 'input_4096' and an output layer named 'output'
- Output layer sizes match the ontology dimensions: 7 pathways, 70 superclasses, 672 classes
- Layer enumeration confirms feed-forward architecture with ReLU-activated hidden layers, batch normalization, and dropout on fully connected layers
- Models load cleanly under tensorflow 2.3.0 without conversion errors

## Limitations

- Conversion to HDF5 TF2 format explicitly requires tensorflow version 2.3.0; other versions may fail or alter the models
- If the model input names change, the serving code must be updated — the README notes 'If the model input names change, then we need to change it in the code'
- Models are separate single-task networks per hierarchy level; loading one file yields only one level's predictions

## Evidence


Output layer's name should be "output""
