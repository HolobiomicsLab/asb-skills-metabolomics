---
name: keras-tensorflow-model-compilation
description: Use when building or modifying a Keras/TensorFlow classification model
  — e.g. extending massNet's two-class normal/tumor output layer to K>1 classes via
  softmax and categorical cross-entropy.
when_to_use_negative:
- The task is the paper's original binary normal/tumor delineation — keep the sigmoid
  output with two classes and binary cross-entropy rather than softmax/categorical
  cross-entropy
- Input spectra have not been converted to the HDF5 format expected for deep learning
  analysis (e.g. still raw imzML from SCiLS lab 2020a)
- The goal is spectral preprocessing or peak picking, which massNet explicitly avoids
  performing prior to classification
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: Keras
  role: Platform used to implement and compile the massNet architecture (classifier
    output layer, activation, and loss)
  repo: https://github.com/wabdelmoula/massNet
- name: tensorflow
  role: Backend behind Keras for the massNet deep learning model implementation
  repo: https://github.com/wabdelmoula/massNet
- name: Python
  role: Language in which the massNet architecture (VAE + classifier) was implemented
- name: massNet
  role: Deep learning architecture whose classifier module is instantiated and compiled;
    scalable probabilistic pixel-based classification of MSI spectra without prior
    peak picking
  repo: https://github.com/wabdelmoula/massNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:2a09ec960aad0cfeb4cc8fa67fabb7ee8a8c22e768ec75a4abc02e0a62af2f27
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - Keras
  - Python
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# keras-tensorflow-model-compilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Instantiate and compile a deep learning model (here, the massNet classifier) in Keras with a TensorFlow backend, matching output-layer activation (sigmoid/softmax) and loss (binary vs categorical cross-entropy) to the number of class labels in the MSI training data.

## When to use

Use when building or modifying a Keras/TensorFlow classification model — e.g. extending massNet's two-class normal/tumor output layer to K>1 classes via softmax and categorical cross-entropy.

## When NOT to use

- The task is the paper's original binary normal/tumor delineation — keep the sigmoid output with two classes and binary cross-entropy rather than softmax/categorical cross-entropy
- Input spectra have not been converted to the HDF5 format expected for deep learning analysis (e.g. still raw imzML from SCiLS lab 2020a)
- The goal is spectral preprocessing or peak picking, which massNet explicitly avoids performing prior to classification

## Inputs

- massNet source code from github.com/wabdelmoula/massNet (classifier module: two fully connected hidden layers + output layer)
- Training data class-label vector (K=2 for the paper's normal/tumor MALDI FT-ICR MSI spectra; 10,024 normal and 3,644 tumor training spectra)
- VAE latent variable input: 5-dimensional Code layer representation of the TIC-normalized MSI spectra

## Outputs

- Compiled Keras/TensorFlow classification model with an output layer of K units and softmax (or sigmoid for binary) activation
- Serialized model architecture JSON (e.g. massnet_multiclass_model.json) with categorical cross-entropy loss for multi-class or binary cross-entropy for the two-class task

## How to apply

Load the massNet source from github.com/wabdelmoula/massNet and locate the classifier module, which the paper defines as two fully connected hidden layers and an output layer of two classes activated with a sigmoid function. Determine the number of class labels K from the training label vector (K=2 for the paper's binary normal/tumor task on MALDI FT-ICR MSI data with 85,062 m/z variables). To generalize to multi-class, replace the sigmoid output activation with a softmax activation over K units and swap binary cross-entropy for categorical cross-entropy. Instantiate the classifier on the VAE 5-dimensional latent Code layer input and compile in Keras (2.2.0) with a TensorFlow backend (1.8.0 per the README), applying batch normalization and dropout for learning stabilization. Serialize the compiled architecture (e.g. to massnet_multiclass_model.json) and verify the output layer has K units with softmax activation before training. The paper reports the full pipeline was implemented in Python using the publicly available Keras and TensorFlow platforms.

## Related tools

- **Keras** (Platform used to implement and compile the massNet architecture (classifier output layer, activation, and loss)) — https://github.com/wabdelmoula/massNet
- **tensorflow** (Backend behind Keras for the massNet deep learning model implementation) — https://github.com/wabdelmoula/massNet
- **Python** (Language in which the massNet architecture (VAE + classifier) was implemented)
- **massNet** (Deep learning architecture whose classifier module is instantiated and compiled; scalable probabilistic pixel-based classification of MSI spectra without prior peak picking) — https://github.com/wabdelmoula/massNet

## Examples

```
from keras.models import model_from_json; model = model_from_json(open('massnet_multiclass_model.json').read()); model.compile(optimizer='adam', loss='categorical_crossentropy')
```

## Evaluation signals

- The saved architecture JSON contains an output layer with softmax activation and a number of units equal to the class-label count (2 for the paper's binary normal/tumor task)
- The compiled loss is categorical cross-entropy for multi-class (replacing binary cross-entropy)
- The classifier input dimensionality matches the 5-dimensional VAE Code (latent) layer
- Training converges stably with batch normalization and dropout regularization (the VAE module converged stably in 100 epochs, ~55.7 minutes, as a reference for pipeline stability)
- Model runs against the README-pinned stack: Python 3.6.12, Keras 2.2.0 with TensorFlow 1.8.0 backend

## Limitations

- The published model demonstrated binary class classification only; multi-class extension is explicitly left as future development
- No software version numbers or hyperparameter values for the trained massNet model are reported in the paper's discussion (versions are only in the README)
- Ground truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity
- No trained model weights availability statement is mentioned in the discussion section

## Evidence

- [full_text] Classifier output layer sigmoid activation: [evidence span withheld — anchor status: not_found]
