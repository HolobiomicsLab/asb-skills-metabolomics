---
name: sigmoid-to-softmax-output-layer-replacement
description: Use when extending a trained binary deep-learning classifier (e.g. massNet's
  sigmoid output layer with two classes, normal and tumor) to multi-class tissue labels
  where the output layer must produce a K-class probabilistic estimate.
when_to_use_negative:
- 'The task remains strictly binary (normal vs. tumor): the original sigmoid output
  layer already performs probabilistic classification and needs no change.'
- The classifier is fed raw mass spectral data or preprocessed peak tables rather
  than the VAE's 5-dimensional Code-layer latent variable, changing the interface
  this modification assumes.
- The multi-class labels are not mutually exclusive per pixel (softmax assumes single-label
  assignment), e.g. mixed intra-tumor heterogeneity annotations.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: 'Source architecture being modified: its second module performs probabilistic
    classification and is activated with a sigmoid function over two classes'
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Framework in which the output layer activation and loss are redefined (softmax,
    categorical_crossentropy) and the model is compiled
  repo: https://github.com/wabdelmoula/massNet
- name: tensorflow
  role: Backend on which the modified multi-class classifier is compiled and executed
- name: Python
  role: Implementation language used to edit the model definition and save the architecture
    JSON
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:0a29243f486b2434c294c9cfb2762d565f3ce1e1dfcd431516f82a063f3db7e7
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sigmoid-to-softmax-output-layer-replacement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generalizes a binary probabilistic classifier such as massNet's two-class output layer to arbitrary K classes by replacing the sigmoid activation with a K-unit softmax and swapping binary cross-entropy for categorical cross-entropy, enabling multi-class MSI tissue labeling beyond the paper's normal/tumor task.

## When to use

Use when extending a trained binary deep-learning classifier (e.g. massNet's sigmoid output layer with two classes, normal and tumor) to multi-class tissue labels where the output layer must produce a K-class probabilistic estimate.

## When NOT to use

- The task remains strictly binary (normal vs. tumor): the original sigmoid output layer already performs probabilistic classification and needs no change.
- The classifier is fed raw mass spectral data or preprocessed peak tables rather than the VAE's 5-dimensional Code-layer latent variable, changing the interface this modification assumes.
- The multi-class labels are not mutually exclusive per pixel (softmax assumes single-label assignment), e.g. mixed intra-tumor heterogeneity annotations.

## Inputs

- massNet source code from github.com/wabdelmoula/massNet (Keras model definition)
- Training data label vector defining the class-label count K (e.g. normal and tumor spectra from MALDI FT-ICR MSI PDX GBM data)
- 5-dimensional latent variable from the VAE Code layer, itself trained on 85,062 m/z variables of TIC-normalized imzML/HDF5 MSI data

## Outputs

- Modified Keras/TensorFlow classifier with a K-unit softmax output layer and categorical cross-entropy loss
- Saved architecture JSON file massnet_multiclass_model.json

## How to apply

Load the massNet source from github.com/wabdelmoula/massNet and locate the classifier module (two fully connected hidden layers and an output layer of two classes). Determine the number of class labels K from the training label vector (K=2 for the paper's normal/tumor task), then replace the sigmoid output activation with a softmax activation of K units and swap the binary cross-entropy loss for categorical cross-entropy. Instantiate the modified classifier on the VAE latent variable input — the 5-dimensional Code layer — and compile it in Keras/TensorFlow, noting the README-pinned versions Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0 backend. Save the modified architecture to massnet_multiclass_model.json for verification.

## Related tools

- **massNet** (Source architecture being modified: its second module performs probabilistic classification and is activated with a sigmoid function over two classes) — https://github.com/wabdelmoula/massNet
- **Keras** (Framework in which the output layer activation and loss are redefined (softmax, categorical_crossentropy) and the model is compiled) — https://github.com/wabdelmoula/massNet
- **tensorflow** (Backend on which the modified multi-class classifier is compiled and executed)
- **Python** (Implementation language used to edit the model definition and save the architecture JSON)

## Examples

```
from tensorflow.keras.layers import Dense; out = Dense(K, activation='softmax'); model.compile(loss='categorical_crossentropy', optimizer='adam')
```

## Evaluation signals

- Saved massnet_multiclass_model.json contains an output layer with softmax activation.
- The output layer has exactly K units, equal to the number of class labels (2 for the paper's binary normal/tumor task).
- Model compilation uses categorical cross-entropy loss instead of binary cross-entropy.
- The classifier input dimensionality matches the 5-dimensional VAE Code-layer latent variable.
- Model still regularizes via batch normalization and dropout as in the original massNet design.

## Limitations

- The published massNet model only demonstrated binary class classification; multi-class classification was explicitly left for future development.
- Ground truth annotation based solely on annotated histology is an approximation that may be suboptimal for intra-tumor heterogeneity, which complicates multi-class label vectors.
- No software version numbers or hyperparameter values for the trained massNet model are reported beyond the README pins (Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0).

## Evidence

- [supplementary] sigmoid output layer of massNet: [evidence span withheld — anchor status: not_found]
