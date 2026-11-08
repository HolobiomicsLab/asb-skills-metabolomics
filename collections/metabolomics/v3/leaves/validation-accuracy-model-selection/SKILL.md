---
name: validation-accuracy-model-selection
description: Reference for choosing hyperparameters (learning rate, number of convolutional
  layers, epochs) and the final checkpoint for a CNN trained on augmented NMR mixture
  spectra, based on validation accuracy.
when_to_use_negative:
- Input spectra are single-compound rather than mixtures, so compound identification
  does not need the pSCNN mixture model being tuned
- No labeled ground truth is available to compute a validation accuracy, so grid selection
  cannot be scored
- The goal is evaluating real-world performance on experimental spectra (e.g., flavor
  mixtures benchmarking) rather than selecting training hyperparameters
tools:
- name: Python 3.8.13
  role: Runtime environment for the hyperparameter sweep and training
- name: TensorFlow 2.5.0-GPU
  role: Framework used to instantiate and train pSCNN variants across the hyperparameter
    grid
- name: Anaconda
  role: Environment manager for installing the pSCNN dependencies (conda env with
    Python 3.8.13)
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress monitoring during per-variant training runs
- name: pSCNN
  role: The pseudo Siamese CNN model whose learning rate, layer count, and epoch settings
    are optimized
  repo: https://github.com/yuxuanliao/pSCNN.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:8c43766e4ebcdd9669a8c8872c0e0f4dc40cff531962863a420a226bc02d94e1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Anaconda
  - Python
  - Python 3.8.13
  - TensorFlow 2.5.0-GPU
  - pSCNN
  - tqdm
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# validation-accuracy-model-selection

## Summary

Selects the final pSCNN model (M3) for compound identification in NMR mixture spectra by sweeping learning rates (10^-2 to 10^-5) and convolutional layer counts (5–10) at 100 epochs, then choosing the configuration with the highest validation accuracy (0.9990).

## When to use

Reference for choosing hyperparameters (learning rate, number of convolutional layers, epochs) and the final checkpoint for a CNN trained on augmented NMR mixture spectra, based on validation accuracy.

## When NOT to use

- Input spectra are single-compound rather than mixtures, so compound identification does not need the pSCNN mixture model being tuned
- No labeled ground truth is available to compute a validation accuracy, so grid selection cannot be scored
- The goal is evaluating real-world performance on experimental spectra (e.g., flavor mixtures benchmarking) rather than selecting training hyperparameters

## Inputs

- 18,000-pair augmented training set of NMR mixture spectra
- 2,000-pair augmented validation set of NMR mixture spectra
- pSCNN.py training script (from Zenodo DOI 10.5281/zenodo.6504814 or github.com/yuxuanliao/pSCNN)
- Hyperparameter grid: learning rates 10^-2 to 10^-5, 5–10 convolutional layers, 100 epochs

## Outputs

- Hyperparameter grid results table reproducing Table 1 (hyperparameter_grid_results.csv)
- Selected model checkpoint pscnn_model_M3.h5 with validation accuracy 0.9990
- Per-epoch validation-accuracy convergence traces (Figure 4b)

## How to apply

Instantiate pSCNN variants in TensorFlow 2.5.0-GPU (Python 3.8.13, Anaconda) sweeping learning rates from 10^-2 down to 10^-5 crossed with 5–10 convolutional layers, with epochs fixed at 100. Train each variant by running pSCNN.py with tqdm-monitored progress on the 18,000-pair augmented training set and 2,000-pair validation set, logging per-epoch validation accuracy to trace convergence (Figure 4b). Tabulate final validation accuracies across the grid to reproduce Table 1 and select the top configuration — expected to be model M3 (epoch = 100, learning rate = 10^-4, 6 convolutional layers) with validation accuracy 0.9990. Save the selected checkpoint as pscnn_model_M3.h5 and the grid-search results as hyperparameter_grid_results.csv.

## Related tools

- **Python 3.8.13** (Runtime environment for the hyperparameter sweep and training)
- **TensorFlow 2.5.0-GPU** (Framework used to instantiate and train pSCNN variants across the hyperparameter grid)
- **Anaconda** (Environment manager for installing the pSCNN dependencies (conda env with Python 3.8.13)) — https://www.anaconda.com/
- **tqdm** (Progress monitoring during per-variant training runs)
- **pSCNN** (The pseudo Siamese CNN model whose learning rate, layer count, and epoch settings are optimized) — https://github.com/yuxuanliao/pSCNN.git

## Examples

```
git clone https://github.com/yuxuanliao/pSCNN.git && python pSCNN.py
```

## Evaluation signals

- Grid-search table reproduces the winning configuration: epoch = 100, learning rate = 10^-4, 6 convolutional layers
- Selected model M3 reaches validation accuracy 0.9990 on the 2,000-pair augmented validation set
- Downstream augmented test performance matches the paper's reported ACC = 99.80%, TPR = 99.70%, FPR = 0.10%
- Convergence traces of validation accuracy per epoch align with Figure 4b behavior
- Selected checkpoint saved as pscnn_model_M3.h5 and grid results saved as hyperparameter_grid_results.csv

## Limitations

- Epochs were fixed at 100 in the reported sweep, so epoch count was not itself optimized in this grid
- The model exceeded repository limits and had to be distributed via Zenodo, so the checkpoint and data must be downloaded separately
- Results were tuned on augmented spectra; real experimental datasets show lower performance (flavor mixtures ACC = 97.62%; additional flavor mixture ACC = 91.67%)

## Evidence

- [intro] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [abstract] Augmented dataset split for training/validation/testing: "The augmented dataset was split and used to train, validate and test the pSCNN model."
- [readme] Run pSCNN.py and download model/data from Zenodo: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [methods] Model and example data reloadable from Zenodo: "Download the model and these example data, pSCNN can be reload and predict easily."
