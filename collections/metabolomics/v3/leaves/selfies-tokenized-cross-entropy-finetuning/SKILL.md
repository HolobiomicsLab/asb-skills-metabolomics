---
name: selfies-tokenized-cross-entropy-finetuning
description: Use when adapting a pretrained fingerprint-to-molecule model to noisy
  experimental spectra (e.g. MassSpecGym or NPLIB1/CANOPUS splits) without simulated
  spectra, using predicted fingerprints as input.
when_to_use_negative:
- The task is pretraining on unlabeled fingerprint–molecule pairs rather than transfer
  to experimental spectra — use the multi-task pretraining objectives (SELFIES denoising,
  fingerprint-to-molecule translation, hybrid denoising) instead.
- Data are simulated spectra under diverse conditions — the skill exists specifically
  to avoid requiring simulated spectra.
- The dataset contains mostly non-[M+H]+ adducts that must be retained for training
  — this pipeline filters to [M+H]+ in the training split.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS-Bart
  role: BART-Base-based seq2seq model finetuned with cross-entropy loss on SELFIES
    token targets
  repo: https://github.com/OpenDFM/MS-BART
- name: MIST
  role: Pretrained spectral inference model that predicts 4096-bit fingerprints from
    experimental spectra, conditioned on chemical formula
  repo: https://github.com/coleygroup/FRIGID
- name: SELFIES
  role: String-based molecular representation providing the 185-token target vocabulary
- name: RDKit
  role: Computes fingerprints and Morgan radius-2, 2048-bit Tanimoto similarities
    for validation
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:514b9b528b8858e5ba08bebabeceaab2d752e1962e5748c8e43b5c385df97642
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - BART-Base
  - MIST
  - MS-Bart
  - RDKit
  - SELFIES
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# SELFIES-tokenized cross-entropy finetuning

## Summary

Finetunes the MS-Bart BART-Base model (185-token SELFIES vocabulary) on experimental mass spectra by feeding MIST-predicted 4096-bit fingerprints binarized into <fp{i:04d}> tokens, using cross-entropy loss between target and predicted SELFIES tokens.

## When to use

Use when adapting a pretrained fingerprint-to-molecule model to noisy experimental spectra (e.g. MassSpecGym or NPLIB1/CANOPUS splits) without simulated spectra, using predicted fingerprints as input.

## When NOT to use

- The task is pretraining on unlabeled fingerprint–molecule pairs rather than transfer to experimental spectra — use the multi-task pretraining objectives (SELFIES denoising, fingerprint-to-molecule translation, hybrid denoising) instead.
- Data are simulated spectra under diverse conditions — the skill exists specifically to avoid requiring simulated spectra.
- The dataset contains mostly non-[M+H]+ adducts that must be retained for training — this pipeline filters to [M+H]+ in the training split.

## Inputs

- MIST-predicted fingerprint probability vectors {p0,...,p4095} conditioned on molecular formula and metadata
- Fingerprint-SELFIES table for the experimental training split ([M+H]+ adducts only)
- Pretrained MS-Bart checkpoint from ./logs/pretrained-model (BART-Base, 185-token SELFIES vocabulary)

## Outputs

- Fine-tuned MS-Bart checkpoint saved under ./logs/<run-name>
- Validation Top-1/Top-10 accuracy, Top-1 Tanimoto similarity (Morgan radius 2, 2048 bits, RDKit), and MCES distance curves

## How to apply

Load MIST probability vectors {p0,...,p4095} predicted from experimental spectra conditioned on chemical formula, binarize into 4096-bit fingerprints with epsilon=0.2 (NPLIB1/CANOPUS) or epsilon=0.11 (MassSpecGym), and convert each active bit into an <fp{i:04d}> token. Split the fingerprint-SELFIES table into train/val/test with preprocess/prepare_test_data.py, keeping only [M+H]+ adducts in the training split while retaining all adducts in val/test. Load the MS-Bart checkpoint from ./logs/pretrained-model (BART-Base backbone, 185-token SELFIES vocabulary, input and output token lengths fixed at 256) and finetune with the cross-entropy loss of Eq. 2 between target and predicted SELFIES tokens. Run bash scripts/msg/finetune.sh (MassSpecGym) or bash scripts/canopus/finetune.sh (NPLIB1/CANOPUS), saving the result to ./logs/<run-name>.

## Related tools

- **MS-Bart** (BART-Base-based seq2seq model finetuned with cross-entropy loss on SELFIES token targets) — https://github.com/OpenDFM/MS-BART
- **MIST** (Pretrained spectral inference model that predicts 4096-bit fingerprints from experimental spectra, conditioned on chemical formula) — https://github.com/coleygroup/FRIGID
- **SELFIES** (String-based molecular representation providing the 185-token target vocabulary)
- **RDKit** (Computes fingerprints and Morgan radius-2, 2048-bit Tanimoto similarities for validation)

## Examples

```
bash scripts/msg/finetune.sh
```

## Evaluation signals

- Fine-tuned checkpoint exists under ./logs/<run-name> after running scripts/msg/finetune.sh or scripts/canopus/finetune.sh
- Recorded validation Top-1 accuracy/Tanimoto curve improves over the Pretrain stage (near-zero Top-1 accuracy of 0.00% on MassSpecGym versus 1.07% after fine-tuning)
- Binarization thresholds applied match the split: epsilon=0.2 for NPLIB1/CANOPUS and epsilon=0.11 for MassSpecGym
- Training split contains only [M+H]+ adducts; val/test retain all adducts
- Input and output token lengths are fixed at 256 during finetuning

## Limitations

- Dependence on external MIST model for fingerprint prediction; its accuracy significantly impacts overall performance
- Formula information is used only for re-ranking and not for training the model
- No software/tool versions are reported, so exact reproduction may be uncertain

## Evidence

- [methods] MIST fingerprint prediction: [evidence span withheld — anchor status: not_found]
- [methods] BART-Base backbone: [evidence span withheld — anchor status: not_found]
- [methods] [M+H]+ adduct filtering: [evidence span withheld — anchor status: not_found]
