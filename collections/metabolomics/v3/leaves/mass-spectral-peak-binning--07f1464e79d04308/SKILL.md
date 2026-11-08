---
name: mass-spectral-peak-binning
description: Use when raw mass spectral peak lists (m/z, intensity) must be converted
  into fixed-length numeric feature vectors for a classifier, or when spectra from
  different acquisition modes (EI-MS vs ESI-MS/MS) must be projected onto a common
  m/z scale for training or comparison.
when_to_use_negative:
- The input is a chemical structure (SMILES/descriptors) rather than a spectrum —
  BitterPredict predicts bitterness from structure, not from spectral data
- Spectra were acquired in negative ion mode — BitterMasS predictions are based on
  EI-MS or ESI-MS/MS spectra obtained in positive ion mode only
- The input is already a fixed-length feature table aligned to a common m/z grid,
  so re-binning would discard or duplicate information
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: BitterMasS
  role: Downstream Random Forest classifier that consumes the 0.1 m/z-binned feature
    matrices; the BitterMasS Random Forest pipeline serves as the reference implementation
    for training the six compared models
- name: MassBank
  role: Source of the experimental EI-MS and ESI-MS/MS positive ion mode mass spectra
    that are binned into the feature matrices
- name: BitterPredict
  role: 'Contrast tool: structure-based bitterness classifier used for the spectrum–structure–bitterness
    strategy that the binned-spectrum direct prediction route is compared against'
  repo: Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:ca5a7664b8e348eb44218a3eaa034b2fb55116633d6c171a32f3dfd50f779873
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  - MassBank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mass-spectral-peak-binning

## Summary

Bins experimental EI-MS and ESI-MS/MS peak intensities onto a shared 0.1 m/z grid to build fixed-length feature matrices for machine-learning bitterness classification, as done for the BitterMasS Random Forest classifier trained on 5414 experimental mass spectra.

## When to use

Use when raw mass spectral peak lists (m/z, intensity) must be converted into fixed-length numeric feature vectors for a classifier, or when spectra from different acquisition modes (EI-MS vs ESI-MS/MS) must be projected onto a common m/z scale for training or comparison.

## When NOT to use

- The input is a chemical structure (SMILES/descriptors) rather than a spectrum — BitterPredict predicts bitterness from structure, not from spectral data
- Spectra were acquired in negative ion mode — BitterMasS predictions are based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode only
- The input is already a fixed-length feature table aligned to a common m/z grid, so re-binning would discard or duplicate information

## Inputs

- Experimental EI-MS peak lists (m/z, intensity) in positive ion mode, e.g. extracted from MassBank
- Experimental ESI-MS/MS peak lists (m/z, intensity) in positive ion mode, e.g. extracted from MassBank
- Bitter/nonbitter class labels for the compounds represented by the spectra

## Outputs

- EI-only binned feature matrix on the 0.1 m/z grid
- ESI-MS/MS-only binned feature matrix on the 0.1 m/z grid
- Combined EI-MS + ESI-MS/MS binned feature matrix on the 0.1 m/z grid
- balanced_accuracy_comparison.csv recording balanced accuracy per algorithm–dataset pair

## How to apply

Extract experimental EI-MS and ESI-MS/MS positive ion mode spectra (BitterMasS sourced these from the MassBank database) and bin each spectrum's peak intensities onto a shared 0.1 m/z grid so spectra of different lengths become comparable feature vectors. Produce three feature matrices: an EI-only set, an ESI-MS/MS-only set, and a combined EI-MS + ESI-MS/MS set, so the contribution of each acquisition mode can be tested separately. Split each matrix into training and held-out test splits, then feed them to cross-validated grid searches (fivefold CV) for candidate algorithms such as Random Forest and XGBoost before training the final models. Evaluate on balanced accuracy; in BitterMasS, the combined-set models outperformed the individual-set models and Random Forest edged out XGBoost by roughly 0.01, which justified both the binning granularity and the algorithm choice.

## Related tools

- **BitterMasS** (Downstream Random Forest classifier that consumes the 0.1 m/z-binned feature matrices; the BitterMasS Random Forest pipeline serves as the reference implementation for training the six compared models)
- **MassBank** (Source of the experimental EI-MS and ESI-MS/MS positive ion mode mass spectra that are binned into the feature matrices)
- **BitterPredict** (Contrast tool: structure-based bitterness classifier used for the spectrum–structure–bitterness strategy that the binned-spectrum direct prediction route is compared against) — Niv-Lab/BitterPredict1

## Examples

```
import numpy as np; binned, _ = np.histogram(mz, bins=np.arange(0, 1000.0, 0.1), weights=intensity)
```

## Evaluation signals

- Three binned matrices (EI-only, ESI-only, combined) are produced on the same 0.1 m/z grid with consistent feature dimensions
- Combined-set balanced accuracy must exceed both the EI-only and ESI-only balanced accuracies; individual-set prediction was poor compared with the combined set
- Random Forest's balanced-accuracy advantage over XGBoost should be approximately 0.01 after fivefold cross-validated grid search on all three datasets
- Six best-configured models (three Random Forest, three XGBoost) are trained and evaluated on held-out test splits, not on the CV folds
- BitterMasS reference performance: precision = 0.83 and recall = 0.90 on the internal test set, giving an expected ballpark for the pipeline output

## Limitations

- Only 30% of the compounds in the initial bitter and nonbitter training set had at least one mass spectrum identified, limiting training data for the binned matrices
- Nonbitter compound spectral representation in the training set is insufficient, so class balance in the binned feature matrix is skewed
- No software/code availability or hosting location is provided for BitterMasS itself, so the binning and training pipeline must be reimplemented; most articles reporting experimental mass spectra do not include the actual spectra, complicating data assembly

## Evidence

- [abstract] Random Forest classifier trained on experimental spectra: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [discussion] Spectra sourced from MassBank in positive ion mode: "Experimental EI-MS and ESI-MS/MS positive ion mode mass spectra for these compounds were extracted from the MassBank database"
- [discussion] Combined-set binning outperforms individual sets: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] Sparse spectral coverage limits training: "only 30% of the compounds in the initial bitter and nonbitter sets in the training set had at least one mass spectrum identified"
