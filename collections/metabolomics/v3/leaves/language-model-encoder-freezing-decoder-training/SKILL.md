---
name: language-model-encoder-freezing-decoder-training
description: 'Use when running the chemical-feedback alignment stage of MS-BART after
  Pretrain-FT: candidates must be re-ranked by Tanimoto similarity and only decoder
  weights should receive gradient updates.'
when_to_use_negative:
- 'Pretraining or fine-tuning stages: these train the full model with multi-task denoising/translation
  objectives or cross-entropy on experimental spectra, not the frozen-encoder rank-loss
  alignment'
- No ground-truth reference structure is available, so Tanimoto-based preference scores
  Ps(Si) and the rank loss cannot be computed
- Molecular formula should guide generation during training — in this method formula
  information is used only for candidate re-ranking, not fed to the model
tools:
- name: MS-BART
  role: Unified mass-spectra/molecule framework whose BART-Base decoder is aligned
    with chemical feedback while the encoder stays frozen
  repo: github.com/OpenDFM/MS-BART
- name: MS-Bart
  role: Model implementation (Fig. 2) receiving fingerprint-token inputs during the
    alignment update
  repo: github.com/OpenDFM/MS-BART
- name: BART-Base
  role: Backbone whose encoder is frozen and decoder updated during the alignment
    stage
- name: MolGen
  role: Prior work whose chemical-feedback alignment practice (frozen encoder, preference-based
    rank loss) is followed
  repo: zjunlp/MolGen
- name: RDKit
  role: Computes Morgan fingerprints (radius 2) from candidates and ground truth to
    derive Tanimoto preference scores
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:05d553ee6b5f2c01d528f88ce14bd1847a1394fbabb81d1497e6f6f406598762
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MS-BART
  - MS-Bart
  - MolGen
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# language-model-encoder-freezing-decoder-training

## Summary

Alignment-stage training recipe for MS-BART in which the pretrained encoder is frozen (following prior-work practice from MolGen's chemical-feedback alignment) and only the BART-Base decoder parameters are updated under the combined L = L_ce + alpha*L_rank objective.

## When to use

Use when running the chemical-feedback alignment stage of MS-BART after Pretrain-FT: candidates must be re-ranked by Tanimoto similarity and only decoder weights should receive gradient updates.

## When NOT to use

- Pretraining or fine-tuning stages: these train the full model with multi-task denoising/translation objectives or cross-entropy on experimental spectra, not the frozen-encoder rank-loss alignment
- No ground-truth reference structure is available, so Tanimoto-based preference scores Ps(Si) and the rank loss cannot be computed
- Molecular formula should guide generation during training — in this method formula information is used only for candidate re-ranking, not fed to the model

## Inputs

- Fine-tuned MS-BART checkpoint (Pretrain-FT stage, BART-Base backbone)
- Thresholded fingerprint–SELFIES finetuning data (bit threshold 0.2, MassSpecGym layout under logs/datasets/MassSpecGym/)
- Ground-truth molecular structures for computing Tanimoto preference scores
- Predicted Morgan fingerprints (4096-bit, radius-2) from MIST conditioned on chemical formula

## Outputs

- Aligned MS-BART checkpoint saved under logs/<run-name>
- Validation metrics: Top-1/Top-10 Tanimoto, Top-1/Top-10 accuracy, MCES distance
- Candidate SELFIES sequences ranked by formula distance with log-probability tie-breaking

## How to apply

Load the fine-tuned MS-BART checkpoint (Pretrain-FT stage, BART-Base backbone) together with the thresholded fingerprint–SELFIES finetuning data (threshold 0.2 on fingerprint bits for MassSpecGym, keeping only [M+H]+ adducts). Generate multiple candidate SELFIES sequences per fingerprint input via beam-search multinomial sampling, then score each candidate's preference Ps(Si) as Tanimoto similarity to the ground-truth structure using RDKit Morgan fingerprints (radius 2, 2048-bit), ranking candidates by molecular-formula distance (sum of per-atom count differences) with log-probability as tie-breaker. Freeze all encoder parameters (requires_grad=False) per prior-work practice from MolGen's chemical-feedback alignment, and update only the decoder with the combined objective L = L_ce + alpha*L_rank, where the rank loss uses rank-scaled margins gamma_ij = (j-i)*gamma to push probability mass toward higher-Tanimoto candidates while token-level cross-entropy is retained. Save the aligned checkpoint under logs/<run-name> and evaluate against the Pretrain-FT baseline (Fig. 3 Stage 3). Rationale: freezing the encoder preserves the pretrained fingerprint→token representations while the decoder learns to prefer chemically closer structures, reducing molecular hallucinations.

## Related tools

- **MS-BART** (Unified mass-spectra/molecule framework whose BART-Base decoder is aligned with chemical feedback while the encoder stays frozen) — github.com/OpenDFM/MS-BART
- **MS-Bart** (Model implementation (Fig. 2) receiving fingerprint-token inputs during the alignment update) — github.com/OpenDFM/MS-BART
- **BART-Base** (Backbone whose encoder is frozen and decoder updated during the alignment stage)
- **MolGen** (Prior work whose chemical-feedback alignment practice (frozen encoder, preference-based rank loss) is followed) — zjunlp/MolGen
- **RDKit** (Computes Morgan fingerprints (radius 2) from candidates and ground truth to derive Tanimoto preference scores)

## Examples

```
for p in model.encoder.parameters(): p.requires_grad = False  # freeze encoder, update decoder with L = L_ce + alpha*L_rank
```

## Evaluation signals

- Verify that all encoder parameter groups have requires_grad=False and appear in zero/frozen optimizer state, while decoder parameters receive updates
- Aligned checkpoint's validation Top-1/Top-10 Tanimoto, accuracy, and MCES must improve over the Pretrain-FT stage (Fig. 3, Stage 3 comparison)
- Combined loss L = L_ce + alpha*L_rank decreases during training; rank-loss term pushes candidate probability mass toward higher-Tanimoto candidates
- Candidate rankings keyed on molecular-formula distance are reproducible with log-probability tie-breaking
- Training data respects the [M+Na]+ filtering and fingerprint-bit threshold epsilon = 0.2 used for MassSpecGym finetuning

## Limitations

- Depends on external MIST model for fingerprint prediction; fingerprint-prediction accuracy limits overall alignment performance
- Chemical formula is used only for candidate re-ranking and not incorporated into model training (deferred to future work)
- Alignment only adjusts decoder behavior toward reference-similar structures; encoder representations remain fixed and cannot adapt to dataset-specific biases during this stage

## Evidence

- [methods] MS-BART [M+Na]+ filtering before fine-tuning and alignment: [evidence span withheld — anchor status: not_found]
