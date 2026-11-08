---
name: spectrum-preprocessing-denoising
description: Use when raw GNPS-derived MGF MS/MS spectra must be normalized and denoised
  before feeding a spectral-similarity model such as TransExION, Cosine, Spec2Vec,
  or MS2DeepScore.
when_to_use_negative:
- Input is already a preprocessed h5py spectrum database produced by ms_convert_mgf2h5py.py
  — re-filtering would double-apply the thresholds.
- The downstream task needs low-mass reporter ions or diagnostic peaks below 10 Da
  or above 1000 Da, which this filter discards.
- The analysis targets raw profile-mode data or peak picking, rather than peak-list-level
  cleaning of centroided library spectra.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: TransExION
  role: Downstream model whose scripts consume the preprocessed h5py spectrum files
  repo: github.com/banhdzui/TransExION
- name: h5py
  role: Storage format for large-scale preprocessed spectrum databases
  repo: github.com/banhdzui/TransExION
- name: pyteomics
  role: Dependency of the TransExION repository supporting spectrum parsing/conversion
  repo: github.com/banhdzui/TransExION
- name: PyTorch
  role: Framework of the TransExION scripts that consume the preprocessed spectra
  repo: github.com/banhdzui/TransExION
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:4db8e6ecfb35ba8157c3fcf2017edc311b4ecbd6528f15240864ac4f8760226d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore
  - Matplotlib
  - PyTorch
  - RDKit
  - Spec2Vec
  - TransExION
  - h5py
  - pyteomics
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# spectrum-preprocessing-denoising

## Summary

Cleans raw MS/MS spectra before similarity modeling by rounding m/z values to two decimals, converting peak intensities to relative abundance percentages, and removing low-intensity and out-of-range peaks (<0.1% intensity, outside 10–1000 Da), with optional MGF-to-h5py conversion for large-scale processing.

## When to use

Use when raw GNPS-derived MGF MS/MS spectra must be normalized and denoised before feeding a spectral-similarity model such as TransExION, Cosine, Spec2Vec, or MS2DeepScore.

## When NOT to use

- Input is already a preprocessed h5py spectrum database produced by ms_convert_mgf2h5py.py — re-filtering would double-apply the thresholds.
- The downstream task needs low-mass reporter ions or diagnostic peaks below 10 Da or above 1000 Da, which this filter discards.
- The analysis targets raw profile-mode data or peak picking, rather than peak-list-level cleaning of centroided library spectra.

## Inputs

- MGF-format MS/MS spectrum files (e.g., GNPS_MassBank_train.mgf, GNPS_MassBank_val.mgf, mergedGNPS test/reference spectra)
- Raw peak lists with m/z and intensity values

## Outputs

- Cleaned spectra with m/z rounded to two decimals, intensities as relative abundance %, and peaks <0.1% intensity or outside 10–1000 Da removed
- h5py-format spectrum databases (e.g., GNPS_MassBank_train.db) ready for TransExION training/testing scripts

## How to apply

Start from MGF-format spectrum files (e.g., GNPS_MassBank_train.mgf or mergedGNPS test/reference spectra) and round every m/z value to two decimals to harmonize peak positions across instruments. Convert absolute intensities to relative abundance percentages so spectra are comparable regardless of acquisition intensity scaling. Remove peaks with relative intensity below 0.1% and peaks outside the 10–1000 Da m/z range, which eliminates noise and detector artifacts without affecting chemically meaningful fragments. For large-scale datasets, convert the cleaned MGF files into h5py format with ms_convert_mgf2h5py.py, because the TransExION training and testing scripts (ms_train_spectal_similarity.py, ms_test_spectral_similarity.py) consume h5py database files.

## Related tools

- **TransExION** (Downstream model whose scripts consume the preprocessed h5py spectrum files) — github.com/banhdzui/TransExION
- **h5py** (Storage format for large-scale preprocessed spectrum databases) — github.com/banhdzui/TransExION
- **pyteomics** (Dependency of the TransExION repository supporting spectrum parsing/conversion) — github.com/banhdzui/TransExION
- **PyTorch** (Framework of the TransExION scripts that consume the preprocessed spectra) — github.com/banhdzui/TransExION

## Examples

```
python ms_convert_mgf2h5py.py --mgf GNPS_MassBank_train.mgf --out GNPS_MassBank_train.db
```

## Evaluation signals

- No remaining peak has relative abundance below 0.1% or m/z outside the 10–1000 Da window.
- All m/z values contain at most two decimal places.
- Intensity vectors per spectrum sum to 100% after relative-abundance conversion.
- The converted h5py database loads without error in ms_train_spectal_similarity.py / ms_test_spectral_similarity.py.
- Downstream squared-error box-plots across Tanimoto bins reproduce the reported low, stable error pattern (TransExION r = 0.811 on mergedGNPS).

## Limitations

- The 10–1000 Da and 0.1% intensity thresholds are specific to the GNPS-derived library spectra used in this study; other acquisition types may need different cutoffs.
- The pipeline applies to centroided peak-list (MGF) data, not raw profile-mode MS/MS.
- The README does not report code versions, environment, or dependency pins for the preprocessing scripts.

## Evidence

- [methods] preprocessed by rounding m/z to two decimals, converting intensities to relative abundance %, and removing peaks <0.1% intensity or outside 10–1000 Da: [evidence span withheld — anchor status: not_found]
- [readme] To work with large scale dataset, the input data files are converted into h5py format.: "To work with large scale dataset, the input data files are converted into h5py format."
- [readme] Our library supports a function to convert a MGF format file into h5py format.: "Our library supports a function to convert a MGF format file into h5py format."
