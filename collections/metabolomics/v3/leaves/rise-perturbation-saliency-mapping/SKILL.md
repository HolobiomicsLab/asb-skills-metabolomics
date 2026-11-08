---
name: rise-perturbation-saliency-mapping
description: Use when you have a trained CNN ensemble on raw LC-MS .npy grids (e.g.
  341×341 RT×m/z arrays) and need per-class metabolite contribution heatmaps or to
  test whether unannotated metabolite signals carry classification-relevant information.
when_to_use_negative:
- Input is an already peak-picked, aligned, annotated feature table — DeepMSProfiler's
  RISE stage operates end-to-end on raw metabolic signals, bypassing peak extraction
  and identification
- No trained ensemble or job directory (e.g. jobs007) exists yet — run_train and run_pred
  must complete before run_feature can attribute contributions
- You need metabolite identities rather than spatial signal-contribution localization
  — the heatmap highlights RT×m/z regions but does not annotate metabolites
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: Implements the RISE ensemble feature-extraction stage (run_feature) and heatmap
    rendering (showFeature.py) on raw LC-MS data
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow
  role: Runtime backend (v2.2.0) executing the DenseNet121 ensemble during RISE masking
    perturbation
- name: Keras
  role: Model API (v2.3.1) hosting the ensemble whose predictions are perturbed by
    RISE masking
- name: CUDA
  role: GPU acceleration for the masking sweeps over the 341×341 grid (CUDA 10.1;
    CPU-only environments also compatible)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:729f5f2eda6920abb9dabc718aedd0bcf1a7149645b8cd8ca4f4c5e3cb7fc381
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
  - DeepMSProfiler
  - Keras
  - Python
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# rise-perturbation-saliency-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply RISE (Randomized Input Sampling for Explanation) masking over raw LC-MS metabolomic arrays to attribute class-discriminative signal regions to a trained DenseNet121 ensemble, producing per-class contribution heatmaps on the retention-time × m/z grid that retain unknown (unannotated) metabolites.

## When to use

Use when you have a trained CNN ensemble on raw LC-MS .npy grids (e.g. 341×341 RT×m/z arrays) and need per-class metabolite contribution heatmaps or to test whether unannotated metabolite signals carry classification-relevant information.

## When NOT to use

- Input is an already peak-picked, aligned, annotated feature table — DeepMSProfiler's RISE stage operates end-to-end on raw metabolic signals, bypassing peak extraction and identification
- No trained ensemble or job directory (e.g. jobs007) exists yet — run_train and run_pred must complete before run_feature can attribute contributions
- You need metabolite identities rather than spatial signal-contribution localization — the heatmap highlights RT×m/z regions but does not annotate metabolites

## Inputs

- Raw LC-MS metabolomics arrays in .npy format (mzML files are auto-converted to .npy)
- Sample disease-label file (e.g. example/label.txt mapping C01.npy→cancer, H01.npy→health, N01.npy→nodule)
- Trained ensemble checkpoint in the job directory (e.g. jobs007: 10 DenseNet121 runs, adam optimizer, lr 1e-4)

## Outputs

- Aggregated per-class RISE contribution heatmap ensemble_RISE.npy, shape (6, 341, 341) on the demo data
- Class-specific heatmap images (ensemble_RISE.png) rendered from ensemble_RISE.npy
- Masked-region ablation accuracy comparison relative to the complete-feature baseline

## How to apply

Load the demo metabolomics .npy arrays (converted automatically from mzML when needed) with the disease-label file, and point the feature-extraction stage at a completed job directory holding the 10-run DenseNet121 ensemble (adam, lr 1e-4). Run the feature stage via 'python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature', which applies the RISE random-masking perturbation procedure in 'ensemble' mode and aggregates per-class contribution maps across the ensemble. The result is saved to feature_results/ensemble_RISE.npy with the demo-logged shape (6, 341, 341) over the 341 × 341 retention-time × m/z grid. Render class-specific heatmap images with 'python showFeature.py' (equivalently show_feature with mode='ensemble'). For the unknown-metabolite ablation, re-run prediction with high-contribution regions masked and compare accuracy against the complete-feature baseline — the article reports 83.5% of detected LC-MS features remain unknown, and excluding them undermines prediction accuracy.

## Related tools

- **DeepMSProfiler** (Implements the RISE ensemble feature-extraction stage (run_feature) and heatmap rendering (showFeature.py) on raw LC-MS data) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow** (Runtime backend (v2.2.0) executing the DenseNet121 ensemble during RISE masking perturbation)
- **Keras** (Model API (v2.3.1) hosting the ensemble whose predictions are perturbed by RISE masking)
- **CUDA** (GPU acceleration for the masking sweeps over the 341×341 grid (CUDA 10.1; CPU-only environments also compatible))

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature
```

## Evaluation signals

- feature_results/ensemble_RISE.npy exists and its array shape matches the demo log value (6, 341, 341)
- showFeature.py produces the heatmap figure ensemble_RISE.png from the saved .npy
- Heatmap channels cover all 6 per-class contribution maps over the 341 × 341 retention-time × m/z grid
- Masking high-contribution regions and re-running prediction shows accuracy degradation relative to the complete-feature baseline, consistent with unknown metabolites carrying classification-relevant information

## Limitations

- RISE saliency requires the trained ensemble from a completed job directory; the pre-trained 859-sample serum model (210 healthy, 323 lung nodules, 326 lung cancer) requires contacting the author for access
- Environment is pinned to Python >= 3.6, TensorFlow 2.2.0, Keras 2.3.1, and was tested on CentOS 7 with CUDA 10.1; newer stacks may break the feature stage
- The demo heatmap grid is fixed at 341 × 341; other input resolutions will change the saved heatmap shape and the demo log cannot be used as a shape check

## Evidence
