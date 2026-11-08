---
name: dual-encoder-representation-learning
description: Use when a query (e.g. an MS/MS spectrum) must be matched against a discrete
  candidate pool (e.g. molecular scaffolds) via embedding-space similarity, rather
  than by generative or rule-based lookup.
when_to_use_negative:
- The correct scaffold is already known (use MADGEN's oracle retrieval lookup table
  instead, which always yields the correct scaffold given the spectrum and chemical
  formula)
- The goal is de novo molecule generation rather than scaffold retrieval — that is
  MADGEN's second stage, which is spectra-conditioned generation starting from the
  retrieved scaffold
- Candidate structures cannot be enumerated into a finite scaffold pool, so a ranking
  formulation over candidates is not applicable
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Implements and trains the dual MLP (spectrum) and GNN (scaffold) encoders
    under the contrastive objective
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit
  role: Extracts the scaffold from each candidate molecule's molecular graph representation
    to build the scaffold candidate set
  repo: https://github.com/HassounLab/MADGEN
- name: AdamW
  role: Optimizer for contrastive training with learning rate 1e-5
- name: MADGEN
  role: Framework hosting the scaffold_retrieval module, including train_msgym.py
    / train_canopus.py and candidate-ranking scripts (cand_rank_msg.py, cand_rank_canopus.py)
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:5bb85fef255507979958dbb1598c03855053f43ad8bd0dc8cf200576e5f5f940
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - AdamW
  - MADGEN
  - PyTorch
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# dual-encoder-representation-learning

## Summary

Train two parallel encoders (an MLP for MS/MS spectra, a GNN for molecular scaffold graphs) with a temperature-scaled contrastive objective so matched spectrum–scaffold pairs embed close together, enabling scaffold retrieval as a cosine-similarity ranking problem.

## When to use

Use when a query (e.g. an MS/MS spectrum) must be matched against a discrete candidate pool (e.g. molecular scaffolds) via embedding-space similarity, rather than by generative or rule-based lookup.

## When NOT to use

- The correct scaffold is already known (use MADGEN's oracle retrieval lookup table instead, which always yields the correct scaffold given the spectrum and chemical formula)
- The goal is de novo molecule generation rather than scaffold retrieval — that is MADGEN's second stage, which is spectra-conditioned generation starting from the retrieved scaffold
- Candidate structures cannot be enumerated into a finite scaffold pool, so a ranking formulation over candidates is not applicable

## Inputs

- Preprocessed MassSpecGym dataset pickle (msgym.pkl) from Zenodo record 15036069, containing MS/MS spectra, chemical formulas, and candidate molecules
- Molecular graph representations of candidate molecules (for RDKit scaffold extraction)
- Scaffold candidate set built from candidate molecules, with target molecules removed

## Outputs

- Spectrum embeddings and scaffold embeddings in a shared d-dimensional latent space
- Ranked candidate scaffolds per test spectrum (ranks_msgym_pred.pkl)
- Scaffold prediction accuracy (SPA) on the held-out test split

## How to apply

Build two encoders in PyTorch: an MLP that embeds the preprocessed MS/MS spectrum (intensity-normalized, low-intensity peaks removed) and a GNN that embeds candidate scaffold graphs extracted with RDKit from each candidate molecule's molecular graph. Train jointly with a temperature-scaled contrastive loss that pulls matched spectrum–scaffold pairs together and pushes mismatched pairs apart, using the AdamW optimizer with learning rate 1e-5. At inference, score every candidate scaffold per spectrum by cosine similarity in the shared d-dimensional latent space, retrieve the top-k ranked candidates, apply frequency-based aggregation over scaffolds per chemical formula, and select the most frequent scaffold as the prediction. Remove target molecules from the candidate pool before ranking. Serialize the ranked candidate scaffolds per test spectrum (e.g. to ranks_msgym_pred.pkl) and report scaffold prediction accuracy (SPA) on the held-out test split.

## Related tools

- **PyTorch** (Implements and trains the dual MLP (spectrum) and GNN (scaffold) encoders under the contrastive objective) — https://github.com/HassounLab/MADGEN
- **RDKit** (Extracts the scaffold from each candidate molecule's molecular graph representation to build the scaffold candidate set) — https://github.com/HassounLab/MADGEN
- **AdamW** (Optimizer for contrastive training with learning rate 1e-5)
- **MADGEN** (Framework hosting the scaffold_retrieval module, including train_msgym.py / train_canopus.py and candidate-ranking scripts (cand_rank_msg.py, cand_rank_canopus.py)) — https://github.com/HassounLab/MADGEN

## Examples

```
cd scaffold_retrieval/ && python train_msgym.py && python cand_rank_msg.py
```

## Evaluation signals

- Scaffold prediction accuracy (SPA) on the held-out test split is the stated success criterion for retrieval quality
- Matched spectrum–scaffold pairs should show higher cosine similarity than mismatched pairs in the shared latent space after training
- Ranked candidate lists per test spectrum must be serialized and schema-consistent (ranks_msgym_pred.pkl), and preprocessing via preprocess_rank.py in data/msgym/raw/ must complete
- Frequency-based aggregation over top-k candidates should select a scaffold consistent with the query's chemical formula

## Limitations

- The retrieval stage constrains but does not guarantee the correct scaffold; MADGEN therefore also reports results with an oracle retriever that always yields the correct scaffold
- Retrieval quality depends on the candidate pool construction — target molecules must be removed from the candidate set to avoid trivial matches
- Results depend on contrastive-embedding quality; the model was validated only on NIST23, CANOPUS, and MassSpecGym datasets

## Evidence

- [abstract] given an MS/MS spectrum, we formulate scaffold retrieval as a ranking problem and employ contrastive learning to align mass spectra with candidate molecular scaffolds: "given an MS/MS spectrum, we formulate scaffold retrieval as a ranking problem and employ contrastive learning to align mass spectra with candidate molecular scaffolds"
- [methods] We employed the AdamW optimizer with a learning rate of 1×10−5.: [evidence span withheld — anchor status: not_found]
- [methods] We construct the lookup table by extracting the scaffold from the molecular graph representation using RDKit.: "We construct the lookup table by extracting the scaffold from the molecular graph representation using RDKit."
- [methods] we introduce a frequency-based aggregation approach. For each data point, we first retrieve the top-k ranked candidate scaffolds.: [evidence span withheld — anchor status: not_found]
