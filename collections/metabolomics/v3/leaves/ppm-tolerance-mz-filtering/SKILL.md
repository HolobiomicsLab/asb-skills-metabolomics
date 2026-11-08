---
name: ppm-tolerance-mz-filtering
description: Decision support for testing whether restricting end-to-end LC-MS deep-learning
  classification to published biomarker m/z windows reduces accuracy versus training
  on complete raw metabolomic data.
when_to_use_negative:
- When the goal is maximum-accuracy disease classification rather than testing the
  contribution of known biomarker signals — full raw data gives the baseline accuracy
  (AUC 0.99).
- When input is already a peak/feature table rather than raw LC-MS data, since DeepMSProfiler
  operates end-to-end on raw signals converted to .npy.
- When biomarker m/z values or publication counts for the disease of interest are
  unavailable, since the filtering depends entirely on the curated PC-annotated biomarker
  list.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: End-to-end pipeline used to convert mzML to .npy, filter signals by biomarker
    m/z tolerance, and train/predict with the DenseNet121 ensemble at each PC threshold
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend for training the DenseNet121 ensemble on the m/z-filtered data (adam,
    lr 1e-4, 10 runs)
- name: Keras 2.3.1
  role: Model-building layer used with TensorFlow for the DenseNet121 ensemble training
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:b63a903a05dbf5aa3341c7e130ac9c3faf030b6bb82a0ed313c892ff883a6175
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ppm-tolerance-mz-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters raw LC-MS metabolomic signals to only those m/z values within a 3 ppm tolerance of curated biomarker m/z values (retained at publication-count thresholds PC>1, PC>3, PC>8), enabling ablation of DeepMSProfiler against the full-data baseline.

## When to use

Decision support for testing whether restricting end-to-end LC-MS deep-learning classification to published biomarker m/z windows reduces accuracy versus training on complete raw metabolomic data.

## When NOT to use

- When the goal is maximum-accuracy disease classification rather than testing the contribution of known biomarker signals — full raw data gives the baseline accuracy (AUC 0.99).
- When input is already a peak/feature table rather than raw LC-MS data, since DeepMSProfiler operates end-to-end on raw signals converted to .npy.
- When biomarker m/z values or publication counts for the disease of interest are unavailable, since the filtering depends entirely on the curated PC-annotated biomarker list.

## Inputs

- Raw serum LC-MS files (.mzML or .npy) from the 859-sample cohort (210 healthy, 323 lung nodules, 326 lung cancer)
- Curated biomarker list with m/z values and publication counts (811 biomarkers from 49 publications)
- PC threshold setting (PC>1, PC>3, or PC>8)

## Outputs

- PC-threshold-filtered .npy datasets containing only signals within 3 ppm of retained biomarker m/z values
- Per-threshold test accuracies and confusion matrices for the DenseNet121 ensemble
- ablation_pc_accuracy.csv comparing PC>1, PC>3, PC>8 accuracies to the full-data baseline

## How to apply

Curate the reported biomarker list (here 811 serum LC-MS lung-cancer biomarkers from 49 screened publications), recording each metabolite's m/z and publication count (PC). Convert raw LC-MS files to .npy format (mzML is auto-converted by the pipeline), then retain only signals falling within 3 ppm of a retained biomarker m/z at the chosen PC threshold (PC>1, then PC>3, then PC>8). Train the DenseNet121 ensemble (adam optimizer, lr 1e-4, 10 runs) on each filtered dataset and predict test labels, then compare accuracy at each PC threshold against the full-data DeepMSProfiler baseline (AUC 0.99, 96.1% early-stage detection). Rationale: if accuracy decreases as the PC threshold tightens, unknown metabolomic signals in serum contribute to classification, which is the expected Fig. 4f ablation trend.

## Related tools

- **DeepMSProfiler** (End-to-end pipeline used to convert mzML to .npy, filter signals by biomarker m/z tolerance, and train/predict with the DenseNet121 ensemble at each PC threshold) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend for training the DenseNet121 ensemble on the m/z-filtered data (adam, lr 1e-4, 10 runs))
- **Keras 2.3.1** (Model-building layer used with TensorFlow for the DenseNet121 ensemble training)

## Examples

```
python mainRun.py -data ../example/pc1_filtered_data/ -label ../example/label.txt -out ../jobs -run_train -run_pred -arch DenseNet121 -opt adam -lr 1e-4 -run 10 -plot_cm
```

## Evaluation signals

- Accuracy at each PC threshold (PC>1, PC>3, PC>8) must fall below the full-data DeepMSProfiler accuracy, reproducing the decreasing trend of Fig. 4f.
- Confusion matrices must be generated for each filtered-data prediction and compared to the baseline (AUC 0.99, 96.1% early-stage detection).
- ablation_pc_accuracy.csv must contain the PC-threshold accuracy table with monotonically sensible ordering across thresholds.
- Filtered datasets should contain only signals within 3 ppm of retained biomarker m/z at the specified PC cutoff — verify no out-of-tolerance signals remain.
- The 10-run DenseNet121 ensemble (adam, lr 1.0e-04) must complete training for each threshold, matching the '[INFO] run model with args: DenseNet121 adam lr 1.0e-04' log pattern.

## Limitations

- The pre-trained 859-sample serum model is not publicly downloadable; access requires contacting the author (<maintainer e-mail, see the repository README>).
- The ablation depends on the quality/completeness of the curated biomarker publication-count list (811 biomarkers from 49 publications); incomplete curation biases the filtered accuracy.
- Results depend on specific pinned versions (Python >= 3.6, TensorFlow == 2.2.0, Keras == 2.3.1, CUDA 10.1); other stacks may not reproduce the baseline.

## Evidence

- [other] Prediction performance of DeepMSProfiler using the complete raw data is highly accurate compared with the model using only the m/z signals of reported biomarkers (Fig. 4f ablation), indicating unknown metabolomic signals in serum contribute to classification accuracy.: [evidence span withheld — anchor status: not_found]
- [other] Using the 859-sample serum cohort (210 healthy, 323 lung nodules, 326 lung cancer), convert raw LC-MS files to .npy and filter to signals within 3 ppm of retained biomarker m/z at threshold PC>1.: [evidence span withheld — anchor status: not_found]
- [other] Train the DenseNet121 ensemble in DeepMSProfiler on the PC>1-filtered data (adam optimizer, lr 1e-4, 10 runs) and predict test labels, recording accuracy.: [evidence span withheld — anchor status: not_found]
