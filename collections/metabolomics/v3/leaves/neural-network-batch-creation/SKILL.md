---
name: neural-network-batch-creation
description: Use when a manually labelled LC-MS peak experiment exists and you need
  balanced train/test/validation batches to retrain or fine-tune the NeatMS neural
  network classifier.
when_to_use_negative:
- You only want to classify peaks with the provided default model (path_to_model.h5,
  threshold 0.22) — no batch creation is needed, just load the model and predict_peaks(threshold)
- Peaks are not manually labelled — batch creation for supervised training requires
  labelled data, e.g. from annotation_tool.launch_annotation_tool()
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: NeatMS
  role: Provides the NN_handler.create_batches method that creates the train/test/validation
    batches
  repo: https://github.com/bihealth/NeatMS
- name: Keras
  role: Underlying framework whose core functions (compile/fit) execute on the created
    batches
- name: Tensorflow
  role: Backend that reports test-set accuracy (called validation in Tensorflow output)
    during training on the batches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c02220
    title: Deep Learning-Assisted Peak Curation for Large-Scale LC-MS Metabolomics
  source_content_hash: sha256:33ec70cca222a09591b2a88c3cd77bf434de35c25d57d0070ca653aa71b7e9dc
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
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# neural-network-batch-creation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Split manually labelled NeatMS peaks into training, testing, and validation batches (80:10:10) via nn_handler.create_batches so a neural network can be retrained or fine-tuned for LC-MS peak quality classification.

## When to use

Use when a manually labelled LC-MS peak experiment exists and you need balanced train/test/validation batches to retrain or fine-tune the NeatMS neural network classifier.

## When NOT to use

- You only want to classify peaks with the provided default model (path_to_model.h5, threshold 0.22) — no batch creation is needed, just load the model and predict_peaks(threshold)
- Peaks are not manually labelled — batch creation for supervised training requires labelled data, e.g. from annotation_tool.launch_annotation_tool()

## Inputs

- Manually labelled training experiment pickle file from the NeatMS tutorial data
- NeatMS Experiment object with instantiated NN_handler (matrice_size=120, margin=1, min_scan_num=5)

## Outputs

- Training, testing, and validation peak batches in an 80:10:10 split, stored on the NN_handler
- Batches ready for Keras/Tensorflow model training and validation AUC evaluation

## How to apply

Attach an ntms.NN_handler object to a NeatMS Experiment built from a manually labelled training experiment (pickle from the NeatMS tutorial data) with default parameters matrice_size=120, margin=1, min_scan_num=5. Call nn_handler.create_batches(validation_split=0.1, normalise_class=False) to yield the documented 80:10:10 training:test:validation split. The validation_split argument controls the size of the test and validation sets, while normalise_class can be set to make every class have the same number of peaks for training; leave it False to preserve the natural class distribution of the labelled data. The resulting batches feed directly into nn_handler.create_model(lr=0.00001, optimizer='Adam') and nn_handler.train_model(1000), with test-set accuracy monitored during training for signs of overfitting.

## Related tools

- **NeatMS** (Provides the NN_handler.create_batches method that creates the train/test/validation batches) — https://github.com/bihealth/NeatMS
- **Keras** (Underlying framework whose core functions (compile/fit) execute on the created batches)
- **Tensorflow** (Backend that reports test-set accuracy (called validation in Tensorflow output) during training on the batches)

## Examples

```
nn_handler.create_batches(validation_split=0.1, normalise_class=False)
```

## Evaluation signals

- The split matches the documented 80:10:10 training:test:validation proportion after create_batches(validation_split=0.1)
- Batches are non-empty and cover both labelled classes (High_quality and noise) from the labelled experiment
- Downstream training (train_model(1000)) runs without shape errors, confirming matrice_size=120 peak matrices were generated
- Test-set accuracy monitored during training shows learning without overfitting, and validation ROC AUC exceeds 0.95

## Limitations

- The normalise_class option changes the class balance; if used, batches no longer reflect the natural labelled class distribution
- The validation_split=0.1 value defines both test and validation sets jointly, so the split proportion cannot be tuned independently for test vs validation
- Transfer learning (fine-tuning the existing model) is an alternative path that also uses these batches, but results depend on the quality of manual labels

## Evidence

- [methods] Create training, testing, and validation batches using nn_handler.create_batches(validation_split=0.1, normalise_class=False): [evidence span withheld — anchor status: not_found]
