---
name: contrastive-scaffold-retrieval-from-mass-spectra
description: Use when an MS/MS spectrum must be assigned a scaffold (not a full structure)
  from a candidate set — e.g. before scaffold-conditioned de novo generation, or to
  compute Scaffold Prediction Accuracy (SPA) on NIST23, CANOPUS, or MassSpecGym.
when_to_use_negative:
- The goal is full de novo molecule generation, not scaffold prediction — use MADGEN's
  Stage 2 scaffold-conditioned generation instead
- The true scaffold is already known and only generation performance is being measured
  — use the oracle retrieval lookup table rather than the predictive contrastive retriever
- Candidate scaffolds are not available for the target spectrum (retrieval requires
  a candidate set to score and rank)
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MADGEN
  role: Two-stage framework whose first stage is the contrastive scaffold retriever
    being applied
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit 2023.09.5
  role: Extracting scaffold from the molecular graph representation to build the scaffold
    lookup table and ground-truth scaffolds
  repo: https://github.com/HassounLab/MADGEN
- name: PyTorch
  role: Training the contrastive neural network that scores and ranks candidate scaffolds
    against MS/MS spectra
- name: conda
  role: Creating the pinned environment (python=3.9, rdkit=2023.09.5 -c conda-forge)
    in which retrieval is run
- name: wget
  role: Downloading processed datasets (msgym.pkl, canopus.pkl) and pre-computed ranking
    pickles (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl) from Zenodo records 15036069
    and 17115068
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:bad8fdc09cf4d3be52051b1d26e58b8fde2820584d14fc061b5c90ed5c0ed4c0
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - MADGEN
  - PyTorch
  - Python
  - RDKit 2023.09.5
  - conda
  - wget
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# Contrastive scaffold retrieval from mass spectra

## Summary

Formulates scaffold retrieval from an MS/MS spectrum as a ranking problem solved with contrastive learning that aligns spectral embeddings with candidate molecular scaffolds; this is Stage 1 of MADGEN's two-stage de novo generation pipeline.

## When to use

Use when an MS/MS spectrum must be assigned a scaffold (not a full structure) from a candidate set — e.g. before scaffold-conditioned de novo generation, or to compute Scaffold Prediction Accuracy (SPA) on NIST23, CANOPUS, or MassSpecGym.

## When NOT to use

- The goal is full de novo molecule generation, not scaffold prediction — use MADGEN's Stage 2 scaffold-conditioned generation instead
- The true scaffold is already known and only generation performance is being measured — use the oracle retrieval lookup table rather than the predictive contrastive retriever
- Candidate scaffolds are not available for the target spectrum (retrieval requires a candidate set to score and rank)

## Inputs

- Preprocessed MS/MS spectra with normalized intensities and low-intensity peaks removed (e.g. msgym.pkl, canopus.pkl from Zenodo record 15036069)
- Pre-computed predictive retrieval rankings (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl from Zenodo record 17115068)
- Candidate scaffold list and chemical formula per spectrum
- Molecular graph representations of candidate molecules for RDKit scaffold extraction
- Ground-truth core scaffolds extracted per dataset split (NIST23, CANOPUS, MassSpecGym)

## Outputs

- Ranked list of candidate scaffolds per test spectrum
- Predicted scaffold per spectrum via frequency-based aggregation of top-k candidates
- Scaffold Prediction Accuracy (SPA) per dataset (e.g. spa_evaluation.csv)

## How to apply

