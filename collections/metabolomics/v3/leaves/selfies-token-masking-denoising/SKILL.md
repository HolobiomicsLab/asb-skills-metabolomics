---
name: selfies-token-masking-denoising
description: Use when pretraining a seq2seq molecular model on unlabeled SELFIES corpora
  and you need a denoising objective, or when reconstructing/reproducing MS-BART-style
  multi-task pretraining from fingerprint–molecule pairs.
when_to_use_negative:
- Finetuning or inference on experimental mass spectra — that stage conditions on
  MIST-predicted fingerprints (with [M+Na]+ adducts filtered out) rather than applying
  token masking
- You need actual structure elucidation from spectra — this objective only teaches
  SELFIES token grammar on unlabeled molecules and produces no spectra-to-structure
  capability by itself
- Your corpus is already label-rich with spectra–structure pairs; supervised translation/finetuning
  objectives are then more appropriate
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MS-BART
  role: Framework whose multi-task pretraining this denoising objective is a component
    of
  repo: https://github.com/OpenDFM/MS-BART
- name: BART-Base
  role: Encoder–decoder backbone initialized from scratch and trained with the masking/recovery
    objective
- name: SELFIES
  role: String-based molecular representation whose tokens are masked and recovered
- name: MolGen
  role: Source of the zinc.npy SELFIES vocabulary (185 tokens) used by the unified
    tokenizer
  repo: https://github.com/zjunlp/MolGen
- name: RDKit
  role: Computes the 4096-bit Morgan fingerprints paired with molecules for the companion
    translation/hybrid pretraining tasks
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:61376c39c5eb1a507f7a555314cdfa82bc68f823bef00fab445f8817eb7ff6b4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# selfies-token-masking-denoising

## Summary

A self-supervised pretraining objective for molecular language models: randomly mask 30% of tokens in SELFIES sequences and train the model to recover the originals. It is one of three multi-task objectives (alongside fingerprint-to-molecule translation and hybrid denoising) used to pretrain MS-Bart on a corpus of fingerprint–molecule pairs.

## When to use

Use when pretraining a seq2seq molecular model on unlabeled SELFIES corpora and you need a denoising objective, or when reconstructing/reproducing MS-BART-style multi-task pretraining from fingerprint–molecule pairs.

## When NOT to use

- Finetuning or inference on experimental mass spectra — that stage conditions on MIST-predicted fingerprints (with [M+Na]+ adducts filtered out) rather than applying token masking
- You need actual structure elucidation from spectra — this objective only teaches SELFIES token grammar on unlabeled molecules and produces no spectra-to-structure capability by itself
- Your corpus is already label-rich with spectra–structure pairs; supervised translation/finetuning objectives are then more appropriate

## Inputs

- pubchem-4M pretraining corpus of fingerprint–molecule pairs (train/val splits)
- SELFIES sequences of unlabeled molecules
- Unified SELFIES/fingerprint tokenizer (185 SELFIES tokens from MolGen zinc.npy vocabulary)
- BART-Base model configuration

## Outputs

- Pretrained BART-Base checkpoints saved under ./logs/pretrained-model
- Per-task training/validation loss curves
- Model weights reusable for fingerprint-to-molecule translation and downstream spectral finetuning

## How to apply

Load the pretraining corpus of fingerprint–molecule pairs (pubchem-4M train/val splits) and tokenize with the unified SELFIES/fingerprint tokenizer (185 SELFIES tokens derived from the MolGen zinc.npy vocabulary). Initialize the BART-Base backbone from scratch with a normal-distribution draw (no pretrained weights). For each SELFIES sequence S={s1,…,sl}, randomly replace 30% of tokens with a Mask token and train the encoder–decoder to output the original sequence, optimizing the conditional-generation cross-entropy loss with maximum sequence length 512. Run this task jointly with task ② (fingerprint-to-molecule translation conditioned on RDKit-computed 4096-bit Morgan fingerprint tokens FP) and task ③ (hybrid denoising, concatenating FP and 30%-masked SELFIES with the <fps_sep> separator in both orders [FP, <fps_sep>, S_masked] and [S_masked, <fps_sep>, FP]). Log per-task train/validation loss curves; the denoising rationale is to teach the model the token grammar and syntax of SELFIES before finetuning on spectra-derived fingerprints.

## Related tools

- **MS-BART** (Framework whose multi-task pretraining this denoising objective is a component of) — https://github.com/OpenDFM/MS-BART
- **BART-Base** (Encoder–decoder backbone initialized from scratch and trained with the masking/recovery objective)
- **SELFIES** (String-based molecular representation whose tokens are masked and recovered)
- **MolGen** (Source of the zinc.npy SELFIES vocabulary (185 tokens) used by the unified tokenizer) — https://github.com/zjunlp/MolGen
- **RDKit** (Computes the 4096-bit Morgan fingerprints paired with molecules for the companion translation/hybrid pretraining tasks)

## Examples

```
mask = torch.rand(seq.shape) < 0.3; seq_masked[mask] = tokenizer.mask_token_id; loss = model(input_ids=seq_masked, labels=seq).loss
```

## Evaluation signals

- Masking rate on each SELFIES sequence is exactly 30% of tokens, replaced with the Mask token
- Tokenizer vocabulary contains 185 SELFIES tokens sourced from the MolGen zinc.npy file
- Training and validation loss curves both decrease monotonically over epochs on the pubchem-4M splits
- Checkpoints are written to ./logs/pretrained-model and reloadable to recover masked tokens
- Maximum sequence length of 512 is enforced during tokenization and batching

## Limitations

- Pretraining data is drawn from a clean 4M subset filtered by Tanimoto similarity > 0.5, so coverage of chemical space is bounded
- The denoising task alone provides no spectral-to-structure mapping; downstream performance depends on MIST fingerprint prediction quality (which the paper lists as a key limitation)
- Molecular formula information is not used during training, only for re-ranking, per the paper's stated limitations

## Evidence

- [methods] SELFIES Denoising pretraining task: [evidence span withheld — anchor status: not_found]
- [methods] Backbone initialization: [evidence span withheld — anchor status: not_found]
