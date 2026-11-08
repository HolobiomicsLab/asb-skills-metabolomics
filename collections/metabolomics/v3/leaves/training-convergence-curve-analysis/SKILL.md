---
name: training-convergence-curve-analysis
description: Use when training a deep neural network (e.g., pSCNN in TensorFlow 2.5.0-GPU)
  and you need to log per-epoch validation accuracy to confirm the model converged
  before locking hyperparameters such as epoch count, learning rate, or layer depth.
when_to_use_negative:
- Input is a single pre-trained model checkpoint to reload and predict (use the Zenodo-hosted
  pscnn_model_M3.h5 directly instead of retraining).
- The experimental data are real NMR mixtures (e.g., flavor mixtures datasets) for
  benchmarking identification performance — that is the prediction/benchmarking workflow,
  not training-convergence analysis.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: The pseudo Siamese convolutional neural network model whose per-epoch validation
    accuracy is traced during training
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Backend used to instantiate and train pSCNN variants across the hyperparameter
    grid
- name: tqdm
  role: Monitors training progress per epoch while logging validation accuracy
- name: Anaconda
  role: Environment manager providing the pinned Python 3.8.13 training environment
  repo: https://www.anaconda.com/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:d01871c2c3d818454c73e4c54a19e12a0ce633ac735bcfff19103fb5c4abd46f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN.git
  tools:
  - Anaconda
  - Python
  - TensorFlow 2.5.0-GPU
  - pSCNN
  - tqdm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# training-convergence-curve-analysis

## Summary

Trace per-epoch validation accuracy while training the pSCNN model (pseudSiamese CNN for compound identification in NMR spectra of mixtures) to verify convergence and justify hyperparameter selection. In this paper the analysis supported choosing epoch = 100, learning rate = 10^-4, and 6 convolutional layers (model M3, validation accuracy 0.9990).

## When to use

Use when training a deep neural network (e.g., pSCNN in TensorFlow 2.5.0-GPU) and you need to log per-epoch validation accuracy to confirm the model converged before locking hyperparameters such as epoch count, learning rate, or layer depth.

## When NOT to use

- Input is a single pre-trained model checkpoint to reload and predict (use the Zenodo-hosted pscnn_model_M3.h5 directly instead of retraining).
- The experimental data are real NMR mixtures (e.g., flavor mixtures datasets) for benchmarking identification performance — that is the prediction/benchmarking workflow, not training-convergence analysis.

## Inputs

- 18,000-pair training set of augmented NMR mixture spectra (superpositions of database spectra with random noise)
- 2000-pair validation set of augmented NMR mixture spectra
- pSCNN.py training script from Zenodo DOI 10.5281/zenodo.6504814
- Hyperparameter grid: learning rates 10^-2 to 10^-5, convolutional layer counts 5–10, epochs = 100

## Outputs

- Per-epoch validation accuracy logs defining the training convergence curve (Figure 4b analog)
- Hyperparameter grid results table (hyperparameter_grid_results.csv) reproducing Table 1
- Selected model checkpoint pscnn_model_M3.h5 with validation accuracy 0.9990

## How to apply

Run the pSCNN.py training script inside an Anaconda environment with Python 3.8.13 and TensorFlow 2.5.0-GPU, using tqdm-monitored progress to watch training. Train 18,000 augmented mixture-spectrum pairs and monitor accuracy on the 2000-pair validation set at each epoch. Sweep the reported grid — learning rates from 10^-2 down to 10^-5 crossed with 5–10 convolutional layers at a fixed 100 epochs — logging final validation accuracy per configuration. Plot the logged per-epoch curves to reproduce the convergence behavior of Figure 4b and tabulate grid accuracies (Table 1). Rationale: the epoch count must be set high enough that the validation-accuracy curve plateaus; here the converged configuration was epoch = 100, learning rate = 10^-4, and 6 convolutional layers, yielding model M3 with validation accuracy 0.9990. Save the selected checkpoint (pscnn_model_M3.h5) and the grid results (hyperparameter_grid_results.csv) for reuse and prediction of mixture spectra.

## Related tools

- **pSCNN** (The pseudo Siamese convolutional neural network model whose per-epoch validation accuracy is traced during training) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Backend used to instantiate and train pSCNN variants across the hyperparameter grid)
- **tqdm** (Monitors training progress per epoch while logging validation accuracy)
- **Anaconda** (Environment manager providing the pinned Python 3.8.13 training environment) — https://www.anaconda.com/

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; pip install tqdm; python pSCNN.py
```

## Evaluation signals

- The per-epoch validation-accuracy curve reaches a plateau by epoch 100 for the selected configuration (model M3).
- The top grid entry has learning rate = 10^-4, 6 convolutional layers, and validation accuracy 0.9990, matching model M3 in Table 1.
- hyperparameter_grid_results.csv contains one final validation accuracy per (learning rate, layer count) combination across the 10^-2–10^-5 × 5–10 grid.
- The saved pscnn_model_M3.h5 checkpoint reloads in TensorFlow 2.5.0-GPU and predicts the example Zenodo mixture-spectrum data.
- Training was run with tqdm progress visible and no per-epoch NaN/decreasing-accuracy divergence in the logs.

## Limitations

- Convergence results depend on the augmented 18,000-pair training / 2000-pair validation split; different splits may shift the selected hyperparameters.
- The paper does not provide discussion-section content on limitations or reproducibility gaps, so failure modes beyond the reported grid are undocumented.
- The full model and NMR data exceed GitHub limits and must be downloaded from Zenodo; training without them is not possible.

## Evidence

- [abstract] The augmented dataset was split and used to train, validate and test the pSCNN model.: "The augmented dataset was split and used to train, validate and test the pSCNN model."
- [methods] pSCNN.py execution entry point: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [readme] Pinned training environment: "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [readme] Zenodo download of model and data: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