Preprocess spectra by normalizing intensities and removing low-intensity peaks below a predefined noise threshold. Train a neural retriever with contrastive learning to score candidate scaffolds against the MS/MS spectrum, treating retrieval as a ranking problem (train via scaffold_retrieval/train_msgym.py or train_canopus.py, then rank with cand_rank_msg.py / cand_rank_canopus.py). For each test spectrum, read the top-k ranked candidate scaffolds from the pre-computed ranking pickle (e.g. ranks_msgym_pred.pkl) and apply frequency-based aggregation: count scaffold frequencies among the top-k and select the most frequently occurring scaffold per chemical formula as the prediction. Extract ground-truth core scaffolds from molecular graph representations with RDKit (scaffolds are unique per split, preventing leakage) and compare predicted vs. true scaffolds to compute SPA. As an upper bound, an oracle lookup table that always yields the correct scaffold given the spectrum and chemical formula can replace the predictive retriever.

## Related tools

- **MADGEN** (Two-stage framework whose first stage is the contrastive scaffold retriever being applied) — https://github.com/HassounLab/MADGEN
- **RDKit 2023.09.5** (Extracting scaffold from the molecular graph representation to build the scaffold lookup table and ground-truth scaffolds) — https://github.com/HassounLab/MADGEN
- **PyTorch** (Training the contrastive neural network that scores and ranks candidate scaffolds against MS/MS spectra)
- **conda** (Creating the pinned environment (python=3.9, rdkit=2023.09.5 -c conda-forge) in which retrieval is run)
- **wget** (Downloading processed datasets (msgym.pkl, canopus.pkl) and pre-computed ranking pickles (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl) from Zenodo records 15036069 and 17115068)

## Examples

```
wget https://zenodo.org/records/17115068/files/ranks_msgym_pred.pkl?download=1 -O ./data/msgym/raw/ranks_msgym_pred.pkl
```

## Evaluation signals

- Reproduced SPA matches the reported pattern: highest on NIST23 (~40.3%, the dataset with the least scaffold diversity) and lower on CANOPUS and MassSpecGym where scaffold diversity is higher; reported range is 13.2%–40.3%
- Ranking pickles load correctly and contain per-spectrum ranked scaffold candidates aligned with the chemical formula of each test spectrum
- Frequency-based aggregation selects exactly one most-frequent scaffold per chemical formula after taking top-k candidates
- RDKit-extracted ground-truth scaffolds are unique per split, confirming no train/test scaffold leakage
- Contrastive training runs complete via train_msgym.py / train_canopus.py and cand_rank_msg.py / cand_rank_canopus.py produce ranked candidate files before preprocess_rank.py is applied in data/{dataset}/raw/

## Limitations

- Predictive retrieval is imperfect: SPA spans only 13.2%–40.3% across the three datasets, so the oracle retriever (exact scaffold lookup given spectrum and chemical formula) is used as an upper bound
- SPA is dataset-dependent — higher scaffold diversity (CANOPUS, MassSpecGym) lowers accuracy relative to NIST23's lower scaffold diversity
- NIST23 is commercially available and requires separate licensing, so full reproduction of that dataset's retrieval requires acquisition from NIST
- Pre-computed predictive rankings are only released for MassSpecGym and CANOPUS; NIST retrieval rankings are not provided in the Zenodo checkpoints/data records described in the README

## Evidence

- [abstract] scaffold retrieval as ranking with contrastive learning: "given an MS/MS spectrum, we formulate scaffold retrieval as a ranking problem and employ contrastive learning to align mass spectra with candidate molecular scaffolds"
- [methods] frequency-based aggregation definition: [evidence span withheld — anchor status: not_found]
- [methods] RDKit scaffold lookup table construction: "We construct the lookup table by extracting the scaffold from the molecular graph representation using RDKit."
- [methods] spectral preprocessing thresholds: "All datasets were preprocessed by normalizing the intensities of the MS/MS spectra and removing low-intensity peaks below a predefined threshold to reduce noise."
- [readme] pre-computed retrieval downloads: "wget https://zenodo.org/records/17115068/files/ranks_msgym_pred.pkl?download=1 -O ./data/msgym/raw/ranks_msgym_pred.pkl"
- [methods] oracle retrieval upper bound: "We maintain a lookup table as an oracle which always yields the correct scaffold given the MS/MS spectrum and the chemical formula."
