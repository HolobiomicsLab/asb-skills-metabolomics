---
name: base-peak-normalization
description: Use when raw MS2 lipid spectra must be put on a common intensity scale
  before model training or filtering — e.g. to compute a base-peak-normalized median
  intensity for a noise-score cutoff.
when_to_use_negative:
- When the goal is absolute quantification across spectra — base-peak normalization
  destroys absolute intensity information and should only be applied for relative,
  pattern-based tasks like lipid identification.
- When spectra are still in profile mode — smoothing, baseline correction, and peak
  picking must precede normalization, as the article applies base peak normalization
  after peak picking.
- When a noise or quality filter is needed but the noise definition differs (e.g.
  TIC-based); this skill's score is specifically base-peak-normalized median intensity
  with a 0.01 threshold.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pyOpenMS
  role: Used upstream of normalization to convert profile spectra to peak-picked spectra
    (smoothing, baseline correction, peak picking) so intensities are suitable for
    base-peak normalization.
- name: LipiDetective
  role: The pipeline within which base-peak normalization is applied to produce the
    normalized MS2 spectra used for transformer model training.
  repo: https://github.com/LipiTUM/lipidetective
- name: h5py
  role: Stores the normalized spectra in an HDF5 file for downstream training and
    prediction.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:80f2b60992b6a30eb14c5fcdd75db27579d079cd49c2f89265cd7deda3c39fb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - h5py
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# base-peak-normalization

## Summary

Intensity normalization of tandem mass spectra by dividing every peak intensity by the spectrum's highest-intensity (base) peak so all intensities lie between 0 and 1, a prerequisite preprocessing step in LipiDetective's training-data pipeline.

## When to use

Use when raw MS2 lipid spectra must be put on a common intensity scale before model training or filtering — e.g. to compute a base-peak-normalized median intensity for a noise-score cutoff.

## When NOT to use

- When the goal is absolute quantification across spectra — base-peak normalization destroys absolute intensity information and should only be applied for relative, pattern-based tasks like lipid identification.
- When spectra are still in profile mode — smoothing, baseline correction, and peak picking must precede normalization, as the article applies base peak normalization after peak picking.
- When a noise or quality filter is needed but the noise definition differs (e.g. TIC-based); this skill's score is specifically base-peak-normalized median intensity with a 0.01 threshold.

## Inputs

- Raw MS2 spectra (already peak-picked, from phospholipid standards and reference datasets such as PNNL, Thermo, HCE, IOBA-NHC, MITOMICS)
- Profile MS2 spectra pre-processed via pyOpenMS (smoothing, baseline correction, peak picking)

## Outputs

- Base-peak-normalized MS2 spectra with intensities in the range 0–1
- Normalized spectra ready for m/z trimming (50–1600) and noise-score filtering, ultimately saved in HDF5 format

## How to apply

For each MS2 spectrum in the training dataset (after smoothing, baseline correction, and peak picking of profile spectra with pyOpenMS), identify the highest-intensity peak (the base peak) and divide all peak intensities in that spectrum by the base-peak intensity so normalized intensities fall in [0, 1]. The normalized spectrum then feeds downstream steps: LipiDetective trims spectra to an m/z range of 50–1600 and computes a noise score as the median intensity divided by the base-peak intensity, excluding spectra whose base-peak normalized median intensity exceeds 0.01. Normalization is applied to all spectra — including phospholipid standard measurements, the PNNL, Thermo, and corneal/conjunctival cell datasets — before storage in an HDF5 file via h5py.

## Related tools

- **pyOpenMS** (Used upstream of normalization to convert profile spectra to peak-picked spectra (smoothing, baseline correction, peak picking) so intensities are suitable for base-peak normalization.)
- **LipiDetective** (The pipeline within which base-peak normalization is applied to produce the normalized MS2 spectra used for transformer model training.) — https://github.com/LipiTUM/lipidetective
- **h5py** (Stores the normalized spectra in an HDF5 file for downstream training and prediction.)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- All peak intensities in each output spectrum lie within [0, 1] after normalization.
- Each spectrum retains exactly one intensity value equal to 1.0 (the base peak).
- Spectra excluded by the downstream noise filter have a base-peak normalized median intensity strictly greater than 0.01; retained spectra are ≤ 0.01.
- Normalized spectra are stored in an HDF5 file and match the 50–1600 m/z trimming window applied afterwards.
- Peak-picking output (not raw profile data) is what gets normalized — spot-check that spectra were smoothed, baseline-corrected, and peak-picked first.

## Limitations

- The article does not report the underlying noise-score distribution or the effect size of the 0.01 threshold on final dataset size.
- The manual adjustment of varying lipid nomenclatures across data sources was performed by hand, so normalized spectra fed into training depend on this non-automated curation step.
- Blank-measurement spectra (2002 added) must also be normalized consistently so the model learns to predict the end-of-sequence token when no lipid species is present.

## Evidence

- [methods] base peak normalization was performed for all spectra: "base peak normalization was performed for all spectra"
- [methods] All profile spectra underwent smoothing, baseline correction, and peak picking: "All profile spectra underwent smoothing, baseline correction, and peak picking"
- [methods] a threshold of 0.01 base peak normalized median intensity was chosen, and any spectra with a higher score were excluded: "a threshold of 0.01 base peak normalized median intensity was chosen, and any spectra with a higher score were excluded"
- [methods] all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] all spectra were saved in a Hierarchical Data Format 5 (HDF5) file: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
