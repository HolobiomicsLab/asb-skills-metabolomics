---
name: noisy-spectra-filtering
description: Use when building a curated MS2 training set from reference-standard
  or raw lipid spectra and noisy profiles must be excluded before model training or
  spectral storage.
when_to_use_negative:
- Input spectra are already centroided and base peak normalized with a known-good
  quality flag — re-filtering would discard valid low-intensity spectra.
- The task concerns model training or prediction itself (transformer configuration,
  mzML prediction) rather than training-data curation.
- The noise characterization of the target data differs from lipid standard measurements,
  where the 0.01 threshold was calibrated; the threshold should not be applied blindly
  to other data types.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pyOpenMS
  role: Smoothing, baseline correction, and peak picking of raw profile MS2 spectra
    prior to the noise-score computation
- name: h5py
  role: Writing the filtered spectra to an HDF5 file for downstream model training
- name: LipiDetective
  role: Pipeline context in which the noise filtering produces the cleaned training
    spectra
  repo: https://github.com/LipiTUM/lipidetective
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:e8958543e1abdbc1ff7aba291b6391d7081dc664e138605145042d32bf2beaa2
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# noisy-spectra-filtering

## Summary

Filters particularly noisy MS2 spectra by computing a base-peak normalized median intensity noise score and excluding spectra above a 0.01 threshold, as used to curate LipiDetective's lipid standard training data.

## When to use

Use when building a curated MS2 training set from reference-standard or raw lipid spectra and noisy profiles must be excluded before model training or spectral storage.

## When NOT to use

- Input spectra are already centroided and base peak normalized with a known-good quality flag — re-filtering would discard valid low-intensity spectra.
- The task concerns model training or prediction itself (transformer configuration, mzML prediction) rather than training-data curation.
- The noise characterization of the target data differs from lipid standard measurements, where the 0.01 threshold was calibrated; the threshold should not be applied blindly to other data types.

## Inputs

- Centroided MS2 spectra from lipid reference standard measurements (post smoothing, baseline correction, and peak picking)
- Per-spectrum peak intensity arrays (profile spectra processed via pyOpenMS before filtering)

## Outputs

- Filtered set of high-quality MS2 spectra with base-peak normalized median intensity <= 0.01
- Noise score per spectrum (median intensity / base peak intensity)
- Cleaned spectra ready for HDF5 storage (HDF5 written via h5py)

## How to apply

First process raw profile MS2 spectra with pyOpenMS, applying smoothing, baseline correction, and peak picking to obtain centroided peaks. Then base peak normalize each spectrum by dividing all peak intensities by the spectrum's highest-intensity peak so intensities lie between zero and one. Compute the noise score as the median intensity divided by the base peak intensity, and exclude any spectrum whose score exceeds the chosen threshold of 0.01. This step was applied specifically to the phospholipid standard measurements to remove particularly noisy spectra before the cleaned spectra were saved to an HDF5 file for downstream training. Retained spectra should also be trimmed to the 50-1600 m/z range used across the preprocessing workflow.

## Related tools

- **pyOpenMS** (Smoothing, baseline correction, and peak picking of raw profile MS2 spectra prior to the noise-score computation)
- **h5py** (Writing the filtered spectra to an HDF5 file for downstream model training)
- **LipiDetective** (Pipeline context in which the noise filtering produces the cleaned training spectra) — https://github.com/LipiTUM/lipidetective

## Examples

```
from pyopenms import *; # after MSSpectrum peak picking: noise_score = sorted(intensities)[len(intensities)//2] / max(intensities); keep = noise_score <= 0.01
```

## Evaluation signals

- All retained spectra have a base-peak normalized median intensity at or below 0.01.
- After normalization, all peak intensities in each spectrum lie between 0 and 1.
- Spectra surviving the filter still cover the 50-1600 m/z range after trimming.
- The filtered spectra serialize cleanly into the HDF5 training file (h5py) without malformed or empty spectra entries.
- Filtering is applied only to the phospholipid standard measurements, matching the paper's scope for this step.

## Limitations

- The 0.01 threshold was chosen for the phospholipid standard measurements; spectra from other sample types (e.g. the HCE, IOBA-NHC, or SRM 1950 datasets) may require recalibration of the cutoff.
- Aggressive filtering can remove genuine low-intensity fragment spectra, reducing training-set diversity for rare lipid species or collision energies.
- Filtering operates on already peak-picked spectra; errors introduced earlier by smoothing, baseline correction, or peak picking propagate into the noise score.

## Evidence

- [methods] 0.01 threshold choice and exclusion criterion: "a threshold of 0.01 base peak normalized median intensity was chosen, and any spectra with a higher score were excluded"
- [methods] profile spectra preprocessing prerequisite: "All profile spectra underwent smoothing, baseline correction, and peak picking"
- [methods] scope of the noisy filtering step: "An additional filtering step was performed for the phospholipid standard measurements to exclude particularly noisy spectra"
- [methods] base peak normalization applied to all spectra: "base peak normalization was performed for all spectra"
- [methods] downstream HDF5 storage of cleaned spectra: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
- [readme] README confirming HDF5-based training input format: [evidence span withheld — anchor status: external_doc]
