---
name: peak-picking-from-profile-spectra
description: Use when raw profile-mode MS2 spectra (e.g. from lipid standard measurements
  stored in mzML) must be converted to centroided peaks prior to normalization, filtering,
  and HDF5 storage for spectral model training.
when_to_use_negative:
- Input spectra are already centroided — profile-only steps (smoothing, baseline correction,
  peak picking) should be skipped, though base peak normalization and 50–1600 m/z
  trimming still apply
- The goal is prediction/annotation on new mzML files rather than rebuilding the training
  dataset — use the LipiDetective prediction workflow instead
- The data are not lipid MS2 tandem spectra (e.g. proteomics DDA raw data), since
  the 0.01 noise threshold and m/z range were calibrated for lipid standard spectra
edam_operation: http://edamontology.org/operation_3215
tools:
- name: pyOpenMS
  role: Python library used to perform smoothing, baseline correction, and peak picking
    on the profile spectra
- name: h5py
  role: Used to save the processed spectra into an HDF5 file for downstream training
- name: LipiDetective
  role: Downstream framework whose training data is produced by this preprocessing
    step; configured via YAML, trained from HDF5 files
  repo: https://github.com/LipiTUM/lipidetective
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:26ba3719166dba26b42c95e3135ec15dcba01fe5cdaa7c4e2b8d9e31dde82335
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - h5py
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# peak-picking-from-profile-spectra

## Summary

Convert raw profile-mode MS2 spectra into centroided peak lists as part of the LipiDetective training-data pipeline: profile spectra from lipid standard measurements are smoothed, baseline-corrected, and peak-picked with pyOpenMS before base peak normalization and noise filtering.

## When to use

Use when raw profile-mode MS2 spectra (e.g. from lipid standard measurements stored in mzML) must be converted to centroided peaks prior to normalization, filtering, and HDF5 storage for spectral model training.

## When NOT to use

- Input spectra are already centroided — profile-only steps (smoothing, baseline correction, peak picking) should be skipped, though base peak normalization and 50–1600 m/z trimming still apply
- The goal is prediction/annotation on new mzML files rather than rebuilding the training dataset — use the LipiDetective prediction workflow instead
- The data are not lipid MS2 tandem spectra (e.g. proteomics DDA raw data), since the 0.01 noise threshold and m/z range were calibrated for lipid standard spectra

## Inputs

- Raw profile-mode MS2 spectra from lipid standard measurements and source datasets (e.g. PNNL dataset with 46,724 MS2 spectra from 1,790 lipids), read as mzML/HDF5 containers
- Lipid nomenclature metadata from the different data sources (manually adjusted to the updated lipid shorthand notation)

## Outputs

- Centroided, base-peak-normalized MS2 peak lists (intensities in 0–1, m/z range 50–1600)
- HDF5 file (Hierarchical Data Format 5) of cleaned spectra for downstream LipiDetective training

## How to apply

Load the raw profile MS2 spectra (the LipiDetective training data came from lipid reference-standard measurements at varying collision energies plus public datasets) and process each profile spectrum with the pyOpenMS library, applying smoothing, baseline correction, and peak picking in that order. Smoothing and baseline correction must precede peak picking so that low-intensity chemical and electronic noise is not picked as peaks. After peak picking, perform base peak normalization by dividing all peak intensities by the highest-intensity peak so intensities lie between zero and one, trim spectra to the m/z range 50–1600, and compute the base peak normalized median intensity as a noise score; spectra with a score above 0.01 are excluded. The resulting centroided spectra are then saved in an HDF5 file (written with h5py) for downstream training of the transformer model.

## Related tools

- **pyOpenMS** (Python library used to perform smoothing, baseline correction, and peak picking on the profile spectra)
- **h5py** (Used to save the processed spectra into an HDF5 file for downstream training)
- **LipiDetective** (Downstream framework whose training data is produced by this preprocessing step; configured via YAML, trained from HDF5 files) — https://github.com/LipiTUM/lipidetective

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- All retained spectra have a base peak normalized median intensity at or below the 0.01 threshold
- Peak intensities after normalization fall within the range 0–1
- All MS2 spectra peaks lie within the trimmed m/z range 50–1600
- No profile-mode spectra remain in the output — every spectrum has undergone peak picking
- Output HDF5 file loads correctly with h5py and matches the spectra counts expected from the source datasets (e.g. 268,720 training spectra plus 2,002 blank spectra)

## Limitations

- The 0.01 base peak normalized median intensity threshold was chosen for phospholipid standard measurements; spectra noisier than this are excluded, so very noisy real-sample spectra may be dropped
- Profile spectra must undergo the full smoothing/baseline-correction/peak-picking chain first; applying the noise filter to raw profile spectra is invalid
- Spectra from blank measurements (no target lipid species) must be retained separately so the model learns to predict the end-of-sequence token

## Evidence

- [methods] profile spectra processing: "All profile spectra underwent smoothing, baseline correction, and peak picking"
- [methods] noise threshold: "a threshold of 0.01 base peak normalized median intensity was chosen, and any spectra with a higher score were excluded"
- [methods] m/z trimming: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] pyOpenMS usage: "the pyOpenMS [21] library was used"
- [readme] HDF5 storage and tool roles: [evidence span withheld — anchor status: not_found]
