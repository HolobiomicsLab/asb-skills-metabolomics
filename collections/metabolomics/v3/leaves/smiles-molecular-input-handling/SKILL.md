---
name: smiles-molecular-input-handling
description: Use when metabolite inputs exist as SMILES strings and must be transformed
  into molecular graphs and molecular images so they can be joined with natural-language
  prompts in MetaboliteChat.
when_to_use_negative:
- The input is already a converted graph/image pair produced by a prior run of smiles2graph_image_demo.py
- The analysis task has no molecular-structure component and only requires plain text
  interaction with a generic LLM (e.g., a GPT-4o-only baseline)
- Only the Vicuna-13B-v1.5 backbone is being tested without the GNN/CNN encoders and
  trained projection layer, since no molecular conversion is needed
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3172
tools:
- name: RDKit
  role: Parses SMILES strings and produces the molecular graph and image representations
    via the conversion script
  repo: https://github.com/13501274828/MetaboliteChat.git
- name: conda
  role: Creates the isolated rdkit environment (`conda create -c conda-forge -n rdkit
    rdkit`) used for the conversion step
- name: MetaboliteChat
  role: Downstream multimodal model (Vicuna-13B LLM + GNN + CNN encoders) that consumes
    the converted graph/image inputs together with SMILES text and prompts
  repo: https://github.com/13501274828/MetaboliteChat.git
- name: torchrun
  role: Launches training of the projection layer that connects the converted GNN
    output to the LLaMA model
  repo: https://github.com/13501274828/MetaboliteChat.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.07.687008
    title: 'MetaboliteChat: A Unified Multimodal Large Language Model for Interactive
      Metabolite Analysis and Functional Insights'
  source_content_hash: sha256:6c71005de8cf1ffcb5628c08b8865943781beac9abbcf2356e598bbb3940c9af
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
  - torchrun
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# smiles-molecular-input-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts metabolite SMILES strings into the graph and image representations required by MetaboliteChat's multimodal LLM (Vicuna-13B-v1.5 backbone with GNN and CNN encoders), using a dedicated RDKit environment.

## When to use

Use when metabolite inputs exist as SMILES strings and must be transformed into molecular graphs and molecular images so they can be joined with natural-language prompts in MetaboliteChat.

## When NOT to use

- The input is already a converted graph/image pair produced by a prior run of smiles2graph_image_demo.py
- The analysis task has no molecular-structure component and only requires plain text interaction with a generic LLM (e.g., a GPT-4o-only baseline)
- Only the Vicuna-13B-v1.5 backbone is being tested without the GNN/CNN encoders and trained projection layer, since no molecular conversion is needed

## Inputs

- SMILES strings for metabolites (sourced from the Human Metabolome Database, HMDB)
- HMDB-derived instruction-tuning records formatted as {SMILES String: [[Question1, Answer1], [Question2, Answer2], ...]}
- Natural-language prompts to be paired with the converted molecular representations

## Outputs

- Molecular graphs (per metabolite) consumed by the GNN encoder
- Molecular images (per metabolite) consumed by the CNN/ImageMol encoder
- Converted dataset files produced by smiles2graph_image_demo.py used for downstream training/evaluation

## How to apply

Create a dedicated conda environment from conda-forge (`conda create -c conda-forge -n rdkit rdkit`) rather than reusing the main metabolitechat environment, because the conversion depends on RDKit. Activate it and run the conversion script `python dataset/smiles2graph_image_demo.py`, which decodes each SMILES string (from the HMDB-based dataset structured as {SMILES String: [[Question1, Answer1], ...]}) into a molecular graph for the GNN encoder and a molecular image for the CNN/ImageMol encoder. These derived representations are what the trained projection layer feeds into the LLaMA-side Vicuna-13B-v1.5 model alongside the raw SMILES text and natural-language prompts. Decision point: if a SMILES fails to parse or convert, it cannot be used for multimodal inference and should be excluded or fixed before running the training (`torchrun --nproc_per_node 1 train.py --cfg-path train_configs/metabolitechat.yaml`) or evaluation (`bash demo.sh`) stages.

## Related tools

- **RDKit** (Parses SMILES strings and produces the molecular graph and image representations via the conversion script) — https://github.com/13501274828/MetaboliteChat.git
- **conda** (Creates the isolated rdkit environment (`conda create -c conda-forge -n rdkit rdkit`) used for the conversion step)
- **MetaboliteChat** (Downstream multimodal model (Vicuna-13B LLM + GNN + CNN encoders) that consumes the converted graph/image inputs together with SMILES text and prompts) — https://github.com/13501274828/MetaboliteChat.git
- **torchrun** (Launches training of the projection layer that connects the converted GNN output to the LLaMA model) — https://github.com/13501274828/MetaboliteChat.git

## Examples

```
conda create -c conda-forge -n rdkit rdkit && conda activate rdkit && python dataset/smiles2graph_image_demo.py
```

## Evaluation signals

- The rdkit conda environment is created successfully from conda-forge and activates without dependency conflicts
- smiles2graph_image_demo.py completes and emits graph and image artifacts for the HMDB-derived SMILES entries without unparseable-SMILES errors
- Training on the converted inputs completes within the stated hardware envelope (at least 70 GB GPU memory, verified on A100 80 GB)
- Downstream `bash demo.sh` evaluation produces predictions whose entries contain a free-form prediction string for each metabolite input (SMILES + image + prompt)
- The released checkpoint trained on 152,222 metabolites loads and runs against the converted inputs, yielding outputs consistent with the reported superiority over GPT-4o

## Limitations

- Training the full pipeline requires at least 70 GB of GPU memory (verified on A100 80 GB), which constrains where conversion-plus-training can be run
- The current model version is fixed to Vicuna-13B-v1.5, so converted inputs must match that backbone's expected configuration
- No discussion/changelog section is provided, so edge-case behavior for invalid or non-HMDB SMILES is not documented

## Evidence

- [intro] SMILES-to-graph/image conversion via RDKit: [evidence span withheld — anchor status: not_found]
