---
name: cnn-architecture-implementation-for-spectra
description: Use when building or reproducing a CNN that classifies compound presence
  in 1D NMR mixture spectra despite chemical shift variation and peak overlap, and
  an augmentable spectral dataset with binary labels is available.
when_to_use_negative:
- The task concerns pure single-compound spectra rather than mixtures — the method
  targets chemical shift variation and peak overlapping among molecules in mixtures
- No spectral database or reference NMR spectra exist to generate the augmented training
  data by superposition with random noises
- Only a small experimental dataset without augmentation is available — reported performance
  degrades on real data (e.g. ACC = 91.67% on the additional flavor mixture dataset)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: The pseudo-Siamese CNN implementation itself; run the file 'pSCNN.py' to train
    and predict mixture spectra
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep learning framework in which the two subnetworks, concatenation, dense/dropout/Sigmoid
    layers, binary cross-entropy loss and Adam optimizer are implemented
- name: Python 3.8.13
  role: Runtime environment (Anaconda env 'pSCNN') for training and inference
- name: Anaconda
  role: Environment manager used to create the pinned python=3.8.13 environment with
    requirements.txt
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress-bar dependency installed via pip for training loops
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:49341f99b9591ca451c79c087b8416fbca7aa691bcab5b74d50d702222b1899b
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
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cnn-architecture-implementation-for-spectra

## Summary

Implements the pseudo-Siamese convolutional neural network (pSCNN) from Wei et al. 2022 for compound identification in NMR spectra of mixtures: two identically architected subnetworks with separately trained weights (6 conv layers, 32 kernels of 5×1, ReLU, max pooling) feeding a concatenated 100-unit dense layer with dropout 0.2 and a Sigmoid output.

## When to use

Use when building or reproducing a CNN that classifies compound presence in 1D NMR mixture spectra despite chemical shift variation and peak overlap, and an augmentable spectral dataset with binary labels is available.

## When NOT to use

- The task concerns pure single-compound spectra rather than mixtures — the method targets chemical shift variation and peak overlapping among molecules in mixtures
- No spectral database or reference NMR spectra exist to generate the augmented training data by superposition with random noises
- Only a small experimental dataset without augmentation is available — reported performance degrades on real data (e.g. ACC = 91.67% on the additional flavor mixture dataset)

## Inputs

- Augmented NMR mixture spectra (superpositions of spectra sampled from a spectral database with random noises), split into training, validation, and test sets
- Binary per-compound presence labels for each augmented mixture spectrum
- Pre-trained model and example NMR data from Zenodo (DOI 10.5281/zenodo.6504814) for direct reload and prediction

## Outputs

- Trained pSCNN model serialized as pscnn_model.h5
- Per-compound binary predictions (Sigmoid outputs) for unseen NMR mixture spectra
- Test-set metrics: accuracy, true positive rate, false positive rate

## How to apply

Set up Python 3.8.13 with TensorFlow 2.5.0-GPU (via Anaconda environment 'pSCNN' plus tqdm per requirements.txt). Augment NMR spectra by superposition of spectra sampled from a spectral database with random noises, then split into training, validation, and test sets. Build two subnetworks with identical topology (6 convolutional layers, 32 kernels of size 5×1, ReLU activation, max pooling) but separately trained weights — this is what makes the network 'pseudo' rather than true Siamese. Concatenate the subnetwork feature maps, feed a 100-unit dense layer with dropout 0.2, and end with a Sigmoid output; compile with binary cross-entropy loss and the Adam optimizer. Train on the augmented training split while monitoring validation, then evaluate on the augmented test set expecting ACC = 99.80%, TPR = 99.70%, FPR = 0.10%. Alternatively, download the pre-trained model and example data from Zenodo (DOI 10.5281/zenodo.6504814) and reload pSCNN to predict mixture spectra directly.

## Related tools

- **pSCNN** (The pseudo-Siamese CNN implementation itself; run the file 'pSCNN.py' to train and predict mixture spectra) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep learning framework in which the two subnetworks, concatenation, dense/dropout/Sigmoid layers, binary cross-entropy loss and Adam optimizer are implemented)
- **Python 3.8.13** (Runtime environment (Anaconda env 'pSCNN') for training and inference)
- **Anaconda** (Environment manager used to create the pinned python=3.8.13 environment with requirements.txt) — https://www.anaconda.com/
- **tqdm** (Progress-bar dependency installed via pip for training loops)

## Examples

```
conda create --name pSCNN python=3.8.13 && conda activate pSCNN && python -m pip install -r requirements.txt && pip install tqdm && python pSCNN.py
```

## Evaluation signals

- Augmented test set must reach approximately ACC = 99.80%, TPR = 99.70%, FPR = 0.10% to match the reported benchmark
- Experimental flavor mixtures dataset should yield approximately ACC = 97.62%, TPR = 96.44%, FPR = 2.29%
- Additional flavor mixture dataset should yield approximately ACC = 91.67%, TPR = 100.00%, FPR = 10.53%
- Architecture check: each subnetwork has 6 conv layers with 32 kernels of size 5×1, ReLU, max pooling; concatenated features pass through a 100-unit dense layer with dropout 0.2 before a Sigmoid output
- Serialization check: the trained model reloads from pscnn_model.h5 (or the Zenodo deposit) and predicts example mixture spectra without retraining

## Limitations

- Performance drops substantially on real experimental data compared with the augmented test set (ACC 99.80% augmented vs 91.67% on the additional flavor mixture dataset, with FPR rising to 10.53%)
- The trained model exceeded repository size limits and must be downloaded from Zenodo (DOI 10.5281/zenodo.6504814) rather than cloned from GitHub
- No discussion-section content was available in the indexed document, so limitations and reproducibility gaps could not be fully extracted

## Evidence

- [methods] Subnetwork topology and training configuration: [evidence span withheld — anchor status: not_found]
- [abstract] Reported benchmark on augmented test set: "the proposed method can achieve good performances in the augmented test set (ACC = 99.80%, TPR = 99.70% and FPR = 0.10%)"
- [intro] Purpose of the pseudo-Siamese design: "We presented a pseudo Siamese convolutional neural network method (pSCNN) to solve the problems of compound identification in NMR spectra of mixtures"
- [readme] Environment and how to run the model: "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [methods] Reload pre-trained model from Zenodo: "Download the model and these example data, pSCNN can be reload and predict easily."
