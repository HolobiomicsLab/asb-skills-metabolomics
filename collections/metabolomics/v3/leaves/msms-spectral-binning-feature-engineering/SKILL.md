---
name: msms-spectral-binning-feature-engineering
description: Use when preparing MS/MS spectra (MoNA, NIST 17, HCD/QqQ/Q-TOF or IT,
  positive mode) as neural-network input vectors for fingerprint prediction, and a
  uniform binning/scaling representation is needed.
when_to_use_negative:
- Input is already a binned, normalized feature matrix (e.g. a saved .h5 model input)
  — re-binning would destroy the trained model's bin alignment
- Spectra are in negative ionization mode or acquired on non-collision-cell, non-IT
  instruments, which fall outside MetFID's training scope
- Spectra contain fewer than five peaks above 2% relative intensity — they should
  be removed rather than binned
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetFID
  role: Downstream ANN (Keras/TensorFlow) that consumes the binned spectrum vectors
    as input to predict 528-bit fingerprints
  repo: https://github.com/aohongyu/MetFID
- name: MoNA
  role: Source repository of MS/MS spectra that are downloaded, filtered, and binned
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Second source library of MS/MS spectra for training-set binning and preprocessing
- name: OpenBabel
  role: Used to determine MACCS/FP3/FP4 fingerprints that form the binned model's
    528-bit training targets
- name: PyFingerprint
  role: Python package aggregating fingerprint implementations (including OpenBabel
    fp3/fp4) usable alongside the binned-spectrum pipeline
  repo: https://github.com/hcji/PyFingerprint
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:8ba1d1a0fb16d03f20763b48053454dd5702749359c899b20c367ef2029247aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - Keras
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  - PyFingerprint
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# msms-spectral-binning-feature-engineering

## Summary

Convert raw MS/MS spectra into fixed-length binned feature vectors (m/z integer bins, intensities scaled 0–100) plus loss features, so they can serve as ANN inputs for compound fingerprint prediction in MetFID.

## When to use

Use when preparing MS/MS spectra (MoNA, NIST 17, HCD/QqQ/Q-TOF or IT, positive mode) as neural-network input vectors for fingerprint prediction, and a uniform binning/scaling representation is needed.

## When NOT to use

- Input is already a binned, normalized feature matrix (e.g. a saved .h5 model input) — re-binning would destroy the trained model's bin alignment
- Spectra are in negative ionization mode or acquired on non-collision-cell, non-IT instruments, which fall outside MetFID's training scope
- Spectra contain fewer than five peaks above 2% relative intensity — they should be removed rather than binned

## Inputs

- Raw MS/MS peak lists (precursor m/z, ionization mode, m/z-intensity pairs) from MoNA or NIST 17 libraries
- Compound identifiers (InChIKey) for merging spectra of the same compound across collision energies

## Outputs

- Fixed-length binned spectrum feature vectors with all-zero bins removed
- Loss-feature vectors (precursor m/z minus each peak m/z)
- Filtered, scaled (0–100 intensity), denoised spectrum set

## How to apply

Start with spectra acquired in positive mode via collision-cell instruments (HCD, QqQ, Q-TOF) or ion trap (IT), with precursor masses 100–1010 Da and H+ or NH4+ adducts. Scale peak intensities so all values fall between 0 and 100, and remove spectra consisting of fewer than five peaks with relative intensity above 2%. Denoise by removing peaks whose intensity is smaller than the highest-intensity peak's value and all peaks with intensity lower than 10; optionally merge spectra of the same compound (matched by InChIKey) across collision energies, merging peaks whose m/z difference is 0.1 or less. Bin the m/z range of each spectrum into prespecified integer bins and drop bins consisting of all 0's across the entire training set; additionally compute loss features as the difference between the precursor ion's m/z and each peak's m/z. The resulting binned vectors (plus loss features) form the ANN input layer feeding MetFID's two-hidden-layer ReLU network with a 528-bit sigmoid fingerprint output.

## Related tools

- **MetFID** (Downstream ANN (Keras/TensorFlow) that consumes the binned spectrum vectors as input to predict 528-bit fingerprints) — https://github.com/aohongyu/MetFID
- **MoNA** (Source repository of MS/MS spectra that are downloaded, filtered, and binned) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Second source library of MS/MS spectra for training-set binning and preprocessing)
- **OpenBabel** (Used to determine MACCS/FP3/FP4 fingerprints that form the binned model's 528-bit training targets)
- **PyFingerprint** (Python package aggregating fingerprint implementations (including OpenBabel fp3/fp4) usable alongside the binned-spectrum pipeline) — https://github.com/hcji/PyFingerprint

## Examples

```
python3 main.py  # run inside the MetFID folder after placing testing_compound.txt and inchikey_list.txt in _files
```

## Evaluation signals

- All intensity values in scaled spectra lie between 0 and 100
- Every retained spectrum has at least five peaks with relative intensity above 2%
- No bin in the final feature matrix consists of all 0's across all training spectra
- Loss features equal precursor m/z minus each peak's m/z for every retained peak
- Binned vector dimensionality matches the MetFID ANN input layer when fed into the Keras/TensorFlow model

## Limitations

- Spectra with fewer than five peaks above 2% relative intensity are discarded, shrinking training coverage
- Merging spectra across collision energies by InChIKey can degrade prediction: the combined collision-energy model showed slightly lower accuracy and lower F1 scores than energy-specific models trained with a 30 eV cutoff
- Positive-mode, 100–1010 Da, H+ or NH4+ restriction limits transferability to other ionization regimes
- No released pre-trained model weights accompany the discussion of future ANN variations, so binned inputs must match the exact binning scheme

## Evidence
