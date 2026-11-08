---
name: molecular-fingerprint-tokenization
description: Use when pretraining or fine-tuning a sequence model over fingerprint–molecule
  pairs and you need Morgan fingerprint bits (from RDKit or MIST predictions) tokenized
  into a unified vocabulary with SELFIES strings.
when_to_use_negative:
- Input mass spectra contain only [M+Na]+ adducts, which are filtered out before fine-tuning
  and alignment (only [M+H]+ is retained)
- The data is already a tokenized fingerprint/SELFIES training corpus — re-tokenization
  would duplicate the preprocessing step
- Access is restricted (e.g. NIST23, commercial licensing) and fingerprints cannot
  be computed or predicted from the underlying structures
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Computes 4096-bit Morgan fingerprints from unlabeled molecular structures
    to build the fingerprint–molecule pretraining corpus
- name: MIST
  role: Predicts molecular fingerprints FP from mass spectra conditioned on the chemical
    formula, providing the spectral-side tokenization signal
  repo: https://github.com/coleygroup/MIST
- name: MS-BART
  role: 'Framework consuming the tokenized pairs: unified vocabulary and multi-task
    pretraining on BART-Base'
  repo: https://github.com/OpenDFM/MS-BART
- name: MolGen
  role: Source of the SELFIES tokenizer vocabulary (zinc.npy, 185 SELFIES tokens)
  repo: https://github.com/zjunlp/MolGen
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:5cea2def49e36f1e5b37a623419a74830eecc3385961179931c7fae461939530
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/MIST
  tools:
  - BART-Base
  - MIST
  - MS-BART
  - MolGen
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# molecular-fingerprint-tokenization

## Summary

Tokenizes molecules and mass spectra into a shared vocabulary of SELFIES structural tokens and 4096-bit Morgan fingerprint tokens, so fingerprint–molecule pairs can drive multi-task pretraining of a BART-Base seq2seq model (MS-BART) for structure elucidation.

## When to use

Use when pretraining or fine-tuning a sequence model over fingerprint–molecule pairs and you need Morgan fingerprint bits (from RDKit or MIST predictions) tokenized into a unified vocabulary with SELFIES strings.

## When NOT to use

- Input mass spectra contain only [M+Na]+ adducts, which are filtered out before fine-tuning and alignment (only [M+H]+ is retained)
- The data is already a tokenized fingerprint/SELFIES training corpus — re-tokenization would duplicate the preprocessing step
- Access is restricted (e.g. NIST23, commercial licensing) and fingerprints cannot be computed or predicted from the underlying structures

## Inputs

- Unlabeled molecular structures (pubchem-4M corpus) as SMILES/SELFIES strings
- Tandem mass spectra with metadata and molecular formulas (MassSpecGym, NPLIB1/GNPS subsets)
- RDKit-computed 4096-bit Morgan fingerprints of molecules
- MIST-predicted fingerprints from spectra (e.g. mist_msg.pt / mist_canopus.pt checkpoints)
- Unified SELFIES/fingerprint tokenizer vocabulary (MolGen zinc.npy)

## Outputs

- Tokenized fingerprint–molecule training pairs (fingerprint token sequence FP + full SELFIES sequence)
- 30%-masked SELFIES sequences for the denoising task
- Hybrid inputs [FP, <fps_sep>, S_masked] and [S_masked, <fps_sep>, FP]
- Pretrained BART-Base checkpoints saved under ./logs/pretrained-model

## How to apply

Compute a 4096-bit radius-2 circular Morgan fingerprint for each molecule with RDKit, or predict it from a mass spectrum with MIST conditioned on the chemical formula. Convert fingerprints to token sequences by embedding the active bit indices, and tokenize molecular structures as SELFIES using the unified tokenizer (185 SELFIES tokens taken from the MolGen zinc.npy vocabulary). For hybrid inputs, concatenate fingerprint tokens and 30%-masked SELFIES with the <fps_sep> separator in both orders, [FP, <fps_sep>, S_masked] and [S_masked, <fps_sep>, FP], capping sequences at length 512. Feed the tokenized pairs to a BART-Base backbone (parameters initialized from scratch with a normal distribution) for the denoising, translation, and hybrid-denoising pretraining tasks. Before finetuning on experimental data, restrict to the dominant [M+H]+ adduct and optionally filter pretrain pairs by Tanimoto similarity > 0.5 (the 'clean' 4M pretrain set in the released data).

## Related tools

- **RDKit** (Computes 4096-bit Morgan fingerprints from unlabeled molecular structures to build the fingerprint–molecule pretraining corpus)
- **MIST** (Predicts molecular fingerprints FP from mass spectra conditioned on the chemical formula, providing the spectral-side tokenization signal) — https://github.com/coleygroup/MIST
- **MS-BART** (Framework consuming the tokenized pairs: unified vocabulary and multi-task pretraining on BART-Base) — https://github.com/OpenDFM/MS-BART
- **MolGen** (Source of the SELFIES tokenizer vocabulary (zinc.npy, 185 SELFIES tokens)) — https://github.com/zjunlp/MolGen

## Examples

```
from rdkit.Chem import AllChem; fp = AllChem.GetMorganFingerprintAsBitVect(mol, radius=2, nBits=4096)
```

## Evaluation signals

- Every fingerprint tensor has exactly 4096 binary dimensions (FP ∈ {0,1}^4096); tokenizer output uses only the 185 SELFIES tokens from the MolGen zinc.npy vocabulary
- Hybrid-denoising inputs contain exactly one <fps_sep> separator token and both order variants are represented in the batch
- Masked SELFIES sequences have ~30% of tokens replaced by the Mask token, and the model recovers the original tokens under cross-entropy loss
- Train and validation loss curves decrease on the pubchem-4M train/val splits and checkpoints appear under ./logs/pretrained-model
- Maximum sequence length does not exceed 512 tokens across all three pretraining tasks

## Limitations

- Accuracy of MIST fingerprint predictions from spectra significantly impacts overall downstream performance
- Formula information is used only for re-ranking, not for training the tokenized model
- Pretrain data must be filtered (e.g. MCES distance < 2 from test-fold molecules excluded; clean variant uses Tanimoto similarity > 0.5) to avoid leakage

## Evidence

- [methods] Tokenize spectra as Morgan fingerprints via MIST: [evidence span withheld — anchor status: not_found]
- [methods] SELFIES masking for denoising task: [evidence span withheld — anchor status: not_found]
- [methods] Backbone initialization for the tokenized pretraining: [evidence span withheld — anchor status: not_found]
- [readme] Fingerprint encoder used for spectral tokenization: [evidence span withheld — anchor status: external_doc]
