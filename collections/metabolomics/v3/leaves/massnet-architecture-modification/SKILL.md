---
name: massnet-architecture-modification
description: Use when adapting the massNet classifier to predict more than the paper's
  two tissue classes, i.e. replacing its sigmoid two-class output layer on the 5-dimensional
  VAE Code layer with a K-unit softmax head trained with categorical cross-entropy.
when_to_use_negative:
- The task remains the paper's original binary normal-vs-tumor classification, where
  the existing sigmoid two-class output layer is already appropriate.
- Input is raw MALDI FT-ICR mass spectra with 85,062 m/z variables rather than the
  VAE Code layer latent variables the classifier consumes.
- Ground-truth class labels beyond binary tumor/normal annotation are unavailable
  — the article notes that establishing ground-truth annotation of MSI data is itself
  the challenging task for other tissues and tumor types.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning model whose classifier module (two fully connected hidden layers,
    sigmoid output layer of two classes) is modified for multi-class softmax output
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Framework in which the modified classifier architecture is defined, compiled
    with categorical cross-entropy, and serialized to JSON
- name: tensorflow
  role: Backend on which the Keras model is executed when compiling and saving the
    multi-class architecture
- name: Python
  role: Implementation language for the modified model; README specifies Python 3.6.12
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:d09b70b50f46f66f0ac80e922ca7ac0084ca1cd61740f8a74462145f807fde71
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# massnet-architecture-modification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Modifies the massNet deep learning classifier—originally built as two fully connected hidden layers with a sigmoid output over two classes (normal and tumor)—to support K-class tissue classification by swapping the sigmoid for a softmax activation and binary cross-entropy for categorical cross-entropy.

## When to use

Use when adapting the massNet classifier to predict more than the paper's two tissue classes, i.e. replacing its sigmoid two-class output layer on the 5-dimensional VAE Code layer with a K-unit softmax head trained with categorical cross-entropy.

## When NOT to use

- The task remains the paper's original binary normal-vs-tumor classification, where the existing sigmoid two-class output layer is already appropriate.
- Input is raw MALDI FT-ICR mass spectra with 85,062 m/z variables rather than the VAE Code layer latent variables the classifier consumes.
- Ground-truth class labels beyond binary tumor/normal annotation are unavailable — the article notes that establishing ground-truth annotation of MSI data is itself the challenging task for other tissues and tumor types.

## Inputs

- massNet source code from github.com/wabdelmoula/massNet (classifier module: two fully connected hidden layers + sigmoid output layer)
- VAE latent variable features from the 5-dimensional Code layer of the optimized massNet model
- Training label vector with K class annotations (K=2 for normal/tumor in the paper, derived from pathologist-annotated H&E transferred to MSI data)

## Outputs

- massnet_multiclass_model.json — serialized Keras/TensorFlow model architecture with a K-unit softmax output layer
- Compiled multi-class massNet model using categorical cross-entropy loss

## How to apply

Load the massNet source from github.com/wabdelmoula/massNet and locate the classification module, which is defined as two fully connected hidden layers and an output layer of two classes activated with a sigmoid function. Determine the number of class labels K from the training data label vector (K=2 for the paper's binary normal/tumor task, with 10,024 normal and 3,644 tumor training spectra). Replace the sigmoid output activation with a softmax activation whose dimensionality matches K, and swap the binary cross-entropy loss for categorical cross-entropy. Instantiate the modified classifier on the VAE latent variable input (the 5-dimensional Code layer described in the paper's optimized model) and compile the full massNet model in Keras (2.2.0) with a TensorFlow (1.8.0) backend under Python 3.6.12. Save the modified architecture to massnet_multiclass_model.json and verify the output layer has K softmax units. The rationale is the paper's own stated limitation: the original model was only demonstrated for binary class classification, with multi-class extension left for future development.

## Related tools

- **massNet** (Deep learning model whose classifier module (two fully connected hidden layers, sigmoid output layer of two classes) is modified for multi-class softmax output) — https://github.com/wabdelmoula/massNet
- **Keras** (Framework in which the modified classifier architecture is defined, compiled with categorical cross-entropy, and serialized to JSON)
- **tensorflow** (Backend on which the Keras model is executed when compiling and saving the multi-class architecture)
- **Python** (Implementation language for the modified model; README specifies Python 3.6.12)

## Examples

```
import keras; m = keras.models.load_model('massNet_classifier.h5'); open('massnet_multiclass_model.json','w').write(m.to_json())
```

## Evaluation signals

- The saved massnet_multiclass_model.json contains an output layer with softmax activation.
- The softmax output layer has a number of units equal to the class-label count K (2 for the paper's binary normal/tumor task).
- The loss is categorical cross-entropy, not the original binary cross-entropy.
- The classifier input dimensionality matches the 5-dimensional VAE Code layer latent variable.
- The modified model compiles successfully in the README-pinned environment: Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0.

## Limitations

- The published paper only demonstrated binary class classification; multi-class classification was explicitly left for future development, so no published multi-class validation exists to compare against.
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article discussion section, so exact reproduction requires the README-pinned versions (Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0).
- Ground-truth class annotation based solely on annotated histology is an approximation that may not be optimal, especially for intra-tumor heterogeneity.

## Evidence
