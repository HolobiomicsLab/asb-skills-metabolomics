---
name: few-shot-task-sampling-from-mass-spectrometry-data
description: Use when SCMS metabolomics data yield too few cells per phenotype for
  conventional deep learning and you need to build N-shot meta-learning tasks (e.g.,
  primary vs metastatic melanoma classification from IGR39/IGR37 single-probe SCMS
  profiles).
when_to_use_negative:
- Input data is raw single-cell mass spectra that still require noise/background subtraction,
  TIC normalization, and peak alignment before task construction
- A large training set of cells is already available, making conventional transfer
  learning or supervised 1D-CNN training sufficient
- 'Phenotypes are already well separated by simpler methods such as PCA (note: PCA
  fails for similar profiles like IGR39/IGR37, which is when meta-learning is warranted)'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: MetaPhenotype
  role: MAML-based meta-learning framework providing dataloade.py (reads dataset.csv),
    meta.py (MAML model code), and base.py entry point for constructing few-shot tasks
    and training the 1D-CNN backbone
  repo: github.com/songyuan93/MetaPhenotype
- name: SCMS metabolomics
  role: Source of the single-cell metabolomics profiles used as the training and evaluation
    data sets for meta-task sampling
- name: Geena2
  role: Online tool used upstream for peak alignment with a mass tolerance of 10 ppm
    before task construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:915288f1b08212e6f4e3e7a5ff94c4ce033f1584dee776e63314193db88ab101
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - SCMS metabolomics
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# few-shot-task-sampling-from-mass-spectrometry-data

## Summary

Constructs MAML meta-training tasks (support/query splits at 1, 5, and 10 shots per class) from pretreated single-cell mass spectrometry (SCMS) metabolomics data so a 1D-CNN MAML model can be trained to classify cell phenotypes with limited samples.

## When to use

Use when SCMS metabolomics data yield too few cells per phenotype for conventional deep learning and you need to build N-shot meta-learning tasks (e.g., primary vs metastatic melanoma classification from IGR39/IGR37 single-probe SCMS profiles).

## When NOT to use

- Input data is raw single-cell mass spectra that still require noise/background subtraction, TIC normalization, and peak alignment before task construction
- A large training set of cells is already available, making conventional transfer learning or supervised 1D-CNN training sufficient
- Phenotypes are already well separated by simpler methods such as PCA (note: PCA fails for similar profiles like IGR39/IGR37, which is when meta-learning is warranted)

## Inputs

- Pretreated SCMS metabolomics feature table (dataset.csv) from single-probe SCMS of untreated IGR39/IGR37 melanoma cells
- Per-cell phenotype labels (primary IGR39 vs metastatic IGR37)

## Outputs

- MAML meta-training task episodes (support/query splits) at 1-, 5-, and 10-shot per class
- Trained MetaPhenotype initializations per backbone and shot setting
- Aggregated AUROC, F1 score, and prediction accuracy results table (meta_training_results.csv)

## How to apply

Start from pretreated SCMS data (peak export, noise/background subtraction, TIC normalization, peak alignment at 10 ppm via Geena2, missing-value removal), stored as dataset.csv. Load the file with the MetaPhenotype dataloader and construct MAML meta-training tasks by sampling support and query cell subsets at 1, 5, and 10 shots per class from the IGR39/IGR37 melanoma cell pair. Instantiate the 1D-CNN backbones (Lecun, LeNet, VGG9) in the MAML framework with an outer-loop optimizer of SGD plus Adam at learning rate 1e-3, and run meta-training for 50 total epochs with 5 inner gradient-descent cycles per task. Rationale: ambient SCMS methods produce relatively small cell numbers, so meta-learning reuses limited data across tasks instead of overfitting a single large training set.

## Related tools

- **MetaPhenotype** (MAML-based meta-learning framework providing dataloade.py (reads dataset.csv), meta.py (MAML model code), and base.py entry point for constructing few-shot tasks and training the 1D-CNN backbone) — github.com/songyuan93/MetaPhenotype
- **SCMS metabolomics** (Source of the single-cell metabolomics profiles used as the training and evaluation data sets for meta-task sampling)
- **Geena2** (Online tool used upstream for peak alignment with a mass tolerance of 10 ppm before task construction)

## Examples

```
python base.py
```

## Evaluation signals

- Meta-evaluation on held-out IGR39/IGR37 tasks reports AUROC, F1 score, and prediction accuracy computed after 50 total epochs of training
- Reproduced MetaPhenotype meta-evaluation reaches high (>90%) classification accuracy with limited new training samples
- LeNet backbone outperforms VGG9 in the 5-shot setting and is selected as the final backbone on efficiency and accuracy grounds
- All AUROC/F1/accuracy values across backbones (Lecun, LeNet, VGG9) and shot settings (1, 5, 10) aggregate into a single meta_training_results.csv

## Limitations

- MAML requires multiple inner-loop gradient steps and backpropagation through those updates in the outer loop, making it memory- and computationally intensive and sensitive to hyperparameters
- Ambient SCMS sample handling constrains live-cell metabolism, so cell numbers per task remain small by design
- Meta-learning is less effective in nonstationary environments

## Evidence

- [intro] 1D-CNN, which is easier to train due to its less computational complexity, would be clearly preferable for analyzing 1D signals, particularly when limited training data are available: [evidence span withheld — anchor status: not_found]
