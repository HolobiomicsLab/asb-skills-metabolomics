---
name: pretraining-dataset-scaling-ablation
description: Use when you must quantify whether scaling a decoder pretraining corpus
  (fingerprint-to-molecule pairs) improves downstream de novo structure generation,
  and you can pretrain/finetune at multiple corpus sizes on a held-out benchmark.
when_to_use_negative:
- You want to ablate the spectrum-encoder pretraining corpus (spectrum-to-fingerprint
  prediction) rather than the fingerprint-to-molecule decoder corpus — this skill
  only varies decoder pretraining size.
- You cannot decontaminate the pretraining pool against the target benchmark (e.g.
  NPLIB1 test/validation molecules must be removed), since leakage invalidates the
  scaling comparison.
- Your task is formula or fingerprint prediction from spectra rather than de novo
  structure generation — use SIRIUS/MIST-CF/MIST formula transformers directly instead.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2258
tools:
- name: DiffMS
  role: Provides the diffusion decoder, fp2mol_main.py pretraining and spec2mol_main.py
    finetuning entry points, and 'fp2mol'/'canopus' dataset configs used at each pretraining
    size
  repo: https://github.com/coleygroup/DiffMS
- name: MIST
  role: Supplies the formula-aware spectrum encoder checkpoints used to initialize
    the end-to-end model before finetuning on NPLIB1
  repo: https://github.com/coleygroup/MIST
- name: PyTorch 2.3.1
  role: Training framework for decoder pretraining and end-to-end finetuning runs
- name: torch_geometric 2.3.1
  role: Graph-based molecular data handling for the diffusion decoder
- name: RDKit 2024.09.4
  role: Validity filtering of sampled molecules (removing invalid/disconnected structures)
    and Tanimoto similarity computation
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:f4da8c0c829e19b0445e6bc3aa22e712eeac05e33acf147c7834187b6b531bfb
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - COCONUT
  - DSSTox
  - DiffMS
  - HMDB
  - MIST
  - MOSES
  - NPLIB1
  - PyTorch
  - PyTorch 2.3.1
  - RDKit 2024.09.4
  - torch_geometric 2.3.1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# Pretraining dataset scaling ablation

## Summary

An ablation protocol that quantifies how the size of the fingerprint-to-molecule pretraining corpus for a diffusion decoder (0, 0.2M, 0.8M, 1.2M, 2.8M pairs) affects downstream spectra-to-molecule generation accuracy after end-to-end finetuning, as done for DiffMS on NPLIB1.

## When to use

Use when you must quantify whether scaling a decoder pretraining corpus (fingerprint-to-molecule pairs) improves downstream de novo structure generation, and you can pretrain/finetune at multiple corpus sizes on a held-out benchmark.

## When NOT to use

- You want to ablate the spectrum-encoder pretraining corpus (spectrum-to-fingerprint prediction) rather than the fingerprint-to-molecule decoder corpus — this skill only varies decoder pretraining size.
- You cannot decontaminate the pretraining pool against the target benchmark (e.g. NPLIB1 test/validation molecules must be removed), since leakage invalidates the scaling comparison.
- Your task is formula or fingerprint prediction from spectra rather than de novo structure generation — use SIRIUS/MIST-CF/MIST formula transformers directly instead.

## Inputs

- Fingerprint-to-molecule pretraining pool ('combined': DSSTox, HMDB, COCONUT, MOSES pairs, decontaminated against NPLIB1 test/validation)
- NPLIB1 spectra-to-molecule finetuning dataset ('canopus', subset of GNPS data used for CANOPUS)
- MIST spectrum encoder checkpoint (Zenodo record 23303289)
- DiffMS decoder/end-to-end checkpoints (Zenodo record 23542112)

## Outputs

- Pretrained diffusion decoders at 0, 0.2M, 0.8M, 1.2M, and 2.8M pair sizes
- Finetuned end-to-end models per pretraining size
- Scaling table of top-1 accuracy, top-10 accuracy, minimum MCES distance, and maximum Tanimoto similarity on NPLIB1

## How to apply

Build the 'combined' fingerprint-molecule pretraining pool from DSSTox, HMDB, COCONUT, and MOSES using the data_processing scripts, removing all NPLIB1 test and validation molecules to prevent leakage. Subsample the pool to 0, 0.2M, 0.8M, 1.2M, and 2.8M fingerprint-molecule pairs and pretrain the diffusion decoder at each size with fp2mol_main.py, setting the dataset in config.yaml to 'fp2mol' so generation is conditioned on molecular fingerprints. For each pretrained decoder, initialize the end-to-end model with the MIST spectrum encoder plus the decoder checkpoint (set general.load_weights in general_default.yaml to the checkpoint path) and finetune on NPLIB1 spectra-to-molecule generation with spec2mol_main.py (dataset 'canopus'). Evaluate each finetuned model by sampling 100 molecules per spectrum, removing invalid or disconnected molecules, and ranking top-k by frequency; report top-1/top-10 accuracy, minimum MCES distance, and maximum Tanimoto similarity to assemble the scaling table. The rationale is that fingerprint-structure pairs are available in virtually unlimited quantities, so scaling this intermediate-representation pretraining stage is the cheapest axis for improving downstream generation.

## Related tools

- **DiffMS** (Provides the diffusion decoder, fp2mol_main.py pretraining and spec2mol_main.py finetuning entry points, and 'fp2mol'/'canopus' dataset configs used at each pretraining size) — https://github.com/coleygroup/DiffMS
- **MIST** (Supplies the formula-aware spectrum encoder checkpoints used to initialize the end-to-end model before finetuning on NPLIB1) — https://github.com/coleygroup/MIST
- **PyTorch 2.3.1** (Training framework for decoder pretraining and end-to-end finetuning runs)
- **torch_geometric 2.3.1** (Graph-based molecular data handling for the diffusion decoder)
- **RDKit 2024.09.4** (Validity filtering of sampled molecules (removing invalid/disconnected structures) and Tanimoto similarity computation)

## Examples

```
bash data_processing/00_download_fp2mol_data.sh; bash data_processing/03_preprocess_fp2mol.sh; python src/fp2mol_main.py  # configs/config.yaml dataset='fp2mol'
```

## Evaluation signals

- The 2.8M-pair variant reaches approximately 8.34% top-1 accuracy and 15.44% top-10 accuracy on NPLIB1, matching the reported DiffMS values.
- Top-1 and top-10 accuracy increase monotonically (or near-monotonically) as pretraining dataset size grows across the 0/0.2M/0.8M/1.2M/2.8M grid.
- The 'combined' pretraining dataset contains no NPLIB1 test or validation molecules, confirming no benchmark leakage.
- Each evaluation samples 100 molecules per spectrum, and invalid or disconnected molecules are removed before top-k frequency ranking.
- Every scaling point uses the same MIST encoder initialization and identical finetuning config on NPLIB1, so only pretraining corpus size varies.

## Limitations

- Exact de novo generation of the precise molecular structure remains challenging; structurally close matches (MCES distance, Tanimoto similarity) are used as secondary signals.
- The scaling result is demonstrated on NPLIB1 (CANOPUS/GNPS subset); behavior on other benchmarks such as MassSpecGym may differ.
- Pretraining data quantity scaling was tested only up to 2.8M fingerprint-structure pairs, so extrapolation beyond that regime is unverified.

## Evidence

- [methods] Evaluation protocol: [evidence span withheld — anchor status: not_found]
