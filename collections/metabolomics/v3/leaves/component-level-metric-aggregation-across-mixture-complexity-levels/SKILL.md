---
name: component-level-metric-aggregation-across-mixture-complexity-levels
description: Use when benchmarking multi-component identification from 1H NMR spectra
  and you need mixture-level metrics broken down by component count (2–5 components)
  rather than a single pooled score.
when_to_use_negative:
- Evaluating single-compound spectra or the augmented test set, where plain ACC/TPR/FPR
  (e.g., FlavorFormer's test-set ACC 98.84 ± 0.86 %) is the appropriate metric rather
  than mixture-level aggregation
- When only component-level (per-compound) metrics are needed and no ground-truth
  component lists per mixture spectrum are available to compute mean TPR across correct
  components
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: FlavorFormer
  role: Model whose predictions on the known-mixture dataset are scored with M-ACC
    and M-TPR per component-count level
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: PyTorch 2.7.0+cu118
  role: Framework used to reload the pretrained FlavorFormer model and run inference
    on mixture spectra
- name: conda
  role: Creates the Python virtual environment from environment.yml to run the evaluation
    pipeline
- name: git
  role: Clones the FlavorFormer repository containing model weights, data, and demo.ipynb
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:090040a4b22531c467ef6272d241ad47fb22dca60cc4a28b9b4254c99cc6bd53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - Anaconda
  - FlavorFormer
  - PyTorch 2.7.0+cu118
  - Python
  - conda
  - git
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# Component-level metric aggregation across mixture complexity levels

## Summary

Evaluates NMR mixture-identification models (FlavorFormer, pSCNN, NMRformer) using per-spectrum mixture accuracy (MA/M-ACC) and mean true-positive rate across correct components (M-TPR/M-PTPR), aggregated both as overall averages and separately at each mixture complexity level (2-, 3-, 4-, and 5-component mixtures).

## When to use

Use when benchmarking multi-component identification from 1H NMR spectra and you need mixture-level metrics broken down by component count (2–5 components) rather than a single pooled score.

## When NOT to use

- Evaluating single-compound spectra or the augmented test set, where plain ACC/TPR/FPR (e.g., FlavorFormer's test-set ACC 98.84 ± 0.86 %) is the appropriate metric rather than mixture-level aggregation
- When only component-level (per-compound) metrics are needed and no ground-truth component lists per mixture spectrum are available to compute mean TPR across correct components

## Inputs

- Known flavor mixture dataset of 1H NMR spectra from the data/ folder (2- to 5-component mixtures)
- Pretrained FlavorFormer model parameters from the model/ folder
- Baseline model predictions for the same spectra from pSCNN and NMRformer
- Ground-truth component labels for each mixture spectrum

## Outputs

- Per-spectrum M-ACC (MA) and M-TPR (M-PTPR) values for each model
- Per-component-count metrics aggregated over 2, 3, 4, and 5 component levels
- Overall average MA and M-PTPR per model (e.g., FlavorFormer MA 46.03%, M-PTPR 82.54%)
- Consolidated metrics table (e.g., reproduction_metrics.csv)

## How to apply

Run each model's inference (e.g., the demo.ipynb inference portion with pretrained weights from the model/ folder) over the known-mixture dataset in the data/ folder. For each mixture spectrum, compute per-spectrum identification accuracy and the mean true-positive rate across the correct components to obtain M-ACC (MA) and M-TPR (M-PTPR). Aggregate these into overall averages and into per-component-count levels (2, 3, 4, 5 components) so that performance degradation with complexity is visible. Assemble per-level and average metrics for all models (e.g., into reproduction_metrics.csv) and compare against reported reference values. Validation: reproduced FlavorFormer averages should fall near the reported MA 46.03% and M-PTPR 82.54%, with the per-level pattern reproduced (e.g., 2-component M-PTPR 100%, 5-component M-PTPR 46.67% vs pSCNN 0%); if only partial reproduction is possible, report exact reproduced values alongside reference values.

## Related tools

- **FlavorFormer** (Model whose predictions on the known-mixture dataset are scored with M-ACC and M-TPR per component-count level) — https://github.com/yfWang01/FlavorFormer.git
- **PyTorch 2.7.0+cu118** (Framework used to reload the pretrained FlavorFormer model and run inference on mixture spectra)
- **conda** (Creates the Python virtual environment from environment.yml to run the evaluation pipeline)
- **git** (Clones the FlavorFormer repository containing model weights, data, and demo.ipynb)

## Examples

```
git clone https://github.com/yfWang01/FlavorFormer.git; cd FlavorFormer; conda env create -f environment.yml; conda activate FlavorFormer; jupyter notebook demo.ipynb
```

## Evaluation signals

- Reproduced FlavorFormer average MA falls within the reported range (46.03%) and average M-PTPR within 82.54%
- Per-level pattern reproduces reported values: 2-component M-PTPR = 100% and 5-component M-PTPR = 46.67%
- Baseline ordering matches the paper: FlavorFormer M-PTPR (82.54%) > pSCNN (47.62%) > NMRformer (14.29%), and MA 46.03% > 28.57% > 4.76%
- Metrics are reported both as overall averages and at each component-count level (2, 3, 4, 5), not only pooled
- Reproduction metrics are exported with reference values side by side (reproduction_metrics.csv), flagging any partial reproduction

## Limitations

- Performance degrades sharply with mixture complexity: FlavorFormer achieves 100% M-PTPR on 2-component mixtures but only 46.67% on 5-component mixtures
- Metrics depend on having exact ground-truth component lists per spectrum; mixture-level accuracy differs from the much higher single-component test-set metrics (ACC 98.84 ± 0.86 %)
- Reported values are dataset-specific (known flavor mixture dataset); unknown-mixture behavior differs (TPR 93.33 ± 11.55 %)

## Evidence

- [other] On the known mixture dataset, FlavorFormer achieves average MA of 46.03% vs pSCNN 28.57% and NMRformer 4.76%: [evidence span withheld — anchor status: not_found]
- [other] Per-level M-PTPR pattern: [evidence span withheld — anchor status: not_found]
- [other] Metric computation procedure: [evidence span withheld — anchor status: not_found]
- [methods] Inference run instructions: "The model parameters is located in the [model](model/) folder and the required data is in the [data](data/) folder, FlavorFormer can be reload and predict easily."
- [abstract] Reference test-set metrics for contrast: "The model achieved consistently high performance metrics on the test set (ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %)"
