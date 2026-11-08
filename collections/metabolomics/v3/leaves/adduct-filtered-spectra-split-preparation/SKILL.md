---
name: adduct-filtered-spectra-split-preparation
description: Use when converting MIST-predicted fingerprint probabilities plus SELFIES
  targets into train/val/test splits for fine-tuning a spectrum-to-structure model,
  and the training split must be restricted to a dominant adduct type (e.g. [M+H]+).
when_to_use_negative:
- The input is already a curated split of fingerprint-SELFIES pairs — re-running thresholding
  or adduct filtering would alter the benchmark's official val/test composition.
- The downstream task requires spectra from adducts other than [M+H]+ in training,
  since this skill explicitly filters out [M+Na]+ and other adducts from the training
  split.
- You are working with simulated or synthetic spectra rather than experimental spectra,
  where the MIST-predicted-fingerprint bridging step is unnecessary.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MIST
  role: Predicts fingerprint probability vectors from experimental spectra conditioned
    on formula/metadata; outputs are binarized and tokenized during split preparation
  repo: github.com/coleygroup/MIST
- name: MS-Bart
  role: Downstream consumer of the prepared splits; fine-tuned on the [M+H]+-only
    training split with 256-token input/output length
  repo: github.com/OpenDFM/MS-BART
- name: SELFIES
  role: Target molecular string representation paired with fingerprint tokens in the
    split files (185-token SELFIES vocabulary)
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:38611f68ff9d3bac8d6e75e2b7d5bd0f1f8e015202e0794f46ed164a3b3c355c
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MIST
  - MS-Bart
  - RDKit
  - SELFIES
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# adduct-filtered-spectra-split-preparation

## Summary

Prepares experimental mass-spectrometry data into train/val/test splits of fingerprint-SELFIES pairs, applying MIST probability thresholding (epsilon=0.2 for NPLIB1/CANOPUS, epsilon=0.11 for MassSpecGym) and adduct filtering so that only [M+H]+ spectra are used for fine-tuning while val/test retain all adduct types.

## When to use

Use when converting MIST-predicted fingerprint probabilities plus SELFIES targets into train/val/test splits for fine-tuning a spectrum-to-structure model, and the training split must be restricted to a dominant adduct type (e.g. [M+H]+).

## When NOT to use

- The input is already a curated split of fingerprint-SELFIES pairs — re-running thresholding or adduct filtering would alter the benchmark's official val/test composition.
- The downstream task requires spectra from adducts other than [M+H]+ in training, since this skill explicitly filters out [M+Na]+ and other adducts from the training split.
- You are working with simulated or synthetic spectra rather than experimental spectra, where the MIST-predicted-fingerprint bridging step is unnecessary.

## Inputs

- MIST probability vectors {p0,...,p4095} predicted from experimental spectra (conditioned on chemical formula and metadata)
- Processed fingerprint-SELFIES table pairing 4096-bit fingerprint tokens with SELFIES target sequences
- Experimental spectra metadata including adduct type ([M+H]+, [M+Na]+) and precursor chemical formula
- MassSpecGym / NPLIB1 (CANOPUS) benchmark spectrum files

## Outputs

- train/val/test split files under data/MassSpecGym/{train,test,val} and data/CANOPUS/{train,test,val}
- Binarized 4096-bit fingerprints tokenized as <fp{i:04d}> tokens
- Training split containing only [M+H]+ adducts; val/test splits retaining all adducts

## How to apply

Load MIST probability vectors {p0,...,p4095} predicted from experimental spectra conditioned on the chemical formula and metadata, binarize them into 4096-bit fingerprints with threshold epsilon=0.2 for NPLIB1/CANOPUS or epsilon=0.11 for MassSpecGym, and convert each active bit into an <fp{i:04d}> token. Build the processed fingerprint-SELFIES table and run preprocess/prepare_test_data.py to generate train/val/test split files (threshold 0.2 for the MSG/CANOPUS split files), placing them under data/MassSpecGym or data/CANOPUS in train/test/val directories. In the training split, filter out [M+Na]+ adducts and retain only the dominant [M+H]+ data, because adduct variability is a major source of spectral heterogeneity; deliberately keep all adducts in val/test so evaluation reflects real experimental conditions. The resulting token sequences (185-token SELFIES vocabulary plus fingerprint tokens) are then consumed by MS-Bart fine-tuning with input and output token lengths fixed at 256 via bash scripts/msg/finetune.sh or bash scripts/canopus/finetune.sh.

## Related tools

- **MIST** (Predicts fingerprint probability vectors from experimental spectra conditioned on formula/metadata; outputs are binarized and tokenized during split preparation) — github.com/coleygroup/MIST
- **MS-Bart** (Downstream consumer of the prepared splits; fine-tuned on the [M+H]+-only training split with 256-token input/output length) — github.com/OpenDFM/MS-BART
- **SELFIES** (Target molecular string representation paired with fingerprint tokens in the split files (185-token SELFIES vocabulary))

## Examples

```
bash scripts/msg/finetune.sh
```

## Evaluation signals

- Training split contains exclusively [M+H]+ adducts while val/test splits retain all adduct types.
- Binarized fingerprints contain exactly 4096 bits with active bits encoded as <fp{i:04d}> tokens, using epsilon=0.2 (NPLIB1/CANOPUS) or epsilon=0.11 (MassSpecGym).
- Fine-tuned checkpoint exists under ./logs/<run-name> after running scripts/msg/finetune.sh or scripts/canopus/finetune.sh.
- Validation Top-1 accuracy curve improves over the Pretrain stage (near-zero Top-1 of 0.00% on MassSpecGym versus 1.07% after fine-tuning).
- Sequence lengths stay within the fixed 256-token input/output budget using the 185-token SELFIES vocabulary.

## Limitations

- Accuracy of the MIST fingerprint predictions significantly impacts overall downstream performance; fine-tuning MIST on task-specific datasets is suggested as future work.
- Filtering out [M+Na]+ adducts discards non-dominant adduct spectra from training, which may limit generalization to other ionization conditions.
- The chemical formula is used for conditioning/re-ranking but not for model training, a limitation the authors defer to future work.

## Evidence

- [methods] we filter out [M+Na]+ adducts before fine-tuning and alignment, retaining only the dominant [M+H]+ data.: [evidence span withheld — anchor status: not_found]
- [methods] we employ the pretrained MIST model [18] to predict molecular fingerprints FP from mass spectra, conditioned on the chemical formula.: [evidence span withheld — anchor status: not_found]
- [readme] Download the preprocessed datasets and original model weights from [Figshare](https://figshare.com/articles/dataset/MS-BART-Model-Weights-Data/30393544) and place them in `data/`.: "Download the preprocessed datasets and original model weights from [Figshare](https://figshare.com/articles/dataset/MS-BART-Model-Weights-Data/30393544) and place them in `data/`."
- [intro] Randomly selected mass spectra of a molecule ... from MassSpecGym [7], acquired under varying collision energies: [evidence span withheld — anchor status: not_found]
