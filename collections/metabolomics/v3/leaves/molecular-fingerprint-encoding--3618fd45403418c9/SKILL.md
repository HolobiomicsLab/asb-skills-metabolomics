---
name: molecular-fingerprint-encoding
description: Use when training or evaluating a machine-learning model that predicts
  compound fingerprints from MS/MS spectra, or when candidate compounds must be ranked
  by fingerprint similarity after spectral prediction.
when_to_use_negative:
- Your annotation task uses electron ionization (EI) spectra or fingerprint types
  beyond MACCS/FP3/FP4 where PyFingerprint's RDKit/CDK types would be more appropriate
  — the MetFID target vector is specifically 528 binary entries from OpenBabel fingerprints
- Input is already a fingerprint table or predicted fingerprint matrix — this skill
  encodes compounds, not spectra, and should not be re-run on encoded features
- Candidate ranking requires calibrated false discovery rates — the article notes
  no FDR discussion and no released pre-trained model weights
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: OpenBabel
  role: Computes the MACCS, FP3, and FP4 fingerprints assembled into the 528-bit binary
    target vector
- name: PyFingerprint
  role: Python wrapper providing babeltypes 'fp3' and 'fp4' fingerprint computation
    used by the MetFID pipeline
  repo: https://github.com/hcji/PyFingerprint
- name: MetFID
  role: ANN model trained to predict the 528-bit fingerprint vector from an MS/MS
    spectrum; ranked candidates via fingerprint similarity
  repo: https://github.com/aohongyu/MetFID
- name: InChIKey
  role: Compound identifier used to merge spectra and enforce structure-disjoint evaluation
    folds
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:b34f47d6f8c2a42176b765f82319b8e1da35b6f65726357fb430d043c2816ad7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/PyFingerprint
  tools:
  - InChIKey
  - Keras
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  - PyFingerprint
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# molecular-fingerprint-encoding

## Summary

Encode compounds as binary molecular fingerprints (MACCS + FP3 + FP4 assembled into a 528-bit vector via OpenBabel) to serve as ANN training targets for compound fingerprint prediction from MS/MS spectra in MetFID-style metabolite annotation.

## When to use

Use when training or evaluating a machine-learning model that predicts compound fingerprints from MS/MS spectra, or when candidate compounds must be ranked by fingerprint similarity after spectral prediction.

## When NOT to use

- Your annotation task uses electron ionization (EI) spectra or fingerprint types beyond MACCS/FP3/FP4 where PyFingerprint's RDKit/CDK types would be more appropriate — the MetFID target vector is specifically 528 binary entries from OpenBabel fingerprints
- Input is already a fingerprint table or predicted fingerprint matrix — this skill encodes compounds, not spectra, and should not be re-run on encoded features
- Candidate ranking requires calibrated false discovery rates — the article notes no FDR discussion and no released pre-trained model weights

## Inputs

- Compound structures (e.g., SMILES/MOL files) for compounds in MoNA and NIST 17 spectral libraries
- InChIKey identifiers used to merge spectra per compound
- PyFingerprint/OpenBabel fingerprint type definitions (MACCS, FP3, FP4)

## Outputs

- 528-bit binary fingerprint vectors (MACCS + FP3 + FP4) for each compound
- Fingerprint similarity scores used to rank candidate compounds

## How to apply

For every compound in the training/testing set, compute MACCS, FP3, and FP4 fingerprints with OpenBabel and assemble them into a single binary vector of 528 entries; this vector is the multi-class multi-label target the ANN regresses toward. Keep fingerprints consistent between training and candidate-ranking stages so that predicted fingerprints can be compared against database candidates via fingerprint similarity scores, with the top-ranked candidate used for annotation. Identify compounds by the first InChIKey block to enforce structure-disjoint splits when evaluating fingerprint prediction. Compute fingerprints programmatically (OpenBabel, or the PyFingerprint package, which exposes babeltypes 'fp3' and 'fp4'), avoiding per-model digit-by-digit prediction that neglects latent relationships between fingerprint digits. Note the article's contrast: unlike prior tools where 'a single model can only predict one digit or one block of the fingerprint', MetFID predicts the whole 528-bit vector at once.

## Related tools

- **OpenBabel** (Computes the MACCS, FP3, and FP4 fingerprints assembled into the 528-bit binary target vector)
- **PyFingerprint** (Python wrapper providing babeltypes 'fp3' and 'fp4' fingerprint computation used by the MetFID pipeline) — https://github.com/hcji/PyFingerprint
- **MetFID** (ANN model trained to predict the 528-bit fingerprint vector from an MS/MS spectrum; ranked candidates via fingerprint similarity) — https://github.com/aohongyu/MetFID
- **InChIKey** (Compound identifier used to merge spectra and enforce structure-disjoint evaluation folds)

## Examples

```
from PyFingerprint.fingerprint import get_fingerprint; fp = get_fingerprint('CCCCN', 'fp3')  # MACCS/FP3/FP4 via OpenBabel babeltypes, assemble into 528-bit vector
```

## Evaluation signals

- Every fingerprint vector contains exactly 528 binary entries (MACCS + FP3 + FP4 concatenated via OpenBabel)
- Structure-disjoint cross-validation confirms no compound (first InChIKey block) appears in both training and testing folds
- Predicted fingerprints, scored by similarity against candidate lists, rank the true compound top in candidate retrieval (e.g., MetaboQuest candidate lists queried via PubChemPy by molecular formula)
- Fingerprint targets are consistent between the ANN training phase and the candidate-ranking phase

## Limitations

- No released pre-trained model weights for MetFID beyond the README's downloadable CNN model MetFID_CNN_40088_5618.h5; the paper's future work notes the architecture is still evolving
- The article does not discuss specificity or false discovery rates in candidate ranking, though it notes a target-decoy approach could estimate FDRs
- The choice of MACCS+FP3+FP4 (528 bits) was not exhaustively validated; 'further investigation is needed to thoroughly evaluate different sets of fingerprints'

## Evidence

- [readme] README lists fp3/fp4 types: [evidence span withheld — anchor status: not_found]
