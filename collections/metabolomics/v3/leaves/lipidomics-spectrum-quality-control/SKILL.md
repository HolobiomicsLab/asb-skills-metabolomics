---
name: lipidomics-spectrum-quality-control
description: Use when raw lipid MS2 spectra (e.g. from phospholipid standard measurements
  or profile-mode mzML) must be cleaned, normalized, and filtered before deep-learning
  training or downstream identification in tools like LipiDetective.
when_to_use_negative:
- Input spectra have already been smoothed, peak-picked, normalized, and stored in
  HDF5 — re-running this skill would discard spectra a second time or distort intensities.
- The analysis target is MS1 precursor survey data or raw chromatographic peak detection
  rather than MS2 fragment spectra for lipid species identification.
- Spectra were acquired on instruments/modes outside the scope of lipid tandem-MS
  (e.g. small-molecule metabolomics spectra not using shorthand lipid nomenclature),
  where the 50–1600 m/z window and lipid-fragment assumptions do not apply.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: pyOpenMS
  role: Smoothing, baseline correction, and peak picking of profile MS2 spectra before
    normalization
- name: h5py
  role: Writing the cleaned spectra into an HDF5 training file
- name: LipiDetective
  role: Downstream consumer of the filtered/normalized spectra for transformer-based
    lipid identification
  repo: https://github.com/LipiTUM/lipidetective
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:5d829120d3b5b7e8fe7f6c0dfe46b2515ee5d3fd4363754eb3441592679e7d7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - h5py
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipidomics-spectrum-quality-control

## Summary

A preprocessing-and-filtering skill that converts raw (including profile-mode) lipid MS2 spectra into a high-quality, normalized peak list — via pyOpenMS smoothing/baseline-correction/peak-picking, base peak normalization, m/z trimming to 50–1600, and exclusion of noisy spectra above a 0.01 median-intensity threshold.

## When to use

Use when raw lipid MS2 spectra (e.g. from phospholipid standard measurements or profile-mode mzML) must be cleaned, normalized, and filtered before deep-learning training or downstream identification in tools like LipiDetective.

## When NOT to use

- Input spectra have already been smoothed, peak-picked, normalized, and stored in HDF5 — re-running this skill would discard spectra a second time or distort intensities.
- The analysis target is MS1 precursor survey data or raw chromatographic peak detection rather than MS2 fragment spectra for lipid species identification.
- Spectra were acquired on instruments/modes outside the scope of lipid tandem-MS (e.g. small-molecule metabolomics spectra not using shorthand lipid nomenclature), where the 50–1600 m/z window and lipid-fragment assumptions do not apply.

## Inputs

- Raw MS2 spectra (profile or centroided) from lipid standard measurements or published datasets, e.g. mzML files
- LipiDetective-compatible spectral input (source datasets such as the PNNL dataset, 46,724 MS2 spectra from 1790 lipids)

## Outputs

- Cleaned, base-peak-normalized, 50–1600 m/z-trimmed MS2 spectra retained by the 0.01 noise-score filter
- HDF5 (.h5) training file consumable by LipiDetective via h5py

## How to apply

Load raw MS2 spectra (profile-mode spectra from mzML or vendor files); for profile spectra, process each with pyOpenMS applying smoothing, baseline correction, and peak picking to yield centroided peaks. Base-peak normalize every spectrum by dividing all peak intensities by the highest-intensity peak so intensities lie in 0–1. Trim each MS2 spectrum to an m/z range of 50–1600 m/z. Compute a noise score per spectrum as the median intensity divided by the base-peak intensity; exclude any spectrum whose score exceeds 0.01 (the threshold chosen for the phospholipid standard measurements to discard particularly noisy spectra). Finally, persist the cleaned spectra in an HDF5 (Hierarchical Data Format 5) file, written via h5py, ready for training LipiDetective's transformer on the resulting 268,720-spectrum dataset.

## Related tools

- **pyOpenMS** (Smoothing, baseline correction, and peak picking of profile MS2 spectra before normalization)
- **h5py** (Writing the cleaned spectra into an HDF5 training file)
- **LipiDetective** (Downstream consumer of the filtered/normalized spectra for transformer-based lipid identification) — https://github.com/LipiTUM/lipidetective

## Examples

```
uv run lipidetective --config config.yaml  # config.yaml points to the HDF5 files produced by this filtering step
```

## Evaluation signals

- Retained spectra have a base-peak normalized median intensity ≤ 0.01.
- All peak intensities fall within [0, 1] after base peak normalization.
- Every retained MS2 spectrum lies within the 50–1600 m/z window.
- Output is a valid HDF5 file readable with h5py and loadable by LipiDetective's training routine.
- Profile-mode inputs show evidence of smoothing, baseline correction, and peak picking (centroided peaks present, no baseline offset).

## Limitations

- The 0.01 base-peak normalized median-intensity cutoff is specific to the phospholipid standard measurements; spectra from other contexts may require a different threshold.
- Manual adjustment of nomenclature to the updated lipid shorthand notation was still required across source datasets — this skill does not standardize lipid identifiers.
- Only MS2 spectra are targeted; blank-measurement spectra (2002 added for training) and other non-lipid spectra require separate handling.
- Even with the filter, downstream identifications by any single software can still carry a high rate of false positives, so independent validation remains necessary.

## Evidence

- [methods] All profile spectra underwent smoothing, baseline correction, and peak picking: "All profile spectra underwent smoothing, baseline correction, and peak picking"
- [methods] base peak normalization was performed for all spectra: "base peak normalization was performed for all spectra"
- [methods] all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] a threshold of 0.01 base peak normalized median intensity was chosen, and any spectra with a higher score were excluded: "a threshold of 0.01 base peak normalized median intensity was chosen, and any spectra with a higher score were excluded"
- [readme] Data Handling: Supports training with HDF5 files and prediction using mzML files.: [evidence span withheld — anchor status: not_found]
