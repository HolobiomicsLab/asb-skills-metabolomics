---
name: transformer-cnn-hybrid-model-training
description: Use when training or reloading a model to identify compounds in 1H NMR
  spectra of flavor mixtures from a repository of standard compound spectra, and when
  evaluated on augmented test and known/unknown mixture datasets.
when_to_use_negative:
- When the analysis goal is something other than identifying compounds in flavor mixtures
  from 1H NMR spectra (e.g. quantitative concentration estimation is not described
  here)
- When no pretrained model parameters or training data are available and training
  from scratch is not feasible in the provided environment
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: FlavorFormer
  role: The hybrid CNN–Transformer model being trained and applied to identify compounds
    in flavor mixtures from 1H NMR spectra
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: PyTorch
  role: Deep learning framework (version 2.7.0+cu118) used to implement and train
    the CNN–Transformer architecture
- name: conda
  role: Environment manager used to create the Python virtual environment from environment.yml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:7f8675546c73f99d82c56e35ff08b1e3574fd8a390111b2b96c2f07d85186d1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - Anaconda
  - FlavorFormer
  - PyTorch
  - Python
  - conda
  - git
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# transformer-cnn-hybrid-model-training

## Summary

Train FlavorFormer, a hybrid CNN–Transformer that captures both local spectral features and global dependencies from 1H NMR spectra of flavor mixtures, using a bi-encoder plus cross-encoder design with fusion pooling and a weighted loss function to identify compounds.

## When to use

Use when training or reloading a model to identify compounds in 1H NMR spectra of flavor mixtures from a repository of standard compound spectra, and when evaluated on augmented test and known/unknown mixture datasets.

## When NOT to use

- When the analysis goal is something other than identifying compounds in flavor mixtures from 1H NMR spectra (e.g. quantitative concentration estimation is not described here)
- When no pretrained model parameters or training data are available and training from scratch is not feasible in the provided environment

## Inputs

- Augmented training dataset of 1H NMR spectra in the data/ folder
- Model parameters in the model/ folder for reloading
- Augmented test set of 1H NMR spectra
- Known flavor mixture dataset
- Unknown flavor mixture dataset

## Outputs

- Trained FlavorFormer model parameters saved to model/
- Compound identification predictions for mixture spectra
- Performance metrics ACC, TPR, FPR (mean ± std) on test and mixture datasets

## How to apply

Clone the FlavorFormer repository and create a Python 3.13.2 virtual environment with conda (`conda env create -f environment.yml; conda activate FlavorFormer`) with PyTorch 2.7.0+cu118. Open `demo.ipynb` and train the hybrid CNN–Transformer on the augmented training data from the `data/` folder, using the bi-encoder and cross-encoder combination with fusion pooling and the weighted loss function that boosts identification accuracy. After training, save model parameters to the `model/` folder so FlavorFormer can be reloaded for prediction. Evaluate the trained model on an augmented test set and on two experimental flavor mixture datasets (known and unknown), reporting ACC, TPR and FPR; the article's baseline is ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 % on the test set. Judge success by matching these metrics within their stated standard deviations and by outperforming the pSCNN baseline in TPR.

## Related tools

- **FlavorFormer** (The hybrid CNN–Transformer model being trained and applied to identify compounds in flavor mixtures from 1H NMR spectra) — https://github.com/yfWang01/FlavorFormer.git
- **PyTorch** (Deep learning framework (version 2.7.0+cu118) used to implement and train the CNN–Transformer architecture)
- **conda** (Environment manager used to create the Python virtual environment from environment.yml)

## Examples

```
cd FlavorFormer; conda env create -f environment.yml; conda activate FlavorFormer; jupyter notebook demo.ipynb
```

## Evaluation signals

- Test-set metrics should approximate ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %
- Known flavor mixture dataset metrics should approximate ACC 95.44 ± 0.00 %, TPR 95.19 ± 1.24 %, FPR 4.31 ± 0.33 %
- Unknown flavor mixture dataset metrics should approximate ACC 94.44 ± 2.41 %, TPR 93.33 ± 11.55 %, FPR 5.26 ± 0.00 %
- Trained parameters should be reloadable from model/ and produce predictions directly from data/ without retraining
- TPR on mixture datasets should exceed the pSCNN state-of-the-art baseline reported in the paper

## Limitations

- Performance on the unknown flavor mixture dataset has high TPR variance (93.33 ± 11.55 %), so repeated runs are needed to characterize stability
- The model was developed and validated specifically for flavor mixture identification from 1H NMR spectra; applicability outside this scope is not demonstrated here
- Requires a specific environment: Python 3.13.2 and PyTorch 2.7.0+cu118

## Evidence

- [abstract] Hybrid architecture description: "FlavorFormer is proposed by incorporating a hybrid CNN and Transformer architecture to capture both local features and global dependencies from 1H NMR spectra."
- [intro] Encoder, pooling and loss components: "it leverages a combination of a bi-encoder and cross-encoder, a fusion pooling strategy, and a weighted loss function to identify compounds correctly."
- [abstract] Baseline test-set metrics: "The model achieved consistently high performance metrics on the test set (ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %)"
- [abstract] Training and evaluation protocol: "After training on augmented datasets, FlavorFormer was tested on an augmented test set and two experimental datasets."
- [readme] Environment setup commands: "cd FlavorFormer
conda env create -f environment.yml
conda activate FlavorFormer"
- [methods] Notebook entry point and reload path: "For training model and predict mixture spectra data, run the file   `demo.ipynb`. The model parameters is located in the [model](model/) folder and the required data is in the [data](data/) folder,"
