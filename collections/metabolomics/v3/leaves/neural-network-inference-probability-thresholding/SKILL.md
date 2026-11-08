---
name: neural-network-inference-probability-thresholding
description: Use when an aligned or unaligned MZmine/XCMS feature table must be scored
  against raw mzML data so that false-positive MS1 peaks reported by the processing
  pipeline can be filtered by confidence label.
when_to_use_negative:
- Peaks originate from non-lipid metabolites outside the curated library panel the
  model was trained on — NeatMS gives no information on peak identity and cannot remove
  such peaks.
- You need peak identity, annotation, or reproducibility metrics — the skill only
  assigns quality labels, not structural or biological interpretation.
- You have no raw mzML data and only the feature table — the classifier needs chromatographic
  peak shapes from raw spectra to score each feature.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: NeatMS
  role: Provides the Experiment object, NN_handler, predict_peaks(threshold) API,
    and csv export with class- and property-based filtering used in this skill.
  repo: https://github.com/bihealth/NeatMS
- name: Tensorflow
  role: Backend that runs the neural network inference and reports validation metrics
    during custom model training/transfer learning.
  repo: https://github.com/bihealth/NeatMS
- name: Keras
  role: Core functions used to build and load the .h5 model inside nn_handler.
- name: mzMine2
  role: Upstream tool that produces the aligned or unaligned feature tables consumed
    by NeatMS for peak labelling.
- name: XCMS
  role: Alternative upstream processing tool whose feature tables (aligned since NeatMS
    0.7) can be fed into the same prediction workflow.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02220
    title: Deep Learning-Assisted Peak Curation for Large-Scale LC-MS Metabolomics
  source_content_hash: sha256:1b0db8d6de390af1e1e2f563b1c3b54ce1db39133a2b4d594a0838521c17cb4b
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# neural-network-inference-probability-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply a trained NeatMS neural network to label each LC-MS peak as High_quality, Low_quality, or Noise, then convert class probabilities into discrete labels using a user-chosen threshold (0.22 for the default model).

## When to use

Use when an aligned or unaligned MZmine/XCMS feature table must be scored against raw mzML data so that false-positive MS1 peaks reported by the processing pipeline can be filtered by confidence label.

## When NOT to use

- Peaks originate from non-lipid metabolites outside the curated library panel the model was trained on — NeatMS gives no information on peak identity and cannot remove such peaks.
- You need peak identity, annotation, or reproducibility metrics — the skill only assigns quality labels, not structural or biological interpretation.
- You have no raw mzML data and only the feature table — the classifier needs chromatographic peak shapes from raw spectra to score each feature.

## Inputs

- Raw LC-MS mzML files folder
- Aligned or unaligned MZmine feature table csv (e.g. aligned_features.csv from the NeatMS test_data)
- Pretrained neural network model file (path_to_model.h5 from the NeatMS data folder)
- Optional: sample exclusion list (NeatMS Sample objects or sample-name strings)

## Outputs

- Per-peak label assignment (High_quality / Low_quality / Noise) stored in the NeatMS Experiment
- Exported csv with columns rt, mz, height, area, label (Noise rows as missing values)
- Optional probability-threshold optimisation output (ROC curve, AUC, best threshold)

## How to apply

Build a NeatMS Experiment object from the raw mzML folder and the feature table csv (input_data='mzmine'), then create an NN_handler and load the pretrained .h5 model with nn_handler.create_model(model='path_to_model.h5'). Select a classification threshold — the optimal value for the default model is 0.22 — and run nn_handler.predict_peaks(threshold), which assigns each peak one of three labels (High_quality, Low_quality, Noise) based on the network's probability output. The threshold is a recall/precision dial: a higher value keeps only peaks the network is very confident are real, while a lower value rescues marginal peaks at the cost of more chemical noise; the article's 0.22 was empirically optimised for the shipped model, so re-optimise with nn_handler.get_threshold or ROC/AUC analysis if a custom model is used. Optionally pass the min_scan_number argument to filter peaks with fewer points/scans before prediction, and use use_annotation=True so manually annotated labels from the dash/Jupyter annotation tool override the predicted label at export time. Export with experiment.export_csv using export_classes to control which labels survive (default exports High_quality and Low_quality, leaving Noise as missing values) and export_properties for rt, mz, height, area columns.

## Related tools

- **NeatMS** (Provides the Experiment object, NN_handler, predict_peaks(threshold) API, and csv export with class- and property-based filtering used in this skill.) — https://github.com/bihealth/NeatMS
- **Tensorflow** (Backend that runs the neural network inference and reports validation metrics during custom model training/transfer learning.) — https://github.com/bihealth/NeatMS
- **Keras** (Core functions used to build and load the .h5 model inside nn_handler.)
- **mzMine2** (Upstream tool that produces the aligned or unaligned feature tables consumed by NeatMS for peak labelling.)
- **XCMS** (Alternative upstream processing tool whose feature tables (aligned since NeatMS 0.7) can be fed into the same prediction workflow.)

## Examples

```
experiment = ntms.Experiment('raw_data_folder_path', 'feature_table.csv', 'mzmine'); nn_handler = ntms.NN_handler(experiment); nn_handler.create_model(model='path_to_model.h5'); nn_handler.predict_peaks(0.22); experiment.export_csv('filtered_peaks.csv')
```

## Evaluation signals

- Output csv contains a `label` column whose values are restricted to High_quality, Low_quality, and Noise (Noise rendered as missing/empty cells).
- Every feature in the input table has a probability-backed label; peaks below the chosen threshold are demoted (e.g. to Noise) rather than silently dropped.
- When threshold=0.22 with the default model, the retained fraction and Noise counts match the tutorial reference run; deviation indicates a wrong model file or feature-table format.
- export_properties columns (rt, mz, height, area) are present and numeric, and sample exclusion via the `exclude` list removes exactly the named samples.
- If min_scan_number is set, no exported peak has fewer points/scans than that cutoff.

## Limitations

- The default model's threshold of 0.22 is not universal — it was optimised for the shipped model and must be re-derived (e.g. via nn_handler.get_threshold or ROC/AUC) when using a retrained or transfer-learned model.
- NeatMS cannot remove peaks originating from non-lipid metabolites or assess peak identity/reproducibility; it only filters chemical noise and low-quality signals.
- For a custom application the tool must be provided with a curated library panel of metabolites of interest.
- From version 0.7 aligned XCMS peaks are supported, but earlier XCMS alignments require the unaligned format.

## Evidence

- [discussion] Cannot remove peaks from non-lipid metabolites or assess identity/reproducibility: [evidence span withheld — anchor status: not_found]
- [discussion] NeatMS removes majority of chemical noise peaks: [evidence span withheld — anchor status: not_found]
- [discussion] Library panel needed for custom peak classification: [evidence span withheld — anchor status: not_found]
