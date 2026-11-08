---
name: ensemble-deep-learning-interpretability
description: Use when a trained ensemble of deep-learning classifiers on raw LC-MS
  data (mzML or .npy arrays over a retention-time × m/z grid) must be interrogated
  to reveal which metabolite signals — including unannotated ones — drive class predictions.
when_to_use_negative:
- Input is already a peak-picked, aligned, and annotated feature table — this skill
  operates end-to-end on raw LC-MS signals without conventional peak extraction and
  identification steps
- Only point predictions are needed (run_predict with plot_auc/plot_cm) and no model-explanation
  or biomarker-region localization is required
- The model is a single network rather than the trained ensemble; the 'ensemble' heatmap
  mode requires the multi-run job directory output
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: End-to-end package that runs the RISE feature-extraction stage (run_feature)
    and renders the ensemble heatmaps (show_feature) from the trained job directory
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend framework in which the DenseNet121 ensemble whose predictions are
    perturbed by RISE is implemented
- name: Keras 2.3.1
  role: Model-building API used with TensorFlow to define the DenseNet121 architecture
    being interpreted
- name: CUDA 10.1
  role: GPU acceleration for the masking-and-inference loops of the RISE feature-extraction
    stage (CPU-only execution also supported)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:fdac05bf75fc938ce45aca598e32670a6c46b63b24168b5214153dcd55ef0b61
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
  - CUDA
  - CUDA 10.1
  - DeepMSProfiler
  - Keras
  - Keras 2.3.1
  - Python
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# ensemble-deep-learning-interpretability

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates per-class feature-contribution heatmaps from an ensemble of DenseNet121 models trained on raw LC-MS metabolomic arrays using the RISE random-masking perturbation procedure, quantifying which retention-time × m/z regions drive disease classification.

## When to use

Use when a trained ensemble of deep-learning classifiers on raw LC-MS data (mzML or .npy arrays over a retention-time × m/z grid) must be interrogated to reveal which metabolite signals — including unannotated ones — drive class predictions.

## When NOT to use

- Input is already a peak-picked, aligned, and annotated feature table — this skill operates end-to-end on raw LC-MS signals without conventional peak extraction and identification steps
- Only point predictions are needed (run_predict with plot_auc/plot_cm) and no model-explanation or biomarker-region localization is required
- The model is a single network rather than the trained ensemble; the 'ensemble' heatmap mode requires the multi-run job directory output

## Inputs

- Raw LC-MS metabolomics arrays in .npy format (341 × 341 retention-time × m/z grid; mzML files are auto-converted to .npy)
- Sample disease-label file (e.g. example/label.txt listing FilePath, Label, Dataset)
- Trained ensemble of 10 DenseNet121 models stored in the job directory (e.g. ../jobs/jobs007)

## Outputs

- feature_results/ensemble_RISE.npy — aggregated per-class RISE contribution heatmaps, shape (6, 341, 341)
- ensemble_RISE.png — class-specific heatmap figure rendered by showFeature.py
- Interpretability evidence (masked-region ablation) that unknown-metabolite signals carry classification-relevant information

## How to apply

Train the ensemble first (10 DenseNet121 runs, adam optimizer, learning rate 1e-4) and store it in a job directory such as jobs007, then execute the feature-extraction stage, which applies the RISE random-masking perturbation procedure to the ensemble in 'ensemble' mode. The aggregated per-class contribution heatmaps are saved to feature_results/ensemble_RISE.npy with the logged shape (6, 341, 341) over the 341 × 341 retention-time × m/z grid. Render class-specific heatmap images with showFeature.py (equivalently show_feature with mode='ensemble'). As the interpretability-rationale step, re-run prediction with high-contribution regions masked: the article's ablation shows excluding unknown metabolites (83.5% of all detected LC-MS features) undermines prediction accuracy, demonstrating that unknown signals carry classification-relevant information. Validate by confirming ensemble_RISE.npy exists with shape (6, 341, 341) and that ensemble_RISE.png is produced by showFeature.py.

## Related tools

- **DeepMSProfiler** (End-to-end package that runs the RISE feature-extraction stage (run_feature) and renders the ensemble heatmaps (show_feature) from the trained job directory) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend framework in which the DenseNet121 ensemble whose predictions are perturbed by RISE is implemented)
- **Keras 2.3.1** (Model-building API used with TensorFlow to define the DenseNet121 architecture being interpreted)
- **CUDA 10.1** (GPU acceleration for the masking-and-inference loops of the RISE feature-extraction stage (CPU-only execution also supported))

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature && python showFeature.py
```

## Evaluation signals

- ensemble_RISE.npy exists in the job's feature_results/ directory
- Array shape of ensemble_RISE.npy matches the demo log value (6, 341, 341)
- ensemble_RISE.png heatmap figure is produced by running showFeature.py / show_feature(mode='ensemble')
- Masking high-contribution regions degrades prediction accuracy relative to the full-raw-feature baseline, confirming the highlighted regions carry classification-relevant signal
- Log emits '[INFO] Save Heatmap to: ../jobs/jobs007/feature_results/ensemble_RISE.npy' and '[INFO] Step2 Feature Done!'

## Limitations

- Pre-trained model access requires contacting the author (<maintainer e-mail, see the repository README>), so external users must first train the ensemble themselves
- Demo/interpretability runs are GPU- and time-intensive (feature step took ~16 min on 2× Tesla P100 in the demo log) and heatmap coverage depends on the RISE masking budget
- Tested with TensorFlow 2.2.0 / Keras 2.3.1 / CUDA 10.1; newer dependency stacks may not be compatible

## Evidence

- [methods] ensemble feature heatmap generation (RISE): [evidence span withheld — anchor status: not_found]
