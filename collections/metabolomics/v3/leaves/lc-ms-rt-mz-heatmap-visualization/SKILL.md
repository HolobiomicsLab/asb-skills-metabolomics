---
name: lc-ms-rt-mz-heatmap-visualization
description: Use when raw LC-MS data (.npy/.mzML) has been classified by a trained
  DeepMSProfiler ensemble and you need per-class importance heatmaps on the 341×341
  RT×m/z grid, e.g. for unknown-metabolite biomarker inspection.
when_to_use_negative:
- No trained model/ensemble is available in the job directory — run_train and run_pred
  must precede run_feature
- Input is already a peak-picked, annotated feature table rather than raw LC-MS RT×m/z
  signal grids; the heatmap operates on raw metabolomic signals
- You only need classification metrics (AUC/confusion matrix) — use run_predict with
  -plot_auc/-plot_cm instead of feature extraction
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: DeepMSProfiler
  role: Provides run_feature (RISE random-masking perturbation in 'ensemble' mode)
    and showFeature.py/show_feature for heatmap generation and rendering on the LC-MS
    RT×m/z grid
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: Python (>= 3.6)
  role: Runtime for mainRun.py -run_feature and showFeature.py heatmap commands
- name: TensorFlow 2.2.0
  role: Backend executing the DenseNet121 ensemble during RISE perturbation-based
    feature attribution
- name: Keras 2.3.1
  role: Model API for loading and perturbing the trained DenseNet121 ensemble
- name: CUDA 10.1
  role: GPU acceleration for RISE masking over the ensemble (CPU-only also compatible)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:5dd885ffe346014c1ce72f11ccef372aab570e89263254b45a4e6d2f0d94d3e2
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
  - Python (>= 3.6)
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lc-ms-rt-mz-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates and renders class-specific feature-contribution heatmaps over the retention-time × m/z grid of raw LC-MS data, using RISE random-masking perturbation on a trained DeepMSProfiler ensemble to visualize which metabolite signals drive disease classification.

## When to use

Use when raw LC-MS data (.npy/.mzML) has been classified by a trained DeepMSProfiler ensemble and you need per-class importance heatmaps on the 341×341 RT×m/z grid, e.g. for unknown-metabolite biomarker inspection.

## When NOT to use

- No trained model/ensemble is available in the job directory — run_train and run_pred must precede run_feature
- Input is already a peak-picked, annotated feature table rather than raw LC-MS RT×m/z signal grids; the heatmap operates on raw metabolomic signals
- You only need classification metrics (AUC/confusion matrix) — use run_predict with -plot_auc/-plot_cm instead of feature extraction

## Inputs

- Trained DeepMSProfiler ensemble in a job directory (e.g. jobs007; 10 DenseNet121 runs, adam, lr 1e-4)
- Raw metabolomics arrays in .npy format (demo downloadable from the example dir or Baidu Netdisk mirror, password acaa; mzML files are auto-converted to .npy)
- Sample disease-type label file (e.g. ../example/label.txt listing cancer/health/nodule classes)

## Outputs

- Aggregated per-class RISE contribution heatmap array: feature_results/ensemble_RISE.npy with shape (6, 341, 341)
- Rendered class-specific heatmap figure ensemble_RISE.png via showFeature.py

## How to apply

First ensure a trained ensemble exists in the job directory (e.g. jobs007, 10 DenseNet121 runs, adam optimizer, lr 1e-4) alongside the demo .npy arrays and disease-label file. Execute the feature-extraction stage with 'python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature', which applies the RISE random-masking perturbation procedure to the ensemble in 'ensemble' mode and aggregates per-class contribution maps. Verify the output ensemble_RISE.npy matches the demo log shape (6, 341, 341) over the 341×341 retention-time × m/z grid. Then render class-specific heatmap images by running 'python showFeature.py' (equivalently show_feature with mode='ensemble'), producing ensemble_RISE.png. As an interpretability check, the resulting high-contribution regions can be masked for ablation re-runs to demonstrate accuracy degradation, confirming the highlighted signals (including unannotated metabolites) carry classification-relevant information.

## Related tools

- **DeepMSProfiler** (Provides run_feature (RISE random-masking perturbation in 'ensemble' mode) and showFeature.py/show_feature for heatmap generation and rendering on the LC-MS RT×m/z grid) — https://github.com/yjdeng9/DeepMSProfiler
- **Python (>= 3.6)** (Runtime for mainRun.py -run_feature and showFeature.py heatmap commands)
- **TensorFlow 2.2.0** (Backend executing the DenseNet121 ensemble during RISE perturbation-based feature attribution)
- **Keras 2.3.1** (Model API for loading and perturbing the trained DenseNet121 ensemble)
- **CUDA 10.1** (GPU acceleration for RISE masking over the ensemble (CPU-only also compatible))

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature; python showFeature.py
```

## Evaluation signals

- ensemble_RISE.npy exists in ../jobs/jobs007/feature_results/ and its array shape equals the logged value (6, 341, 341) over the 341×341 RT×m/z grid
- Log line '[INFO] Save Heatmap to: ../jobs/jobs007/feature_results/ensemble_RISE.npy' appears and Step2 Feature completes
- showFeature.py (or show_feature with mode='ensemble') produces the class-specific heatmap figure ensemble_RISE.png
- Masking high-contribution regions and re-running prediction degrades accuracy relative to the unknown-metabolite baseline, confirming the heatmaps highlight classification-relevant signals
- The number of heatmap channels (6) is consistent with the ensemble-mode aggregation over model runs and classes

## Limitations

- Pre-trained model access requires contacting the author; without it you must train the ensemble yourself before run_feature
- Demo runs were validated on specific pinned versions (TensorFlow 2.2.0, Keras 2.3.1, CUDA 10.1); other stacks may behave differently
- Heatmap dimensions are fixed by the input grid (341×341 in the demo); other data shapes will yield different heatmap dimensions
- Feature extraction is computationally non-trivial (RISE masking of a 10-model ensemble; demo feature step ran ~15 min on dual Tesla P100 GPUs)

## Evidence
