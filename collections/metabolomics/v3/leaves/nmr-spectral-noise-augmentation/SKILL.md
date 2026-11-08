---
name: nmr-spectral-noise-augmentation
description: Use when training a supervised NMR mixture-identification model and labeled
  experimental mixture spectra are too scarce — synthesize realistic mixtures from
  measured pure-compound 1H NMR spectra with variability simulation.
when_to_use_negative:
- Evaluation on real experimental mixtures where synthetic weighted-sum spectra with
  Gaussian shift/noise would not match true measurement conditions — use the two experimental
  flavor-mixture datasets for that purpose.
- Analysis tasks requiring quantified concentrations rather than binary presence/absence
  identification of compounds.
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Implements the synthetic mixture sampling, weighted-sum spectrum computation,
    Gaussian shift/noise perturbation, and label assignment
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: PyTorch
  role: Framework in which the augmented spectral-pair data are consumed for training
    FlavorFormer
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: FlavorFormer
  role: Downstream model trained on the augmented datasets and evaluated on the augmented
    test set and two experimental datasets
  repo: https://github.com/yfWang01/FlavorFormer.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:f218e0189e23e277f929ffa0d644e2d14b63feae5e2b3d4775f5c9b94dcb5521
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - FlavorFormer
  - PyTorch
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# nmr-spectral-noise-augmentation

## Summary

A synthetic data-augmentation strategy for 1H NMR mixture identification that builds simulated mixture spectra by weighted-summing 1–7 of 24 flavor-standard spectra with coefficients in [0.2, 1.0], perturbing them with Gaussian chemical-shift deviations and random noise, and assigning binary per-compound labels to form positive/negative spectral pairs for training FlavorFormer.

## When to use

Use when training a supervised NMR mixture-identification model and labeled experimental mixture spectra are too scarce — synthesize realistic mixtures from measured pure-compound 1H NMR spectra with variability simulation.

## When NOT to use

- Evaluation on real experimental mixtures where synthetic weighted-sum spectra with Gaussian shift/noise would not match true measurement conditions — use the two experimental flavor-mixture datasets for that purpose.
- Analysis tasks requiring quantified concentrations rather than binary presence/absence identification of compounds.

## Inputs

- 24 standard-compound 1H NMR spectra from the FlavorFormer data/ folder
- augmentation parameters: number of components (1–7 per mixture), mixing coefficient range [0.2, 1.0], Gaussian chemical-shift deviation, random spectral noise

## Outputs

- augmented_mixtures.npz containing simulated mixture spectra and per-compound binary label arrays (positive if the target compound spectrum is present in the mixture, negative otherwise)
- augmented training and test sets of simulated mixtures with binary labels for all 24 flavor compounds

## How to apply

Load the 24 standard-compound 1H NMR spectra from the data/ folder of the FlavorFormer repository. For each synthetic mixture, randomly sample 1–7 of the 24 standards and draw a mixing coefficient in the range 0.2–1.0 for each selected standard, then compute the weighted-sum mixture spectrum. Simulate measurement variability by perturbing the mixture with Gaussian chemical-shift deviations and adding random spectral noise. Assign binary labels for all 24 candidate compounds (positive if the compound's spectrum was included in the mixture, negative otherwise), and repeat to build the augmented dataset saved as augmented_mixtures.npz. The resulting spectra–label pairs are what allow the bi-encoder/cross-encoder FlavorFormer to learn compound identification; the augmented test set drawn from the same synthetic process is used alongside two experimental datasets for evaluation.

## Related tools

- **Python** (Implements the synthetic mixture sampling, weighted-sum spectrum computation, Gaussian shift/noise perturbation, and label assignment) — https://github.com/yfWang01/FlavorFormer.git
- **PyTorch** (Framework in which the augmented spectral-pair data are consumed for training FlavorFormer) — https://github.com/yfWang01/FlavorFormer.git
- **FlavorFormer** (Downstream model trained on the augmented datasets and evaluated on the augmented test set and two experimental datasets) — https://github.com/yfWang01/FlavorFormer.git

## Examples

```
git clone https://github.com/yfWang01/FlavorFormer.git; cd FlavorFormer; conda env create -f environment.yml; conda activate FlavorFormer; # run demo.ipynb to train on augmented_mixtures.npz
```

## Evaluation signals

- Output file augmented_mixtures.npz exists and contains aligned spectra and binary label arrays covering all 24 candidate compounds.
- Each synthetic mixture contains 1–7 components, and every mixing coefficient falls within [0.2, 1.0].
- Label of a target compound is positive exactly when that standard's spectrum contributed to the weighted sum, negative otherwise.
- Perturbed spectra differ from the clean weighted sum by Gaussian chemical-shift deviations plus added noise, simulating measurement variability.
- A FlavorFormer trained on the augmented data reaches reported test-set performance on the augmented test set (ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %).

## Limitations

- Synthetic mixtures are linear weighted sums with Gaussian shift and noise perturbations; they may not capture all real measurement artifacts, which is why evaluation also uses two experimental datasets (known mixtures ACC 95.44 ± 0.00 %, unknown mixtures ACC 94.44 ± 2.41 %).
- The augmentation is tied to the specific set of 24 flavor standards; new compound panels require regenerating the augmented dataset.
- Labels are strictly binary (compound present/absent); the synthetic pairs carry no concentration ground truth beyond the mixing coefficients.

## Evidence

- [other] The augmentation synthesizes mixture spectra by randomly combining 1–7 of the 24 flavor standards, each weighted by a random coefficient between [0.2, 1.0]: [evidence span withheld — anchor status: not_found]
- [other] spectra then receive Gaussian chemical-shift adjustments and random noise, and the pair is labeled positive if the mixture contains the target compound spectrum: [evidence span withheld — anchor status: not_found]
- [methods] Load the 24 standard-compound 1H NMR spectra from the data/ folder: "The model parameters is located in the [model](model/) folder and the required data is in the [data](data/) folder, FlavorFormer can be reload and predict easily."
- [abstract] After training on augmented datasets, FlavorFormer was tested on an augmented test set and two experimental datasets.: "After training on augmented datasets, FlavorFormer was tested on an augmented test set and two experimental datasets."
