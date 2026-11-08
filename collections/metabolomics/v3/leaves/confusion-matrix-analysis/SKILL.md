---
name: confusion-matrix-analysis
description: Use when evaluating classification performance of the DenseNet121 ensemble
  on raw LC-MS (.npy/.mzML) test samples — especially when comparing full-data vs
  biomarker-filtered (PC>1/3/8, 3 ppm) models via Fig. 4f-style ablation.
when_to_use_negative:
- Input is raw mzML/.npy data that has not yet been through run_train and run_predict
  — no predictions exist to tabulate.
- The goal is metabolite feature interpretation (RISE heatmaps, metabolite-protein
  networks) rather than classification performance; use run_feature/show_feature instead.
- Evaluating a single model run rather than the ensemble mode, since the paper attributes
  background-category removal and stable performance to the ensemble strategy.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DeepMSProfiler
  role: Generates the confusion matrix via run_predict's plot_cm flag after ensemble
    training; the tool whose ablated accuracies are tabulated.
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend that trains the DenseNet121 ensemble whose test predictions feed the
    confusion matrix.
- name: Keras 2.3.1
  role: Model API layer used with TensorFlow for the DenseNet121 ensemble runs.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:052eb0e6afaf258f7b2b4f1f1339ca654258d1e98921009f325ed7182117a644
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# confusion-matrix-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates and interprets confusion matrices from DeepMSProfiler ensemble predictions on LC-MS serum metabolomics data, comparing class-level prediction errors (healthy vs lung nodule vs lung cancer) across model variants or data-filtering conditions.

## When to use

Use when evaluating classification performance of the DenseNet121 ensemble on raw LC-MS (.npy/.mzML) test samples — especially when comparing full-data vs biomarker-filtered (PC>1/3/8, 3 ppm) models via Fig. 4f-style ablation.

## When NOT to use

- Input is raw mzML/.npy data that has not yet been through run_train and run_predict — no predictions exist to tabulate.
- The goal is metabolite feature interpretation (RISE heatmaps, metabolite-protein networks) rather than classification performance; use run_feature/show_feature instead.
- Evaluating a single model run rather than the ensemble mode, since the paper attributes background-category removal and stable performance to the ensemble strategy.

## Inputs

- DeepMSProfiler prediction outputs for the test split (from run_predict on trained ensemble jobs)
- datalist.txt / label.txt mapping .npy sample files to labels (health, nodule, cancer) and train/test split
- Biomarker-filtered .npy data at PC>1, PC>3, PC>8 (signals within 3 ppm of published biomarker m/z)

## Outputs

- Confusion matrix plot (e.g. SVG) per model/PC-threshold condition
- Per-class accuracy counts across healthy, lung nodule, and lung cancer classes
- Ablation accuracy table written to ablation_pc_accuracy.csv comparing PC thresholds vs full-data baseline

## How to apply

Run run_predict with plot_cm=True after ensemble training (DenseNet121, adam, lr 1e-4, 10 runs) so the tool emits the confusion matrix alongside the AUC curve for the held-out test split defined in datalist.txt. For the ablation, repeat train/predict on data filtered to signals within 3 ppm of the 811 curated serum biomarker m/z values at PC>1, PC>3, and PC>8, generating a confusion matrix per threshold. Compare each matrix's accuracy against the full-data DeepMSProfiler baseline (AUC 0.99, 96.1% early-stage detection); the validation signal is a monotonic accuracy decrease as the PC threshold tightens, reproducing the Fig. 4f trend. Write the per-threshold accuracies to ablation_pc_accuracy.csv and inspect the matrices for which class pairs (e.g. nodule vs cancer) absorb the misclassifications.

## Related tools

- **DeepMSProfiler** (Generates the confusion matrix via run_predict's plot_cm flag after ensemble training; the tool whose ablated accuracies are tabulated.) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend that trains the DenseNet121 ensemble whose test predictions feed the confusion matrix.)
- **Keras 2.3.1** (Model API layer used with TensorFlow for the DenseNet121 ensemble runs.)

## Examples

```
run_predict(job_dir='DeepMSProfiler/example/out/jobs007',plot_auc=True,plot_cm=True)
```

## Evaluation signals

- Full-data baseline accuracy must exceed accuracy at every PC threshold (PC>1, PC>3, PC>8) with a decreasing trend, matching Fig. 4f.
- Full-data reference performance: AUC 0.99 and 96.1% accuracy on early-stage lung adenocarcinoma detection.
- Matrix dimensions must cover all three classes (healthy, nodule, cancer) and test rows in datalist.txt assigned to 'test'.
- Confusion matrix is produced only when plot_cm=True is passed to run_predict; confirm the flag and output artifact exist in the job directory.

## Limitations

- The pre-trained 859-sample serum model for prediction-only runs requires contacting the author for access, so matrix generation may require full local retraining.
- Demo/default training uses epoch=2 and 10 runs; accuracy values are reproducible only with the same DenseNet121, adam, lr 1e-4 ensemble settings.
- Biomarker-restricted inputs (PC>1/3/8) intentionally degrade accuracy; a confusion matrix from filtered data reflects unknown-metabolite loss, not model failure per se.

## Evidence

- [other] Generate confusion matrices for each filtered-data prediction and compare accuracies against the full-data DeepMSProfiler baseline: [evidence span withheld — anchor status: not_found]
- [other] Prediction performance of DeepMSProfiler using the complete raw data is highly accurate compared with the model using only the m/z signals of reported biomarkers: [evidence span withheld — anchor status: not_found]
