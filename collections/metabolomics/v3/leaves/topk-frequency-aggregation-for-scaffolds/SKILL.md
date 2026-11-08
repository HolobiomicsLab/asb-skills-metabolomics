---
name: topk-frequency-aggregation-for-scaffolds
description: Use when you must predict a molecular scaffold from a query MS/MS spectrum
  by ranking a candidate scaffold set with a dual encoder (MLP spectrum encoder, GNN
  scaffold encoder) trained contrastively.
when_to_use_negative:
- When the true scaffold is already known (MADGEN's oracle retrieval setting uses
  a lookup table that always yields the correct scaffold given the spectrum and chemical
  formula, so contrastive ranking is unnecessary)
- When the goal is the generation stage itself — scaffold-conditioned de novo molecule
  generation with spectra guidance (MADGEN stage 2) — rather than scoring/ranking
  scaffold candidates
- When candidate molecules cannot be converted to molecular graphs/scaffolds with
  RDKit, since the scaffold candidate set is built entirely from RDKit-extracted scaffolds
tools:
- name: MADGEN
  role: Framework containing the scaffold_retrieval module (train_msgym.py / train_canopus.py
    for contrastive training, cand_rank_msg.py / cand_rank_canopus.py for candidate
    ranking)
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit
  role: Extracts the scaffold from each candidate molecule's molecular graph representation
    to build the scaffold candidate set and lookup table
- name: PyTorch
  role: Implements and trains the MLP spectrum encoder and GNN scaffold encoder under
    the temperature-scaled contrastive objective
- name: AdamW
  role: Optimizer for contrastive training of the dual encoders (learning rate 1e-5)
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:0e845c0c02343e422f9891fa61f781148c3ae28f94203ccd8c0be68d78b040b8
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# Contrastive spectrum–scaffold retrieval

## Summary

MADGEN's first stage predicts a molecular scaffold for a query MS/MS spectrum by formulating scaffold retrieval as a ranking problem: a contrastively trained dual encoder (MLP spectrum encoder + GNN scaffold encoder) scores candidate scaffolds via cosine similarity so the predicted scaffold can constrain downstream de novo generation.

## When to use

Use when you must predict a molecular scaffold from a query MS/MS spectrum by ranking a candidate scaffold set with a dual encoder (MLP spectrum encoder, GNN scaffold encoder) trained contrastively.

## When NOT to use

- When the true scaffold is already known (MADGEN's oracle retrieval setting uses a lookup table that always yields the correct scaffold given the spectrum and chemical formula, so contrastive ranking is unnecessary)
- When the goal is the generation stage itself — scaffold-conditioned de novo molecule generation with spectra guidance (MADGEN stage 2) — rather than scoring/ranking scaffold candidates
- When candidate molecules cannot be converted to molecular graphs/scaffolds with RDKit, since the scaffold candidate set is built entirely from RDKit-extracted scaffolds

## Inputs

- Preprocessed MassSpecGym dataset (msgym.pkl from Zenodo record 15036069)
- scaffold_retrieval datasets (massspecgym.zip or canopus.zip from Zenodo record 17115068)
- Candidate molecular scaffolds extracted from molecular graphs with RDKit (target molecules removed from the pool)
- Preprocessed MS/MS spectra (intensity-normalized, low-intensity peaks removed)

## Outputs

- Ranked candidate scaffolds per test spectrum (e.g., ranks_msgym_pred.pkl)
- Most frequently occurring scaffold per chemical formula as the scaffold prediction
- Scaffold prediction accuracy (SPA) on the held-out test split

## How to apply

Extract the scaffold of each candidate molecule from its molecular graph with RDKit to build the scaffold candidate set, explicitly removing target molecules from the pool. Jointly train an MLP encoder for the (preprocessed, intensity-normalized) MS/MS spectra and a GNN encoder for scaffold graphs with a temperature-scaled contrastive objective that pulls matched spectrum–scaffold pairs together and pushes mismatched pairs apart, using AdamW with learning rate 1e-5. Score every candidate scaffold per spectrum by cosine similarity between the two embeddings in the shared d-dimensional latent space. Then apply frequency-based aggregation: retrieve the top-k ranked candidate scaffolds per data point, compute scaffold frequencies per chemical formula, and select the most frequently occurring scaffold as the prediction. Serialize the ranked candidates per test spectrum (e.g., ranks_msgym_pred.pkl) and judge retrieval quality by scaffold prediction accuracy (SPA) on the held-out test split.

## Related tools

- **MADGEN** (Framework containing the scaffold_retrieval module (train_msgym.py / train_canopus.py for contrastive training, cand_rank_msg.py / cand_rank_canopus.py for candidate ranking)) — https://github.com/HassounLab/MADGEN
- **RDKit** (Extracts the scaffold from each candidate molecule's molecular graph representation to build the scaffold candidate set and lookup table)
- **PyTorch** (Implements and trains the MLP spectrum encoder and GNN scaffold encoder under the temperature-scaled contrastive objective)
- **AdamW** (Optimizer for contrastive training of the dual encoders (learning rate 1e-5))

## Examples

```
cd scaffold_retrieval/ && python train_msgym.py; python cand_rank_msg.py
```

## Evaluation signals

- Scaffold prediction accuracy (SPA) computed on the held-out test split is the stated success criterion for retrieval quality
- Every candidate scaffold per spectrum has a cosine-similarity score between the spectrum embedding and scaffold embedding in the shared d-dimensional latent space
- Frequency aggregation invariant: for each data point the top-k ranked candidate scaffolds are retrieved and scaffold frequencies are computed per chemical formula, with the most frequent scaffold selected
- Candidate-pool integrity check: target molecules are removed from the scaffold candidate set before ranking
- Output artifact check: ranked candidate scaffolds per test spectrum are serialized (e.g., ranks_msgym_pred.pkl), matching the pre-computed retrieval files hosted on Zenodo record 17115068

## Limitations

- The frequency-based aggregation is a heuristic post-processing step over the top-k ranked scaffolds and may fail when the correct scaffold is consistently outranked across the top-k
- The NIST dataset is commercially available and requires separate licensing, so contrastive retrieval training on NIST23 is gated externally
- The article's discussion section provides no text in the available source, so caveats on retrieval failure modes must be inferred from results rather than explicit discussion

## Evidence
