---
name: biomarker-literature-curation
description: Decision support for testing whether restricting raw LC-MS metabolomic
  input to literature-reported biomarker m/z windows (3 ppm, PC>1/3/8) lowers a trained
  classifier's accuracy versus the full raw-data baseline.
when_to_use_negative:
- When the goal is biomarker discovery or annotating unknown metabolites via interpretability
  (RISE heatmaps, metabolite-protein networks) rather than testing whether known biomarker
  signals alone suffice for classification
- When input is already a peak/feature table rather than raw LC-MS data, since the
  3 ppm m/z-window filtering is defined on raw metabolomic signals
- When the data modality is not serum LC-MS metabolomics with the corresponding lung-cancer
  biomarker literature available for curation
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: DeepMSProfiler
  role: Provides the DenseNet121 ensemble training/prediction pipeline used to evaluate
    each PC-threshold-filtered dataset against the full-data baseline; installed via
    pip or git clone
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Deep-learning backend that runs the DenseNet121 ensemble training on the filtered
    .npy datasets
- name: Keras 2.3.1
  role: Model-definition layer over TensorFlow for the DenseNet121 architecture used
    in each ablation run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:069d852589b4c4f5f3bcb852715f002832d86f13b4b47fa6e73b278735578d20
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
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# biomarker-literature-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates published disease biomarker m/z values (with publication counts, PC) from screened literature and uses them to filter raw LC-MS signals within a 3 ppm tolerance, enabling ablation experiments that test how much unannotated metabolomic signal contributes to DeepMSProfiler classification accuracy.

## When to use

Decision support for testing whether restricting raw LC-MS metabolomic input to literature-reported biomarker m/z windows (3 ppm, PC>1/3/8) lowers a trained classifier's accuracy versus the full raw-data baseline.

## When NOT to use

- When the goal is biomarker discovery or annotating unknown metabolites via interpretability (RISE heatmaps, metabolite-protein networks) rather than testing whether known biomarker signals alone suffice for classification
- When input is already a peak/feature table rather than raw LC-MS data, since the 3 ppm m/z-window filtering is defined on raw metabolomic signals
- When the data modality is not serum LC-MS metabolomics with the corresponding lung-cancer biomarker literature available for curation

## Inputs

- 811 serum LC-MS lung-cancer biomarker m/z values with publication counts (PC) curated from 49 screened publications
- 859-sample serum LC-MS cohort (210 healthy, 323 lung nodules, 326 lung cancer) as raw .mzML or .npy files
- PC thresholds to apply: PC>1, PC>3, PC>8
- 3 ppm mass tolerance for retaining signals near biomarker m/z

## Outputs

- PC-threshold-filtered .npy datasets (PC>1, PC>3, PC>8) containing only signals within 3 ppm of retained biomarker m/z
- Per-threshold classification accuracies from the 10-run DenseNet121 ensemble
- Confusion matrices for each filtered-data prediction
- ablation_pc_accuracy.csv table comparing PC-threshold accuracies to the full-data baseline

## How to apply

Curate the 811 serum LC-MS lung-cancer biomarkers reported across 49 screened publications, recording each metabolite's m/z and publication count (PC). Convert the 859-sample serum cohort (210 healthy, 323 lung nodules, 326 lung cancer) raw LC-MS files (mzML auto-converts to .npy) and retain only signals within 3 ppm of biomarker m/z values at threshold PC>1. Train the DenseNet121 ensemble (adam optimizer, lr 1e-4, 10 runs) on the PC>1-filtered data and record test accuracy, then repeat filtering plus identical retraining at PC>3 and PC>8. Generate confusion matrices per threshold and compare against the full-data DeepMSProfiler baseline; the rationale is that the paper's Fig. 4f ablation shows full raw data is far more accurate than biomarker-restricted data, so unknown metabolomic signals contribute to classification.

## Related tools

- **DeepMSProfiler** (Provides the DenseNet121 ensemble training/prediction pipeline used to evaluate each PC-threshold-filtered dataset against the full-data baseline; installed via pip or git clone) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Deep-learning backend that runs the DenseNet121 ensemble training on the filtered .npy datasets)
- **Keras 2.3.1** (Model-definition layer over TensorFlow for the DenseNet121 architecture used in each ablation run)

## Examples

```
python mainRun.py -data ../pc1_filtered_data/ -label ../example/label.txt -out ../jobs/pc1 -run_train -run_pred -plot_cm
```

## Evaluation signals

- Accuracy at each PC threshold (PC>1, PC>3, PC>8) must fall below the full-data DeepMSProfiler accuracy (baseline AUC 0.99, 96.1% early-stage detection)
- Accuracy should show a decreasing trend as the PC threshold increases (PC>1 > PC>3 > PC>8), reproducing the Fig. 4f pattern
- Each filtered .npy dataset must contain only signals within 3 ppm of retained biomarker m/z values at the specified PC threshold
- Confusion matrices must be generated for every filtered-data prediction and compared against the full-data baseline confusion matrix
- Final accuracy table must be written to ablation_pc_accuracy.csv with one row per PC threshold

## Limitations

- Access to the pre-trained 859-sample serum model requires contacting the author, so the full-data baseline comparison may need retraining from scratch
- The filter retains only signals within 3 ppm of published biomarkers, discarding unknown/unannotated metabolomic signals that the paper shows contribute most of the classification accuracy
- Results depend on the curated 811-biomarker list from 49 publications; incomplete or changing literature coverage changes the PC thresholds and filtered inputs

## Evidence

- [abstract] unknown metabolomic signals contribute to accuracy: "Model flow and ablation experiments demonstrate that DeepMSProfiler overcomes inter-hospital variability and effects of unknown metabolites signals"
- [readme] 859-sample serum cohort composition: "a pre-trained model based on 859 serum metabolomics samples (210 healthy individuals, 323 lung nodules, 326 lung cancer) for academic use"
- [readme] raw file format handling: "If you upload a file in. mzML format, and the script will automatically convert to. npy format automatically."
- [abstract] end-to-end raw signal analysis: "Here, we present DeepMSProfiler, an explainable deep-learning-based method, enabling end-to-end analysis on raw metabolic signals"
