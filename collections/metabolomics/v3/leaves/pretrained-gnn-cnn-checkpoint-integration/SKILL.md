---
name: pretrained-gnn-cnn-checkpoint-integration
description: Use when SMILES-derived molecular graphs and molecular images must be
  encoded by pretrained GNN/CNN encoders before training the projection layer that
  connects to the LLaMA model.
when_to_use_negative:
- Inputs are natural-language or tabular metabolite data with no SMILES strings, so
  no molecular graphs or images can be produced for GNN/CNN encoding.
- The molecular graphs and images already have encoded features and the projection
  layer has been trained — use the trained model directly via demo.sh instead.
- Only fine-tuning or evaluating a MetaboliteChat checkpoint (e.g. the one trained
  on 152,222 metabolites) without rebuilding the graph/image encoding pipeline.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MetaboliteChat
  role: Repository whose train.py and metabolitechat.yaml config train the projection
    layer consuming the GNN-encoded graph features; evaluated via demo.sh
  repo: https://github.com/13501274828/MetaboliteChat.git
- name: RDKit
  role: Parses each SMILES string into a molecular graph and renders the molecular
    image that the GNN and CNN checkpoints then encode
- name: snap-stanford/pretrain-gnns
  role: Source of the pretrained GNN checkpoint used to encode molecular graph features
  repo: https://github.com/snap-stanford/pretrain-gnns/
- name: ImageMol
  role: Source of the pretrained CNN checkpoint used to encode molecular image features
  repo: https://github.com/HongxinXiang/ImageMol
- name: torchrun
  role: 'Launches projection-layer training: torchrun --nproc_per_node 1 train.py
    --cfg-path train_configs/metabolitechat.yaml'
- name: conda
  role: Creates the separate rdkit environment (from conda-forge) in which RDKit featurisation
    runs before checkpoint encoding
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.07.687008
    title: 'MetaboliteChat: A Unified Multimodal Large Language Model for Interactive
      Metabolite Analysis and Functional Insights'
  source_content_hash: sha256:067e1ade8c8c57a00ecdd4a8951e8b1bb2dc565d83599e94bfd5e86152e1fe72
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
  - ImageMol
  - MetaboliteChat
  - Miniconda
  - RDKit
  - conda
  - numpy
  - pip
  - snap-stanford/pretrain-gnns
  - torchrun
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# pretrained-gnn-cnn-checkpoint-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Downloads and integrates pretrained GNN and CNN checkpoints into MetaboliteChat so that RDKit-derived molecular graphs and molecular images can be encoded into features that feed the projection layer connected to the LLaMA (Vicuna-13B-v1.5) model.

## When to use

Use when SMILES-derived molecular graphs and molecular images must be encoded by pretrained GNN/CNN encoders before training the projection layer that connects to the LLaMA model.

## When NOT to use

- Inputs are natural-language or tabular metabolite data with no SMILES strings, so no molecular graphs or images can be produced for GNN/CNN encoding.
- The molecular graphs and images already have encoded features and the projection layer has been trained — use the trained model directly via demo.sh instead.
- Only fine-tuning or evaluating a MetaboliteChat checkpoint (e.g. the one trained on 152,222 metabolites) without rebuilding the graph/image encoding pipeline.

## Inputs

- SMILES strings from HMDB-derived instruction-tuning JSON files keyed as {SMILES String: [[Question, Answer], ...]}
- RDKit-derived molecular graphs (atoms/bonds) per SMILES
- RDKit-rendered molecular images per SMILES
- Pretrained GNN checkpoints (snap-stanford/pretrain-gnns) from the Google Drive folder
- Pretrained CNN checkpoints (ImageMol) from the Google Drive folder
- Vicuna-13B-v1.5 weights with path set in the config file

## Outputs

- Encoded molecular graph features from the pretrained GNN
- Encoded molecular image features from the pretrained CNN
- Trained projection layer connecting the GNN output to the LLaMA model

## How to apply

First run the RDKit featurisation step (dataset/smiles2graph_image_demo.py in a dedicated rdkit conda environment) so every HMDB-derived SMILES string in the instruction-tuning JSON ({SMILES String: [[Question, Answer], ...]}) yields both a molecular graph and a molecular image. Download the pretrained GNN and CNN checkpoints from the MetaboliteChat Google Drive folder (the GNN corresponds to snap-stanford/pretrain-gnns and the CNN to ImageMol) and keep them available in the environment. Train the projection layer that connects the GNN output to the LLaMA model by launching train.py with torchrun and the train_configs/metabolitechat.yaml config; the CNN encodes the rendered molecular images in parallel. Finally, evaluate the assembled multimodal model by running bash demo.sh.

## Related tools

- **MetaboliteChat** (Repository whose train.py and metabolitechat.yaml config train the projection layer consuming the GNN-encoded graph features; evaluated via demo.sh) — https://github.com/13501274828/MetaboliteChat.git
- **RDKit** (Parses each SMILES string into a molecular graph and renders the molecular image that the GNN and CNN checkpoints then encode)
- **snap-stanford/pretrain-gnns** (Source of the pretrained GNN checkpoint used to encode molecular graph features) — https://github.com/snap-stanford/pretrain-gnns/
- **ImageMol** (Source of the pretrained CNN checkpoint used to encode molecular image features) — https://github.com/HongxinXiang/ImageMol
- **torchrun** (Launches projection-layer training: torchrun --nproc_per_node 1 train.py --cfg-path train_configs/metabolitechat.yaml)
- **conda** (Creates the separate rdkit environment (from conda-forge) in which RDKit featurisation runs before checkpoint encoding)

## Examples

```
torchrun --nproc_per_node 1 train.py --cfg-path train_configs/metabolitechat.yaml
```

## Evaluation signals

- The smiles2graph_image_demo.py conversion completes without errors and every SMILES entry in the HMDB dataset yields both a molecular graph output and a molecular image output.
- Both the GNN and CNN checkpoints from the Google Drive folder are downloaded and available before starting projection-layer training.
- Training (torchrun with metabolitechat.yaml) runs on a GPU with at least 70 GB memory (verified on A100 80 GB) without OOM failure.
- The trained projection layer connects the GNN output to the LLaMA model, confirmed by running the evaluation via bash demo.sh.

## Limitations

- Requires at least 70 GB GPU memory for training (verified only on A100 80 GB).
- Checkpoints are distributed only as Google Drive downloads; the GNN and CNN encoders must be kept available alongside the RDKit conversion outputs.
- The RDKit featurisation must run in a dedicated rdkit conda environment separate from the main metabolitechat environment, adding environment-management overhead.
- The provided checkpoint was trained on 152,222 metabolites from HMDB; generalisation beyond that coverage is not established by this workflow.

## Evidence

- [other] RDKit featurisation executes in dedicated rdkit environment: [evidence span withheld — anchor status: not_found]
