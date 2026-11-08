---
name: msms-spectral-preprocessing
description: Use when raw MS/MS spectra from spectral libraries (e.g., MoNA or NIST
  17) need to be cleaned, merged per compound, and converted into integer m/z bin
  feature vectors before training or applying a machine-learning metabolite annotation
  model.
when_to_use_negative:
- When spectra are already peak-binned or preprocessed feature vectors, applying this
  skill again would re-denose and re-bin them incorrectly
- When spectra are from negative ion mode or use adducts other than H+ or NH4+, since
  the paper's homogeneity selection criteria exclude them
- When the goal is de novo spectral library matching (not ANN fingerprint prediction),
  since these steps are tailored to the MetFID feature-extraction pipeline
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetFID
  role: The tool that applies this preprocessing pipeline to convert raw spectra into
    merged, binned feature vectors for CNN fingerprint prediction
  repo: https://github.com/aohongyu/MetFID
- name: PyFingerprint
  role: Dependency that provides compound fingerprint generation (RDKit, CDK, OpenBabel)
    used downstream in the MetFID training pipeline
  repo: https://github.com/hcji/PyFingerprint
- name: Open Babel
  role: Used in the MetFID training pipeline to determine MACCS, FP3, and FP4 molecular
    fingerprints of training compounds
  repo: https://openbabel.org/wiki/Python
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:f22c6d5a3bc7085258dee34de8688ed79f098314f7f568d46af9ff5514fbf036
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MoNA
  - NIST 17
  - Open Babel
  - PyFingerprint
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# msms-spectral-preprocessing

## Summary

A set of steps described in the MetFID paper for converting raw MS/MS spectra (from MoNA and NIST 17 libraries) into merged, denoised, and binned feature vectors suitable for ANN-based fingerprint prediction.

## When to use

Use when raw MS/MS spectra from spectral libraries (e.g., MoNA or NIST 17) need to be cleaned, merged per compound, and converted into integer m/z bin feature vectors before training or applying a machine-learning metabolite annotation model.

## When NOT to use

- When spectra are already peak-binned or preprocessed feature vectors, applying this skill again would re-denose and re-bin them incorrectly
- When spectra are from negative ion mode or use adducts other than H+ or NH4+, since the paper's homogeneity selection criteria exclude them
- When the goal is de novo spectral library matching (not ANN fingerprint prediction), since these steps are tailored to the MetFID feature-extraction pipeline

## Inputs

- Raw MS/MS spectra files from MoNA repositories (Vaniya/Fiehn Natural Products Library, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC)
- MS/MS spectra from the NIST 17 library
- Precursor ion m/z and collision-energy metadata per spectrum
- InChIKey identifiers linking spectra to compounds

## Outputs

- Merged, denoised spectra per compound (InChIKey) ready for fingerprint prediction
- Per-peak loss features (precursor m/z minus peak m/z, intensity as proxy)
- Binned feature table of continuous integer m/z bins with accumulated intensity values, bins that are all zeros removed

## How to apply

Scale peak intensities to 0–100 and drop spectra with fewer than five peaks above 2% relative intensity. Denoise by locating the highest-intensity peak with mass greater than the precursor mass, removing all peaks less intense than it, then discarding remaining peaks with intensity below 10. Select only spectra meeting homogeneity criteria: positive mode, collision-cell (HCD, QqQ, Q-TOF) or ion-trap (IT) fragmentation, 100–1010 Da mass range, and H+ or NH4+ adduct only. Merge spectra of the same compound (matched via InChIKey) by combining peaks whose m/z difference is 0.1 or less and assigning the merged peak the mean intensity; compute loss features for each peak as the difference between precursor ion m/z and the peak m/z. Finally bin each spectrum's m/z range into continuous integer m/z bins with accumulated intensities as feature values, and drop bins that are all zeros across the training set.

## Related tools

- **MetFID** (The tool that applies this preprocessing pipeline to convert raw spectra into merged, binned feature vectors for CNN fingerprint prediction) — https://github.com/aohongyu/MetFID
- **PyFingerprint** (Dependency that provides compound fingerprint generation (RDKit, CDK, OpenBabel) used downstream in the MetFID training pipeline) — https://github.com/hcji/PyFingerprint
- **Open Babel** (Used in the MetFID training pipeline to determine MACCS, FP3, and FP4 molecular fingerprints of training compounds) — https://openbabel.org/wiki/Python

## Examples

```
python3 main.py  # from the MetFID repo root, after placing MetFID_CNN_40088_5618.h5 and _files/testing_compound.txt
```

## Evaluation signals

- Verify merged spectra counts against paper-reported pre-merge counts: 11,784 MoNA spectra / 5,667 compounds and 122,481 NIST 17 spectra / 10,730 compounds
- Check the denoising cutoff: no peaks remain with intensity below 10 after the denoising step
- Confirm that all retained spectra use positive ion mode and H+ or NH4+ adduct, and that m/z falls within the 100–1010 Da range
- Verify no binned feature column is all zeros across the training set (all-zero bins removed)
- Loss features must be present: for each peak, a precursor-minus-peak m/z difference should exist and serve as the loss-feature proxy with peak intensity

## Limitations

- Spectra with fewer than five peaks of relative intensity above 2% are removed, so extremely sparse spectra cannot be preprocessed by this skill
- The m/z difference of 0.1 used for peak merging is a fixed threshold; it may over- or under-merge peaks for compounds with very similar fragment m/z values
- Negative ion mode and non-H+/NH4+ adducts are excluded, limiting applicability to the paper's homogeneous data subset
- Future work notes the model may benefit from considering variations of ANNs to enhance fingerprint prediction performance, implying the preprocessing may not be optimal for all compounds

## Evidence
