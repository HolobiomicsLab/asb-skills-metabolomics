---
name: spectral-noise-filtering
description: Use when preparing raw MS/MS peak lists (e.g. MoNA, NIST 17 spectra with
  precursor mass and ionization mode) for fingerprint-prediction training, and low-intensity
  peaks and precursor-over-mass peaks must be removed.
when_to_use_negative:
- Input is already a merged, binned spectral feature table (spectral_feature_table.csv)
  — filtering has already been applied upstream
- Spectra are negative-mode or generated with adducts other than H+ or NH4+, which
  the MetFID training set excludes
- MS/MS were acquired outside the 100–1010 Da mass range or with instrument types
  not using collision cell or ion trap fragmentation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetFID
  role: Implements the scaling, denoising, and spectrum-selection steps in its preprocessing
    pipeline before merging and binning
  repo: https://github.com/aohongyu/MetFID
- name: MoNA
  role: Source of raw MS/MS spectra that are scaled, filtered, and denoised
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Secondary spectral library whose spectra undergo the same scaling and denoising
    workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:eca7c2f70867f041843c2d52efd0b74d8325af74b28feb6cb64d71bd5f40948d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MoNA
  - NIST 17
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spectral-noise-filtering

## Summary

MetFID's preprocessing routine that scales peak intensities to 0–100, removes low-information spectra, denoises peaks, and selects collision-cell/ion-trap spectra before merging MS/MS data for ANN fingerprint prediction.

## When to use

Use when preparing raw MS/MS peak lists (e.g. MoNA, NIST 17 spectra with precursor mass and ionization mode) for fingerprint-prediction training, and low-intensity peaks and precursor-over-mass peaks must be removed.

## When NOT to use

- Input is already a merged, binned spectral feature table (spectral_feature_table.csv) — filtering has already been applied upstream
- Spectra are negative-mode or generated with adducts other than H+ or NH4+, which the MetFID training set excludes
- MS/MS were acquired outside the 100–1010 Da mass range or with instrument types not using collision cell or ion trap fragmentation

## Inputs

- Raw MS/MS spectra from MoNA libraries (Vaniya/Fiehn Natural Products Library, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC) and NIST 17
- Peak lists with precursor mass, ionization mode, and intensity pairs (e.g. testing_compound.txt format: 'precursor mass mode' followed by 'm/z intensity' rows)

## Outputs

- Cleaned peak lists scaled to 0–100 intensity range with peaks below intensity 10 removed
- Filtered spectra that meet homogeneity criteria (positive mode, HCD/QqQ/Q-TOF or IT, 100–1010 Da, H+ or NH4+ adduct)
- Spectra suitable for merging into compound-level profiles keyed by InChIKey

## How to apply

First scale peak intensities so all values fall between 0 and 100, then discard spectra with fewer than five peaks above 2% relative intensity. For denoising, identify peaks with the highest intensities that also have mass greater than the precursor mass, remove peaks whose intensity is smaller than that highest-intensity peak, and then drop all remaining peaks with intensity below 10. Finally, select spectra that are positive-mode, acquired with collision-cell instruments (HCD, QqQ, Q-TOF) or ion trap (IT), spanning 100–1010 Da, and bearing only H+ or NH4+ adducts. These steps precede merging spectra of the same InChIKey across collision energies, so noise and non-fragmentation peaks do not corrupt the merged profile used for loss-feature computation and m/z binning.

## Related tools

- **MetFID** (Implements the scaling, denoising, and spectrum-selection steps in its preprocessing pipeline before merging and binning) — https://github.com/aohongyu/MetFID
- **MoNA** (Source of raw MS/MS spectra that are scaled, filtered, and denoised) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Secondary spectral library whose spectra undergo the same scaling and denoising workflow)

## Examples

```
cd MetFID && python3 main.py
```

## Evaluation signals

- Every spectrum in the cleaned set should have peak intensities within 0–100 after scaling
- No remaining spectrum has fewer than five peaks with relative intensity above 2%
- No peak with mass greater than the precursor mass remains (these are removed during denoising)
- No peak with intensity lower than 10 survives in the denoised spectra
- All retained spectra are positive-mode, use HCD/QqQ/Q-TOF or IT fragmentation, span 100–1010 Da, and carry only H+ or NH4+ adducts

## Limitations

- Spectra with fewer than five peaks above 2% relative intensity are discarded entirely, which may remove genuinely informative but sparse spectra
- Denoising removes all peaks whose mass exceeds the precursor mass, assuming such peaks are noise; this assumption could mis-handle adducts or calibration artifacts
- The 100–1010 Da mass range and H+/NH4+ adduct restriction exclude many real-world analyte spectra (e.g. negative-mode or Na+ adducts)
- Filtering is tuned to the collision-cell vs. ion-trap instruments the authors validated; other instrument types are not addressed

## Evidence
