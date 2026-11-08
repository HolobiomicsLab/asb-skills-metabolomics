---
name: torch-geometric-model-finetuning
description: Use when a pretrained PyTorch Geometric graph model must be adapted to
  a new dataset (e.g. NPLIB1/CANOPUS or MassSpecGym spectra) and pretrained checkpoints
  exist to initialize from.
when_to_use_negative:
- You are training a graph model from scratch on a generic graph dataset with no pretrained
  checkpoint — use the upstream DiGress main.py workflow instead
- The task is fingerprint prediction from spectra rather than molecular structure
  generation — use the MIST encoder pretraining path (coleygroup/MIST) instead of
  end-to-end fine-tuning
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_3474
tools:
- name: DiffMS
  role: Codebase being fine-tuned; encoder-decoder generative network adapted from
    DiGress
  repo: https://github.com/coleygroup/DiffMS
- name: torch_geometric 2.3.1
  role: Graph neural network framework underlying the discrete graph diffusion decoder
- name: PyTorch 2.3.1
  role: Deep learning framework (tested with cuda 11.8) for training/finetuning
- name: MIST
  role: Formula transformer spectrum encoder whose pretrained checkpoints initialize
    the encoder before fine-tuning
  repo: https://github.com/coleygroup/MIST
- name: RDKit 2024.09.4
  role: Installed in the conda environment for molecule validity/connectedness checks
    on generated graphs
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:131c389b891d21fbd915e1be48f9ac74b9bbdceb23bf6d0ce2b9bd2c6b4a74fc
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - DiffMS
  - MIST
  - PyTorch
  - PyTorch 2.3.1
  - RDKit 2024.09.4
  - SIRIUS
  - torch_geometric
  - torch_geometric 2.3.1
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# torch-geometric-model-finetuning

## Summary

Fine-tune a pretrained graph-based generative model (DiffMS's discrete graph diffusion decoder, built on the DiGress/PyTorch Geometric stack) end-to-end on domain-specific spectra-molecule pairs, after a fingerprint-molecule pretraining stage, by adjusting config.yaml dataset settings and loading pretrained checkpoint weights.

## When to use

Use when a pretrained PyTorch Geometric graph model must be adapted to a new dataset (e.g. NPLIB1/CANOPUS or MassSpecGym spectra) and pretrained checkpoints exist to initialize from.

## When NOT to use

- You are training a graph model from scratch on a generic graph dataset with no pretrained checkpoint — use the upstream DiGress main.py workflow instead
- The task is fingerprint prediction from spectra rather than molecular structure generation — use the MIST encoder pretraining path (coleygroup/MIST) instead of end-to-end fine-tuning

## Inputs

- Spectra-molecule pair dataset (NPLIB1/CANOPUS or MassSpecGym) prepared by data_processing/ scripts 00-03
- Fingerprint-molecule pretraining dataset ('combined': DSSTox, HMDB, COCONUT, MOSES; 2.8M pairs)
- Pretrained DiffMS decoder/end-to-end checkpoints (Zenodo 23542112) and MIST encoder checkpoints (Zenodo 23303289)
- Hydra config files: configs/config.yaml, configs/dataset/fp2mol.yaml, configs/general/general_default.yaml

## Outputs

- Fine-tuned end-to-end DiffMS model checkpoints (spectrum encoder + graph diffusion decoder)
- Generated molecules for evaluation: 100 sampled molecules per spectrum, filtered to valid, connected molecules, ranked by frequency into top-k

## How to apply

First pretrain the diffusion decoder on fingerprint-molecule pairs (DSSTox, HMDB, COCONUT, MOSES; 2.8M pairs) via fp2mol_main.py with the dataset in configs/config.yaml set to 'fp2mol' and 'combined' as the pretraining dataset in fp2mol.yaml. Then fine-tune the end-to-end model on spectra-molecule pairs with src/spec2mol_main.py, setting the dataset in config.yaml to 'canopus' for NPLIB1 or 'msg' for MassSpecGym, and pointing `general.load_weights` in configs/general/general_default.yaml to the pretrained decoder/encoder checkpoint from Zenodo (records 23542112 for DiffMS, 23303289 for MIST encoder). Rationale: pretraining bridges latent fingerprint embeddings and molecular structures so downstream performance scales with pretraining dataset size. The stack tested with PyTorch 2.3.1, cuda 11.8, and torch_geometric 2.3.1, installed in a conda env with rdkit=2024.09.4 and `pip install -e .`.

## Related tools

- **DiffMS** (Codebase being fine-tuned; encoder-decoder generative network adapted from DiGress) — https://github.com/coleygroup/DiffMS
- **torch_geometric 2.3.1** (Graph neural network framework underlying the discrete graph diffusion decoder)
- **PyTorch 2.3.1** (Deep learning framework (tested with cuda 11.8) for training/finetuning)
- **MIST** (Formula transformer spectrum encoder whose pretrained checkpoints initialize the encoder before fine-tuning) — https://github.com/coleygroup/MIST
- **RDKit 2024.09.4** (Installed in the conda environment for molecule validity/connectedness checks on generated graphs)

## Examples

```
conda activate diffms; set dataset='canopus' in configs/config.yaml and general.load_weights to the Zenodo decoder checkpoint, then python src/spec2mol_main.py
```

## Evaluation signals

- Fine-tuned model, when sampled 100 times per spectrum, yields valid connected molecules whose top-1/top-10 accuracy, MCES distance, and Tanimoto similarity match or improve over the pretrained-only baseline
- Downstream performance should scale with fingerprint-structure pretraining dataset size (scaling ablation behavior reported in the paper)
- config.yaml dataset key resolves to 'canopus' or 'msg' without path errors and the checkpoint at general.load_weights loads without key mismatches
- Environment versions match tested stack: PyTorch 2.3.1, torch_geometric 2.3.1, rdkit 2024.09.4, cuda 11.8

## Limitations

- Fine-tuning is designed for the two open-source de novo benchmarks NPLIB1 and MassSpecGym; other datasets require writing a new dataset config
- Exact molecular structure from de novo generation remains challenging — fine-tuned outputs are structurally close matches, not guaranteed exact structures
- Graph-tool and torch_geometric can conflict on MacOS (noted in the upstream DiGress README)

## Evidence

- [methods] Sampling 100 molecules per spectrum and filtering: [evidence span withheld — anchor status: not_found]
