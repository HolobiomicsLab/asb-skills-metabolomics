---
name: spectrum-pair-data-augmentation
description: Use when training MS2DeepScore (or a similar Siamese spectral model)
  on annotated MS/MS spectra and you need to expand limited spectral data into diverse
  training pairs with Tanimoto-score labels.
when_to_use_negative:
- When the goal is only to compute similarities with an already-trained MS2DeepScore
  model — augmentation is a training-time technique, not needed for inference
- When the input spectra have not first been cleaned and binned per the matchms/MS2DeepScore
  pipeline (peaks <0.1% max intensity removed, ≤1000 peaks, 10,000 bins over 10–1000
  m/z)
- When you cannot verify pair sampling quality on a small training set — the README
  warns pair sampling can be suboptimal if not checked for smaller datasets
tools:
- name: MS2DeepScore
  role: Siamese neural network trained on the augmented spectrum pairs to predict
    Tanimoto scores
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Spectrum cleaning pipeline recommended before augmentation and pair generation
- name: RDKit
  role: Computes Daylight 2048-bit fingerprints and Tanimoto labels for the sampled
    spectrum pairs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:54046a947756f2d377b62b784b1a4d90605feb15f47a8338f3c8d7bfc3420d5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - RDKit
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Spectrum pair data augmentation

## Summary

Generates training pairs for MS2DeepScore's Siamese network by applying three data augmentations (low-intensity peak removal, intensity jitter, new peak addition) to binned GNPS-derived MS/MS spectra, expanding >100,000 spectra of ~15,062 unique InChIKeys into diverse Tanimoto-labelled training pairs.

## When to use

Use when training MS2DeepScore (or a similar Siamese spectral model) on annotated MS/MS spectra and you need to expand limited spectral data into diverse training pairs with Tanimoto-score labels.

## When NOT to use

- When the goal is only to compute similarities with an already-trained MS2DeepScore model — augmentation is a training-time technique, not needed for inference
- When the input spectra have not first been cleaned and binned per the matchms/MS2DeepScore pipeline (peaks <0.1% max intensity removed, ≤1000 peaks, 10,000 bins over 10–1000 m/z)
- When you cannot verify pair sampling quality on a small training set — the README warns pair sampling can be suboptimal if not checked for smaller datasets

## Inputs

- Cleaned annotated MS/MS spectra (e.g., GNPS-derived MGF; Zenodo record 4699300, 210,407 spectra)
- RDKit Daylight 2048-bit molecular fingerprints (from SMILES/InChI) for Tanimoto label computation
- InChIKey-based molecule-level split definitions (train/validation/test)

## Outputs

- Augmented binned spectrum pairs with corresponding Tanimoto similarity labels
- Trained Siamese model weights (e.g., ms2deepscore_model.hdf5)

## How to apply

First clean and bin each spectrum per the MS2DeepScore pipeline: remove peaks <0.1% of maximum intensity, keep at most the 1000 highest-intensity peaks, square-root transform intensities, and bin peaks into 10,000 equal-width bins from 10 to 1000 m/z. On these binned spectra, apply the three augmentations — low-intensity peak removal, intensity jitter, and new peak addition — to generate variability within each compound's spectra. Sample spectrum pairs across molecules and compute Tanimoto labels with RDKit Daylight 2048-bit fingerprints; split by unique InChIKey (14,062 train / 500 validation / 500 test) so augmentation never leaks across the split. Before committing to training, verify pair sampling quality (e.g., Tanimoto-bin coverage), since the README warns pair sampling can be suboptimal if not re-optimized for new datasets, particularly for smaller training sets. Train with Adam optimizer, MSE loss, batch size 32, learning rate 0.001, and early stopping after 5 epochs without validation-loss decrease.

## Related tools

- **MS2DeepScore** (Siamese neural network trained on the augmented spectrum pairs to predict Tanimoto scores) — https://github.com/matchms/ms2deepscore
- **matchms** (Spectrum cleaning pipeline recommended before augmentation and pair generation)
- **RDKit** (Computes Daylight 2048-bit fingerprints and Tanimoto labels for the sampled spectrum pairs)

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; settings = SettingsMS2Deepscore(spectrum_file_path="./combined_libraries.mgf", ionisation_mode="both")
```

## Evaluation signals

- Test-set RMSE between predicted and true Tanimoto scores falls in the 0.13–0.2 range on 3601 spectra of 500 unseen compounds
- Predicted Tanimoto scores span roughly 0.1–0.9, with the largest errors in the extreme (highest/lowest) similarity bins
- Pair sampling check shows adequate Tanimoto-bin coverage before training; quality must be re-verified for new datasets
- Early stopping triggers within 5 epochs of no validation-loss decrease, indicating the augmented pairs are not overfit
- Augmented spectra respect binning constraints (10,000 equal-width bins, 10–1000 m/z)

## Limitations

- Pair sampling quality depends on the dataset and can be suboptimal, particularly for smaller training sets
- The authors could not significantly decrease the overall Tanimoto prediction error with deeper/wider networks or less restrictive binning
- GNPS-derived training data are of varying quality, from different instrumentation types, and biased toward certain compound classes
- Trained scores are not deterministic and may change when a model is retrained with different parameters or data

## Evidence

- [discussion] Data augmentation applied to binned spectra: "we applied three forms of data augmentation on the binned spectra."
- [discussion] Preprocessing parameters for spectra: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [discussion] Binning specification: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [readme] Pair sampling quality caveat from README: [evidence span withheld — anchor status: external_doc]
- [discussion] Molecule-level data split: "The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500)."
