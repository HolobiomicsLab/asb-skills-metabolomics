---
name: gnn-spectral-property-prediction-workflow
description: 'Use when you want to train or apply a graph neural network over molecular
  graphs to predict a structure-dependent property — retention time, collision cross
  section (CCS), or an MS2 spectrum — and use that predicted property to filter or
  re-rank candidate structures for untargeted metabolomics annotation: build RDKit/PyTorch-Geometric
  molecular graphs from SMILES, design and train a GNN against a labelled property
  dataset, run inference to predict the property for candidate structures, and rescore
  a candidate pool by comparing predicted vs observed property values.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - chemical-graph-parsing-and-featurization
  - molecular-graph-construction-from-smiles
  - graph-attention-network-implementation
  - molecular-graph-neural-network-implementation
  - molecular-graph-gcn-encoder-training
  - retention-time-regression-model-training--8849e07a1d6b4a9c
  - molecular-graph-representation-learning
  - graph-neural-network-training--5c016ac055bd20cd
  - graph-neural-network-training--ea0bd47505660ff7
  - pretrained-gnn-inference
  - neural-network-model-inference
  - nmr-spectrum-inference-decoding
  - putative-identity-ranking-by-rt-error
  - candidate-frequency-ranking-for-denovo-annotation
  - precursor-mass-candidate-screening
  - retention-order-score-integration
  - casmi-challenge-candidate-ranking
  member_tools:
  - Cython
  - Python
  - Pytorch Geometric
  - RDKit
  - PyTorch
  - RT-Transformer
  - GNN-RT repository (Preprocess.py, Train.py, Transferlearning.py)
  - HMDB
  - Keras for R
  - METLIN SMRT dataset
  - PredRet database
  - R
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# GNN Molecular Property Prediction for Annotation Rescoring (RDKit graph -> PyTorch GNN -> candidate re-ranking)

## Summary

SMILES + labelled property dataset in, a property-rescored candidate table out: RDKit/PyG molecular graph construction, GNN architecture design and supervised training, trained-model inference on candidate structures, and predicted-vs-observed candidate re-ranking.


## When to use

Use when you want to train or apply a graph neural network over molecular graphs to predict a structure-dependent property — retention time, collision cross section (CCS), or an MS2 spectrum — and use that predicted property to filter or re-rank candidate structures for untargeted metabolomics annotation: build RDKit/PyTorch-Geometric molecular graphs from SMILES, design and train a GNN against a labelled property dataset, run inference to predict the property for candidate structures, and rescore a candidate pool by comparing predicted vs observed property values.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — featurize

**Goal:** SMILES structures -> attributed molecular graphs (RDKit + PyTorch Geometric)

**EDAM operation:** http://edamontology.org/operation_0292

**Inputs:** tsv · **Outputs:** pyg-graph-dataset

**Candidate leaf skills:** `chemical-graph-parsing-and-featurization` (primary), `molecular-graph-construction-from-smiles`

**Tools (primary):** Cython, Python, Pytorch Geometric, RDKit

**Other candidate tools:** CCSbase, ISiCLE, SigmaCCS, Spektral, TensorFlow

**Grounding:** 2 KB(s); DOIs: 10.1038/s42004-023-00939-w, 10.48550/arxiv.2404.02360

### Stage 2 — architect

**Goal:** molecular graph representation -> GNN architecture (message-passing layers + property head)

**EDAM operation:** http://edamontology.org/operation_0337

**Inputs:** pyg-graph-dataset · **Outputs:** model-architecture

**Candidate leaf skills:** `graph-attention-network-implementation` (primary), `molecular-graph-neural-network-implementation`, `molecular-graph-gcn-encoder-training`

**Tools (primary):** PyTorch, PyTorch Geometric, RDKit, RT-Transformer

**Other candidate tools:** Cython, Python, Pytorch Lightning, DGL, Lightning, MVP, massspecgym

**Grounding:** 3 KB(s); DOIs: 10.1093/bioinformatics/btae084, 10.1101/2025.11.12.688047, 10.48550/arxiv.2404.02360

### Stage 3 — train

**Goal:** GNN architecture + labelled property dataset -> trained model checkpoint

**EDAM operation:** http://edamontology.org/operation_3927

**Inputs:** model-architecture, pyg-graph-dataset · **Outputs:** model-checkpoint

**Candidate leaf skills:** `retention-time-regression-model-training--8849e07a1d6b4a9c` (primary), `molecular-graph-representation-learning`, `graph-neural-network-training--5c016ac055bd20cd`

**Tools (primary):** GNN-RT repository (Preprocess.py, Train.py, Transferlearning.py), PyTorch, RDKit

**Other candidate tools:** Hugging Face Spaces, PyTorch Geometric (torch_geometric), RDKit (rdkit-pypi), RT-Transformer, FIORA, ICEBERG

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.0c04071, 10.1038/s41467-025-57422-4, 10.1093/bioinformatics/btae084

### Stage 4 — predict

**Goal:** trained GNN + candidate structures -> predicted property values

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** model-checkpoint, tsv · **Outputs:** tsv

**Candidate leaf skills:** `graph-neural-network-training--ea0bd47505660ff7` (primary), `pretrained-gnn-inference`, `neural-network-model-inference`, `nmr-spectrum-inference-decoding`

**Tools (primary):** RT-Transformer

**Other candidate tools:** Jupyter Notebook, Python, RDkit, NEIMS, RDKit Cheminformatics package, Tensorflow, Anaconda, NMR2Struct, convolutional neural network (CNN), transformer (encoder-decoder)

**Grounding:** 4 KB(s); DOIs: 10.1002/anie.202507483, 10.1021/acscentsci.4c01132, 10.1021/acscentsci.9b00085, 10.1093/bioinformatics/btae084

### Stage 5 — rescore

**Goal:** predicted property + experimental evidence -> re-ranked / filtered annotation candidates

**EDAM operation:** http://edamontology.org/operation_3800

**Inputs:** tsv, feature-table · **Outputs:** tsv

**Candidate leaf skills:** `putative-identity-ranking-by-rt-error` (primary), `candidate-frequency-ranking-for-denovo-annotation`, `precursor-mass-candidate-screening`, `retention-order-score-integration`, `casmi-challenge-candidate-ranking`

**Tools (primary):** HMDB, Keras for R, METLIN SMRT dataset, PredRet database, R

**Other candidate tools:** DiGress, DiffMS, MIST, PyTorch, RDKit, GNN-RT, SigmaCCS, IOKR, Python and, json >=, networkx, networkx >=, numpy, pandas, rcdkTools, ROASMI, chemprop, msms_rt_score_integration

**Grounding:** 5 KB(s); DOIs: 10.1038/s41467-019-13680-7, 10.1038/s42004-023-00939-w, 10.1093/bioinformatics/bty590, 10.1186/s13321-025-00968-8 …

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
