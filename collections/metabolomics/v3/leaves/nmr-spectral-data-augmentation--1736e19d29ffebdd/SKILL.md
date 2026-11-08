---
name: nmr-spectral-data-augmentation
description: Use when you need to construct an augmented training/validation/test
  set of NMR spectral pairs from a small flavor-standard spectral database, or need
  pseudo-mixture spectra (e.g., augmented F10) to benchmark a component-identification
  model.
when_to_use_negative:
- Input spectra are already experimental mixture spectra for real-application benchmarking
  — use direct prediction (pSCNN.py on the pretrained model) instead of synthesizing
  pairs
- The goal is testing chemical-shift robustness via translated spectra — use the chemical-shift
  variation scheme (35 shifts within ±0.052 ppm) rather than superposition augmentation
- No component-standard spectral database is available to sample spectra from
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: Python 3.8.13
  role: Runtime environment for implementing the spectral superposition and pair-construction
    code
- name: TensorFlow 2.5.0-GPU
  role: Array format and deep-learning backend for the augmented training/validaton/test
    data consumed by pSCNN
- name: Anaconda
  role: Environment manager used to create the pSCNN Python 3.8.13 environment
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress tracking during the augmentation/pair-generation loops
- name: pSCNN
  role: Model trained on the augmented spectral pairs via the pSCNN.py script
  repo: https://github.com/yuxuanliao/pSCNN.git
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:7fabc2f72570d6b4da93fc81325a6d1d4e8c3642df64fec9d3c3fa6e878b7dbe
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
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nmr-spectral-data-augmentation

## Summary

Generates synthetic NMR spectral pairs for training a pseudo Siamese CNN (pSCNN) by superposing flavor-standard database spectra at random mixing ratios (e.g., 0.2–1.0) with added random noise, producing positive pairs (matching compound compositions) and negative pairs (differing compositions) so the model can identify mixture components despite chemical shift variation and peak overlap.

## When to use

Use when you need to construct an augmented training/validation/test set of NMR spectral pairs from a small flavor-standard spectral database, or need pseudo-mixture spectra (e.g., augmented F10) to benchmark a component-identification model.

## When NOT to use

- Input spectra are already experimental mixture spectra for real-application benchmarking — use direct prediction (pSCNN.py on the pretrained model) instead of synthesizing pairs
- The goal is testing chemical-shift robustness via translated spectra — use the chemical-shift variation scheme (35 shifts within ±0.052 ppm) rather than superposition augmentation
- No component-standard spectral database is available to sample spectra from

## Inputs

- Flavor-standard NMR spectral database (from Zenodo deposit 10.5281/zenodo.6504814)
- Experimental flavor mixtures NMR spectra (e.g., F10 mixture spectrum)
- Component standard spectra (Sulcatone, L-Menthone, Citronellal, Leaf acetate)
- Mixing-ratio range specification (e.g., 0.2–1.0)

## Outputs

- Augmented TensorFlow-compatible arrays of positive NMR spectral pairs (matching compositions)
- Augmented TensorFlow-compatible arrays of negative NMR spectral pairs (differing compositions)
- Train/validation/test splits of the augmented dataset
- Plotted comparison of augmented vs. experimental F10 spectrum (Figure 3 style)

## How to apply

Load the flavor-standard NMR spectral database and experimental flavor-mixture spectra from the Zenodo deposit (DOI 10.5281/zenodo.6504814) using Python 3.8.13 with TensorFlow 2.5.0-GPU (installed via Anaconda, tqdm for progress tracking). Randomly sample several flavor-standard spectra and superimpose them at random mixing ratios in the range 0.2–1.0 to synthesize pseudo-mixture spectra, then add random noise to each superposed spectrum to mimic experimental acquisition variability. Construct positive spectral pairs (matching compound compositions) and negative pairs (differing compositions, same superposition scheme but without the target compound's spectrum) as TensorFlow-compatible arrays. Split the augmented dataset into training, validation, and test sets for pSCNN training. To validate the augmentation, build the augmented F10 mixture spectrum by superposing Sulcatone, L-Menthone, Citronellal, and Leaf acetate standard spectra at the documented ratios and check that its peaks visually align with the experimental F10 spectrum; the augmented test set should support ACC = 99.80%, TPR = 99.70%, FPR = 0.10%.

## Related tools

- **Python 3.8.13** (Runtime environment for implementing the spectral superposition and pair-construction code)
- **TensorFlow 2.5.0-GPU** (Array format and deep-learning backend for the augmented training/validaton/test data consumed by pSCNN)
- **Anaconda** (Environment manager used to create the pSCNN Python 3.8.13 environment) — https://www.anaconda.com/
- **tqdm** (Progress tracking during the augmentation/pair-generation loops)
- **pSCNN** (Model trained on the augmented spectral pairs via the pSCNN.py script) — https://github.com/yuxuanliao/pSCNN.git

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; pip install tqdm; git clone https://github.com/yuxuanliao/pSCNN.git && python pSCNN.py
```

## Evaluation signals

- Augmented F10 spectrum's peaks visually align with the experimental F10 spectrum in the Figure 3 style comparison
- Augmented test set reproduces ACC = 99.80%, TPR = 99.70%, FPR = 0.10% when evaluated with pSCNN
- Positive and negative pairs are balanced and correctly labeled (positive pairs contain the specific compound's spectrum; negative pairs omit it)
- Superposition mixing ratios fall within the documented range (e.g., 0.2–1.0) and noise is applied per superposed spectrum
- Dataset splits cleanly into training, validation, and test sets without leakage of pairs between splits

## Limitations

- Augmentation relies on superposing database spectra, so performance on truly novel compounds not represented in the flavor-standard database is not guaranteed (real-application performance drops: ACC = 97.62% on flavor mixtures, 91.67% on the additional flavor mixture dataset)
- Model file exceeded repository limits, so the model and all NMR data must be fetched from Zenodo (10.5281/zenodo.6504814) rather than the GitHub repo
- Peak overlapping among molecules and chemical shift variation remain intrinsic challenges that augmentation alone mitigates rather than removes

## Evidence

- [abstract] superposition at random ratios: "A data augmentation method was implemented for the superposition of several NMR spectra sampled from a spectral database with random noises."
- [abstract] augmented dataset splits: "The augmented dataset was split and used to train, validate and test the pSCNN model."
- [abstract] augmented test set metrics: "the proposed method can achieve good performances in the augmented test set (ACC = 99.80%, TPR = 99.70% and FPR = 0.10%)"
- [methods] run pSCNN.py: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [readme] environment setup: "conda create --name pSCNN python=3.8.13
conda activate pSCNN
python -m pip install -r requirements.txt
pip install tqdm"
- [readme] chemical shift challenge: "is difficult to identify compounds in mixtures because of chemical shift variation of the same compound in different mixtures and peak overlapping among molecules"
