---
name: language-model-finetuning-and-alignment-checkpoint-inference
description: Decision support for running the finetuning and alignment stages of MS-BART
  (scripts/msg/finetune.sh, align.sh) and generating final molecule predictions at
  the aligned checkpoint for benchmark evaluation.
when_to_use_negative:
- Input spectra contain only [M+Na]+ adducts or other non-[M+H]+ precursor types —
  the paper explicitly filters these out before finetuning and alignment
- No pretrained MS-BART checkpoint is available and only the raw BART-Base backbone
  exists — finetuning without the multi-task pretraining stage will not match the
  reported SOTA numbers
- The goal is de novo generation without any fingerprint intermediate — use a direct
  spectrum-to-structure model instead; MS-BART depends on MIST fingerprint accuracy
  and degrades if predictions are poor
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0091
tools:
- name: MS-BART
  role: The finetuned/aligned language model whose checkpoint is loaded for beam-search
    molecule generation
  repo: https://github.com/OpenDFM/MS-BART
- name: MIST
  role: Pretrained spectral inference model that produces the 4096-bit Morgan fingerprint
    predictions used as finetuning input
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, 2048 bits) used both for the alignment
    Tanimoto-similarity rank loss and for evaluating Top-1/Top-10 accuracy, MCES distance,
    and Tanimoto similarity of generated structures
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:03331687cdda48a1c2ac04f20f9a3c3deed4f8796f3dae4b993614a85be981e0
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - BART-Base
  - MIST
  - MS-BART
  - MassSpecGym
  - NPLIB1
  - RDKit
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# language-model-finetuning-and-alignment-checkpoint-inference

## Summary

End-to-end pipeline for MS-BART that finetunes the BART-Base backbone on MIST-predicted 4096-bit Morgan fingerprints from MassSpecGym/NPLIB1 spectra, applies Tanimoto-similarity rank-loss chemical alignment, and runs beam-search inference at the aligned checkpoint to generate candidate molecular SELFIES structures.

## When to use

Decision support for running the finetuning and alignment stages of MS-BART (scripts/msg/finetune.sh, align.sh) and generating final molecule predictions at the aligned checkpoint for benchmark evaluation.

## When NOT to use

- Input spectra contain only [M+Na]+ adducts or other non-[M+H]+ precursor types — the paper explicitly filters these out before finetuning and alignment
- No pretrained MS-BART checkpoint is available and only the raw BART-Base backbone exists — finetuning without the multi-task pretraining stage will not match the reported SOTA numbers
- The goal is de novo generation without any fingerprint intermediate — use a direct spectrum-to-structure model instead; MS-BART depends on MIST fingerprint accuracy and degrades if predictions are poor

## Inputs

- Pretrained MS-BART checkpoint (BART-Base backbone, from scripts/pretrain.sh output ./logs/pretrained-model)
- MIST-predicted 4096-bit Morgan fingerprint tables for MassSpecGym and NPLIB1/CANOPUS train/val/test splits (produced by preprocess/fp_pred_main.py)
- MGF-format spectra files with associated chemical formula and metadata labels (generate_mgf_and_lables.py / generate_canopus_and_lables.py output)
- SELFIES-encoded ground-truth molecular structures for train/val/test folds

## Outputs

- Finetuned MS-BART checkpoint adapted to experimental spectra fingerprint inputs
- Aligned MS-BART checkpoint (data/MassSpecGym/model-weights or data/CANOPUS/model-weights) after chemical-feedback alignment
- Beam-search generated SELFIES candidate sequences per test spectrum (beam widths up to 100)
- Top-1/Top-10 accuracy (full InChIKey exact match), minimum MCES distance, and maximum Tanimoto similarity metrics computed with RDKit Morgan fingerprints (radius 2, 2048 bits)

## How to apply

Start from the pretrained MS-BART checkpoint saved under ./logs/pretrained-model after multi-task pretraining (SELFIES denoising, fingerprint-to-molecule translation, hybrid denoising). For each benchmark, run MIST (mist_msg weights for MassSpecGym, mist_canopus for NPLIB1/CANOPUS, batch-size 32) to produce 4096-bit Morgan fingerprint predictions conditioned on the chemical formula, filter out [M+Na]+ adducts so only [M+H]+ spectra are retained, then execute scripts/msg/finetune.sh (MassSpecGym) or scripts/canopus/finetune.sh (NPLIB1) to finetune on those predicted fingerprints. Next, run the chemical-feedback alignment step via scripts/msg/align.sh or scripts/canopus/align.sh, which applies a contrastive Tanimoto-similarity rank loss (Eqs. 4-5, margin gamma scaled by rank difference) to assign higher probabilities to generated structures closer to the ground-truth fingerprint/molecule. At inference, load the aligned model checkpoint from data/MassSpecGym/model-weights (or data/CANOPUS/model-weights) and run beam-search multinomial sampling with beam widths up to 100 (temperature ablated in Appendix D), producing SELFIES sequences that are decoded for Top-1/Top-10 accuracy, MCES distance, and Tanimoto similarity evaluation with RDKit (radius 2, 2048-bit Morgan fingerprints). Correct the known upstream eval-script bugs (test-path filename and NUM_BEAMS/NUM_BEAM mismatch) before scoring.

## Related tools

- **MS-BART** (The finetuned/aligned language model whose checkpoint is loaded for beam-search molecule generation) — https://github.com/OpenDFM/MS-BART
- **MIST** (Pretrained spectral inference model that produces the 4096-bit Morgan fingerprint predictions used as finetuning input)
- **RDKit** (Computes Morgan fingerprints (radius 2, 2048 bits) used both for the alignment Tanimoto-similarity rank loss and for evaluating Top-1/Top-10 accuracy, MCES distance, and Tanimoto similarity of generated structures)

## Examples

```
bash scripts/msg/finetune.sh && bash scripts/msg/align.sh && python eval.py --checkpoint data/MassSpecGym/model-weights --num_beams 100
```

## Evaluation signals

- Reproduced NPLIB1 Top-1 accuracy reaches 7.45%, MCES 9.66, Tanimoto 0.44 (MS-BART row in the benchmark table)
- Reproduced MassSpecGym Top-1 accuracy reaches 1.07%, MCES 16.47, Tanimoto 0.28 (MS-BART row)
- Gold-fingerprint upper bounds: 73.50% Top-1 accuracy on NPLIB1 and 47.56% on MassSpecGym — the aligned model should stay well below these but above diffusion baselines like DiffMS (8.34% NPLIB1 Top-1)
- Alignment margin gamma scales with rank difference per Eqs. 4-5; verify the contrastive Tanimoto rank loss decreases during alignment training
- Only [M+H]+ spectra appear in the finetuning and alignment training set after adduct filtering

## Limitations

- Fingerprint prediction accuracy from MIST is the bottleneck — the article states 'The accuracy of these fingerprint predictions significantly impacts the overall performance.'
- Chemical formula information is used only for re-ranking (Eq. 6, log-probability tie-breaking) and not for training the model
- Known bugs in the upstream MassSpecGym eval script (test-path filename and NUM_BEAMS/NUM_BEAM variable mismatch) must be corrected before metrics are trustworthy
- The [M+Na]+ adduct subset is discarded, so performance on sodium-adduct spectra is unmeasured

## Evidence

- [methods] MIST conditions on chemical formula: [evidence span withheld — anchor status: not_found]
- [methods] [M+Na]+ adduct filtering: [evidence span withheld — anchor status: not_found]
- [readme] README repo availability: [evidence span withheld — anchor status: not_found]
