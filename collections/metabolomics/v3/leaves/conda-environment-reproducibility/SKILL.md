---
name: conda-environment-reproducibility
description: Use when reproducing the MetaboliteChat checkpoint evaluation or training
  pipeline and a Python 3.9+ environment matching the authors' environment.yml must
  be instantiated before running demo.sh or train.py.
when_to_use_negative:
- You only need to query GPT-4o or another generic LLM baseline via API — no local
  conda environment from environment.yml is required.
- The target machine has no NVIDIA GPU and the goal is training (needs >=70 GB GPU
  memory, verified on A100 80 GB); inference-only or CPU-only inspection should be
  considered instead.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: conda
  role: Create and activate the pinned metabolitechat environment from environment.yml
    and the separate rdkit environment from conda-forge
- name: Miniconda
  role: Provides the Conda distribution used to provision the Python 3.9+ environment
- name: pip
  role: Installs Python dependencies inside the environment (e.g. pip install numpy)
- name: numpy
  role: Pinned dependency installed via pip inside the metabolitechat environment
- name: RDKit
  role: Runs in an isolated rdkit environment to convert SMILES to graph/image inputs
    via smiles2graph_image_demo.py
- name: nvidia-smi
  role: Verifies GPU memory availability (>=70 GB) before training
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.07.687008
    title: 'MetaboliteChat: A Unified Multimodal Large Language Model for Interactive
      Metabolite Analysis and Functional Insights'
  source_content_hash: sha256:33404222540b627cd4c44f1c20e7148774d7f1a6b1bebc2e17c02237d3406d42
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
  - Miniconda
  - RDKit
  - conda
  - numpy
  - nvidia-smi
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 26
---

# conda-environment-reproducibility

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Set up a pinned, reproducible Python environment for MetaboliteChat by cloning the daniel-guooo/MetaboliteChat repository and creating the metabolitechat conda environment from its environment.yml (Python 3.9+ via Miniconda/Conda), plus a separate RDKit environment for dataset conversion.

## When to use

Use when reproducing the MetaboliteChat checkpoint evaluation or training pipeline and a Python 3.9+ environment matching the authors' environment.yml must be instantiated before running demo.sh or train.py.

## When NOT to use

- You only need to query GPT-4o or another generic LLM baseline via API — no local conda environment from environment.yml is required.
- The target machine has no NVIDIA GPU and the goal is training (needs >=70 GB GPU memory, verified on A100 80 GB); inference-only or CPU-only inspection should be considered instead.

## Inputs

- environment.yml from the MetaboliteChat repository (git clone https://github.com/13501274828/MetaboliteChat.git)
- Miniconda/Conda installation with Python 3.9+
- NVIDIA GPU with driver visible to nvidia-smi (>=70 GB memory needed for training)

## Outputs

- Active 'metabolitechat' conda environment with Python 3.9+ and all pinned dependencies
- Separate 'rdkit' conda environment for running dataset/smiles2graph_image_demo.py
- Reproducible base for executing train.py (torchrun) and demo.sh evaluation

## How to apply

Clone the MetaboliteChat repository (git clone https://github.com/13501274828/MetaboliteChat.git) and create the environment with 'conda env create -f environment.yml', then 'conda activate metabolitechat'. The environment.yml pins Python 3.9+ and dependencies (e.g. numpy installed via pip, RDKit) so the multimodal Vicuna-13B-v1.5 model with GNN and CNN encoders runs as the authors built it. A separate rdkit environment is created from conda-forge ('conda create -c conda-forge -n rdkit rdkit') specifically to run the dataset conversion script smiles2graph_image_demo.py, isolating RDKit dependency conflicts from the main training/evaluation stack. Before training, verify GPU capacity with nvidia-smi, since training requires at least 70 GB GPU memory (verified on A100 80 GB).

## Related tools

- **conda** (Create and activate the pinned metabolitechat environment from environment.yml and the separate rdkit environment from conda-forge)
- **Miniconda** (Provides the Conda distribution used to provision the Python 3.9+ environment)
- **pip** (Installs Python dependencies inside the environment (e.g. pip install numpy))
- **numpy** (Pinned dependency installed via pip inside the metabolitechat environment)
- **RDKit** (Runs in an isolated rdkit environment to convert SMILES to graph/image inputs via smiles2graph_image_demo.py)
- **nvidia-smi** (Verifies GPU memory availability (>=70 GB) before training)

## Examples

```
git clone https://github.com/13501274828/MetaboliteChat.git && conda env create -f environment.yml && conda activate metabolitechat
```

## Evaluation signals

- 'conda activate metabolitechat' succeeds and 'python --version' reports 3.9+ inside the environment.
- The separate rdkit environment activates and dataset/smiles2graph_image_demo.py runs without import errors.
- nvidia-smi reports a GPU with at least 70 GB memory before attempting train.py.
- demo.sh can be launched from the activated environment, confirming dependencies (torch, transformers, LAVIS/MiniGPT-4 code paths) resolve correctly.

## Limitations

- Training requires at least 70 GB GPU memory (verified on A100 80 GB), so smaller GPUs cannot run the full training step even with a correct environment.
- RDKit dependency conflicts motivate a second, isolated conda environment; running dataset conversion inside the main environment may fail.

## Evidence

- [intro] Clone and create environment: [evidence span withheld — anchor status: not_found]
- [intro] Python via Miniconda: "Python: 3.9+ via Miniconda/Conda."
- [intro] Separate rdkit environment: [evidence span withheld — anchor status: not_found]
- [intro] GPU memory requirement: "**You need at least 70 GB GPU memory for the training.**"
- [readme] Repository purpose from README: "This repository holds the code of MetaboliteChat. Read the biorxiv [preprint](https://doi.org/10.1101/2025.11.07.687008)."
