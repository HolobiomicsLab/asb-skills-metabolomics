---
name: gpu-memory-management-for-large-model-training
description: Use when instruction-tuning a large multimodal LLM (e.g., 13B-parameter
  backbone plus encoder projection layers) and you must size GPU memory, batch size,
  and iteration counts before launching training.
when_to_use_negative:
- GPU has less than 70 GB VRAM (e.g., consumer 24–48 GB cards) — training will not
  fit and the released 13B-backbone pipeline cannot be run as documented
- Goal is inference/evaluation only with the provided checkpoint — no training memory
  budgeting or iters_per_epoch tuning is needed
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: torchrun
  role: Launches single-process distributed training (torchrun --nproc_per_node 1
    train.py) that trains the projection layer within the single-GPU memory budget
- name: nvidia-smi
  role: Used to check GPU memory availability (at least 70 GB required) before training
- name: conda
  role: Creates the metabolitechat environment (Python 3.9+ via Miniconda) in which
    training runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.07.687008
    title: 'MetaboliteChat: A Unified Multimodal Large Language Model for Interactive
      Metabolite Analysis and Functional Insights'
  source_content_hash: sha256:b5efa175c6b54ef339b08518547a39ea26708c06c12ac1970f313b556d7d764e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/daniel-guooo/MetaboliteChat
  tools:
  - MetaboliteChat
  - conda
  - nvidia-smi
  - pip
  - torchrun
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4642857142857143
n_claims: 28
---

# gpu-memory-management-for-large-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Provisioning GPU resources and structuring training so a multimodal LLM (MetaboliteChat on Vicuna-13B-v1.5 with a GNN encoder) can be trained within a single GPU's memory budget, verified on an A100 80 GB card.

## When to use

Use when instruction-tuning a large multimodal LLM (e.g., 13B-parameter backbone plus encoder projection layers) and you must size GPU memory, batch size, and iteration counts before launching training.

## When NOT to use

- GPU has less than 70 GB VRAM (e.g., consumer 24–48 GB cards) — training will not fit and the released 13B-backbone pipeline cannot be run as documented
- Goal is inference/evaluation only with the provided checkpoint — no training memory budgeting or iters_per_epoch tuning is needed

## Inputs

- train_configs/metabolitechat.yaml training configuration file
- Pretrained Vicuna-13B-v1.5 weights from Hugging Face
- Pretrained GNN and CNN encoder checkpoints (Google Drive)
- HMDB-derived instruction tuning JSON files ({SMILES String: [[Question, Answer], ...]}, 152,222 metabolites)

## Outputs

- Trained MetaboliteChat checkpoint comparable to the released checkpoint trained on 152,222 metabolites
- Trained projection layer connecting the GNN output to the LLaMA model

## How to apply

Before training, check the GPU with nvidia-smi and confirm at least 70 GB of VRAM is available, since MetaboliteChat training was verified on an A100 80 GB. Edit train_configs/metabolitechat.yaml to set hyperparameters (max_epoch, init_lr, min_lr, warmup_steps, batch_size_train) and adjust iters_per_epoch so that iters_per_epoch * batch_size_train equals the training set size — with the released dataset this is 152,222 metabolites. Launch single-process distributed training with torchrun --nproc_per_node 1 so the run fits on one large-memory GPU while training the projection layer connecting the GNN output to the LLaMA model. If memory is insufficient, the run will fail, so verifying the hardware budget and the iters/batch arithmetic before launch is the core of this skill.

## Related tools

- **torchrun** (Launches single-process distributed training (torchrun --nproc_per_node 1 train.py) that trains the projection layer within the single-GPU memory budget)
- **nvidia-smi** (Used to check GPU memory availability (at least 70 GB required) before training)
- **conda** (Creates the metabolitechat environment (Python 3.9+ via Miniconda) in which training runs)

## Examples

```
torchrun --nproc_per_node 1 train.py --cfg-path train_configs/metabolitechat.yaml
```

## Evaluation signals

- nvidia-smi reports ≥70 GB free GPU memory before launch (verified configuration: A100 80 GB)
- iters_per_epoch * batch_size_train equals the training set size of 152,222 metabolites
- Training completes under torchrun --nproc_per_node 1 with the metabolitechat.yaml config without out-of-memory errors
- Resulting checkpoint's scale and training coverage match the released checkpoint trained on 152,222 metabolites

## Limitations

- Training requires at least 70 GB GPU memory (verified only on A100 80 GB); smaller GPUs are not supported by the documented pipeline
- Training is configured as single-process distributed (--nproc_per_node 1); multi-GPU scaling is not documented
- Memory requirements scale with the Vicuna-13B-v1.5 backbone; swapping in a different LLM would invalidate the documented VRAM budget

## Evidence

- [intro] at least 70 GB VRAM requirement: "**You need at least 70 GB GPU memory for the training.**"
- [other] iters_per_epoch arithmetic: [evidence span withheld — anchor status: not_found]
- [intro] torchrun launch command: "Start training the projection layer that connects the GNN output and the LLaMA model by running: torchrun --nproc_per_node 1 train.py --cfg-path train_configs/metabolitechat.yaml"
- [intro] checkpoint scale: "We provide a checkpoint [here](https://drive.google.com/drive/folders/18KRWlZddQh3wfwPr9bzKB-ieQ8X9Pdkd?usp=sharing) by training on 152,222 metabolites."
