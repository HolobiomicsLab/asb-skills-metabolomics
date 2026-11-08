---
name: molecular-fingerprint-vector-assembly
description: Use when preparing compound fingerprint labels for machine-learning training
  or scoring fingerprint similarity between a predicted fingerprint and database candidate
  structures.
when_to_use_negative:
- When the goal is direct spectral-library matching against curated MS/MS reference
  spectra (e.g., GNPS, NIST) rather than fingerprint-based candidate ranking.
- When only a single fingerprint digit or block must be predicted — the multi-label
  formulation is designed for the whole 528-bit vector at once.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: OpenBabel
  role: Computes the MACCS, FP3, and FP4 fingerprints that are concatenated into the
    528-bit binary vector.
- name: PyFingerprint
  role: Alternative Python package that can retrieve MACCS, fp3, and fp4 fingerprints
    via get_fingerprint() / get_fingerprints().
  repo: https://github.com/hcji/PyFingerprint
- name: MetFID
  role: Consumes the assembled fingerprint vectors as supervised labels and emits
    the predicted 528-bit vector for candidate ranking.
  repo: https://github.com/aohongyu/MetFID
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:cb496ed2126924757abc764af5df27c2f325ae0dcbba885a420d1f2edcbca29e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/PyFingerprint
  tools:
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  - PyFingerprint
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# molecular-fingerprint-vector-assembly

## Summary

Assemble a 528-bit binary molecular fingerprint vector by concatenating MACCS, FP3, and FP4 fingerprints computed with OpenBabel, providing the supervised target for MetFID's ANN-based fingerprint prediction from MS/MS spectra.

## When to use

Use when preparing compound fingerprint labels for machine-learning training or scoring fingerprint similarity between a predicted fingerprint and database candidate structures.

## When NOT to use

- When the goal is direct spectral-library matching against curated MS/MS reference spectra (e.g., GNPS, NIST) rather than fingerprint-based candidate ranking.
- When only a single fingerprint digit or block must be predicted — the multi-label formulation is designed for the whole 528-bit vector at once.

## Inputs

- SMILES or InChI strings for training compounds (e.g., 5,667 MoNA and 10,730 NIST 17 compounds)
- OpenBabel installation supporting MACCS, FP3, and FP4 fingerprint types
- Predicted fingerprint vector from the MetFID ANN when scoring candidates

## Outputs

- 528-bit binary fingerprint vector per compound
- Similarity scores between predicted and candidate fingerprints for compound ranking

## How to apply

For every training or candidate compound, compute its MACCS, FP3, and FP4 fingerprints with OpenBabel and concatenate them into a single 528-entry binary vector so all fingerprint digits are predicted simultaneously by one ANN rather than a digit-per-model scheme. Apply a sigmoid activation with binary cross-entropy loss at the fingerprint layer instead of softmax/categorical cross-entropy, since the multi-label formulation captures latent relationships among fingerprint bits. Remove bits that are all-zero across the training set before building the ANN input mapping. Use the assembled vectors as labels for Keras/TensorFlow training (30 epochs, batch size 100, Adam optimizer) and later compute Tanimoto-style similarity between a predicted 528-bit vector and candidate database fingerprints to rank compounds.

## Related tools

- **OpenBabel** (Computes the MACCS, FP3, and FP4 fingerprints that are concatenated into the 528-bit binary vector.)
- **PyFingerprint** (Alternative Python package that can retrieve MACCS, fp3, and fp4 fingerprints via get_fingerprint() / get_fingerprints().) — https://github.com/hcji/PyFingerprint
- **MetFID** (Consumes the assembled fingerprint vectors as supervised labels and emits the predicted 528-bit vector for candidate ranking.) — https://github.com/aohongyu/MetFID

## Examples

```
from PyFingerprint.fingerprint import get_fingerprint; fp_maccs = get_fingerprint(smi, 'maccs'); fp3 = get_fingerprint(smi, 'fp3'); fp4 = get_fingerprint(smi, 'fp4'); vec = np.concatenate([fp_maccs.to_numpy(), fp3.to_numpy(), fp4.to_numpy()])
```

## Evaluation signals

- Every assembled vector must contain exactly 528 binary entries (MACCS + FP3 + FP4 counts).
- Bits that are all-zero across the training set should be removed before training the ANN.
- Training convergence: binary cross-entropy loss decreases over 30 epochs with batch size 100 using the Adam optimizer.
- Similarity scores between the predicted fingerprint and the true compound's fingerprint should exceed scores for decoy candidates on structure-disjoint test compounds.
- Macro-average accuracy and F1 score on structure-disjoint test sets should be comparable to the reported MetFID performance.

## Limitations

- The 528-bit vector is defined by MACCS, FP3, and FP4; substituting other fingerprint types changes the vector length and is not covered by the MetFID trained model.
- Spectra from different collision energies (e.g., 14 eV vs. 38 eV) yield different fragmentation patterns; fingerprint prediction performance can vary with collision energy (30 eV was found to be a reasonable cutoff for low/high).
- The MetFID trained model (.h5) is tied to this specific 528-bit definition, so reusing the vectors with a different predictor requires retraining.
- No released pre-trained weights exist for the ANN unless the user downloads MetFID_CNN_40088_5618.h5 separately.

## Evidence

- [methods] Fingerprints computed with OpenBabel: [evidence span withheld — anchor status: not_found]
- [methods] Sigmoid + binary cross-entropy over softmax/categorical CE: [evidence span withheld — anchor status: not_found]
- [methods] Removing all-zero bins across training spectra: [evidence span withheld — anchor status: not_found]
- [readme] README requires PyFingerprint, Open Babel, and the trained .h5 model: [evidence span withheld — anchor status: not_found]
- [readme] PyFingerprint exposes OpenBabel fp3/fp4 and MACCS types: [evidence span withheld — anchor status: external_doc]
