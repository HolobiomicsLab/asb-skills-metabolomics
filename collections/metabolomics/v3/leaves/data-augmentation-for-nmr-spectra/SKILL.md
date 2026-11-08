---
name: data-augmentation-for-nmr-spectra
description: Use when training FlavorFormer (or a similar CNN–Transformer classifier)
  on 1H NMR spectra of flavor mixtures and the labelled training set is too small
  to achieve the reported test ACC ≈ 98.84 % and TPR ≈ 99.90 %.
when_to_use_negative:
- When the goal is to identify compounds in mixtures without any labelled pure-standard
  NMR data to augment — FlavorFormer requires a library of standards to train on.
- When spectral data are not 1H NMR spectra (e.g. LC-MS, FTIR) — the augmentation
  and architecture are specific to 1H NMR feature extraction.
- When the unknown mixture contains a compound outside the training standard library
  and no retraining/augmentation plan has been prepared — the original closed library
  will not cover it.
tools:
- name: FlavorFormer
  role: Hybrid CNN–Transformer model that consumes the augmented 1H NMR spectra; trained
    and evaluated with this augmentation strategy
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: PyTorch 2.7.0+cu118
  role: Deep-learning backend for training FlavorFormer on augmented NMR datasets
- name: conda
  role: Creates the reproducible Python virtual environment from environment.yml for
    the augmentation and training pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:d73592e37b754d26d94075e2b67d181ddda7b9d2f4e4be532fa596d2aac8252e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# data-augmentation-for-nmr-spectra

## Summary

Augments 1H NMR spectra training data so that a hybrid CNN–Transformer identifier (FlavorFormer) learns robust local and global features without overfitting, enabling high TPR/ACC on both synthetic and experimental mixture datasets.

## When to use

Use when training FlavorFormer (or a similar CNN–Transformer classifier) on 1H NMR spectra of flavor mixtures and the labelled training set is too small to achieve the reported test ACC ≈ 98.84 % and TPR ≈ 99.90 %.

## When NOT to use

- When the goal is to identify compounds in mixtures without any labelled pure-standard NMR data to augment — FlavorFormer requires a library of standards to train on.
- When spectral data are not 1H NMR spectra (e.g. LC-MS, FTIR) — the augmentation and architecture are specific to 1H NMR feature extraction.
- When the unknown mixture contains a compound outside the training standard library and no retraining/augmentation plan has been prepared — the original closed library will not cover it.

## Inputs

- 1H NMR spectra of pure flavor standards (the 24-compound training library)
- 1H NMR spectra of flavor mixtures (known and unknown experimental datasets)
- environment.yml (Python 3.13.2, PyTorch 2.7.0+cu118) and data/ folder contents

## Outputs

- Augmented training dataset and augmented test set ready for FlavorFormer training
- Trained FlavorFormer model parameters saved to the model/ folder
- Evaluation metrics (ACC, TPR, FPR) reported as mean ± standard deviation on test and mixture datasets

## How to apply

Generate augmented spectra variants before model training so the hybrid CNN–Transformer (bi-encoder + cross-encoder with fusion pooling and weighted loss) sees diverse perturbations of each 1H NMR standard and of mixture spectra. Train FlavorFormer on the augmented datasets, then evaluate on an augmented test set and on two experimental datasets (known and unknown flavor mixtures) reporting ACC, TPR and FPR as mean ± standard deviation over repeated runs. The augmentation supports generalisation beyond the closed 24-standard compound library — as demonstrated by retraining on 21 standards (Lactic acid, Isopentyl acetate, γ-Nonanoic lactone removed) with test ACC 99.27 ± 0.33 % and TPR 99.87 ± 0.11 %. Use conda-managed Python 3.13.2 with PyTorch 2.7.0+cu118 and the model/data folders in the repository for reloading and predicting.

## Related tools

- **FlavorFormer** (Hybrid CNN–Transformer model that consumes the augmented 1H NMR spectra; trained and evaluated with this augmentation strategy) — https://github.com/yfWang01/FlavorFormer.git
- **PyTorch 2.7.0+cu118** (Deep-learning backend for training FlavorFormer on augmented NMR datasets)
- **conda** (Creates the reproducible Python virtual environment from environment.yml for the augmentation and training pipeline)

## Examples

```
cd FlavorFormer; conda env create -f environment.yml; conda activate FlavorFormer; jupyter notebook demo.ipynb
```

## Evaluation signals

- Test-set metrics after training on augmented data should match the article's reported values: ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %.
- Known flavor mixture dataset should yield ACC 95.44 ± 0.00 %, TPR 95.19 ± 1.24 %, FPR 4.31 ± 0.33 %.
- Unknown flavor mixture dataset should yield ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %.
- Retraining on 21 standards (removing Lactic acid, Isopentyl acetate, γ-Nonanoic lactone) should still give test ACC ≈ 99.27 % and TPR ≈ 99.87 %, confirming augmentation supports generalisation beyond the closed library.
- Trained model parameters should be present in the model/ folder and be reloadable for easy prediction as stated in the README.

## Limitations

- FlavorFormer's original library is closed to the 24 flavor standards; compounds outside this set require retraining with augmented data on a reduced standard set.
- Unknown mixtures show higher variance (TPR 93.33 ± 11.55 %), indicating augmentation does not fully stabilise performance on truly novel samples.
- Augmentation supports training but does not replace the need for a bi-encoder + cross-encoder, fusion pooling, and weighted loss function to achieve the reported accuracy.

## Evidence

- [abstract] abstract quote: "After training on augmented datasets, FlavorFormer was tested on an augmented test set and two experimental datasets."
- [methods] methods quote: [evidence span withheld — anchor status: not_found]
- [readme] readme quote: "For training model and predict mixture spectra data, run the file `demo.ipynb`."
- [intro] intro quote: "it leverages a combination of a bi-encoder and cross-encoder, a fusion pooling strategy, and a weighted loss function to identify compounds correctly."
