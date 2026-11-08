---
name: keras-model-summary-inspection
description: Use when a pretrained .h5 model must be replicated or audited and its
  architecture (layer names, shapes, parameter counts) needs to be verified programmatically.
when_to_use_negative:
- The goal is peak prediction or filtering itself (e.g. labelling LCMS peaks as noise,
  low confidence, or high quality) rather than architecture inspection — use NeatMS's
  predict_peaks workflow instead.
- Only a single layer's weight matrix is needed and the .h5 file is already loaded
  — layer inspection alone does not require rebuilding the full architecture.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3372
- http://edamontology.org/topic_0003
tools:
- name: Keras
  role: Load the .h5 model and rebuild the fixed architecture, then emit the model.summary()
    table
  repo: https://github.com/bihealth/NeatMS
- name: Tensorflow
  role: Underlying backend whose validation output reports the test (called validation)
    split during training of the inspected model
  repo: https://github.com/bihealth/NeatMS
- name: NeatMS
  role: Distributes the pretrained neural network model in its data folder and defines
    the target architecture being inspected
  repo: https://github.com/bihealth/NeatMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c02220
    title: Deep Learning-Assisted Peak Curation for Large-Scale LC-MS Metabolomics
  source_content_hash: sha256:1d5a52a366ec6b41a5dbd538b2e60d969e3105304ed1ed4e0827df33829ec1b3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/bihealth/NeatMS
  repo_url: https://github.com/bihealth/NeatMS
  tools:
  - Keras
  - NeatMS
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3372
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# keras-model-summary-inspection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct and inspect a pretrained Keras neural network (e.g. NeatMS's 1,002,883-parameter CNN) layer by layer with model.summary(), verifying each layer's output shapes and parameter counts against the shipped .h5 file.

## When to use

Use when a pretrained .h5 model must be replicated or audited and its architecture (layer names, shapes, parameter counts) needs to be verified programmatically.

## When NOT to use

- The goal is peak prediction or filtering itself (e.g. labelling LCMS peaks as noise, low confidence, or high quality) rather than architecture inspection — use NeatMS's predict_peaks workflow instead.
- Only a single layer's weight matrix is needed and the .h5 file is already loaded — layer inspection alone does not require rebuilding the full architecture.

## Inputs

- Pretrained Keras .h5 model file (e.g. path_to_model.h5 from the NeatMS data folder)
- Rebuilt Keras model with documented layer sequence

## Outputs

- model_architecture_summary.txt containing the model.summary() table with layer names, output shapes, and parameter counts
- Match confirmation that rebuilt input_1 and dense_prediction shapes equal the pretrained .h5 shapes

## How to apply

Load the pretrained model file with Keras (e.g. NeatMS's path_to_model.h5) to obtain the reference layer list, then rebuild the fixed architecture in Keras: InputLayer with shape (None, 2, 120, 1), a Conv2D with 32 filters (output (None, 2, 120, 32), 832 params), MaxPooling2D (output (None, 2, 60, 32)), a second Conv2D with 64 filters (output (None, 2, 60, 64), 18,496 params), Flatten to (None, 7680), Dense to (None, 128) with 983,168 params, Dropout, and a final Dense to (None, 3) with 387 params. Call model.summary() to emit the table (total params 1,002,883; trainable 1,002,883; non-trainable 0) and save it as model_architecture_summary.txt. Validation consists of confirming that the rebuilt model's input_1 and dense_prediction shapes match those of the pretrained .h5 file and that the summary text contains the documented layer names, output shapes, and parameter counts.

## Related tools

- **Keras** (Load the .h5 model and rebuild the fixed architecture, then emit the model.summary() table) — https://github.com/bihealth/NeatMS
- **Tensorflow** (Underlying backend whose validation output reports the test (called validation) split during training of the inspected model) — https://github.com/bihealth/NeatMS
- **NeatMS** (Distributes the pretrained neural network model in its data folder and defines the target architecture being inspected) — https://github.com/bihealth/NeatMS

## Examples

```
import keras; m = keras.models.load_model('path_to_model.h5'); m.summary(); open('model_architecture_summary.txt','w').write(str(m.summary()))
```

## Evaluation signals

- Rebuilt model's total parameter count equals 1,002,883 (trainable 1,002,883; non-trainable 0)
- Per-layer parameter counts match the reference: conv2d_1 = 832, conv2d_2 = 18,496, dense_1 = 983,168, dense_prediction = 387
- Output shapes match the documented sequence: (None, 2, 120, 32), (None, 2, 60, 32), (None, 2, 60, 64), (None, 7680), (None, 128), (None, 3)
- Rebuilt input_1 and dense_prediction shapes are identical to those of the pretrained .h5 file
- model_architecture_summary.txt contains the exact documented layer names (conv2d_1, max_pooling2d_1, conv2d_2, flatten_1, dense_1, dropout_1, dense_prediction)

## Limitations

- NeatMS's model architecture is fixed, so the summary is only meaningful for this exact layer sequence; a modified architecture will not reproduce the documented counts.
- NeatMS only performs signal labelling and filtering — the summary does not provide peak identity or reproducibility information, and non-lipid metabolite peaks require a curated library panel.
- The pretrained model is distributed in the NeatMS data folder, so architecture inspection depends on access to that repository asset.

## Evidence

- [methods] Load the .h5 model and rebuild the fixed architecture, then emit the model.summary() table: "nn_handler.create_model(model="path_to_model.h5")"
- [methods] Parameter-count expectations come from the pretrained model distributed with NeatMS: "We can now load the neural network model that is provided with the tool in the data folder of [NeatMS github repository](https://github.com/bihealth/NeatMS)"
- [methods] Keras functions used for the summary and layer inspection are Keras core APIs: "Note that these are Keras core functions, refer to the Keras documentation for more details"
- [abstract] The model is a Tensorflow-trained classifier whose data-quality accuracy is the motivation for inspecting the architecture: [evidence span withheld — anchor status: not_found]
- [readme] Source code and the neural network model asset are distributed on the NeatMS repository: "The source code and related materials (e.g. tutorials, example data, neural network model) are available at [https://github.com/bihealth/NeatMS](https://github.com/bihealth/NeatMS)."
