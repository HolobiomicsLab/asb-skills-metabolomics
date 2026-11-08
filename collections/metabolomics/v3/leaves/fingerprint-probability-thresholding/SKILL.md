---
name: fingerprint-probability-thresholding
description: Use when experimental MS/MS spectra must be turned into discrete fingerprint
  tokens for finetuning or inference, and a pretrained spectral inference model (MIST)
  outputs per-bit probabilities {p0,...,p4095} rather than binary fingerprints.
when_to_use_negative:
- When fingerprints are already computed directly from known molecular structures
  with RDKit (the pretraining fingerprint-molecule pairs) — no spectral inference
  or thresholding is needed
- When no pretrained MIST model is available for the spectra's acquisition conditions
  — fingerprint prediction accuracy significantly limits overall downstream performance
- When the task is spectrum simulation or forward prediction (spectrum-to-fingerprint
  direction is reversed); this skill only handles the spectra-to-fingerprint direction
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MIST
  role: Pretrained spectral inference model that predicts the 4096-bit Morgan fingerprint
    probabilities from spectra, conditioned on metadata and molecular formula, prior
    to thresholding
  repo: https://github.com/coleygroup/FRIGID
- name: MS-Bart
  role: 'Downstream consumer of the binarized fingerprints: active bits become <fp{i:04d}>
    tokens in its unified vocabulary for finetuning and inference'
  repo: https://github.com/OpenDFM/MS-BART
- name: RDKit
  role: Defines the 4096-bit radius-2 circular (Morgan) fingerprint representation
    that the thresholded probabilities approximate
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:4de2a4bd2b20ab40fa1000db1cf959bcc513751f8997579f8166441909dff673
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/FRIGID
  tools:
  - BART-Base
  - MIST
  - MS-Bart
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# fingerprint-probability-thresholding

## Summary

Converts MIST-predicted per-bit probability vectors for a 4096-bit circular Morgan fingerprint into binary fingerprints and discrete <fp{i:04d}> tokens, bridging noisy experimental mass spectra into the fingerprint-token vocabulary that the fingerprint-to-molecule model consumes.

## When to use

Use when experimental MS/MS spectra must be turned into discrete fingerprint tokens for finetuning or inference, and a pretrained spectral inference model (MIST) outputs per-bit probabilities {p0,...,p4095} rather than binary fingerprints.

## When NOT to use

- When fingerprints are already computed directly from known molecular structures with RDKit (the pretraining fingerprint-molecule pairs) — no spectral inference or thresholding is needed
- When no pretrained MIST model is available for the spectra's acquisition conditions — fingerprint prediction accuracy significantly limits overall downstream performance
- When the task is spectrum simulation or forward prediction (spectrum-to-fingerprint direction is reversed); this skill only handles the spectra-to-fingerprint direction

## Inputs

- MIST probability vectors {p0,...,p4095} predicted from experimental mass spectra, conditioned on chemical formula and associated metadata
- Dataset-specific binarization threshold epsilon (0.2 for NPLIB1/CANOPUS; 0.11 for MassSpecGym)
- Alternative MIST weight dictionaries mist_msg.pt / mist_canopus.pt from the FRIGID release

## Outputs

- Binarized 4096-bit circular Morgan fingerprints (radius 2)
- Token sequences of <fp{i:04d}> tokens, one per active bit (i in 0..4095)
- Fingerprint-SELFIES table ready for train/val/test splitting via preprocess/prepare_test_data.py

## How to apply

Load the MIST probability vectors {p0,...,p4095} predicted from experimental spectra conditioned on the associated metadata and chemical formula, then binarize each vector into a 4096-bit fingerprint by thresholding at dataset-specific epsilon: 0.2 for NPLIB1/CANOPUS and 0.11 for MassSpecGym. Each active bit i is emitted as a single <fp{i:04d}> token, producing a fingerprint-SELFIES table that is subsequently split into train/val/test with preprocess/prepare_test_data.py (threshold 0.2 for the MSG/CANOPUS split files). The rationale is that fingerprints are invariant to experimental conditions (collision energies, adduct types, instrument settings), so binarized MIST predictions let a pretrained fingerprint-to-molecule model be finetuned on real spectra without simulating spectra under diverse conditions. Optionally use the FRIGID-provided mist_msg.pt / mist_canopus.pt parameter dictionaries as alternative MIST weights in the fingerprint preprocessing script.

## Related tools

- **MIST** (Pretrained spectral inference model that predicts the 4096-bit Morgan fingerprint probabilities from spectra, conditioned on metadata and molecular formula, prior to thresholding) — https://github.com/coleygroup/FRIGID
- **MS-Bart** (Downstream consumer of the binarized fingerprints: active bits become <fp{i:04d}> tokens in its unified vocabulary for finetuning and inference) — https://github.com/OpenDFM/MS-BART
- **RDKit** (Defines the 4096-bit radius-2 circular (Morgan) fingerprint representation that the thresholded probabilities approximate)

## Evaluation signals

- Threshold value matches the benchmark: epsilon=0.2 for NPLIB1/CANOPUS-derived vectors and epsilon=0.11 for MassSpecGym vectors
- Every emitted token is a well-formed <fp{i:04d}> token with index i in 0..4095, one token per active bit, and the binarized vector is a valid 4096-bit Morgan fingerprint
- The resulting fingerprint-SELFIES table loads successfully in preprocess/prepare_test_data.py with the MSG/CANOPUS split files using threshold 0.2
- Finetuning on the thresholded fingerprints improves validation Top-1 accuracy over the Pretrain stage (0.00% -> 1.07% Top-1 on MassSpecGym)
- Training uses only [M+H]+ adducts after filtering, while val/test retain all adducts, confirming the binarized table split correctly

## Limitations

- Accuracy of the MIST fingerprint predictions significantly impacts overall downstream performance; finetuning MIST on task-specific datasets is noted as future work
- The optimal threshold epsilon is dataset-dependent (0.2 vs 0.11), so reusing a threshold across benchmarks can degrade bit precision/recall
- Formula information is not used in model training, only for re-ranking, so thresholded fingerprints carry the molecular-conditioning signal alone at this stage

## Evidence

- [methods] Load MIST probability vectors {p0,...,p4095} predicted from experimental spectra conditioned on the chemical formula, and binarize them into 4096-bit fingerprints using threshold epsilon=0.2 for NPLIB1/CANOPUS and epsilon=0.11 for MassSpecGym, converting each active bit into an <fp{i:04d}> token.: [evidence span withheld — anchor status: not_found]
- [methods] we represent each spectrum as a 4096-bit circular Morgan fingerprint FP∈{0,1}4096: [evidence span withheld — anchor status: not_found]
- [intro] using MIST [18] to predict fingerprints from spectra, conditioned on associated metadata and molecular formulas: [evidence span withheld — anchor status: not_found]
