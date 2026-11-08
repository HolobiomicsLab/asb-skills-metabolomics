---
name: fingerprint-prediction-metrics-evaluation
description: Use when comparing fingerprint-prediction model variants (e.g. low ≤30
  eV vs high ≥30 eV collision-energy models vs one combined model) on MS/MS spectra
  and need structure-disjoint CV with accuracy/F1 metrics.
when_to_use_negative:
- Comparing IT vs HCD instrument-type models — the paper found combining IT and HCD
  does not affect prediction performance, so splitting by instrument type is unnecessary
- Evaluating candidate-ranking / annotation performance against MetFrag, ChemDistiller,
  or CSI:FingerID — this skill only scores fingerprint prediction itself, not end-to-end
  metabolite annotation
- 'Random (non-disjoint) cross-validation is the wrong choice here: compounds with
  the same first InChIKey block must never span train and test folds'
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetFID
  role: ANN fingerprint-prediction framework whose low-/high-/combined collision-energy
    models are being trained and evaluated
  repo: https://github.com/aohongyu/MetFID
- name: OpenBabel
  role: Computes MACCS, FP3, and FP4 fingerprints assembled into the 528-bit binary
    target vector
- name: PyFingerprint
  role: Python wrapper used by MetFID to obtain RDKit/CDK/OpenBabel fingerprint types
    including fp3 and fp4
  repo: https://github.com/hcji/PyFingerprint
- name: TensorFlow/Keras
  role: Implements the ANNs (two ReLU hidden layers, sigmoid output, binary cross-entropy,
    Adam, 30 epochs, batch size 100)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:c65244b09b173e9b1f9ac44fc9eb0f5862edc8571b577713123379ed13bfb2e1
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
  - TensorFlow/Keras
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# fingerprint-prediction-metrics-evaluation

## Summary

Evaluation protocol for ANN compound-fingerprint prediction models (MetFID) using structure-disjoint five-fold cross-validation grouped by first InChIKey block, comparing collision-energy-specific vs combined models with macro-averaged accuracy and F1 on HCD MS/MS spectra.

## When to use

Use when comparing fingerprint-prediction model variants (e.g. low ≤30 eV vs high ≥30 eV collision-energy models vs one combined model) on MS/MS spectra and need structure-disjoint CV with accuracy/F1 metrics.

## When NOT to use

- Comparing IT vs HCD instrument-type models — the paper found combining IT and HCD does not affect prediction performance, so splitting by instrument type is unnecessary
- Evaluating candidate-ranking / annotation performance against MetFrag, ChemDistiller, or CSI:FingerID — this skill only scores fingerprint prediction itself, not end-to-end metabolite annotation
- Random (non-disjoint) cross-validation is the wrong choice here: compounds with the same first InChIKey block must never span train and test folds

## Inputs

- Positive-mode HCD/QqQ/Q-TOF or IT MS/MS spectra from MoNA and NIST 17 (m/z 100–1010 Da, H+ or NH4+ adducts)
- Collision energy annotations per spectrum (eV values for low/high partitioning)
- Compound structures identified by InChIKey (first block used for structure-disjoint fold assignment)
- 528-bit binary fingerprint vectors (MACCS + FP3 + FP4) computed with OpenBabel

## Outputs

- collision_energy_model_comparison.csv with macro-averaged accuracy (Eq. 2) and F1 (Eq. 3) for the low-energy, high-energy, and combined ANN models
- Per-fold train/test assignments with structure-disjoint (InChIKey block 1) groupings

## How to apply

Retrieve positive-mode MS/MS spectra (HCD/QqQ/Q-TOF or IT instruments, 100–1010 Da, H+ or NH4+ adducts), scale intensities to 0–100, remove spectra with fewer than five peaks above 2% relative intensity, and bin m/z values into integer bins dropping all-zero bins. Compute the 528-bit target fingerprint vector (MACCS + FP3 + FP4 via OpenBabel) for each compound. Split spectra by collision energy at the 30 eV cutoff identified by cluster analysis of NIST 17 spectra, then train three ANNs (low-energy, high-energy, combined) — each with two ReLU hidden layers and a sigmoid output with binary cross-entropy, Adam optimizer, 30 epochs, batch size 100 in TensorFlow/Keras. Evaluate with structure-disjoint five-fold cross-validation on the 6677 HCD compounds, ensuring no compound (first InChIKey block) appears in both training and testing folds. Compute macro-averaged accuracy (Eq. 2) and F1 (Eq. 3) per model and write results to collision_energy_model_comparison.csv.

## Related tools

- **MetFID** (ANN fingerprint-prediction framework whose low-/high-/combined collision-energy models are being trained and evaluated) — https://github.com/aohongyu/MetFID
- **OpenBabel** (Computes MACCS, FP3, and FP4 fingerprints assembled into the 528-bit binary target vector)
- **PyFingerprint** (Python wrapper used by MetFID to obtain RDKit/CDK/OpenBabel fingerprint types including fp3 and fp4) — https://github.com/hcji/PyFingerprint
- **TensorFlow/Keras** (Implements the ANNs (two ReLU hidden layers, sigmoid output, binary cross-entropy, Adam, 30 epochs, batch size 100))

## Examples

```
from PyFingerprint.fingerprint import get_fingerprint; fp = get_fingerprint('CCCCN', 'fp4')
```

## Evaluation signals

- The combined model's accuracy on low-energy testing spectra must be slightly lower than the low-energy model's accuracy
- The combined model's F1 scores must be lower than both the low-energy and high-energy separate models
- No compound (first InChIKey block) may appear in both training and testing folds in any cross-validation split
- All spectra retained must have at least five peaks with relative intensity above 2%, intensities scaled to 0–100
- Fingerprint targets must be 528-bit binary vectors assembled from MACCS + FP3 + FP4

## Limitations

- Only tested on positive-mode spectra (100–1010 Da, H+ or NH4+ adducts) from HCD/QqQ/Q-TOF and IT instruments; negative-mode generalization is untested here
- The paper notes no released pre-trained model weights for MetFID and no explicit data-split strategy discussion beyond the InChIKey-block disjoint CV described
- The 30 eV low/high cutoff was empirically derived from cluster analysis of 916 NIST 17 compounds (>30 spectra per compound); other datasets may need re-derivation

## Evidence

- [results] Combined vs low-energy model result: [evidence span withheld — anchor status: not_found]
- [readme] PyFingerprint fingerprint types: [evidence span withheld — anchor status: external_doc]
