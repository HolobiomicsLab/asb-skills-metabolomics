---
name: fingerprint-to-molecule-sequence-translation
description: Use when constructing a cross-modal pretraining corpus of fingerprint–molecule
  pairs and you need a sequence-to-sequence model that maps fingerprint tokens (FP)
  to SELFIES structures, or when reproducing MS-BART's task ② pretraining objective.
when_to_use_negative:
- The goal is de novo generation from experimental mass spectra — after pretraining,
  spectra must be converted via MIST-predicted fingerprints and the model finetuned
  on experimental data before use
- The target representation is SMILES or SAFE rather than the SELFIES vocabulary used
  by MS-BART's unified tokenizer
- A pretrained checkpoint already exists and the task is finetuning/alignment (e.g.,
  [M+H]+ filtering, chemical feedback alignment) rather than from-scratch multi-task
  pretraining
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3318
tools:
- name: MS-BART
  role: Framework implementing the multi-task pretraining (denoising + fingerprint-to-molecule
    translation) over the fingerprint–molecule corpus
  repo: https://github.com/OpenDFM/MS-BART
- name: BART-Base
  role: Backbone seq2seq model initialized from scratch with a normal distribution
    for the translation task
- name: RDKit
  role: Computes the 4096-bit Morgan fingerprints used as conditioning tokens from
    unlabeled molecules
- name: SELFIES
  role: String-based molecular representation generated as the translation target
- name: MolGen
  role: Source of the zinc.npy SELFIES vocabulary (185 tokens) used by the unified
    tokenizer
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:5245658d2896ed695cda0ebec98fd4b07cf9eb1530acd188b2c27845e946224a
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - BART-Base
  - MS-BART
  - MolGen
  - RDKit
  - SELFIES
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fingerprint-to-molecule-sequence-translation

## Summary

Multi-task pretraining objective of MS-BART that trains a BART-Base backbone to generate SELFIES molecular sequences conditioned on 4096-bit Morgan fingerprint tokens, learned over ~4M fingerprint–molecule pairs computed with RDKit.

## When to use

Use when constructing a cross-modal pretraining corpus of fingerprint–molecule pairs and you need a sequence-to-sequence model that maps fingerprint tokens (FP) to SELFIES structures, or when reproducing MS-BART's task ② pretraining objective.

## When NOT to use

- The goal is de novo generation from experimental mass spectra — after pretraining, spectra must be converted via MIST-predicted fingerprints and the model finetuned on experimental data before use
- The target representation is SMILES or SAFE rather than the SELFIES vocabulary used by MS-BART's unified tokenizer
- A pretrained checkpoint already exists and the task is finetuning/alignment (e.g., [M+H]+ filtering, chemical feedback alignment) rather than from-scratch multi-task pretraining

## Inputs

- Fingerprint–molecule pair corpus (e.g., pubchem-4M train/val splits; 4 million unlabeled molecules with RDKit-computed fingerprints)
- Unified SELFIES/fingerprint tokenizer (185 SELFIES tokens from MolGen zinc.npy vocabulary)
- 4096-bit radius-2 circular Morgan fingerprint token sequences FP per molecule

## Outputs

- Pretrained BART-Base checkpoint under ./logs/pretrained-model
- Per-task training/validation cross-entropy loss curves for the fingerprint-to-molecule translation objective

## How to apply

Build fingerprint–molecule pairs by computing 4096-bit radius-2 circular Morgan fingerprints from unlabeled molecules with RDKit and tokenizing structures as SELFIES with the unified vocabulary (185 SELFIES tokens from the MolGen zinc.npy vocabulary, shared with fingerprint tokens). Initialize a BART-Base backbone from scratch with a normal distribution, then train task ②: generate the full SELFIES sequence conditioned solely on fingerprint tokens FP. Train jointly with task ① (SELFIES denoising: mask 30% of tokens with a Mask token and recover them) and task ③ (hybrid denoising: concatenate FP and 30%-masked SELFIES via <fps_sep> in both input orders [FP, <fps_sep>, S_masked] and [S_masked, <fps_sep>, FP], predicting the full SELFIES). Optimize all tasks with conditional-generation cross-entropy loss at a maximum sequence length of 512, saving checkpoints under ./logs/pretrained-model; MS-BART later conditions this same fingerprint-to-molecule capability on MIST-predicted fingerprints during finetuning on experimental spectra.

## Related tools

- **MS-BART** (Framework implementing the multi-task pretraining (denoising + fingerprint-to-molecule translation) over the fingerprint–molecule corpus) — https://github.com/OpenDFM/MS-BART
- **BART-Base** (Backbone seq2seq model initialized from scratch with a normal distribution for the translation task)
- **RDKit** (Computes the 4096-bit Morgan fingerprints used as conditioning tokens from unlabeled molecules)
- **SELFIES** (String-based molecular representation generated as the translation target)
- **MolGen** (Source of the zinc.npy SELFIES vocabulary (185 tokens) used by the unified tokenizer)

## Evaluation signals

- Training and validation cross-entropy loss curves decrease monotonically on both pubchem-4M train and validation splits for the translation task
- Conditioning on fingerprint tokens FP alone (no other input) yields valid, chemically parseable SELFIES sequences from the validation split
- Model output tokenizes/round-trips with the unified 185-token SELFIES vocabulary (zinc.npy) without unknown-token errors
- Saved checkpoints appear under ./logs/pretrained-model and load into the BART-Base architecture with max sequence length 512
- The jointly trained checkpoint also performs on sibling tasks (30% SELFIES denoising recovery and hybrid denoising with <fps_sep> in both input orders), indicating multi-task convergence

## Limitations

- Downstream accuracy depends on the quality of fingerprint predictions: 'The accuracy of these fingerprint predictions significantly impacts the overall performance.'
- Formula information is used only for re-ranking, not for training the model, per the paper's discussion
- Pretrain data should be filtered (e.g., Tanimoto similarity > 0.5 clean pretrain data in the released corpus; MCES-distance exclusion of molecules within distance <2 of test folds) to avoid test leakage

## Evidence

- [methods] Fingerprint-to-Molecule Translation pretraining task: [evidence span withheld — anchor status: not_found]
- [methods] backbone initialization: [evidence span withheld — anchor status: not_found]
- [methods] fingerprint computation: [evidence span withheld — anchor status: not_found]
