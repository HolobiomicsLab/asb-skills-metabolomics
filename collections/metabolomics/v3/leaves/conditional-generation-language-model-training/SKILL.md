---
name: conditional-generation-language-model-training
description: Use when pretraining a sequence-to-sequence generative model on paired
  structured representations (e.g., 4096-bit Morgan fingerprint tokens and SELFIES
  strings) to learn cross-modal conditional generation before downstream fine-tuning.
when_to_use_negative:
- Your training data are experimental mass spectra rather than fingerprint–molecule
  pairs — the spectra-to-fingerprint mapping requires MIST-predicted fingerprints
  and a separate fine-tuning/alignment stage, not this pretraining skill
- You only have a small labeled corpus (e.g., 231k MassSpecGym spectra) without access
  to a large unlabeled molecular corpus (4 million molecules) from which fingerprints
  can be computed
- The target output is not a SELFIES string (e.g., diffusion-based SAFE generation
  as in FRIGID uses a masked diffusion language model, not BART conditional generation)
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: MS-BART
  role: Unified pretraining framework whose multi-task objectives (denoising, translation)
    are trained on the fingerprint–molecule corpus
  repo: https://github.com/OpenDFM/MS-BART
- name: BART-Base
  role: Backbone encoder-decoder model initialized from scratch with a normal distribution
    and trained with the multi-task conditional-generation objectives
  repo: https://github.com/OpenDFM/MS-BART
- name: SELFIES
  role: String-based molecular representation whose tokens are masked and reconstructed
    in the denoising and translation tasks
- name: RDKit
  role: Computes the 4096-bit circular Morgan fingerprints used as conditional FP
    tokens for the translation and hybrid-denoising tasks
- name: MolGen
  role: Source of the zinc.npy SELFIES vocabulary (185 tokens) for the unified tokenizer
  repo: https://github.com/zjunlp/MolGen
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:fa9c16b00512c35b0637d6abc1e8befe84265861d4f78b5c6ff66547e6c1ce7a
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
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# conditional-generation-language-model-training

## Summary

Multi-task conditional pretraining of a BART-Base language model on fingerprint–molecule pairs, combining SELFIES denoising, fingerprint-to-molecule translation, and hybrid denoising so the model learns cross-modal mapping between fingerprint tokens and SELFIES sequences.

## When to use

Use when pretraining a sequence-to-sequence generative model on paired structured representations (e.g., 4096-bit Morgan fingerprint tokens and SELFIES strings) to learn cross-modal conditional generation before downstream fine-tuning.

## When NOT to use

- Your training data are experimental mass spectra rather than fingerprint–molecule pairs — the spectra-to-fingerprint mapping requires MIST-predicted fingerprints and a separate fine-tuning/alignment stage, not this pretraining skill
- You only have a small labeled corpus (e.g., 231k MassSpecGym spectra) without access to a large unlabeled molecular corpus (4 million molecules) from which fingerprints can be computed
- The target output is not a SELFIES string (e.g., diffusion-based SAFE generation as in FRIGID uses a masked diffusion language model, not BART conditional generation)

## Inputs

- Fingerprint–molecule pair corpus (e.g., pubchem-4M train/val splits) with 4096-bit circular Morgan fingerprints computed via RDKit
- Unified SELFIES/fingerprint tokenizer (185 SELFIES tokens from the MolGen zinc.npy vocabulary plus FP tokens and <fps_sep>)

## Outputs

- Pretrained BART-Base checkpoint saved under ./logs/pretrained-model
- Per-task training/validation loss curves for the three pretraining objectives

## How to apply

Load a corpus of fingerprint–molecule pairs (e.g., pubchem-4M train/val splits) with a unified tokenizer combining 185 SELFIES tokens from the MolGen zinc.npy vocabulary and fingerprint tokens. Initialize a BART-Base backbone from scratch with a normal-distribution init. Train three conditional-generation tasks jointly: (①) SELFIES denoising — randomly mask 30% of tokens with <Mask> and recover the originals; (②) fingerprint-to-molecule translation — generate the full SELFIES sequence conditioned solely on RDKit-computed 4096-bit Morgan fingerprint tokens; (③) hybrid denoising — concatenate fingerprint tokens and 30%-masked SELFIES via the <fps_sep> separator in both input orders, [FP, <fps_sep>, S_masked] and [S_masked, <fps_sep>, FP], and predict the full SELFIES sequence. Optimize all tasks with the conditional-generation cross-entropy loss at a maximum sequence length of 512, and monitor per-task train/validation loss curves.

## Related tools

- **MS-BART** (Unified pretraining framework whose multi-task objectives (denoising, translation) are trained on the fingerprint–molecule corpus) — https://github.com/OpenDFM/MS-BART
- **BART-Base** (Backbone encoder-decoder model initialized from scratch with a normal distribution and trained with the multi-task conditional-generation objectives) — https://github.com/OpenDFM/MS-BART
- **SELFIES** (String-based molecular representation whose tokens are masked and reconstructed in the denoising and translation tasks)
- **RDKit** (Computes the 4096-bit circular Morgan fingerprints used as conditional FP tokens for the translation and hybrid-denoising tasks)
- **MolGen** (Source of the zinc.npy SELFIES vocabulary (185 tokens) for the unified tokenizer) — https://github.com/zjunlp/MolGen

## Examples

```
python scripts/train.py --task pretrain --corpus data/CANOPUS/pretrain-data --backbone bart-base --tasks denoising,translation,hybrid --mask-rate 0.3 --max-len 512 --output ./logs/pretrained-model
```

## Evaluation signals

- Training converges: per-task training and validation cross-entropy loss curves decrease on both the train and validation splits of pubchem-4M
- Checkpoints are saved under ./logs/pretrained-model after each epoch without NaN losses
- Masked-token recovery accuracy on the 30% SELFIES-denoising task improves monotonically during training
- Fingerprint-conditioned generation produces valid SELFIES strings decodable to molecules (sanity check on a held-out FP-token batch)
- Both hybrid-denoising input orders ([FP, <fps_sep>, S_masked] and [S_masked, <fps_sep>, FP]) yield comparable loss trajectories, confirming order-robust conditioning

## Limitations

- Pretraining relies on fingerprints computed from unlabeled structures; downstream performance on experimental spectra depends on the accuracy of MIST-predicted fingerprints at fine-tuning time
- The pretraining corpus is synthetic (RDKit-computed fingerprints on 4M unlabeled PubChem molecules), so the model must be adapted to dataset-specific noise and biases via fine-tuning on experimental data
- Maximum sequence length of 512 constrains the size of molecules and fingerprint token sequences that can be processed

## Evidence

- [abstract] Multi-task pretraining objectives enhance generalization: "Multi-task pretraining objectives further enhance MS-BART's generalization by jointly"
- [methods] SELFIES denoising task definition: [evidence span withheld — anchor status: not_found]
- [methods] Fingerprint-to-molecule translation task: [evidence span withheld — anchor status: not_found]
- [methods] Hybrid denoising task with separator and both input orders: [evidence span withheld — anchor status: not_found]
- [methods] BART-Base backbone and initialization: [evidence span withheld — anchor status: not_found]
- [intro] Pretraining corpus construction from unlabeled molecules: "We first construct a large-scale pretraining dataset consisting of fingerprint–molecule pairs"
