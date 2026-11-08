---
name: lcms-peak-quality-classification
description: Use when unaligned or aligned mzMine2/XCMS feature tables must be labelled
  and filtered to remove false-positive MS1 peaks before downstream statistical analysis
  of large-scale LC-MS metabolomics data.
when_to_use_negative:
- The goal is to remove peaks from non-lipid metabolites or peaks that need identity
  and reproducibility assessment — NeatMS gives no information on peak identity or
  reproducibility
- A curated library panel of metabolites of interest is required to classify peaks
  rather than remove chemical noise
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: NeatMS
  role: Core Python package providing the Experiment and NN_handler classes for peak
    labelling, classification, and CSV export
  repo: https://github.com/bihealth/NeatMS
- name: Tensorflow
  role: Backend used to run and train the neural network model that classifies peak
    quality
- name: Keras
  role: Core functions for building and loading the NN model (e.g. create_model) inside
    NN_handler
- name: pandas
  role: Used by the CSV export method (experiment.export_csv) to write filtered peak
    tables
- name: mzMine2
  role: Upstream pipeline whose aligned feature table CSV serves as the input to peak
    labelling
- name: XCMS
  role: Upstream pipeline whose feature table serves as input; supported since NeatMS
    version 0.7
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c02220
    title: Deep Learning-Assisted Peak Curation for Large-Scale LC-MS Metabolomics
  source_content_hash: sha256:cf892a83eb68b85a77ef04b8738713d8a6484105358b254da01962b0c8a94dda
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
  - XCMS
  - mzMine2
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lcms-peak-quality-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies untargeted LC-MS peaks as High_quality, Low_quality, or Noise using NeatMS's pretrained neural network, then filters out false-positive MS1 peaks reported by upstream pipelines such as mzMine2 and XCMS. Rationale: label-driven peak curation with a probability threshold (0.22 for the default model) outperforms rule-based noise removal on large-scale metabolomics data.

## When to use

Use when unaligned or aligned mzMine2/XCMS feature tables must be labelled and filtered to remove false-positive MS1 peaks before downstream statistical analysis of large-scale LC-MS metabolomics data.

## When NOT to use

- The goal is to remove peaks from non-lipid metabolites or peaks that need identity and reproducibility assessment — NeatMS gives no information on peak identity or reproducibility
- A curated library panel of metabolites of interest is required to classify peaks rather than remove chemical noise

## Inputs

- Raw LC-MS data folder containing mzML files
- Aligned or unaligned feature table CSV from mzMine2 or XCMS
- Pretrained default NeatMS model file (path_to_model.h5)

## Outputs

- CSV file with peak properties (rt, mz, height, area) and a label column containing High_quality, Low_quality, or Noise (Noise exported as missing values)

## How to apply

Create a NeatMS Experiment from the raw mzML data folder and the aligned/unaligned feature table CSV (input_data='mzmine' or 'xcms'), then instantiate an NN_handler on the experiment and load the pretrained default model with nn_handler.create_model(model='path_to_model.h5'). Set the classification threshold to 0.22 — the optimal value for the default model — and run nn_handler.predict_peaks(threshold) so every peak receives one of three labels (High_quality, Low_quality, Noise). Export with experiment.export_csv using export_properties (rt, mz, height, area, label), leaving Noise-labelled peaks as missing values; optionally restrict output via export_classes, min_group_size with min_group_classes, use_annotation, and an exclude list of sample names. Rationale: a deep neural network generalises from manually labelled training peaks to remove the majority of chemical noise, and analysis time drops roughly 25% when only high-confidence features are kept.

## Related tools

- **NeatMS** (Core Python package providing the Experiment and NN_handler classes for peak labelling, classification, and CSV export) — https://github.com/bihealth/NeatMS
- **Tensorflow** (Backend used to run and train the neural network model that classifies peak quality)
- **Keras** (Core functions for building and loading the NN model (e.g. create_model) inside NN_handler)
- **pandas** (Used by the CSV export method (experiment.export_csv) to write filtered peak tables)
- **mzMine2** (Upstream pipeline whose aligned feature table CSV serves as the input to peak labelling)
- **XCMS** (Upstream pipeline whose feature table serves as input; supported since NeatMS version 0.7)

## Examples

```
import NeatMS as ntms; experiment = ntms.Experiment(raw_data_folder_path, feature_table_path, input_data='mzmine'); nn_handler = ntms.NN_handler(experiment); nn_handler.create_model(model='path_to_model.h5'); nn_handler.predict_peaks(0.22); experiment.export_csv('results.csv')
```

## Evaluation signals

- Output CSV exists and contains a label column whose values are among High_quality, Low_quality, and Noise
- Noise-labelled peaks are exported as missing values, not as the string 'Noise'
- Peak property columns rt, mz, height, area are present in the exported CSV
- When training from scratch, validation ROC AUC computed via sklearn.metrics.auc is > 0.95
- Classification threshold is set to 0.22, the documented optimal value for the default model

## Limitations

- Tool gives no information on peak identity and reproducibility; it cannot remove peaks originated from non-lipid metabolites or other types of peaks
- A curated library panel of metabolites of interest is needed for peak classification beyond chemical noise removal
- Optimal threshold is model-specific — 0.22 applies to the provided default model; other models require their own threshold tuning
- Python version is not documented in the article (missing signal)
- NeatMS version 0.7 or later is required for XCMS aligned peak support

## Evidence

- [methods] Optimal threshold for default model is 0.22: "The optimal value for the default model that we provide is 0.22"
- [intro] NeatMS filters false positive MS1 peaks: "**NeatMS** enables automated filtering of false positive MS<sup>1</sup> peaks reported by commonly used LCMS data processing pipelines."
- [readme] NeatMS labels via NN classification: [evidence span withheld — anchor status: not_found]
- [methods] Workflow core API calls: "experiment = ntms.Experiment(raw_data_folder_path, feature_table_path, input_data)"
- [discussion] Analysis time reduction: [evidence span withheld — anchor status: not_found]
