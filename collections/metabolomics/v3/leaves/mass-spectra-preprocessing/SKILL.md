---
name: mass-spectra-preprocessing
description: Use when raw profile MS2 lipid spectra must be converted into clean peak
  lists for downstream model training or spectral-library building, particularly when
  noisy spectra should be filtered by a base-peak-normalized median intensity threshold.
when_to_use_negative:
- Input spectra are already centroided and peak-picked — the smoothing/baseline-correction/peak-picking
  step is only for profile spectra
- The analysis targets annotated MS1/feature tables rather than raw tandem MS2 spectra
  requiring model-ready cleanup
- The downstream tool requires a different m/z range or its own normalization scheme
  incompatible with 50–1600 m/z and base-peak normalization
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipiDetective
  role: Target pipeline whose preprocessing conventions (0.01 noise threshold, 50–1600
    m/z trimming, HDF5 storage) this skill reproduces
  repo: https://github.com/LipiTUM/lipidetective
- name: pyOpenMS
  role: Used for smoothing, baseline correction, and peak picking of profile spectra
- name: h5py
  role: Writes the processed spectra into an HDF5 training file
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:f20df2009d50b5338d1dd5fc0c06c0594c304d0e681dd5b51807d0109fa6d519
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - h5py
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mass-spectra-preprocessing

## Summary

Preprocesses raw MS2 spectra for LipiDetective training: profile-spectrum peak picking with pyOpenMS, base peak normalization, m/z trimming to 50–1600, and exclusion of noisy spectra above a 0.01 base-peak-normalized median intensity threshold.

## When to use

Use when raw profile MS2 lipid spectra must be converted into clean peak lists for downstream model training or spectral-library building, particularly when noisy spectra should be filtered by a base-peak-normalized median intensity threshold.

## When NOT to use

- Input spectra are already centroided and peak-picked — the smoothing/baseline-correction/peak-picking step is only for profile spectra
- The analysis targets annotated MS1/feature tables rather than raw tandem MS2 spectra requiring model-ready cleanup
- The downstream tool requires a different m/z range or its own normalization scheme incompatible with 50–1600 m/z and base-peak normalization

## Inputs

- Raw profile MS2 spectra (e.g. from Thermo/phospholipid standard measurements, PNNL, HCE, IOBA-NHC, MITOMICS datasets)
- Raw mzML files containing profile spectra

## Outputs

- Cleaned peak-list spectra (centroided, normalized, trimmed to 50–1600 m/z)
- HDF5 (.h5) file of processed spectra suitable for LipiDetective training

## How to apply

Load raw profile MS2 spectra and process each with pyOpenMS, applying smoothing, baseline correction, and peak picking. Base peak normalize every spectrum by dividing each peak intensity by the spectrum's highest intensity so values lie in 0–1, then trim MS2 spectra to the m/z range 50–1600 as LipiDetective does. Compute each spectrum's noise score as the median intensity divided by the base peak intensity and exclude any spectrum whose score exceeds 0.01 (the threshold chosen in the paper). Optionally harmonize lipid nomenclature across sources to the updated lipid shorthand notation, then save the cleaned spectra in an HDF5 file via h5py for LipiDetective training. Retained spectra should show base-peak-normalized median intensity ≤ 0.01 and all intensities within 0–1.

## Related tools

- **LipiDetective** (Target pipeline whose preprocessing conventions (0.01 noise threshold, 50–1600 m/z trimming, HDF5 storage) this skill reproduces) — https://github.com/LipiTUM/lipidetective
- **pyOpenMS** (Used for smoothing, baseline correction, and peak picking of profile spectra)
- **h5py** (Writes the processed spectra into an HDF5 training file)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- Retained spectra have base-peak-normalized median intensity ≤ 0.01 (noise score below threshold)
- All peak intensities in each output spectrum fall within 0–1 after base peak normalization
- Output spectra span the 50–1600 m/z range with no peaks outside it
- Output is stored in a valid HDF5 file readable by h5py and usable by LipiDetective's YAML-configured training pipeline

## Limitations

- The 0.01 threshold was tuned for phospholipid standard measurements; datasets with intrinsically noisier spectra may need threshold recalibration
- Profile spectra require the pyOpenMS smoothing/baseline-correction/peak-picking path; already-processed spectra may be degraded if re-run
- Peaks relevant to prediction (e.g. 224.0 m/z) lack annotations in Alex123, so preprocessing does not resolve interpretability gaps
- Nomenclature harmonization across data sources was performed manually in the paper and may require expert curation

## Evidence

- [methods] Profile-spectrum peak picking and preprocessing: "All profile spectra underwent smoothing, baseline correction, and peak picking"
- [methods] Noise threshold: "a threshold of 0.01 base peak normalized median intensity was chosen, and any spectra with a higher score were excluded"
- [methods] m/z trimming: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] Normalization rationale: "base peak normalization was performed for all spectra"
- [methods] Output format: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
- [methods] Library support: "the pyOpenMS [21] library was used"
- [readme] Training data configuration: [evidence span withheld — anchor status: external_doc]
