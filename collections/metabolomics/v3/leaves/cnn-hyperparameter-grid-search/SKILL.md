---
name: cnn-hyperparameter-grid-search
description: Use when training the pSCNN model on augmented NMR mixture spectra and
  you must determine which epochs, learning rate, and convolutional layer count yield
  the validation accuracy before final model selection.
when_to_use_negative:
- The pretrained pSCNN model from Zenodo is already adequate — download it and predict
  directly instead of re-running the grid search
- The model architecture or task is unrelated to 1D-CNN classification of NMR mixture
  spectra, so the specific grid (learning rate 10^-2 to 10^-5, 5-10 conv layers) does
  not apply
- Only benchmarking pSCNN on the experimental flavor mixtures datasets is needed —
  no retraining or hyperparameter tuning is involved
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0593
tools:
- name: pSCNN
  role: The pseudo Siamese convolutional neural network model whose variants are trained
    across the hyperparameter grid
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep-learning framework in which pSCNN variants are instantiated and trained
- name: Anaconda
  role: Environment manager used to create the Python 3.8.13 environment for the grid
    search
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress monitoring during per-variant training runs
- name: Python 3.8.13
  role: Runtime interpreter pinned for the pSCNN environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:005d9985aac33bccfb896e8358c1616d83fa424320a9c503ded9455f4d7331e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN.git
  tools:
  - Anaconda
  - Python
  - Python 3.8.13
  - TensorFlow 2.5.0-GPU
  - pSCNN
  - tqdm
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cnn-hyperparameter-grid-search

## Summary

Grid-search hyperparameters (epochs, learning rate, number of convolutional layers) for the pSCNN 1D-CNN used to identify compounds in NMR spectra of mixtures, and select the best model by validation accuracy.

## When to use

Use when training the pSCNN model on augmented NMR mixture spectra and you must determine which epochs, learning rate, and convolutional layer count yield the best validation accuracy before final model selection.

## When NOT to use

- The pretrained pSCNN model from Zenodo is already adequate — download it and predict directly instead of re-running the grid search
- The model architecture or task is unrelated to 1D-CNN classification of NMR mixture spectra, so the specific grid (learning rate 10^-2 to 10^-5, 5-10 conv layers) does not apply
- Only benchmarking pSCNN on the experimental flavor mixtures datasets is needed — no retraining or hyperparameter tuning is involved

## Inputs

- 18,000-pair training set of augmented NMR mixture spectra (superposition of spectra sampled from a spectral database with random noises)
- 2,000-pair validation set of augmented mixture spectra
- pSCNN.py training script from Zenodo DOI 10.5281/zenodo.6504814
- Hyperparameter grid: learning rates 10^-2 to 10^-5, 5-10 convolutional layers, epochs fixed at 100

## Outputs

- Table 1-style grid of final validation accuracies per configuration (hyperparameter_grid_results.csv)
- Selected model checkpoint pscnn_model_M3.h5 (epoch = 100, learning rate = 10^-4, 6 convolutional layers)
- Per-epoch validation accuracy logs tracing convergence behavior (Figure 4b)

## How to apply

Set up an Anaconda environment with Python 3.8.13 and TensorFlow 2.5.0-GPU, install tqdm via pip, and download the pSCNN model, augmented NMR data, and pSCNN.py training script from Zenodo (DOI 10.5281/zenodo.6504814). Instantiate pSCNN variants sweeping learning rates from 10^-2 down to 10^-5 crossed with 5 to 10 convolutional layers, with epochs fixed at 100, training on 18,000 pairs of augmented mixture spectra and monitoring per-epoch validation accuracy on 2,000 validation pairs with tqdm. Tabulate final validation accuracies across the grid to reproduce Table 1 and select the top configuration. In this study the search selected epoch = 100, learning rate = 10^-4, and 6 convolutional layers, yielding model M3 with a validation accuracy of 0.9990, saved as pscnn_model_M3.h5.

## Related tools

- **pSCNN** (The pseudo Siamese convolutional neural network model whose variants are trained across the hyperparameter grid) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep-learning framework in which pSCNN variants are instantiated and trained)
- **Anaconda** (Environment manager used to create the Python 3.8.13 environment for the grid search) — https://www.anaconda.com/
- **tqdm** (Progress monitoring during per-variant training runs)
- **Python 3.8.13** (Runtime interpreter pinned for the pSCNN environment)

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; pip install tqdm; python pSCNN.py
```

## Evaluation signals

- Grid includes the full reported sweep: learning rates 10^-2, 10^-3, 10^-4, 10^-5 crossed with 5-10 convolutional layers at 100 epochs
- Top configuration reproduces M3: epoch = 100, learning rate = 10^-4, 6 convolutional layers, validation accuracy = 0.9990
- Best augmented test-set performance of the selected model is consistent with ACC = 99.80%, TPR = 99.70%, FPR = 0.10%
- Selected checkpoint saved as pscnn_model_M3.h5 and grid results exported as hyperparameter_grid_results.csv
- Training runs complete under the pinned environment: Python 3.8.13 with TensorFlow 2.5.0-GPU

## Limitations

- The grid is bounded to the paper's reported ranges (learning rates 10^-2 to 10^-5, 5-10 convolutional layers, 100 epochs); configurations outside it were not explored
- The pretrained model exceeded repository size limits and is distributed via Zenodo rather than GitHub, so data/model download is a prerequisite
- Hyperparameter selection is based on augmented (simulated) mixture spectra; real-application accuracy is lower (97.62% on flavor mixtures, 91.67% on the additional flavor mixture dataset)

## Evidence

- [intro] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [methods] Run pSCNN.py; model and data on Zenodo: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [methods] Zenodo DOI for model and NMR data: "zenodo.org/badge/DOI/10.5281/zenodo.6504814.svg)](https://doi.org/10.5281/zenodo.6504814)"
- [readme] Conda environment creation command: "conda create --name pSCNN python=3.8.13"
- [abstract] Data augmentation and dataset splitting for training pSCNN: "The augmented dataset was split and used to train, validate and test the pSCNN model."
