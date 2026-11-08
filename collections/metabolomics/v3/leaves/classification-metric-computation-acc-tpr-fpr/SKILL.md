---
name: classification-metric-computation-acc-tpr-fpr
description: Use when a binary compound-identification model (e.g., FlavorFormer on
  1H NMR spectra) must be evaluated and its variants compared using ACC, TPR, and
  FPR reported as mean ± std across runs.
when_to_use_negative:
- The task requires metrics beyond ACC/TPR/FPR at the operating threshold (e.g., AUC,
  precision-recall curves, per-component confusion matrices) that the FlavorFormer
  evaluation does not report
- Predictions come from a single run with no replicates, so mean ± standard deviation
  across runs cannot be computed
- The model under evaluation is not trained with the weighted loss function under
  identical hyperparameters, making metric comparisons against the FlavorFormer baselines
  invalid
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: FlavorFormer
  role: Source model whose bi-encoder + cross-encoder predictions are evaluated for
    ACC/TPR/FPR across pooling strategies
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: PyTorch
  role: Framework for re-instantiating the model with each pooling head and running
    inference to generate predictions for metric computation
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: Python 3.13.2
  role: Runtime for the demo.ipynb training/inference pipeline that produces the prediction
    outputs used in metric calculation
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: conda
  role: Creates the reproducible virtual environment (environment.yml) in which metric
    computations are run
  repo: https://github.com/yfWang01/FlavorFormer.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:cd94518f9fb9f44e5c935e106bd112921255e78fef7b9ace8dee296b9e3ce262
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - Anaconda
  - FlavorFormer
  - PyTorch
  - Python
  - Python 3.13.2
  - conda
  - git
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# classification-metric-computation-acc-tpr-fpr

## Summary

Computes and reports ACC, TPR, and FPR (as mean ± standard deviation) for NMR-based compound identification classifiers such as FlavorFormer, on an augmented test set and experimental mixture datasets, enabling head-to-head comparison of model variants (e.g., fusion vs. mean vs. weighted vs. attention vs. CLS pooling).

## When to use

Use when a binary compound-identification model (e.g., FlavorFormer on 1H NMR spectra) must be evaluated and its variants compared using ACC, TPR, and FPR reported as mean ± std across runs.

## When NOT to use

- The task requires metrics beyond ACC/TPR/FPR at the operating threshold (e.g., AUC, precision-recall curves, per-component confusion matrices) that the FlavorFormer evaluation does not report
- Predictions come from a single run with no replicates, so mean ± standard deviation across runs cannot be computed
- The model under evaluation is not trained with the weighted loss function under identical hyperparameters, making metric comparisons against the FlavorFormer baselines invalid

## Inputs

- Pretrained FlavorFormer model parameters (model/ folder) and preprocessed 1H NMR spectral data (data/ folder)
- Predicted vs. ground-truth compound-identification labels for the augmented test set
- Predicted vs. ground-truth labels for the 21-component known flavor mixture dataset
- Per-variant prediction outputs from each pooling head (fusion, mean, weighted, attention, CLS-only)

## Outputs

- Table of ACC (%), TPR (%), FPR (%) with standard deviations per pooling strategy and dataset
- Reproduced baseline row for fusion pooling (test set ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %)
- Per-dataset metric breakdown distinguishing augmented test set from known-mixture dataset performance

## How to apply

Run the trained model's inference pipeline (demo.ipynb in the FlavorFormer repo, with pretrained parameters from model/ and data from data/) to obtain predictions on the augmented test set and the 21-component known-mixture dataset. For each pooling-strategy variant (fusion, mean, weighted, attention, CLS-only), retrained under identical hyperparameters with the weighted loss function, compute ACC, TPR, and FPR and report each as mean ± standard deviation across repeated runs. The fusion-pooling baseline must reproduce the reported figures — test set ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %, and known-mixture dataset ACC 95.44 ± 0.00 %, TPR 95.19 ± 1.24 %, FPR 4.31 ± 0.33 % — within reported standard deviations. Assemble per-strategy rows into a single comparison table (pooling strategy, dataset, ACC %, TPR %, FPR %, std) and verify alternative pooling rows show the expected relative decrease in accuracy, e.g., weighted pooling at ACC 96.70 ± 2.79 % and attention pooling dropping to TPR 88.02 ± 3.41 % on the known mixture dataset versus fusion's 95.19 ± 1.24 %.

## Related tools

- **FlavorFormer** (Source model whose bi-encoder + cross-encoder predictions are evaluated for ACC/TPR/FPR across pooling strategies) — https://github.com/yfWang01/FlavorFormer.git
- **PyTorch** (Framework for re-instantiating the model with each pooling head and running inference to generate predictions for metric computation) — https://github.com/yfWang01/FlavorFormer.git
- **Python 3.13.2** (Runtime for the demo.ipynb training/inference pipeline that produces the prediction outputs used in metric calculation) — https://github.com/yfWang01/FlavorFormer.git
- **conda** (Creates the reproducible virtual environment (environment.yml) in which metric computations are run) — https://github.com/yfWang01/FlavorFormer.git

## Examples

```
conda env create -f environment.yml; conda activate FlavorFormer; jupyter notebook demo.ipynb
```

## Evaluation signals

- Fusion-pooling row reproduces reported test-set metrics within standard deviations: ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %
- Fusion-pooling row reproduces known-mixture metrics within standard deviations: ACC 95.44 ± 0.00 %, TPR 95.19 ± 1.24 %, FPR 4.31 ± 0.33 %
- Alternative pooling rows (weighted: ACC 96.70 ± 2.79 %; mean: ACC 97.94 ± 1.88 %) show the expected relative decrease versus fusion
- Attention pooling exhibits the characteristic generalization gap: test TPR 99.73 ± 0.08 % but known-mixture TPR 88.02 ± 3.41 % vs. fusion's 95.19 ± 1.24 %
- All metrics reported as mean ± standard deviation over repeated runs, in a table with columns: pooling strategy, dataset, ACC (%), TPR (%), FPR (%), standard deviation

## Limitations

- Metrics are threshold-dependent ACC/TPR/FPR from the FlavorFormer operating point; no ROC/AUC-style threshold sweep is described
- Standard deviations can be extreme on small experimental sets (e.g., unknown-mixture dataset TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 % with 0.00 std), so single-dataset comparisons may be unstable
- Comparability requires identical hyperparameters and the weighted loss function across all pooling variants; deviations invalidate the expected relative ordering

## Evidence

- [abstract] The model achieved consistently high performance metrics on the test set (ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %): "The model achieved consistently high performance metrics on the test set (ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %)"
- [abstract] Known-mixture dataset baseline metrics: "a known flavor mixture dataset (ACC 95.44 ± 0.00 %, TPR 95.19 ± 1.24 %, FPR 4.31 ± 0.33 %)"
- [methods] Where the evaluation pipeline runs: "For training model and predict mixture spectra data, run the file   `demo.ipynb`."
- [readme] Pretrained parameters and data locations for reloading the model before evaluation: [evidence span withheld — anchor status: not_found]
