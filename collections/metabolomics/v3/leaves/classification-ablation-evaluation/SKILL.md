---
name: classification-ablation-evaluation
description: Use when you need to test whether restricting raw LC-MS input to published
  lung-cancer serum biomarker m/z signals (3 ppm window, PC>1/3/8 thresholds) degrades
  classifier accuracy versus the full-data model.
when_to_use_negative:
- Input data are already peak-picked, aligned, annotated feature tables — this ablation
  requires raw LC-MS signals (mzML/.npy) so the 3 ppm biomarker filtering can be applied
  end-to-end
- The goal is biomarker identification or annotation (e.g., via peak extraction) rather
  than quantifying the classification contribution of unknown metabolite signals
- No held-out test labels or disease classes exist, so accuracy of each PC-threshold
  model cannot be compared against the full-data baseline
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: DeepMSProfiler
  role: End-to-end pipeline used to train and predict with the DenseNet121 ensemble
    on PC-filtered versus full raw data and plot confusion matrices/AUC
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend framework required by DeepMSProfiler for training the DenseNet121
    ensemble runs
- name: Keras 2.3.1
  role: Keras version pinned by the package for building the DenseNet121 models in
    each ensemble run
- name: Python (>= 3.6)
  role: Runtime for running DeepMSProfiler's run_train/run_predict loop and writing
    ablation_pc_accuracy.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:233ad3f4226a2fa8171dc4acbb22c3e9ce51d6647f301f85231331180df2b4d6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - DeepMSProfiler
  - Keras
  - Keras 2.3.1
  - Python
  - Python (>= 3.6)
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# classification-ablation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An ablation procedure that retrains a DenseNet121 ensemble on raw LC-MS data filtered to only literature-reported biomarker m/z signals (within 3 ppm, at publication-count thresholds PC>1, PC>3, PC>8) and compares accuracy against the full-data DeepMSProfiler baseline to quantify how much unknown metabolomic signals contribute to classification.

## When to use

Use when you need to test whether restricting raw LC-MS input to published lung-cancer serum biomarker m/z signals (3 ppm window, PC>1/3/8 thresholds) degrades classifier accuracy versus the full-data model.

## When NOT to use

- Input data are already peak-picked, aligned, annotated feature tables — this ablation requires raw LC-MS signals (mzML/.npy) so the 3 ppm biomarker filtering can be applied end-to-end
- The goal is biomarker identification or annotation (e.g., via peak extraction) rather than quantifying the classification contribution of unknown metabolite signals
- No held-out test labels or disease classes exist, so accuracy of each PC-threshold model cannot be compared against the full-data baseline

## Inputs

- Raw serum LC-MS files (mzML, auto-converted to .npy) from the 859-sample cohort (210 healthy, 323 lung nodules, 326 lung cancer)
- Curated biomarker list: 811 lung-cancer serum biomarker m/z values with publication counts (PC) from 49 screened publications
- Label file with sample class and train/test split (e.g., datalist.txt / label.txt with cancer, health, nodule labels)

## Outputs

- ablation_pc_accuracy.csv: accuracy table per PC threshold (PC>1, PC>3, PC>8) versus full-data baseline
- Per-threshold confusion matrices for the filtered-data predictions
- Trained DenseNet121 ensemble model checkpoints per filtered dataset

## How to apply

First curate the 811 serum LC-MS lung-cancer biomarkers reported across 49 screened publications, recording each metabolite's m/z and publication count (PC). Convert the raw mzML files of the 859-sample serum cohort (210 healthy, 323 lung nodules, 326 lung cancer) to .npy and filter to signals within 3 ppm of retained biomarker m/z values at threshold PC>1. Train the DenseNet121 ensemble via DeepMSProfiler with adam optimizer, lr 1e-4, 10 runs (batch size 8, 2 epochs per README defaults), predict test labels, and record accuracy; repeat the 3 ppm filtering and retraining at PC>3 and PC>8 with identical settings. Rationale: if accuracies at PC>1, PC>3, PC>8 all fall below the full-data baseline (AUC 0.99, 96.1% early-stage detection) with a rising trend toward the baseline as PC increases, this reproduces the Fig. 4f ablation finding that unknown metabolomic signals contribute to classification accuracy. Write the threshold-accuracy comparison to ablation_pc_accuracy.csv and generate confusion matrices per threshold via plot_cm.

## Related tools

- **DeepMSProfiler** (End-to-end pipeline used to train and predict with the DenseNet121 ensemble on PC-filtered versus full raw data and plot confusion matrices/AUC) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend framework required by DeepMSProfiler for training the DenseNet121 ensemble runs)
- **Keras 2.3.1** (Keras version pinned by the package for building the DenseNet121 models in each ensemble run)
- **Python (>= 3.6)** (Runtime for running DeepMSProfiler's run_train/run_predict loop and writing ablation_pc_accuracy.csv)

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_train -run_pred -run_feature
```

## Evaluation signals

- Accuracy at each PC threshold (PC>1, PC>3, PC>8) must fall below the full-data DeepMSProfiler baseline (AUC 0.99, 96.1% early-stage detection)
- Accuracy should trend upward as the PC threshold increases (PC>1 → PC>3 → PC>8), approaching the full-data baseline — reproducing the decreasing-accuracy trend of Fig. 4f
- Each ensemble run must consist of exactly 10 DenseNet121 models with adam optimizer and lr 1.0e-04 (log lines 0–9)
- Confusion matrices for each filtered prediction must be generated for the 3-class scheme (healthy, nodule, cancer) and be consistent with the train/test split in the label file
- ablation_pc_accuracy.csv must contain one accuracy entry per threshold plus the full-data reference row

## Limitations

- The pre-trained full-data baseline model requires contacting the author for access; it cannot be downloaded directly
- Filtering to known biomarkers intentionally discards unknown signals, so filtered models will underperform the full-data model — this skill is a diagnostic, not a path to better accuracy
- Results depend on the quality/coverage of the curated 811-biomarker list from 49 publications and the fixed 3 ppm tolerance
- The package is pinned to TensorFlow 2.2.0, Keras 2.3.1 and CUDA 10.1 (CPU-only environments supported); newer environments may need adaptation

## Evidence

- [abstract] Ablation rationale: unknown signals drive accuracy: "Model flow and ablation experiments demonstrate that DeepMSProfiler overcomes inter-hospital variability and effects of unknown metabolites signals"
- [abstract] Full-data baseline performance: "DeepMSProfiler successfully differentiates the metabolomic profiles of different groups (AUC 0.99)"
- [intro] Early-stage detection accuracy baseline: "an accuracy of 96.1% in detecting early-stage lung adenocarcinoma"
- [methods] Cohort definition for the ablation: "a pre-trained model based on 859 serum metabolomics samples (210 healthy individuals, 323 lung nodules, 326 lung cancer) for academic use"
- [methods] Ensemble training configuration: "[INFO] run model with args: DenseNet121 adam lr 1.0e-04 0"
- [readme] Number of ensemble runs: "-run`: Specifies the number of runs, default is `10`."
