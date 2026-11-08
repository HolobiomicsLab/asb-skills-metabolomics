---
name: spectral-noise-simulation
description: Use when building synthetic NMR mixture training data from pure-compound
  standard spectra to simulate chemical shift variation and acquisition variability
  for training a spectral classifier such as pSCNN.
when_to_use_negative:
- Benchmarking pSCNN on real experimental datasets (flavor mixtures or additional
  flavor mixture) — augmentation is only for building the synthetic training corpus
- Data types other than one-dimensional NMR spectra of small-molecule mixtures (the
  superposition model assumes additive spectral mixing)
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: Python 3.8.13
  role: Runtime environment for the augmentation script
- name: TensorFlow 2.5.0-GPU
  role: Consumes the TensorFlow-compatible augmented arrays for pSCNN training
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: tqdm
  role: Progress tracking during pair generation
- name: Anaconda
  role: Environment manager used to create the pSCNN Python 3.8.13 environment
  repo: https://www.anaconda.com/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:7fcb9593328809b0cd78774368639485317aedae31fe749fac52f90f74460a7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN.git
  tools:
  - Anaconda
  - Python
  - Python 3.8.13
  - TensorFlow 2.5.0-GPU
  - tqdm
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Spectral Noise Simulation for NMR Data Augmentation

## Summary

Generates augmented NMR mixture spectra by superposing flavor-standard spectra sampled from a spectral database at random mixing ratios (e.g., 0.2–1.0) with added random noise, producing positive and negative spectral pairs used to train, validate, and test the pSCNN model.

## When to use

Use when building synthetic NMR mixture training data from pure-compound standard spectra to simulate chemical shift variation and acquisition variability for training a spectral classifier such as pSCNN.

## When NOT to use

- Benchmarking pSCNN on real experimental datasets (flavor mixtures or additional flavor mixture) — augmentation is only for building the synthetic training corpus
- Data types other than one-dimensional NMR spectra of small-molecule mixtures (the superposition model assumes additive spectral mixing)

## Inputs

- Flavor-standard NMR spectral database (from Zenodo deposit 10.5281/zenodo.6504814)
- Experimental flavor mixtures NMR spectra (e.g., F10 mixture spectrum)
- Random mixing-ratio range (e.g., 0.2–1.0) and random-noise settings

## Outputs

- Augmented pseudo-mixture NMR spectra with superimposed random noise
- Positive spectral pairs (matching compound compositions)
- Negative spectral pairs (differing compositions)
- TensorFlow-compatible training/validation/test arrays

## How to apply

Load the flavor-standard NMR spectral database and experimental flavor mixture spectra from the Zenodo deposit (10.5281/zenodo.6504814) using Python 3.8.13. Randomly sample several flavor-standard spectra and superimpose them at random mixing ratios in a range such as 0.2–1.0 to synthesize pseudo-mixture spectra, then add random noise to mimic experimental acquisition variability. Construct positive spectral pairs whose components match (e.g., the same compound superposed with other randomly sampled database spectra) and negative pairs generated the same way but without the specific compound of interest. Split the augmented dataset into training, validation, and test sets and export TensorFlow-compatible arrays, tracking progress with tqdm. Rationale: the augmentation lets the pSCNN convolutional network exploit translational invariance to handle chemical shift variation of the same compound across mixtures and peak overlapping among molecules.

## Related tools

- **Python 3.8.13** (Runtime environment for the augmentation script)
- **TensorFlow 2.5.0-GPU** (Consumes the TensorFlow-compatible augmented arrays for pSCNN training) — https://github.com/yuxuanliao/pSCNN.git
- **tqdm** (Progress tracking during pair generation)
- **Anaconda** (Environment manager used to create the pSCNN Python 3.8.13 environment) — https://www.anaconda.com/

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; pip install tqdm; python pSCNN.py
```

## Evaluation signals

- Augmented F10 spectrum peaks visually align with the experimental F10 spectrum in a Figure 3-style comparison plot
- The augmented test set reproduces reported performance: ACC = 99.80%, TPR = 99.70%, FPR = 0.10%
- Positive/negative pairs contain the expected composition difference (negative pairs lack the specific target compound's spectrum)
- Mixing ratios fall within the documented range (e.g., 0.2–1.0)
- Arrays are loadable in TensorFlow 2.5.0-GPU without shape errors

## Limitations

- Synthetic augmentation performance (99.80% ACC) does not transfer fully to real mixtures: experimental flavor mixtures reach 97.62% ACC and additional flavor mixtures only 91.67% ACC (FPR 10.53%)
- Model exceeded size limits and was distributed via Zenodo rather than the GitHub repo
- No discussion-section limitations were extractable (missing changelog/content in the source document)

## Evidence

- [abstract] Superposition with random noise: "A data augmentation method was implemented for the superposition of several NMR spectra sampled from a spectral database with random noises."
- [abstract] Augmented dataset split: "The augmented dataset was split and used to train, validate and test the pSCNN model."
- [abstract] Translational invariance rationale: "We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra."
- [abstract] Performance benchmark: "the proposed method can achieve good performances in the augmented test set (ACC = 99.80%, TPR = 99.70% and FPR = 0.10%)"
- [readme] Zenodo data deposit: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
