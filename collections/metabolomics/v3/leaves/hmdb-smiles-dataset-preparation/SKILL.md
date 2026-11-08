---
name: hmdb-smiles-dataset-preparation
description: 'Use when preparing HMDB-derived SMILES datasets in the {SMILES String:
  [[Question, Answer], ...]} format so each metabolite yields a molecular graph and
  image that downstream GNN/CNN encoders can encode for MetaboliteChat training.'
when_to_use_negative:
- The molecular graphs and images have already been generated and the pretrained checkpoints
  are in place — re-running the featurisation is redundant
- The dataset is not SMILES-keyed HMDB metabolite data (e.g. non-molecular text corpora)
  — RDKit cannot parse it into molecular graphs
- You are only running inference with the released checkpoint (152,222-metabolite
  model) and not rebuilding the training dataset
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3316
tools:
- name: RDKit
  role: Parses each SMILES string into a molecular graph (atoms/bonds) and renders
    each molecule as a molecular image via dataset/smiles2graph_image_demo.py
- name: conda
  role: Creates the dedicated rdkit environment from conda-forge, kept separate from
    the main metabolitechat environment
- name: Miniconda
  role: Distribution used to set up the conda environments (Python 3.9+)
- name: numpy
  role: Dependency installed via pip into the rdkit environment before running the
    conversion script
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.07.687008
    title: 'MetaboliteChat: A Unified Multimodal Large Language Model for Interactive
      Metabolite Analysis and Functional Insights'
  source_content_hash: sha256:2fc1512108c9c2e33516fdca1e0356662f24b7a6eadba1049fad163415a1bc83
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
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# hmdb-smiles-dataset-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts HMDB-derived SMILES instruction-tuning data (keyed {SMILES String: [[Question, Answer], ...]}) into per-molecule molecular graphs and rendered molecular images using RDKit inside a dedicated rdkit conda environment, so pretrained GNN and CNN encoders can featurize them for MetaboliteChat's projection layer.

## When to use

Use when preparing HMDB-derived SMILES datasets in the {SMILES String: [[Question, Answer], ...]} format so each metabolite yields a molecular graph and image that downstream GNN/CNN encoders can encode for MetaboliteChat training.

## When NOT to use

- The molecular graphs and images have already been generated and the pretrained checkpoints are in place — re-running the featurisation is redundant
- The dataset is not SMILES-keyed HMDB metabolite data (e.g. non-molecular text corpora) — RDKit cannot parse it into molecular graphs
- You are only running inference with the released checkpoint (152,222-metabolite model) and not rebuilding the training dataset

## Inputs

- HMDB-derived instruction-tuning JSON files keyed as {SMILES String: [[Question, Answer], ...]}
- Pretrained GNN checkpoints from the MetaboliteChat Google Drive folder
- Pretrained CNN (ImageMol) checkpoints from the MetaboliteChat Google Drive folder

## Outputs

- One molecular graph (atoms/bonds) per SMILES entry, for GNN encoding
- One rendered molecular image per SMILES entry, for CNN encoding
- Graph and image features consumable by the projection layer connected to the LLaMA model

## How to apply

First create a dedicated rdkit conda environment from conda-forge, separate from the main metabolitechat environment, and install numpy into it with pip. Load the HMDB-derived instruction-tuning JSON files whose keys are SMILES strings and whose values are [Question, Answer] pair lists. Run 'python dataset/smiles2graph_image_demo.py' so RDKit parses every SMILES string into a molecular graph (atoms/bonds) and renders each molecule as a molecular image. Keep the pretrained GNN (snap-stanford/pretrain-gnns) and CNN (ImageMol) checkpoints downloaded from the Google Drive folder available, because the generated graph and image features are encoded downstream by the projection layer that connects the GNN output to the LLaMA model. Validate that the conversion script completes without errors and that every SMILES entry yields both a graph and an image output.

## Related tools

- **RDKit** (Parses each SMILES string into a molecular graph (atoms/bonds) and renders each molecule as a molecular image via dataset/smiles2graph_image_demo.py)
- **conda** (Creates the dedicated rdkit environment from conda-forge, kept separate from the main metabolitechat environment)
- **Miniconda** (Distribution used to set up the conda environments (Python 3.9+))
- **numpy** (Dependency installed via pip into the rdkit environment before running the conversion script)

## Examples

```
conda create -c conda-forge -n rdkit rdkit; conda activate rdkit; pip install numpy; python dataset/smiles2graph_image_demo.py
```

## Evaluation signals

- The conversion script dataset/smiles2graph_image_demo.py completes without errors
- Every SMILES entry in the HMDB JSON yields both a molecular graph output and a molecular image output
- Each JSON key parses as a valid SMILES string under RDKit (no failed parses silently skipped)
- Generated graph features are loadable by the pretrained GNN checkpoint (snap-stanford/pretrain-gnns) and image features by the ImageMol CNN checkpoint
- The rdkit environment is isolated from the main metabolitechat environment (separate conda env from conda-forge)

## Limitations

- The featurisation must run in a dedicated rdkit conda environment from conda-forge, not the main metabolitechat environment, or dependency conflicts can occur
- Downstream encoding requires the pretrained GNN and CNN checkpoints from the Google Drive deposit to be downloaded and available
- Training the downstream model requires at least 70 GB GPU memory (verified on A100 80 GB), so the prepared features are only usable on suitable hardware

## Evidence

- [intro] RDKit featurisation workflow: [evidence span withheld — anchor status: not_found]
- [intro] HMDB dataset format: "Our dataset is based on the [Human Metabolome Database(HMDB)](https://hmdb.ca/). In order to train MetaboliteChat, We transformed their data to the following format:"
- [intro] Projection layer training: "Start training the projection layer that connects the GNN output and the LLaMA model by running: torchrun --nproc_per_node 1 train.py --cfg-path train_configs/metabolitechat.yaml"
- [readme] Multimodal model scope: "MetaboliteChat is a multimodal large language model (LLM) designed for metabolite research. It integrates molecular structures (SMILES), molecular images, and natural language prompts to generate"
