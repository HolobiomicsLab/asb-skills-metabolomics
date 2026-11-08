---
name: mass-spectra-fingerprint-tokenization
description: Use when preparing experimental MS/MS spectra (e.g. MassSpecGym or NPLIB1)
  as token inputs for a fingerprint-to-structure generative model, and spectra must
  be converted into a condition-invariant discrete representation.
when_to_use_negative:
- Input is already discrete molecular structure (SELFIES/SMILES) without associated
  spectra — this skill converts spectra→fingerprints, not molecules→fingerprints.
- The task requires simulated spectra for training data — MS-BART uses fingerprints
  instead to avoid simulating mass spectra, so if you need spectral simulation use
  ICEBERG/other forward models.
- The model consumes raw peak lists or continuous spectra directly (e.g. diffusion
  or CNN encoders that do not require fingerprint tokenization).
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3421
tools:
- name: MIST
  role: Pretrained spectral inference model that predicts 4096-bit Morgan fingerprints
    from spectra, conditioned on metadata and molecular formula; its checkpoint weights
    (mist-msg.pt / mist_canopus.pt from FRIGID) are the tool used for the fingerprint
    prediction step.
  repo: https://github.com/coleygroup/FRIGID
- name: RDKit
  role: Cheminformatics toolkit used to compute reference fingerprints from molecular
    structures (for verification, Tanimoto evaluation, and the alignment rank loss),
    not for the spectra→fingerprint prediction itself.
- name: MS-BART
  role: 'Downstream consumer: the unified-vocabulary BART-Base model that takes thresholded
    fingerprint tokens as spectra input during finetuning and alignment.'
  repo: https://github.com/OpenDFM/MS-BART
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_content_hash: sha256:9d997ebd1b122f25085aa2a2b25e23d13feb98092eef9312e6c75a1eaacdae66
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/FRIGID
  tools:
  - BART-Base
  - MIST
  - MS-BART
  - MassSpecGym
  - NPLIB1
  - RDKit
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3421
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mass-spectra-fingerprint-tokenization

## Summary

Tokenize experimental mass spectra as fixed-length 4096-bit circular Morgan fingerprints (predicted with MIST, conditioned on metadata and molecular formula), providing the invariant spectral input representation consumed by MS-BART's finetuning and alignment stages.

## When to use

Use when preparing experimental MS/MS spectra (e.g. MassSpecGym or NPLIB1) as token inputs for a fingerprint-to-structure generative model, and spectra must be converted into a condition-invariant discrete representation.

## When NOT to use

- Input is already discrete molecular structure (SELFIES/SMILES) without associated spectra — this skill converts spectra→fingerprints, not molecules→fingerprints.
- The task requires simulated spectra for training data — MS-BART uses fingerprints instead to avoid simulating mass spectra, so if you need spectral simulation use ICEBERG/other forward models.
- The model consumes raw peak lists or continuous spectra directly (e.g. diffusion or CNN encoders that do not require fingerprint tokenization).

## Inputs

- Experimental MS/MS spectra with metadata (adduct, collision energy, precursor formula) from MassSpecGym or NPLIB1
- MIST fingerprint probability vectors (4096-bit radius-2 Morgan)
- Molecular formula annotations
- Train/val/test fold splits preserving the original fold structure

## Outputs

- Thresholded 4096-bit binary fingerprint token sequences (epsilon=0.11 for MassSpecGym, epsilon=0.2 for NPLIB1)
- Fingerprint–SELFIES input tables for train/val/test splits in the unified vocabulary
- Fingerprint tables ready for finetuning (scripts/msg/finetune.sh / scripts/canopus/finetune.sh)

## How to apply

Represent each spectrum as a 4096-bit radius-2 circular Morgan fingerprint FP∈{0,1}^4096 predicted with the pretrained MIST model, conditioned on associated metadata (adduct, collision energy, precursor formula) and molecular formula, then serialize the fingerprint into a unified token vocabulary shared with SELFIES molecules. Threshold the predicted probabilities with an ε parameter (MassSpecGym uses ε=0.11, NPLIB1 uses ε=0.2) to convert probability vectors into binary bit strings, and split the fingerprint–spectrum table into train/val/test folds, retaining only [M+H]+ adducts for training since these dominate the benchmarks. For finetuning, feed the thresholded fingerprint tokens to the pretrained BART-Base backbone; the rationale is that fingerprints are invariant to experimental conditions (collision energies, adduct types, instrument settings), eliminating the need to simulate mass spectra. This skill is the prerequisite for scripts/msg/finetune.sh and scripts/canopus/finetune.sh, and its output feeds the contrastive alignment with chemical feedback (Tanimoto-similarity rank loss).

## Related tools

- **MIST** (Pretrained spectral inference model that predicts 4096-bit Morgan fingerprints from spectra, conditioned on metadata and molecular formula; its checkpoint weights (mist-msg.pt / mist_canopus.pt from FRIGID) are the tool used for the fingerprint prediction step.) — https://github.com/coleygroup/FRIGID
- **RDKit** (Cheminformatics toolkit used to compute reference fingerprints from molecular structures (for verification, Tanimoto evaluation, and the alignment rank loss), not for the spectra→fingerprint prediction itself.)
- **MS-BART** (Downstream consumer: the unified-vocabulary BART-Base model that takes thresholded fingerprint tokens as spectra input during finetuning and alignment.) — https://github.com/OpenDFM/MS-BART

## Examples

```
python preprocess/fp_pred_main.py --mist mist_msg.pt --batch-size 32 --data data/MassSpecGym/mist
```

## Evaluation signals

- Predicted fingerprint bit-vectors must have length exactly 4096 and be binary after thresholding (values ∈ {0,1}).
- Thresholded table must reproduce the paper's chosen epsilon: 0.11 for MassSpecGym and 0.2 for NPLIB1 — verify by checking Table 4 values (Top-1 Tanimoto ≈ 0.23 for MSG, 0.44 for NPLIB1).
- Train/val/test fold splits must preserve the original fold structure; only [M+H]+ adducts retained for training, [M+Na]+ filtered out.
- Fingerprint-SELFIES input tables must be serialized with the <fps_sep> separator token for hybrid denoising pretraining and translated-token format for finetuning.
- End-to-end check: the finetuned model's Top-1 accuracy on the MSG test fold should be 1.07% (ε=0.11) and NPLIB1 Top-1 accuracy 7.45%, matching Table 1 when downstream finetuning + alignment is also applied correctly.

## Limitations

- The accuracy of the MIST fingerprint predictions significantly impacts overall performance — this skill is bottlenecked by the upstream MIST encoder, so retraining MIST on task-specific data can shift results.
- Threshold ε is a tunable hyperparameter; the paper reports results for ε∈{0.10, 0.11, 0.15, 0.20} and shows the model is relatively insensitive, but very different values could degrade performance.
- The representation discards peak intensities and fine spectral structure — it is a discrete fingerprint tokenization, so models requiring raw spectral signals cannot use it directly.

## Evidence

- [methods] we represent each spectrum as a 4096-bit circular Morgan fingerprint FP∈{0,1}4096: [evidence span withheld — anchor status: not_found]
- [methods] we employ the pretrained MIST model [18] to predict molecular fingerprints FP from mass spectra, conditioned on the chemical formula.: [evidence span withheld — anchor status: not_found]
- [methods] fingerprint tokens FP. Generate SELFIES sequences conditioned on fingerprint tokens FP.: [evidence span withheld — anchor status: not_found]
