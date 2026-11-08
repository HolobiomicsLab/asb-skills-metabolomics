---
name: peak-label-csv-export
description: Use when an aligned or unaligned MZmine/XCMS feature table must be labelled
  as High_quality, Low_quality, or Noise by the pretrained NeatMS model and exported
  to CSV for downstream use.
when_to_use_negative:
- Goal is peak identification or reproducibility assessment — NeatMS gives no information
  on peak identity and cannot remove non-lipid metabolite peaks without a curated
  library panel.
- Input is already a labelled/curated table and you only need group filtering, not
  NN classification.
- Peak quality depends on properties other than MS2-derived feature quality (e.g.
  non-lipid metabolites) that the pretrained model was not trained on.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: NeatMS
  role: Provides Experiment, NN_handler (create_model, predict_peaks) and export_csv
    used to load data, label peaks and write the CSV.
  repo: https://github.com/bihealth/NeatMS
- name: pandas
  role: Underlies the CSV export method used to write the labelled feature table.
- name: Tensorflow
  role: Backend used by the pretrained Keras/Tensorflow neural network model loaded
    via create_model.
- name: mzMine2
  role: Source of the aligned feature table CSV that serves as input_data to the Experiment.
- name: XCMS
  role: Alternative source of the aligned feature table CSV consumed by the Experiment
    (aligned peaks supported from version 0.7).
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02220
    title: Deep Learning-Assisted Peak Curation for Large-Scale LC-MS Metabolomics
  source_content_hash: sha256:79110f532e25241b684a8021b850cb7dd7962cc7b00748c5889181abb8236298
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
  - NeatMS
  - Tensorflow
  - XCMS
  - mzMine2
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# peak-label-csv-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads the pretrained NeatMS neural network into an NN_handler, labels every LC-MS MS1 peak at the 0.22 threshold, and writes a CSV whose columns (rt, mz, height, area, label) are controlled by export_properties.

## When to use

Use when an aligned or unaligned MZmine/XCMS feature table must be labelled as High_quality, Low_quality, or Noise by the pretrained NeatMS model and exported to CSV for downstream use.

## When NOT to use

- Goal is peak identification or reproducibility assessment — NeatMS gives no information on peak identity and cannot remove non-lipid metabolite peaks without a curated library panel.
- Input is already a labelled/curated table and you only need group filtering, not NN classification.
- Peak quality depends on properties other than MS2-derived feature quality (e.g. non-lipid metabolites) that the pretrained model was not trained on.

## Inputs

- Raw LC-MS mzML data folder
- MZmine2 or XCMS aligned feature table CSV (e.g. aligned_features.csv)
- Unaligned per-sample feature tables (e.g. unaligned_features directory)
- Pretrained default NeatMS neural network model (path_to_model.h5)
- Optional manual annotation set for use_annotation export

## Outputs

- CSV with columns rt, mz, height, area and a label column whose values are High_quality, Low_quality, or missing (Noise)
- Filtered feature table restricted to selected export_classes and/or min_group_classes / min_group_size criteria

## How to apply

Create a NeatMS Experiment from the raw mzML folder and the feature-table CSV with input_data='mzmine' (unaligned tables also supported since version 0.7). Instantiate an NN_handler and call create_model(model='path_to_model.h5') to load the pretrained default model shipped in the repository's data folder. Set the classification threshold to 0.22 — the optimal value for the default model — and run nn_handler.predict_peaks(threshold) so each peak receives one of three labels. Export with experiment.export_csv(filename): use the default export_properties (rt, mz, height, area, label); by default Noise-labelled peaks appear as missing values. Rationale: label-based export removes false-positive MS1 peaks from the processing pipeline without rerunning it, cutting analysis time by ~25% when only high-confidence features are kept.

## Related tools

- **NeatMS** (Provides Experiment, NN_handler (create_model, predict_peaks) and export_csv used to load data, label peaks and write the CSV.) — https://github.com/bihealth/NeatMS
- **pandas** (Underlies the CSV export method used to write the labelled feature table.)
- **Tensorflow** (Backend used by the pretrained Keras/Tensorflow neural network model loaded via create_model.)
- **mzMine2** (Source of the aligned feature table CSV that serves as input_data to the Experiment.)
- **XCMS** (Alternative source of the aligned feature table CSV consumed by the Experiment (aligned peaks supported from version 0.7).)

## Examples

```
experiment = ntms.Experiment(raw_data_folder_path, feature_table_path, 'mzmine'); nn_handler.create_model(model='path_to_model.h5'); nn_handler.predict_peaks(0.22); experiment.export_csv(filename)
```

## Evaluation signals

- Output CSV exists and contains a label column whose values are among High_quality, Low_quality, and missing (Noise exported as missing)
- Peak property columns rt, mz, height, area are present as expected from export_properties
- All peaks classified at threshold 0.22; default model optimal value respected when reproducing paper settings
- If min_group_size/min_group_classes are set, retained features appear in at least min_group_size% of samples and carry one of the allowed labels

## Limitations

- Removing most chemical noise does not guarantee removal of other peak types, e.g. peaks from non-lipid metabolites or contaminants without a curated library panel
- The pretrained default model's optimal threshold (0.22) and workflow parameters were tuned for the paper's setting; other thresholds or models may need re-optimisation via get_threshold / ROC–AUC analysis
- Noise-labelled peaks are exported as missing values by default, which changes downstream schema assumptions
- No peak-identity information is provided — the tool is a labelling/filtering step, not an annotation method

## Evidence
