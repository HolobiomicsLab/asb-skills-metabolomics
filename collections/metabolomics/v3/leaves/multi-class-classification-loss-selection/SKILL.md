---
name: multi-class-classification-loss-selection
description: Use when a trained Keras/TensorFlow classifier has a sigmoid output layer
  with two classes (e.g. massNet's normal/tumor head) but the training label vector
  contains K > 2 class labels, and a probabilistic multi-class output is required.
when_to_use_negative:
- The task remains the paper's binary normal/tumor delineation problem — the original
  sigmoid + binary cross-entropy configuration is already adequate and should not
  be changed.
- The classifier consumes raw m/z spectra directly rather than the VAE 5-dimensional
  Code layer — this skill modifies only the output head, not the feature representation.
- There is no multi-class ground-truth annotation available; the article notes that
  annotating MSI data from histology alone is an approximation and that ground-truth
  establishment is the challenging task for other tissue and tumor types.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: massNet
  role: Deep learning model whose classifier module (sigmoid, 2-class output) is extended
    to multi-class softmax classification
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Platform in which the classifier architecture is defined, compiled with the
    categorical cross-entropy loss, and serialized to JSON
- name: tensorflow
  role: Backend used to train and run the compiled Keras model
- name: Python
  role: Implementation language for the architecture modification and serialization
    workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:3b280793041065dd77e50fe1b3afbef74c5c18d6827482e21dc7851fa8876abb
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# multi-class-classification-loss-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Adapts the massNet classifier — originally a Keras/TensorFlow model with two fully connected hidden layers and a 2-class sigmoid output — to multi-class probabilistic classification by switching the output layer to a K-unit softmax and replacing binary cross-entropy with categorical cross-entropy.

## When to use

Use when a trained Keras/TensorFlow classifier has a sigmoid output layer with two classes (e.g. massNet's normal/tumor head) but the training label vector contains K > 2 class labels, and a probabilistic multi-class output is required.

## When NOT to use

- The task remains the paper's binary normal/tumor delineation problem — the original sigmoid + binary cross-entropy configuration is already adequate and should not be changed.
- The classifier consumes raw m/z spectra directly rather than the VAE 5-dimensional Code layer — this skill modifies only the output head, not the feature representation.
- There is no multi-class ground-truth annotation available; the article notes that annotating MSI data from histology alone is an approximation and that ground-truth establishment is the challenging task for other tissue and tumor types.

## Inputs

- massNet classifier source code (two fully connected hidden layers plus a 2-class sigmoid output layer) from github.com/wabdelmoula/massNet
- Training label vector y with K class labels (paper's binary case: 10,024 normal and 3,644 tumor spectra)
- VAE latent variable input: 5-dimensional Code layer activations from the massNet VAE trained on TIC-normalized MALDI FT-ICR MSI data (85,062 m/z variables)

## Outputs

- massnet_multiclass_model.json — modified Keras model architecture with a K-unit softmax output layer
- Compiled Keras/TensorFlow massNet model configured with categorical cross-entropy loss instead of binary cross-entropy

## How to apply

Load the massNet source from github.com/wabdelmoula/massNet and locate the classifier module: two fully connected hidden layers whose output layer currently uses a sigmoid activation with two classes (normal and tumor). Determine K from the training label vector (K=2 in the paper's binary task, but the code must generalize to arbitrary K). Replace the sigmoid activation with a softmax activation whose dimensionality matches K, and swap the binary cross-entropy loss for categorical cross-entropy — the paper's loss function choice is the paired decision with the activation change. Instantiate the modified classifier on the VAE latent variable input (the 5-dimensional Code layer captured by the massNet VAE, which reconstructs TIC-normalized MSI data with MSE 5.69 × 10^-8) and compile the full model in Keras with a TensorFlow backend. Save the modified architecture to massnet_multiclass_model.json and verify the saved JSON contains an output layer with softmax activation and exactly K units.

## Related tools

- **massNet** (Deep learning model whose classifier module (sigmoid, 2-class output) is extended to multi-class softmax classification) — https://github.com/wabdelmoula/massNet
- **Keras** (Platform in which the classifier architecture is defined, compiled with the categorical cross-entropy loss, and serialized to JSON)
- **tensorflow** (Backend used to train and run the compiled Keras model)
- **Python** (Implementation language for the architecture modification and serialization workflow)

## Examples

```
model.compile(optimizer='adam', loss='categorical_crossentropy'); model.save('massnet_multiclass_model.json')
```

## Evaluation signals

- The saved massnet_multiclass_model.json contains an output layer with exactly K units (K=2 for the paper's binary task) and softmax activation.
- The compiled model's loss function is categorical cross-entropy, replacing the original binary cross-entropy.
- The model's input dimension matches the 5-dimensional Code layer of the VAE.
- Forward-pass outputs are K-class probability distributions (non-negative, summing to 1) rather than independent sigmoid probabilities per class.
- Architecture round-trips through Keras model_from_json without errors using the README-pinned environment (Keras 2.2.0, TensorFlow 1.8.0, Python 3.6.12).

## Limitations

- The paper only demonstrated binary class classification; multi-class classification is explicitly left for future development, so no published multi-class results or hyperparameters exist to validate against.
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article's discussion; only the README pins versions (Keras 2.2.0, TensorFlow 1.8.0), which are outdated and may complicate reproduction.
- Ground-truth multi-class annotation derived solely from annotated histology is an approximation that may not be optimal for intra-tumor heterogeneity.

## Evidence

- [results] The two output layers of the massNet architecture, including the class-prediction output layer, were activated using the sigmoid function.: [evidence span withheld — anchor status: not_found]
