---
name: msms-spectrum-preprocessing-binning
description: Use when raw MS/MS peak lists (precursor m/z, ionization mode, m/z-intensity
  pairs) must be converted into MetFID's fixed-length binned feature vectors before
  loading them into the fingerprint-prediction ANN.
when_to_use_negative:
- The input is already a binned, scaled feature matrix or the spectra are being submitted
  directly to the pre-trained .h5 model after prior preprocessing
- The task requires merging across collision energies for testing spectra — MetFID's
  testing workflow applies preprocessing but without merging across collision energies
- The data are chromatographic peak tables or other non-MS/MS feature lists rather
  than per-analyte peak lists with precursor mass and ionization mode
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3332
tools:
- name: MetFID
  role: Implements the scaling, denoising, loss-feature, and binning preprocessing
    inside its pipeline before spectrum-to-fingerprint prediction
  repo: https://github.com/aohongyu/MetFID
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:0035a97b8a7f44b3dc5da6ae33d41b08bf1f111ffbbfc5d8db8e0a68aa8dca32
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MetaboQuest
  - MoNA
  - PubChemPy
  - PyFingerprint
  - TensorFlow
  - tabulate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# msms-spectrum-preprocessing-binning

## Summary

Preprocessing workflow from MetFID that converts raw MS/MS peak lists into fixed-length numeric feature vectors via intensity scaling (0–100), denoising (remove peaks with intensity < 10), neutral-loss feature calculation, and integer-bin m/z binning so spectra can be fed to the fingerprint-predicting ANN.

## When to use

Use when raw MS/MS peak lists (precursor m/z, ionization mode, m/z-intensity pairs) must be converted into MetFID's fixed-length binned feature vectors before loading them into the fingerprint-prediction ANN.

## When NOT to use

- The input is already a binned, scaled feature matrix or the spectra are being submitted directly to the pre-trained .h5 model after prior preprocessing
- The task requires merging across collision energies for testing spectra — MetFID's testing workflow applies preprocessing but without merging across collision energies
- The data are chromatographic peak tables or other non-MS/MS feature lists rather than per-analyte peak lists with precursor mass and ionization mode

## Inputs

- MS/MS peak lists in testing_compound.txt format: blocks prefixed #digit, first row = precursor mass and ionization mode (e.g. '389.1626 positive'), followed by m/z intensity pairs
- Training-set spectra from MoNA libraries and NIST 17 (positive mode; HCD, QqQ, Q-TOF, or IT instruments)

## Outputs

- Fixed-length binned feature vectors (m/z bins plus loss features, scaled intensities 0–100) ready for ANN fingerprint prediction
- Merged per-compound spectra keyed by InChIKey (training only)

## How to apply

Scale peak intensities so all values fall between 0 and 100, and discard any spectrum with fewer than five peaks at relative intensity above 2%. Denoise by removing peaks whose intensity is smaller than the highest-intensity peak's value, then remove all peaks with intensity lower than 10. Compute neutral-loss features from the difference between the precursor ion's m/z and each fragment peak's m/z, and bin the m/z range of each spectrum into prespecified integer bins. When constructing the training set, merge multiple MS/MS spectra of the same compound (identified by InChIKey) across collision energies by merging peaks whose m/z difference is ≤ 0.1, and remove bins that are all zeros across all training spectra; for testing spectra (e.g. from testing_compound.txt), apply the same scaling/denoising/loss/binning steps but do NOT merge across collision energies. The resulting vectors are the direct input to the pre-trained MetFID_CNN_40088_5618.h5 model in TensorFlow/Keras.

## Related tools

- **MetFID** (Implements the scaling, denoising, loss-feature, and binning preprocessing inside its pipeline before spectrum-to-fingerprint prediction) — https://github.com/aohongyu/MetFID

## Examples

```
python3 main.py
```

## Evaluation signals

- All scaled intensity values in the output vectors lie within 0–100
- No fragment peaks with intensity below 10 remain after denoising
- Loss features equal precursor m/z minus each fragment peak m/z
- No bins that are all zeros across the training set appear in the feature vectors
- Block counter (#digit) in testing_compound.txt matches the counter in inchikey_list.txt so vectors align with candidate lists when fed to the model

## Limitations

- Spectra consisting of fewer than five peaks with relative intensity above 2% are removed entirely
- Merging spectra across collision energies is used in the training phase; testing spectra must not be merged across collision energies
- For the m/z = 758.5697 HCC analyte case, structurally similar candidates could not be distinguished — a limitation downstream that preprocessing cannot resolve

## Evidence
