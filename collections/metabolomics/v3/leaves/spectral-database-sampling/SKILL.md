---
name: spectral-database-sampling
description: Use when building a training/validation/test corpus of pseudo-mixture
  NMR spectra for a Siamese-style compound identifier, when only a standard-spectrum
  database and experimental mixture spectra are available.
when_to_use_negative:
- The input is already a labeled pair dataset ready for pSCNN training — no augmentation
  needed
- The goal is benchmarking on real experimental mixtures (flavor mixtures or additional
  flavor mixture datasets), which is done without synthetic augmentation
- The spectral data are not from an NMR standard-spectrum database of small molecules,
  so superposition semantics do not hold
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: Python 3.8.13
  role: Runtime environment for implementing the superposition/noise/sampling augmentation
    pipeline
  repo: https://github.com/yuxuanliao/pSCNN
- name: TensorFlow 2.5.0-GPU
  role: Consumes the augmented pair arrays for pSCNN training, validation and testing
  repo: https://github.com/yuxuanliao/pSCNN
- name: tqdm
  role: Progress tracking during iterative spectrum sampling and pair construction
  repo: https://github.com/yuxuanliao/pSCNN
- name: Anaconda
  role: Environment manager used to install the pSCNN conda environment with Python
    3.8.13
- name: Zenodo deposit 10.5281/zenodo.6504814
  role: Source of the model and all NMR data (standard database and example mixtures)
    used by the augmentation
  repo: https://doi.org/10.5281/zenodo.6504814
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:3ea0809401cd7a1e1a508c72e42790c883eeaaf0e73472dcda990d2fe9d1bb00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN
  tools:
  - Anaconda
  - Python
  - Python 3.8.13
  - TensorFlow 2.5.0-GPU
  - Zenodo deposit 10.5281/zenodo.6504814
  - tqdm
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# spectral-database-sampling

## Summary

Generates augmented NMR spectral pairs for pSCNN training by superposing randomly sampled flavor-standard database spectra at random mixing ratios (e.g., 0.2–1.0) with added random noise; positive pairs contain a given compound, negative pairs omit it.

## When to use

Use when building a training/validation/test corpus of pseudo-mixture NMR spectra for a Siamese-style compound identifier, when only a standard-spectrum database and experimental mixture spectra are available.

## When NOT to use

- The input is already a labeled pair dataset ready for pSCNN training — no augmentation needed
- The goal is benchmarking on real experimental mixtures (flavor mixtures or additional flavor mixture datasets), which is done without synthetic augmentation
- The spectral data are not from an NMR standard-spectrum database of small molecules, so superposition semantics do not hold

## Inputs

- flavor-standard NMR spectral database (from Zenodo 10.5281/zenodo.6504814)
- experimental flavor mixtures NMR spectra (benchmark dataset)
- additional flavor mixture NMR dataset (for benchmarking)

## Outputs

- Augmented training, validation and test spectral pair sets (positive and negative pairs) as TensorFlow-compatible arrays
- Augmented pseudo-mixture spectra (e.g., synthetic F10 built from Sulcatone, L-Menthone, Citronellal, Leaf acetate at documented ratios)

## How to apply

Load the flavor-standard NMR spectral database and experimental flavor mixture spectra from the Zenodo deposit (10.5281/zenodo.6504814) with Python 3.8.13. Randomly sample several standard spectra and superimpose them at random mixing ratios in the 0.2–1.0 range to synthesize pseudo-mixture spectra, adding random noise to mimic experimental acquisition variability. For each target compound, form positive pairs by superposing its spectrum with other randomly sampled database spectra; form negative pairs identically but without that compound's spectrum. Split the augmented dataset into training, validation and test sets and export as TensorFlow-compatible arrays (version 2.5.0-GPU), tracking progress with tqdm. To reproduce the F10 example, superpose Sulcatone, L-Menthone, Citronellal and Leaf acetate standard spectra at the documented ratios and compare peak positions/intensities against the experimental F10 spectrum.

## Related tools

- **Python 3.8.13** (Runtime environment for implementing the superposition/noise/sampling augmentation pipeline) — https://github.com/yuxuanliao/pSCNN
- **TensorFlow 2.5.0-GPU** (Consumes the augmented pair arrays for pSCNN training, validation and testing) — https://github.com/yuxuanliao/pSCNN
- **tqdm** (Progress tracking during iterative spectrum sampling and pair construction) — https://github.com/yuxuanliao/pSCNN
- **Anaconda** (Environment manager used to install the pSCNN conda environment with Python 3.8.13)
- **Zenodo deposit 10.5281/zenodo.6504814** (Source of the model and all NMR data (standard database and example mixtures) used by the augmentation) — https://doi.org/10.5281/zenodo.6504814

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; pip install -r requirements.txt; pip install tqdm; git clone https://github.com/yuxuanliao/pSCNN.git; python pSCNN.py
```

## Evaluation signals

- Augmented F10 spectrum's peaks visually align with the experimental F10 spectrum when plotted side-by-side (Figure 3 style)
- Positive pairs contain the target compound's spectrum; negative pairs generated the same way lack it
- Mixing ratios fall within the documented 0.2–1.0 range and noise is added to every superposed spectrum
- Downstream pSCNN performance on the augmented test set reaches ACC = 99.80%, TPR = 99.70%, FPR = 0.10%
- Arrays are loadable by TensorFlow 2.5.0-GPU without shape/dtype errors

## Limitations

- Augmented performance (99.80% ACC) does not transfer directly to real mixtures: flavor mixtures dataset drops to ACC = 97.62% and additional flavor mixture dataset to ACC = 91.67% (FPR = 10.53%)
- Synthetic superposition may not fully capture chemical shift variation and peak overlapping seen in experimental spectra
- Model exceeds repository size limits, so data and model must be fetched from Zenodo rather than the GitHub repo

## Evidence

- [abstract] Data augmentation by superposition of several NMR spectra sampled from a spectral database with random noises: "A data augmentation method was implemented for the superposition of several NMR spectra sampled from a spectral database with random noises."
- [abstract] Performance on augmented test set: "the proposed method can achieve good performances in the augmented test set (ACC = 99.80%, TPR = 99.70% and FPR = 0.10%)"
- [full_text] Augmentation procedure detail from source card: [evidence span withheld — anchor status: not_found]
- [readme] README install/run instructions: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
