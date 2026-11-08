---
name: bcewithlogitsloss-weighted-classifier-training
description: Use when training a binary NMR spectral-pair classifier (compound-present
  vs. absent) on imbalanced labeled data in PyTorch, where a sigmoid-constrained probability
  output and a weighted positive-class loss are required to reach high TPR.
when_to_use_negative:
- The task is multi-class classification or regression rather than binary presence/absence
  — BCEWithLogitsLoss with a single positive weight is designed for the paper's two-class
  formulation.
- Inputs are not paired spectral data (e.g. no mixture spectrum to compare against),
  since the bi-encoder/cross-encoder pipeline expects both a compound and a mixture
  spectrum.
- The dataset is already balanced and a weighted loss would distort calibration —
  the weight of 3 is specific to the paper's imbalanced positive-class scenario.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Provides BCEWithLogitsLoss and the model modules (conv layers, TransformerEncoder)
    used to build and train the classifier
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: Python 3.13.2
  role: Runtime environment for running demo.ipynb training and prediction
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: conda
  role: Creates the FlavorFormer virtual environment from environment.yml before training
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: git
  role: Clones the FlavorFormer repository containing demo.ipynb, model/, and data/
    folders
  repo: https://github.com/yfWang01/FlavorFormer.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:246005bc76e0f3149edbbb5d3df443ade3bf7a73193d5eaf53de1f647d95f6c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - FlavorFormer
  - PyTorch
  - Python
  - Python 3.13.2
  - conda
  - git
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bcewithlogitsloss-weighted-classifier-training

## Summary

A training procedure for the FlavorFormer hybrid CNN–Transformer classifier that uses PyTorch's BCEWithLogitsLoss with a positive-class weight of 3 to counter class imbalance when predicting whether a compound is present in an NMR mixture, combined with fusion pooling of the cross-encoder's [CLS] token and mean-pooled sequence features.

## When to use

Use when training a binary NMR spectral-pair classifier (compound-present vs. absent) on imbalanced labeled data in PyTorch, where a sigmoid-constrained probability output and a weighted positive-class loss are required to reach high TPR.

## When NOT to use

- The task is multi-class classification or regression rather than binary presence/absence — BCEWithLogitsLoss with a single positive weight is designed for the paper's two-class formulation.
- Inputs are not paired spectral data (e.g. no mixture spectrum to compare against), since the bi-encoder/cross-encoder pipeline expects both a compound and a mixture spectrum.
- The dataset is already balanced and a weighted loss would distort calibration — the weight of 3 is specific to the paper's imbalanced positive-class scenario.

## Inputs

- Paired 1H NMR spectra (compound spectrum + mixture spectrum) from the data/ folder
- Binary presence/absence labels for each compound–mixture pair
- Augmented training datasets generated per the paper's augmentation workflow

## Outputs

- Trained FlavorFormer model parameters saved to the model/ folder
- Sigmoid-constrained probability scores in [0,1] for compound presence in mixtures
- Test-set metrics (ACC, TPR, FPR) reported per the paper's evaluation

## How to apply

Assemble the FlavorFormer model from the repository's model/ folder: a CNN bi-encoder (input dimension 39,042; three convolutional layers with 64 output channels, kernel size 5, stride 3, batch normalization, ReLU, max-pool of 2, dropout 0.1, and a 1×1 convolution to 128 channels), a Transformer cross-encoder (hidden size 64, 4 layers, 4 attention heads, learnable [CLS]/[SEP] tokens), and a fusion-pooling module combining the [CLS] output with mean pooling over sequence features. Train in demo.ipynb within the conda environment created from environment.yml (Python 3.13.2, PyTorch 2.7.0+cu118) on augmented datasets, using BCEWithLogitsLoss with a positive-class weight of 3 so the minority 'compound-present' class dominates the gradient signal; the classifier head emits a sigmoid score in [0,1]. Save trained weights into the model/ folder so they can be reloaded for prediction on mixture spectra in the data/ folder. The weighted loss plus fusion pooling is what the paper attributes its accuracy gains to, so omitting either changes the reported performance.

## Related tools

- **PyTorch** (Provides BCEWithLogitsLoss and the model modules (conv layers, TransformerEncoder) used to build and train the classifier) — https://github.com/yfWang01/FlavorFormer.git
- **Python 3.13.2** (Runtime environment for running demo.ipynb training and prediction) — https://github.com/yfWang01/FlavorFormer.git
- **conda** (Creates the FlavorFormer virtual environment from environment.yml before training) — https://github.com/yfWang01/FlavorFormer.git
- **git** (Clones the FlavorFormer repository containing demo.ipynb, model/, and data/ folders) — https://github.com/yfWang01/FlavorFormer.git

## Examples

```
git clone https://github.com/yfWang01/FlavorFormer.git; cd FlavorFormer; conda env create -f environment.yml; conda activate FlavorFormer; jupyter notebook demo.ipynb
```

## Evaluation signals

- The model runs forward and backward passes on sample spectra from data/ without errors.
- On the augmented test set, reported metrics fall in the paper's range: ACC ≈ 98.84 ± 0.86 %, TPR ≈ 99.90 ± 0.10 %, FPR ≈ 2.24 ± 1.77 %.
- On the known flavor mixture dataset, ACC ≈ 95.44 ± 0.00 %, TPR ≈ 95.19 ± 1.24 %, FPR ≈ 4.31 ± 0.33 %.
- On the unknown flavor mixture dataset, ACC ≈ 94.44 ± 2.41 %, TPR ≈ 93.33 ± 11.55 %, FPR ≈ 5.26 ± 0.00 %.
- Classifier outputs are bounded in [0,1] (sigmoid-constrained) and match the architecture spec (4 cross-encoder layers, 4 heads, hidden size 64).

## Limitations

- Requires paired compound/mixture NMR spectra; performance degrades on the unknown mixture dataset (TPR variance ± 11.55 %) compared to known mixtures.
- The positive-class weight of 3 is tuned for the paper's class balance; other datasets may need re-tuning.
- Training depends on augmented datasets as described in the paper; training on raw unaugmented data may not reproduce the reported metrics.

## Evidence

- [methods] Training criterion uses BCEWithLogitsLoss with a positive-class weight of 3: [evidence span withheld — anchor status: not_found]
- [abstract] Weighted loss and fusion pooling boost accuracy: "Fusion pooling and weighted loss function significantly boost identification accuracy."
- [abstract] Test-set performance metrics: "The model achieved consistently high performance metrics on the test set (ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %)"
- [readme] Run demo.ipynb for training and prediction: "For training model and predict mixture spectra data, run the file   `demo.ipynb`."
- [readme] Conda environment creation: "Use this command to create python virtual environment."
