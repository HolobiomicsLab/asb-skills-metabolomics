---
name: siamese-network-weight-training
description: Use when training a two-branch CNN on paired/augmented NMR mixture spectra
  where chemical shift variation and peak overlapping complicate compound identification,
  and each branch must learn its own weights.
when_to_use_negative:
- Input is a single-compound (pure) NMR spectrum rather than a mixture — the method
  targets compound identification in mixtures
- No augmented training corpus exists and only a handful of real mixture spectra are
  available — the model depends on data augmentation by spectral superposition for
  training
- The task requires a true Siamese network with shared weights; pSCNN deliberately
  trains the two subnetworks' weights separately
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: The pseudo-Siamese CNN model itself; 'pSCNN.py' implements training and mixture-spectrum
    prediction
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep-learning framework used to build and train the two-branch pSCNN architecture
- name: Python 3.8.13
  role: Runtime environment for the training script, provisioned via conda environment
    'pSCNN'
- name: Anaconda
  role: Environment manager used to create the Python 3.8.13 environment and install
    requirements.txt
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress-bar dependency installed during environment setup
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:f7ce76deedcb60540998cbe8460068a80efd35e2a7de091fba450456d14d3584
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# siamese-network-weight-training

## Summary

Reconstruct and train the pseudo-Siamese convolutional neural network (pSCNN) — two identically architected subnetworks with separately trained weights — for identifying compounds in NMR spectra of mixtures using TensorFlow 2.5.0-GPU.

## When to use

Use when training a two-branch CNN on paired/augmented NMR mixture spectra where chemical shift variation and peak overlapping complicate compound identification, and each branch must learn its own weights.

## When NOT to use

- Input is a single-compound (pure) NMR spectrum rather than a mixture — the method targets compound identification in mixtures
- No augmented training corpus exists and only a handful of real mixture spectra are available — the model depends on data augmentation by spectral superposition for training
- The task requires a true Siamese network with shared weights; pSCNN deliberately trains the two subnetworks' weights separately

## Inputs

- Augmented NMR training spectra (superposition of spectra sampled from a spectral database with random noises), split into training/validation/test sets
- Pre-trained model and example NMR data from Zenodo deposit 10.5281/zenodo.6504814
- Experimental benchmark datasets (flavor mixtures dataset, additional flavor mixture dataset)

## Outputs

- Trained pSCNN model serialized as pscnn_model.h5
- Predicted compound identities for input mixture spectra
- Test-set performance metrics (ACC/TPR/FPR) for comparison against the benchmark

## How to apply

Build two subnetworks with identical topology but separately trained weights in TensorFlow 2.5.0-GPU on Python 3.8.13: each branch stacks 6 convolutional layers with 32 kernels of size 5x1 and ReLU activation, each followed by max pooling. Concatenate the two subnetwork feature maps and feed them to a dense layer of 100 units with dropout 0.2, terminating in a Sigmoid output. Compile with binary cross-entropy loss and the Adam optimizer, then train on the augmented training split (spectra superposed from a spectral database with random noises) while monitoring the validation split. Evaluate on the augmented test set against the published benchmark (ACC = 99.80%, TPR = 99.70%, FPR = 0.10%) and serialize the trained model to pscnn_model.h5 for reloading and mixture-spectrum prediction.

## Related tools

- **pSCNN** (The pseudo-Siamese CNN model itself; 'pSCNN.py' implements training and mixture-spectrum prediction) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep-learning framework used to build and train the two-branch pSCNN architecture)
- **Python 3.8.13** (Runtime environment for the training script, provisioned via conda environment 'pSCNN')
- **Anaconda** (Environment manager used to create the Python 3.8.13 environment and install requirements.txt) — https://www.anaconda.com/
- **tqdm** (Progress-bar dependency installed during environment setup)

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; pip install tqdm; python pSCNN.py
```

## Evaluation signals

- Augmented test set performance matches the reported benchmark: ACC = 99.80%, TPR = 99.70%, FPR = 0.10%
- Both subnetworks have identical topology (6 conv layers, 32 kernels of 5x1, ReLU, max pooling) but separately trained weight tensors
- Compiled model uses binary cross-entropy loss with the Adam optimizer and ends in a Sigmoid output layer
- Trained model serializes cleanly to pscnn_model.h5 and can be reloaded from the Zenodo deposit for prediction
- Experimental benchmark: ACC = 97.62%, TPR = 96.44%, FPR = 2.29% on the flavor mixtures dataset and ACC = 91.67%, TPR = 100.00%, FPR = 10.53% on the additional flavor mixture dataset

## Limitations

- Performance degrades on real experimental data relative to the augmented test set (e.g., ACC = 91.67% and FPR = 10.53% on the additional flavor mixture dataset)
- The trained model exceeded GitHub's size limit, so the model and NMR data are distributed via Zenodo rather than the repository
- Chemical shift variation and peak overlapping among molecules remain the core difficulty the architecture addresses; results depend on the data augmentation strategy (spectral superposition with random noises)

## Evidence

- [abstract] Architecture: "We presented a pseudo Siamese convolutional neural network method (pSCNN) to solve the problems of compound identification in NMR spectra of mixtures"
- [abstract] Benchmark metrics: "the proposed method can achieve good performances in the augmented test set (ACC = 99.80%, TPR = 99.70% and FPR = 0.10%)"
- [methods] Run script: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [readme] Environment: "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [readme] Model download: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
