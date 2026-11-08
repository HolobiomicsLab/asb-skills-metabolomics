---
name: cross-modal-hybrid-denoising-pretraining
description: Use when pretraining a sequence-to-sequence model on a corpus of fingerprint–SELFIES
  pairs (e.g. 4M PubChem molecules) to teach cross-modal mapping between Morgan fingerprint
  tokens and molecular structures before finetuning on spectra.
when_to_use_negative:
- The downstream data are already experimental spectra rather than computed fingerprint–molecule
  pairs — this skill covers pretraining only, not the MIST-based finetuning or chemical-feedback
  alignment stages.
- Only one modality is available (e.g. SELFIES sequences with no RDKit-computed fingerprints),
  so the translation and hybrid denoising tasks cannot be constructed.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: MS-BART
  role: Framework whose multi-task pretraining objectives this skill implements on
    the fingerprint–molecule corpus
  repo: github.com/OpenDFM/MS-BART
- name: BART-Base
  role: Seq2seq backbone initialized from scratch with a normal distribution before
    multi-task pretraining
- name: SELFIES
  role: String-based molecular representation whose tokens are masked and reconstructed
    during denoising tasks
- name: RDKit
  role: Computes the 4096-bit Morgan fingerprints paired with each unlabeled molecule
- name: MolGen
  role: Source of the zinc.npy vocabulary providing the 185 SELFIES tokens for the
    unified tokenizer
  repo: github.com/zjunlp/MolGen
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:684b36e2e0dbd3f6cc3404a29c3d3cf430b915c3debd331397d6dd7ec45db4bb
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MS-BART
  - MolGen
  - RDKit
  - SELFIES
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-modal-hybrid-denoising-pretraining

## Summary

Multi-task pretraining of a BART-Base seq2seq model on fingerprint–molecule pairs, combining SELFIES denoising (30% masking), fingerprint-to-molecule translation, and hybrid denoising that concatenates fingerprint tokens with masked SELFIES via a <fps_sep> separator in both input orders.

## When to use

Use when pretraining a sequence-to-sequence model on a corpus of fingerprint–SELFIES pairs (e.g. 4M PubChem molecules) to teach cross-modal mapping between Morgan fingerprint tokens and molecular structures before finetuning on spectra.

## When NOT to use

- The downstream data are already experimental spectra rather than computed fingerprint–molecule pairs — this skill covers pretraining only, not the MIST-based finetuning or chemical-feedback alignment stages.
- Only one modality is available (e.g. SELFIES sequences with no RDKit-computed fingerprints), so the translation and hybrid denoising tasks cannot be constructed.

## Inputs

- Fingerprint–molecule pair corpus (pubchem-4M train/val splits; fingerprints are 4096-bit RDKit Morgan fingerprints)
- Unified SELFIES/fingerprint tokenizer (185 SELFIES tokens from the MolGen zinc.npy vocabulary)
- BART-Base model configuration (parameters initialized from scratch)

## Outputs

- Pretrained MS-BART checkpoint saved under ./logs/pretrained-model
- Per-task training and validation loss curves (selfies-denoising, fp2mol-translation, hybrid-denoising)

## How to apply

Load a fingerprint–molecule corpus (pubchem-4M train/val splits) with a unified tokenizer covering 185 SELFIES tokens (from the MolGen zinc.npy vocabulary) plus fingerprint tokens. Initialize a BART-Base backbone from scratch with a normal-distribution initialization. Run three tasks with a conditional-generation cross-entropy loss and max sequence length 512: (1) SELFIES denoising — randomly mask 30% of SELFIES tokens and recover originals; (2) fingerprint-to-molecule translation — generate the full SELFIES sequence conditioned solely on the 4096-bit Morgan fingerprint tokens; (3) hybrid denoising — concatenate fingerprint tokens and 30%-masked SELFIES via <fps_sep> in both orders, [FP, <fps_sep>, S_masked] and [S_masked, <fps_sep>, FP], training to predict the full SELFIES sequence. Joint multi-task training strengthens cross-modal grounding so the later finetuning stage can consume MIST-predicted fingerprints as spectra surrogates.

## Related tools

- **MS-BART** (Framework whose multi-task pretraining objectives this skill implements on the fingerprint–molecule corpus) — github.com/OpenDFM/MS-BART
- **BART-Base** (Seq2seq backbone initialized from scratch with a normal distribution before multi-task pretraining)
- **SELFIES** (String-based molecular representation whose tokens are masked and reconstructed during denoising tasks)
- **RDKit** (Computes the 4096-bit Morgan fingerprints paired with each unlabeled molecule)
- **MolGen** (Source of the zinc.npy vocabulary providing the 185 SELFIES tokens for the unified tokenizer) — github.com/zjunlp/MolGen

## Examples

```
python pretrain.py --task hybrid_denoising --corpus data/pubchem-4M --tokenizer zinc.npy --mask_ratio 0.30 --max_seq_len 512 --output_dir ./logs/pretrained-model
```

## Evaluation signals

- Training and validation loss curves decrease and converge for each of the three pretraining tasks (denoising, translation, hybrid denoising).
- Checkpoints are saved under ./logs/pretrained-model after training completes.
- Model recovers original SELFIES tokens from 30%-masked inputs (task ①) and generates full SELFIES sequences from fingerprint tokens alone (task ②).
- Hybrid-denoising inputs use both orderings [FP, <fps_sep>, S_masked] and [S_masked, <fps_sep>, FP] with the <fps_sep> separator, and outputs are full SELFIES sequences.
- Max sequence length is capped at 512 with the conditional-generation cross-entropy loss across all tasks.

## Limitations

- Pretraining uses fingerprints computed from unlabeled molecules via RDKit; the spectra-to-fingerprint step depends on the external MIST model, whose prediction accuracy limits downstream performance.
- Pretraining data quality matters — the released pipeline filters pretraining data by Tanimoto similarity > 0.5 and excludes molecules with MCES distance less than two from test-fold molecules.
- Formula information is not used during model training, only for re-ranking.

## Evidence

- [methods] Randomly mask 30% of tokens in SELFIES sequence S={s1,…,sl} with Mask token and recover original tokens.: [evidence span withheld — anchor status: not_found]
- [methods] Hybrid denoising with separator: [evidence span withheld — anchor status: not_found]
- [methods] BART-Base backbone initialization: [evidence span withheld — anchor status: not_found]
