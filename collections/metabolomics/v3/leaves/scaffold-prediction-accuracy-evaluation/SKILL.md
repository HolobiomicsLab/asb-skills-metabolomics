---
name: scaffold-prediction-accuracy-evaluation
description: Use when evaluating a mass-spectrometry scaffold retriever (e.g., contrastive-ranking
  models on NIST23, CANOPUS, or MassSpecGym pkl datasets) and you need a per-dataset
  accuracy for how often the true core scaffold is retrieved from ranked candidates.
when_to_use_negative:
- The goal is end-to-end generation accuracy (Top-1/Tanimoto/MCES) rather than the
  retrieval stage — use generation-evaluation workflows instead
- Only an oracle scaffold lookup table is used (correct scaffold always provided),
  so retrieval accuracy is not the measured quantity
- Spectra are not preprocessed (intensities not normalized, low-intensity peaks not
  removed), which the retriever rankings assume
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MADGEN
  role: Provides the predictive scaffold retriever whose top-k candidate rankings
    are scored for SPA
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit
  role: Extracts ground-truth scaffolds from the molecular graph representation for
    the predicted-vs-truth comparison
- name: wget
  role: Downloads processed datasets and pre-computed predictive retrieval rankings
    from Zenodo
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_content_hash: sha256:b9301967c9c3f1bc568da021bb1c7c4265c46c425980e91d4efa918355c3c24f
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - MADGEN
  - Python
  - RDKit
  - conda
  - wget
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# scaffold-prediction-accuracy-evaluation

## Summary

Computes Scaffold Prediction Accuracy (SPA) of MADGEN's first-stage contrastive scaffold retriever on MS/MS datasets, using frequency-based aggregation of top-k ranked candidate scaffolds and RDKit-extracted ground-truth scaffolds. It quantifies how well the retriever stage constrains the subsequent de novo generation stage.

## When to use

Use when evaluating a mass-spectrometry scaffold retriever (e.g., contrastive-ranking models on NIST23, CANOPUS, or MassSpecGym pkl datasets) and you need a per-dataset accuracy for how often the true core scaffold is retrieved from ranked candidates.

## When NOT to use

- The goal is end-to-end generation accuracy (Top-1/Tanimoto/MCES) rather than the retrieval stage — use generation-evaluation workflows instead
- Only an oracle scaffold lookup table is used (correct scaffold always provided), so retrieval accuracy is not the measured quantity
- Spectra are not preprocessed (intensities not normalized, low-intensity peaks not removed), which the retriever rankings assume

## Inputs

- Processed MS/MS spectra pickles (msgym.pkl, canopus.pkl) with normalized intensities and low-intensity peaks removed
- Pre-computed predictive retrieval rankings (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl)
- Ground-truth molecular graph representations (SMILES/molecular graphs) for RDKit scaffold extraction

## Outputs

- Per-dataset Scaffold Prediction Accuracy (SPA) values (expected: 40.3% NIST23, 17.7% CANOPUS, 13.2% MassSpecGym)
- spa_evaluation.csv results table

## How to apply

Load the preprocessed spectra (msgym.pkl, canopus.pkl from Zenodo record 15036069) with normalized intensities and low-intensity peaks removed, and the pre-computed retrieval rankings (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl from Zenodo record 17115068). For each test spectrum, take the top-k ranked candidate scaffolds, count the frequency of each scaffold, and select the most frequently occurring scaffold per chemical formula as the prediction (the paper's frequency-based aggregation). Extract ground-truth core scaffolds from the molecular graph representation with RDKit (scaffolds are unique to each split, so no train/test leakage occurs). Compare predicted versus ground-truth scaffolds and compute SPA per dataset; write the results to a table such as spa_evaluation.csv. Reference SPA values to reproduce: 40.3% (NIST23, least scaffold diversity), 17.7% (CANOPUS), 13.2% (MassSpecGym).

## Related tools

- **MADGEN** (Provides the predictive scaffold retriever whose top-k candidate rankings are scored for SPA) — https://github.com/HassounLab/MADGEN
- **RDKit** (Extracts ground-truth scaffolds from the molecular graph representation for the predicted-vs-truth comparison)
- **wget** (Downloads processed datasets and pre-computed predictive retrieval rankings from Zenodo)

## Examples

```
wget https://zenodo.org/records/17115068/files/ranks_msgym_pred.pkl?download=1 -O ./data/msgym/raw/ranks_msgym_pred.pkl
```

## Evaluation signals

- Reproduced SPA per dataset must match the reported pattern: highest SPA (~40.3%) on NIST23, the dataset with least scaffold diversity; lower SPA (~17.7% CANOPUS, ~13.2% MassSpecGym) where scaffold diversity is higher
- Ground-truth scaffolds extracted with RDKit must be unique to each data split (no leakage between train/test scaffolds)
- Frequency-based aggregation must select the most frequent scaffold per chemical formula among the top-k ranked candidates, not simply the rank-1 candidate
- Spectra loaded from the pkl files must already have normalized intensities and low-intensity peaks removed below the predefined noise threshold

## Limitations

- NIST23 data is commercially available and requires separate licensing; only msgym.pkl and canopus.pkl are downloadable from Zenodo record 15036069
- Pre-computed rankings (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl) cover CANOPUS and MassSpecGym/MSGym; running retrieval yourself requires training via scaffold_retrieval scripts
- Low SPA directly constrains downstream predictive-retriever generation (Top-1 drops to 4.6%/2.10%/1.31% vs oracle 49.0%/18.7%/10.5%), so SPA should be reported alongside oracle-vs-predictive gaps

## Evidence

- [methods] frequency-based aggregation: [evidence span withheld — anchor status: not_found]
