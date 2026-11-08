---
name: tensorflow-model-training-and-serialization
description: Use when building, training, or saving the pSCNN architecture (dual 6-conv
  subnetworks, 32 kernels of 5x1, ReLU, max pooling, 100-unit dense with dropout 0.2,
  Sigmoid output) for NMR mixture compound identification.
when_to_use_negative:
- The analysis goal is classifying individual pure-compound spectra rather than identifying
  compounds in NMR spectra of mixtures.
- The input data are not NMR spectra and no augmented mixture-spectrum training split
  exists; the architecture, loss, and benchmarks are NMR-specific.
- Only inference is needed on existing mixtures — download the pre-trained model from
  Zenodo and reload it instead of retraining.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0593
tools:
- name: TensorFlow 2.5.0-GPU
  role: Framework used to build, compile, train, and serialize the pSCNN pseudo-Siamese
    CNN
- name: Python 3.8.13
  role: Runtime environment for the pSCNN training and prediction scripts
- name: pSCNN
  role: Implementation containing pSCNN.py, which trains the model and predicts mixture
    spectra data
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: Anaconda
  role: Environment manager used to create the pSCNN environment (python=3.8.13) and
    install requirements.txt
  repo: https://www.anaconda.com/
- name: Zenodo (10.5281/zenodo.6504814)
  role: Hosts the pre-trained model and all NMR data since the model exceeded repository
    size limits
  repo: https://doi.org/10.5281/zenodo.6504814
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:499e39f15da8054c7e69e74f1ffb86adb2bb300d984d94726135acba769d0ddb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN.git
  tools:
  - Anaconda
  - Python
  - Python 3.8.13
  - TensorFlow 2.5.0-GPU
  - Zenodo (10.5281/zenodo.6504814)
  - pSCNN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tensorflow-model-training-and-serialization

## Summary

Train the pSCNN pseudo-Siamese CNN for compound identification in NMR spectra of mixtures in TensorFlow 2.5.0-GPU (Python 3.8.13), then serialize the trained model to pscnn_model.h5 for later reload and mixture-spectrum prediction.

## When to use

Use when building, training, or saving the pSCNN architecture (dual 6-conv subnetworks, 32 kernels of 5x1, ReLU, max pooling, 100-unit dense with dropout 0.2, Sigmoid output) for NMR mixture compound identification.

## When NOT to use

- The analysis goal is classifying individual pure-compound spectra rather than identifying compounds in NMR spectra of mixtures.
- The input data are not NMR spectra and no augmented mixture-spectrum training split exists; the architecture, loss, and benchmarks are NMR-specific.
- Only inference is needed on existing mixtures — download the pre-trained model from Zenodo and reload it instead of retraining.

## Inputs

- Augmented training, validation, and test NMR spectra (superposition of spectra from a spectral database with random noises)
- Pre-trained model and example NMR data from Zenodo deposit 10.5281/zenodo.6504814
- pSCNN repository code (github.com/yuxuanliao/pSCNN), specifically the file pSCNN.py

## Outputs

- Trained pSCNN model serialized as pscnn_model.h5
- Trained model reloadable for mixture spectra prediction
- Test-set metrics (ACC, TPR, FPR) comparable to the augmented test set benchmark (ACC = 99.80%, TPR = 99.70%, FPR = 0.10%)

## How to apply

Set up a conda environment with Python 3.8.13 and TensorFlow 2.5.0-GPU (install Anaconda, then requirements.txt plus tqdm). Build two identically architected subnetworks with separately trained weights, each with 6 convolutional layers (32 kernels of size 5x1, ReLU activation, followed by max pooling); concatenate the feature maps, feed a 100-unit dense layer with dropout 0.2, and end with a Sigmoid output. Compile with binary cross-entropy loss and the Adam optimizer, then train on the augmented training split while monitoring the validation split. Evaluate on the augmented test set against the reported benchmark (ACC = 99.80%, TPR = 99.70%, FPR = 0.10%) and serialize the trained model to pscnn_model.h5; the pre-trained model and example NMR data can be downloaded from Zenodo (DOI 10.5281/zenodo.6504814) and reloaded for prediction.

## Related tools

- **TensorFlow 2.5.0-GPU** (Framework used to build, compile, train, and serialize the pSCNN pseudo-Siamese CNN)
- **Python 3.8.13** (Runtime environment for the pSCNN training and prediction scripts)
- **pSCNN** (Implementation containing pSCNN.py, which trains the model and predicts mixture spectra data) — https://github.com/yuxuanliao/pSCNN.git
- **Anaconda** (Environment manager used to create the pSCNN environment (python=3.8.13) and install requirements.txt) — https://www.anaconda.com/
- **Zenodo (10.5281/zenodo.6504814)** (Hosts the pre-trained model and all NMR data since the model exceeded repository size limits) — https://doi.org/10.5281/zenodo.6504814

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; python pSCNN.py
```

## Evaluation signals

- Augmented test set metrics match the reported benchmark: ACC = 99.80%, TPR = 99.70%, FPR = 0.10%.
- Both subnetworks have identical topology (6 conv layers, 32 kernels of 5x1, ReLU, max pooling) but separately trained weights, and their concatenated features feed a 100-unit dense layer with dropout 0.2 and a Sigmoid output.
- The model compiles with binary cross-entropy loss and the Adam optimizer, and validation loss is monitored during training on the augmented split.
- The trained model serializes to pscnn_model.h5 and can be reloaded from the Zenodo deposit to predict mixture spectra without retraining.

## Limitations

- The model exceeded the repository size limit, so it and all NMR data are distributed via Zenodo rather than GitHub.
- Performance degrades on real experimental data: flavor mixtures dataset (ACC = 97.62%, TPR = 96.44%, FPR = 2.29%) and additional flavor mixture dataset (ACC = 91.67%, TPR = 100.00%, FPR = 10.53%).
- No discussion-section content was available in the indexed document, so limitations and reproducibility gaps beyond reported metrics were not extracted.

## Evidence

- [intro] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [methods] pSCNN.py training and Zenodo model/data distribution: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [abstract] Benchmark metrics on augmented test set: "the proposed method can achieve good performances in the augmented test set (ACC = 99.80%, TPR = 99.70% and FPR = 0.10%)"
- [abstract] Data augmentation and model training workflow: "A data augmentation method was implemented for the superposition of several NMR spectra sampled from a spectral database with random noises."
- [readme] Model exceeds repo limit, hosted on Zenodo: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
