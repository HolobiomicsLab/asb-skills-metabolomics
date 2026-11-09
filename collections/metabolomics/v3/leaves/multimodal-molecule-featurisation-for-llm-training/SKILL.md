---
name: multimodal-molecule-featurisation-for-llm-training
description: 'Use when SMILES strings from a metabolite instruction-tuning dataset
  (e.g. HMDB-derived JSON keyed {SMILES String: [[Question, Answer], ...]}) must be
  featurised into molecular graphs and molecular images before GNN/CNN encoding for
  LLM training.'
when_to_use_negative:
- The molecular graphs and images have already been generated and you only need to
  train the projection layer with torchrun (train.py --cfg-path train_configs/metabolitechat.yaml)
  or evaluate via demo.sh
- The data lacks SMILES strings (e.g. text-only or image-only inputs), so RDKit has
  nothing to parse into a molecular graph
- You are working outside the dedicated rdkit environment, since RDKit is intentionally
  installed separately from the main metabolitechat conda environment
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: RDKit
  role: Parses each SMILES string into a molecular graph (atoms/bonds) and renders
    each molecule as a molecular image via dataset/smiles2graph_image_demo.py
- name: conda
  role: Creates the dedicated rdkit environment from conda-forge, kept separate from
    the main metabolitechat environment
- name: numpy
  role: Installed via pip into the rdkit environment to support the conversion script
- name: MetaboliteChat
  role: Repository hosting dataset/smiles2graph_image_demo.py and the downstream projection
    layer that connects the GNN output to the LLaMA model
  repo: https://github.com/13501274828/MetaboliteChat.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.07.687008
    title: 'MetaboliteChat: A Unified Multimodal Large Language Model for Interactive
      Metabolite Analysis and Functional Insights'
  source_content_hash: sha256:87ae4f37f818a1b8f785c0ad567ef26e99e84000df204e800cc13f0b2715b5a2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/daniel-guooo/MetaboliteChat
  repo_url: https://github.com/13501274828/MetaboliteChat.git
  tools:
  - MetaboliteChat
  - Miniconda
  - RDKit
  - conda
  - numpy
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multimodal-molecule-featurisation-for-llm-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts SMILES strings from the HMDB-derived instruction-tuning data into dual molecular representations — a molecular graph (atoms/bonds) and a rendered molecular image — using RDKit, so that pretrained GNN and CNN encoders can feed the projection layer connected to the LLaMA model in MetaboliteChat.

## When to use

Use when SMILES strings from a metabolite instruction-tuning dataset (e.g. HMDB-derived JSON keyed {SMILES String: [[Question, Answer], ...]}) must be featurised into molecular graphs and molecular images before GNN/CNN encoding for LLM training.

## When NOT to use

- The molecular graphs and images have already been generated and you only need to train the projection layer with torchrun (train.py --cfg-path train_configs/metabolitechat.yaml) or evaluate via demo.sh
- The data lacks SMILES strings (e.g. text-only or image-only inputs), so RDKit has nothing to parse into a molecular graph
- You are working outside the dedicated rdkit environment, since RDKit is intentionally installed separately from the main metabolitechat conda environment

## Inputs

- HMDB-derived instruction-tuning JSON files keyed as {SMILES String: [[Question, Answer], ...]} (from the Google Drive deposit)
- Individual SMILES strings parsed by RDKit
- Pretrained GNN and CNN checkpoints (downloaded from the Google Drive folder) for downstream feature encoding

## Outputs

- Molecular graph representation (atoms/bonds) per SMILES string
- Molecular image render per SMILES string
- Encoded graph and image features feeding the projection layer that connects the GNN output to the LLaMA model

## How to apply

Create and activate a dedicated rdkit conda environment built from conda-forge ('conda create -c conda-forge -n rdkit rdkit'), separate from the main metabolitechat environment, and install numpy into it via pip. Load the HMDB-derived instruction-tuning JSON files from the Google Drive deposit, whose entries are keyed as {SMILES String: [[Question, Answer], ...]}. Run the conversion script 'python dataset/smiles2graph_image_demo.py' so RDKit parses every SMILES string into a molecular graph of atoms and bonds and renders the molecule as a molecular image. Keep the pretrained GNN (snap-stanford/pretrain-gnns) and CNN (ImageMol) checkpoints available, because the generated graph and image features are consumed downstream by the projection layer connecting the GNN output to the LLaMA model. Rationale: MetaboliteChat ingests molecular graphs, molecular images, and natural-language prompts, so each SMILES entry must yield both modalities before projection-layer training.

## Related tools

- **RDKit** (Parses each SMILES string into a molecular graph (atoms/bonds) and renders each molecule as a molecular image via dataset/smiles2graph_image_demo.py)
- **conda** (Creates the dedicated rdkit environment from conda-forge, kept separate from the main metabolitechat environment)
- **numpy** (Installed via pip into the rdkit environment to support the conversion script)
- **MetaboliteChat** (Repository hosting dataset/smiles2graph_image_demo.py and the downstream projection layer that connects the GNN output to the LLaMA model) — https://github.com/13501274828/MetaboliteChat.git

## Examples

```
conda activate rdkit && python dataset/smiles2graph_image_demo.py
```

## Evaluation signals

- The conversion script 'python dataset/smiles2graph_image_demo.py' completes without errors in the dedicated rdkit conda environment
- Every SMILES entry in the HMDB-derived JSON dataset yields both a molecular graph output and a molecular image output
- The rdkit environment was created from conda-forge and is separate from the main metabolitechat environment, with numpy installed via pip
- Generated graph and image features can be encoded downstream by the projection layer connecting the GNN output to the LLaMA model, given the pretrained GNN and CNN checkpoints are present

## Limitations

- The featurisation requires a dedicated rdkit environment created from conda-forge; it is not run inside the main metabolitechat environment
- The generated graph and image features are only useful if the pretrained GNN and CNN checkpoints from the Google Drive folder are downloaded and available for downstream encoding
- Training the downstream projection layer needs at least 70 GB GPU memory (verified on A100 80 GB), which constrains the full pipeline beyond featurisation itself

## Evidence

- [intro] Create a separate rdkit environment and run the dataset conversion script (smiles2graph_image_demo.py).: [evidence span withheld — anchor status: not_found]
- [intro] conda create -c conda-forge -n rdkit rdkit ... conda activate rdkit ... python dataset/smiles2graph_image_demo.py: [evidence span withheld — anchor status: not_found]
- [intro] Start training the projection layer that connects the GNN output and the LLaMA model: "Start training the projection layer that connects the GNN output and the LLaMA model"
- [readme] MetaboliteChat is a multimodal large language model (LLM) designed for metabolite research. It integrates molecular structures (SMILES), molecular images: "MetaboliteChat is a multimodal large language model (LLM) designed for metabolite research. It integrates molecular structures (SMILES), molecular images"
- [intro] Our dataset is based on the [Human Metabolome Database(HMDB)](https://hmdb.ca/).: "Our dataset is based on the [Human Metabolome Database(HMDB)](https://hmdb.ca/)."
