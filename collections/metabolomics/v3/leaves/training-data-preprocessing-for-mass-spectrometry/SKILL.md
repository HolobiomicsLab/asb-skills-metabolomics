---
name: training-data-preprocessing-for-mass-spectrometry
description: 'Use when raw MS/MS spectra (e.g. GNPS MGF/mzML) must be converted into
  model-ready training tensors: peaks must be filtered, intensities transformed, spectra
  binned (10-1000 m/z, 10,000 bins), or augmentation applied before training a spectral
  embedding model.'
when_to_use_negative:
- You only want similarity scores for a few spectra using the pre-trained MS2DeepScore
  model — filtering/binning/augmentation here is for training-time data preparation,
  not inference scoring
- Spectra are already cleaned and binned in the model's expected 10,000-bin format
  — reapplying the filter pipeline would discard information
- The dataset is too small to train a model at all — the README recommends training
  only with > 100,000 spectra of sufficiently diverse types, so preprocessing for
  training is premature
tools:
- name: matchms
  role: Provides the spectrum cleaning pipeline (peak-intensity filtering, peak-count
    limiting) recommended before MS2DeepScore training; MS2DeepScore is integrated
    in matchms
  repo: https://github.com/matchms/matchms
- name: MS2DeepScore
  role: Defines the target data format (binned spectra, sqrt-transformed intensities)
    and implements the three data augmentations applied on binned spectra during training
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:18948eae798a79d82a07bcfa4c2bbbeaba0b5f7b88dfe54fef71138da4175e22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# training-data-preprocessing-for-mass-spectrometry

## Summary

Cleans, filters, bins, and augments MS/MS spectra into the exact input format MS2DeepScore expects before Siamese-network training: intensity filtering, square-root transformation, 10,000-bin binning over 10-1000 m/z, and three binned-spectrum augmentations to make the model robust.

## When to use

Use when raw MS/MS spectra (e.g. GNPS MGF/mzML) must be converted into model-ready training tensors: peaks must be filtered, intensities transformed, spectra binned (10-1000 m/z, 10,000 bins), or augmentation applied before training a spectral embedding model.

## When NOT to use

- You only want similarity scores for a few spectra using the pre-trained MS2DeepScore model — filtering/binning/augmentation here is for training-time data preparation, not inference scoring
- Spectra are already cleaned and binned in the model's expected 10,000-bin format — reapplying the filter pipeline would discard information
- The dataset is too small to train a model at all — the README recommends training only with > 100,000 spectra of sufficiently diverse types, so preprocessing for training is premature

## Inputs

- Raw MS/MS peak lists (m/z, intensity) in MGF, MSP, mzML, mzXML, JSON or USI format, e.g. the GNPS annotated dataset (210,407 spectra)
- Structure annotations (InChIKey and SMILES and/or InChI) per spectrum for compound-level splitting

## Outputs

- Cleaned, binned spectra: 10,000 equal-width bins over 10-1000 m/z with square-root transformed intensities
- Augmented binned spectra regenerated per epoch/batch (random peak removal, intensity jitter, random peak addition)
- Training/validation/test splits defined by unique InChIKeys (14,062/500/500)

## How to apply

First clean spectra in matchms by removing peaks with intensities < 0.1% of the maximum peak intensity and keeping at most the 1000 highest-intensity peaks, then square-root-transform intensities. Bin the peaks into 10,000 equally-sized bins ranging from 10 to 1000 m/z; operate on these binned spectra for all subsequent steps. Apply three augmentations in sequence with independent random draws each epoch: (1) randomly remove a random fraction (0-20%) of non-zero bins whose pre-transformation intensity is below 0.4 (transformed intensity < sqrt(0.4)); (2) multiply non-zero bins by a random factor within 0.6-1.4 (jitter up to +/-40%); (3) insert 0-10 random peaks with intensities drawn uniformly between 0 and 0.01. Split data by unique InChIKeys (training/validation/test = 14,062/500/500) before preprocessing so no compound leaks across splits, then train with Adam optimizer on MSE loss (batch size 32, learning rate 0.001, early stopping after 5 epochs without validation-loss decrease).

## Related tools

- **matchms** (Provides the spectrum cleaning pipeline (peak-intensity filtering, peak-count limiting) recommended before MS2DeepScore training; MS2DeepScore is integrated in matchms) — https://github.com/matchms/matchms
- **MS2DeepScore** (Defines the target data format (binned spectra, sqrt-transformed intensities) and implements the three data augmentations applied on binned spectra during training) — https://github.com/matchms/ms2deepscore

## Examples

```
from ms2deepscore.wrapper_functions.training_wrapper_functions import train_ms2deepscore_wrapper; train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Binned output has exactly 10,000 equal-width bins spanning 10-1000 m/z with square-root-transformed intensities; augmentations leave spectrum shape/dtype unchanged
- Filtering invariant: peaks below 0.1% of maximum intensity removed and no more than the 1000 highest-intensity peaks retained per spectrum
- Augmentation bounds hold per draw: removal fraction <= 0.2 (only bins with intensity < 0.4 pre-transformation), jitter factor within 0.6-1.4, added peaks <= 10 with intensities <= 0.01
- Split check: no InChIKey appears in more than one of the training (n = 14,062), validation (n = 500), and test (n = 500) sets
- Each of the three augmentations is configurable and independently re-drawn per epoch/batch during training

## Limitations

- Spectral libraries used for training are inherently biased toward certain compound classes and must be considered when applying the trained model to experimental data
- Dataset spectra come from different instrumentation types, are of varying quality, and are very unbalanced with regard to represented compound classes
- Trying deeper/wider networks or less restrictive binning could not significantly decrease the overall Tanimoto prediction error, so preprocessing choices are not the bottleneck for the residual RMSE (~0.13-0.2)

## Evidence

- [discussion] removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks.: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [discussion] Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z.: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] we applied three forms of data augmentation on the binned spectra.: "we applied three forms of data augmentation on the binned spectra."
- [discussion] Models were trained with the Adam optimizer [44, 45] that optimized the mean squared error (MSE) loss. We used a batch size of 32, and a learning rate of 0.001.: "Models were trained with the Adam optimizer [44, 45] that optimized the mean squared error (MSE) loss. We used a batch size of 32, and a learning rate of 0.001."
- [readme] Please first ensure cleaning your spectra. We recommend using the cleaning pipeline in [matchms](https://github.com/matchms/matchms).: [evidence span withheld — anchor status: external_doc]
