---
name: positive-negative-pair-generation
description: 'Use when constructing an augmented training corpus for a Siamese/CNN
  compound-identification model from a library of pure-component spectra: pairs of
  spectra with matching vs. differing compound compositions must be synthesized before
  training pSCNN.'
when_to_use_negative:
- The goal is benchmarking pSCNN on real experimental data (flavor mixtures or additional
  flavor mixture datasets) rather than synthesizing training pairs — use the experimental
  datasets directly instead.
- Input spectra are already paired or labeled training examples; regeneration of synthetic
  pairs would duplicate existing augmentation.
- A target compound's pure-standard spectrum is absent from the database, since negative
  pairs are defined by omitting that compound's spectrum.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: Python 3.8.13
  role: Runtime environment in which the spectral superposition and pair-construction
    code is executed.
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Provides the array/dataset format into which the augmented positive and negative
    spectral pairs are assembled for pSCNN training.
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: tqdm
  role: Progress tracking for the random-sampling and superposition loops during pair
    generation.
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: Anaconda
  role: Environment manager used to create the pSCNN conda environment (python=3.8.13)
    with requirements.txt before pair generation.
  repo: https://www.anaconda.com/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:9d96e43b746461fff1682f08be7a40b13045444320104c606950c1c2604260dc
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# positive-negative-pair-generation

## Summary

Generates the positive and negative NMR spectral pairs used to train, validate and test the pSCNN model by superposing flavor-standard database spectra at random mixing ratios (0.2–1.0) with random noise, so that the Siamese network can learn compound identification despite chemical shift variation and peak overlap.

## When to use

Use when constructing an augmented training corpus for a Siamese/CNN compound-identification model from a library of pure-component spectra: pairs of spectra with matching vs. differing compound compositions must be synthesized before training pSCNN.

## When NOT to use

- The goal is benchmarking pSCNN on real experimental data (flavor mixtures or additional flavor mixture datasets) rather than synthesizing training pairs — use the experimental datasets directly instead.
- Input spectra are already paired or labeled training examples; regeneration of synthetic pairs would duplicate existing augmentation.
- A target compound's pure-standard spectrum is absent from the database, since negative pairs are defined by omitting that compound's spectrum.

## Inputs

- flavor-standard NMR spectral database (pure-component spectra) from Zenodo 10.5281/zenodo.6504814
- experimental flavor mixtures NMR spectra (e.g., F10) for comparison and benchmarking
- component standard spectra (Sulcatone, L-Menthone, Citronellal, Leaf acetate) for the augmented F10 example
- mixing-ratio range specification (e.g., 0.2–1.0) and noise model parameters

## Outputs

- positive spectral pairs (spectra with matching compound compositions) as TensorFlow-compatible arrays
- negative spectral pairs (spectra omitting the target compound) as TensorFlow-compatible arrays
- augmented dataset split into training, validation and test sets for pSCNN
- augmented F10 mixture spectrum and comparison plot against the experimental F10 spectrum (Figure 3 style)

## How to apply

Load the flavor-standard NMR spectral database and experimental flavor mixture spectra from the Zenodo deposit (10.5281/zenodo.6504814) in a Python 3.8.13 / TensorFlow 2.5.0 environment. Randomly sample several flavor-standard spectra and superimpose them at random mixing ratios in the 0.2–1.0 range to synthesize pseudo-mixture spectra, adding random noise to each superposition to mimic experimental acquisition variability (use tqdm to track the sampling loop). Form positive spectral pairs whose members share the same compound composition, and negative pairs generated the same way but omitting the spectrum of the specific target compound; assemble the pairs into TensorFlow-compatible arrays. Split the augmented dataset into training, validation and test sets for pSCNN; for demonstration, superpose Sulcatone, L-Menthone, Citronellal and Leaf acetate standards at documented ratios to build the augmented F10 mixture spectrum and plot it against the experimental F10 spectrum to check that peak positions and intensities align.

## Related tools

- **Python 3.8.13** (Runtime environment in which the spectral superposition and pair-construction code is executed.) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Provides the array/dataset format into which the augmented positive and negative spectral pairs are assembled for pSCNN training.) — https://github.com/yuxuanliao/pSCNN.git
- **tqdm** (Progress tracking for the random-sampling and superposition loops during pair generation.) — https://github.com/yuxuanliao/pSCNN.git
- **Anaconda** (Environment manager used to create the pSCNN conda environment (python=3.8.13) with requirements.txt before pair generation.) — https://www.anaconda.com/

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; pip install tqdm; python pSCNN.py
```

## Evaluation signals

- The augmented F10 spectrum's peak positions and intensities visually align with the experimental F10 spectrum in the Figure 3 style comparison plot.
- The augmented test set supports the reported pSCNN performance level: ACC = 99.80%, TPR = 99.70%, FPR = 0.10%.
- Positive pairs contain the target compound's spectrum in both members, while negative pairs are generated the same way but without that compound's spectrum.
- All superposed spectra use mixing ratios drawn from the documented range (0.2–1.0) and include added random noise.
- Pair arrays load cleanly as TensorFlow inputs and the augmented dataset splits into training, validation and test sets without errors (tqdm loops complete).

## Limitations

- Synthetic pairs are based on random superposition plus noise, so real-world performance degrades on experimental data: ACC drops to 97.62% (TPR 96.44%, FPR 2.29%) on flavor mixtures and 91.67% (TPR 100.00%, FPR 10.53%) on the additional flavor mixture dataset.
- The pretrained model exceeded repository size limits and had to be hosted on Zenodo, so pair generation depends on separately downloading the model and NMR data.
- Chemical shift variation and peak overlap among molecules remain the underlying challenges; translation invariance of the CNN (not the pair scheme alone) addresses chemical shift variation.

## Evidence

- [abstract] Data augmentation by superposition of several NMR spectra sampled from a spectral database with random noises: "A data augmentation method was implemented for the superposition of several NMR spectra sampled from a spectral database with random noises."
- [abstract] Augmented dataset split for pSCNN training: "The augmented dataset was split and used to train, validate and test the pSCNN model."
- [methods] Positive/negative pair construction from source card finding: [evidence span withheld — anchor status: not_found]
- [readme] Zenodo model and data upload: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
- [readme] Environment specification: "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
