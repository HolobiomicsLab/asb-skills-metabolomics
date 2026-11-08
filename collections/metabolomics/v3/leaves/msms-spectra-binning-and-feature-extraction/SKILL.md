---
name: msms-spectra-binning-and-feature-extraction
description: Use when raw MS/MS spectra (MoNA or NIST 17, positive mode, HCD/QqQ/Q-TOF
  or IT) must be preprocessed into binned, denoised feature vectors to train a multi-class
  multi-label ANN like MetFID.
when_to_use_negative:
- Spectra were already converted to a fixed-length fingerprint or feature table by
  another pipeline — re-binning would destroy bin metadata.
- Only negative-mode or in-source-fragmentation spectra are available, since the method
  filters to positive-mode collision-cell or ion-trap data.
- Candidate ranking via spectral library matching rather than ANN fingerprint prediction
  is the goal.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetFID
  role: Downstream CNN/ANN that consumes the binned, denoised feature vectors to predict
    528-bit compound fingerprints
  repo: https://github.com/aohongyu/MetFID
- name: MoNA
  role: Source of the MS/MS training spectra that are binned and feature-extracted
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Additional MS/MS library source used alongside MoNA for training-set binning
- name: PyFingerprint
  role: Used downstream to generate fingerprints (RDKit/CDK/OpenBabel) once binned
    spectra have been transformed by the ANN
  repo: https://github.com/hcji/PyFingerprint
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:89d2e61abef6ea3d7957c6672b45fa3dc397cf0fe5a6bea3392f677664b2623c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MoNA
  - NIST 17
  - PyFingerprint
  - RDKit
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# MS/MS spectra binning and feature extraction

## Summary

Converts raw MS/MS spectra into fixed-length feature vectors for ANN fingerprint prediction by binning the m/z axis, computing loss features from precursor-ion m/z differences, and scaling/filtering peak intensities.

## When to use

Use when raw MS/MS spectra (MoNA or NIST 17, positive mode, HCD/QqQ/Q-TOF or IT) must be preprocessed into binned, denoised feature vectors to train a multi-class multi-label ANN like MetFID.

## When NOT to use

- Spectra were already converted to a fixed-length fingerprint or feature table by another pipeline — re-binning would destroy bin metadata.
- Only negative-mode or in-source-fragmentation spectra are available, since the method filters to positive-mode collision-cell or ion-trap data.
- Candidate ranking via spectral library matching rather than ANN fingerprint prediction is the goal.

## Inputs

- MoNA MS/MS spectra (Vaniya/Fiehn Natural Products Library, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC)
- NIST 17 MS/MS library spectra
- Peak lists of precursor mass, ionization mode, and m/z–intensity pairs (testing_compound.txt format)
- InChIKey compound identifiers for spectra merging

## Outputs

- Binned peak-intensity feature vector per spectrum (m/z bins with all-zero bins removed)
- Loss-feature vector (precursor m/z minus each peak m/z)
- Filtered, denoised, scaled spectra ready for ANN training/prediction

## How to apply

After selecting spectra acquired in positive mode via collision-cell (HCD, QqQ, Q-TOF) or ion-trap instruments, bin each spectrum's m/z range into prespecified bins and remove bins that are all zeros across the training set. Scale peak intensities to 0–100, then denoise by removing peaks with intensity lower than 10 after discarding spectra with fewer than five peaks above 2% relative intensity. Compute loss features as the m/z difference between the precursor ion and each peak, and concatenate these with the binned intensities to form the ANN input vector. Optionally merge spectra of the same compound (same InChIKey) across collision energies by merging peaks whose m/z difference is ≤ 0.1, though MetFID later favors keeping collision energies separate (30 eV cutoff).

## Related tools

- **MetFID** (Downstream CNN/ANN that consumes the binned, denoised feature vectors to predict 528-bit compound fingerprints) — https://github.com/aohongyu/MetFID
- **MoNA** (Source of the MS/MS training spectra that are binned and feature-extracted) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Additional MS/MS library source used alongside MoNA for training-set binning)
- **PyFingerprint** (Used downstream to generate fingerprints (RDKit/CDK/OpenBabel) once binned spectra have been transformed by the ANN) — https://github.com/hcji/PyFingerprint

## Examples

```
python3 main.py  # after placing testing_compound.txt in _files; MetFID bins m/z values and computes loss features per spectrum
```

## Evaluation signals

- After binning, verify that no bin is entirely zero across all training-set spectra (all-zero bins are removed).
- Confirm that every retained spectrum has at least five peaks with relative intensity above 2%.
- Check that all peak intensities fall within 0–100 after scaling, and no peak below intensity 10 survives denoising.
- Merged spectra must share the same first InChIKey block and contain only peaks with m/z differences ≤ 0.1 from an existing peak.
- Total feature-vector length equals the number of non-empty m/z bins plus the number of loss features.

## Limitations

- The 30 eV collision-energy cutoff is an empirical choice; combining low- and high-energy spectra into one ANN slightly reduces accuracy and F1 compared to separate models.
- Spectra with fewer than five peaks above 2% relative intensity are discarded, potentially dropping sparse but informative spectra.
- Merging spectra across collision energies (peaks with m/z difference ≤ 0.1) discards energy-specific fragmentation patterns, which the paper showed to be highly variable.
- In source-collision (IT) versus collision-cell (HCD, QqQ, Q-TOF) instruments, ion-trap spectra clustered together in analyses despite different fragmentation mechanisms, so cross-instrument generalization is imperfect.

## Evidence

- [methods] Binning removes empty bins: [evidence span withheld — anchor status: not_found]
- [methods] Loss features definition: [evidence span withheld — anchor status: not_found]
