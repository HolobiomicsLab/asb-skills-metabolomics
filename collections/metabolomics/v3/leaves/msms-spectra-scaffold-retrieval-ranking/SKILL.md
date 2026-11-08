---
name: msms-spectra-scaffold-retrieval-ranking
description: Use when a de novo molecular structure generation pipeline needs to select
  a molecular scaffold from a candidate set given an MS/MS spectrum, either predictively
  (contrastive ranking) or via an oracle scaffold lookup keyed by spectrum and chemical
  formula.
when_to_use_negative:
- When NIST23 data is required but not separately licensed — only CANOPUS and MassSpecGym
  processed pickles are openly downloadable from Zenodo
- When the task is full molecule generation without a scaffold constraint — this skill
  only solves the retrieval/ranking stage, not spectra-to-structure generation
- When spectra lack chemical formula annotations, since the oracle lookup requires
  both the MS/MS spectrum and the chemical formula
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MADGEN
  role: Two-stage framework whose first stage is contrastive scaffold retrieval ranking;
    retrieved scaffold seeds the second-stage spectra-guided generation
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit
  role: Extracts scaffolds from molecular graph representations to build the oracle
    lookup table (rdkit=2023.09.5 in the madgen conda env)
- name: PyTorch
  role: Implements and trains the contrastive scaffold-ranking neural network in the
    scaffold_retrieval module
- name: wget
  role: Downloads processed dataset pickles and pre-computed rank files from Zenodo
    records 15036069 and 17115068
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:48b4efd921634fa94a45e0e21844e790bff869fa0ea8fc34d119e0e627043c6f
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
  - RDKit
  - conda
  - pip
  - wget
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# msms-spectra-scaffold-retrieval-ranking

## Summary

Formulates scaffold retrieval from MS/MS spectra as a ranking problem using contrastive learning to align spectra with candidate molecular scaffolds, providing the scaffold seed for MADGEN's spectra-guided de novo molecule generation.

## When to use

Use when a de novo molecular structure generation pipeline needs to select a molecular scaffold from a candidate set given an MS/MS spectrum, either predictively (contrastive ranking) or via an oracle scaffold lookup keyed by spectrum and chemical formula.

## When NOT to use

- When NIST23 data is required but not separately licensed — only CANOPUS and MassSpecGym processed pickles are openly downloadable from Zenodo
- When the task is full molecule generation without a scaffold constraint — this skill only solves the retrieval/ranking stage, not spectra-to-structure generation
- When spectra lack chemical formula annotations, since the oracle lookup requires both the MS/MS spectrum and the chemical formula

## Inputs

- Preprocessed MS/MS spectra pickles (msgym.pkl, canopus.pkl from Zenodo record 15036069)
- Scaffold candidate set S derived from candidate molecules
- Oracle scaffold lookup table keyed by spectrum and chemical formula (built with RDKit)
- Pre-trained MADGEN checkpoints (Zenodo record 17115068)
- Pre-computed predictive rank files (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl)

## Outputs

- Ranked scaffold candidate lists per spectrum (top-k scaffolds)
- Retrieved/known scaffold SMILES consumed by scaffold-conditioned generation
- Prediction CSVs with true, pred, and scaffold columns for downstream Top-K accuracy and Tanimoto evaluation

## How to apply

Given a set of scaffold candidates S, score each candidate with a neural network trained via contrastive learning to align mass spectra with scaffolds, and rank candidates to retrieve the top scaffold. For the oracle setting, maintain a lookup table built by extracting the scaffold from the molecular graph representation with RDKit (conda env: python=3.9, rdkit=2023.09.5), keyed so the correct scaffold is always returned given the MS/MS spectrum and chemical formula. For the predictive setting, either train via scaffold_retrieval/train_msgym.py or train_canopus.py and rank with cand_rank_msg.py / cand_rank_canopus.py, or reuse the pre-computed rank files ranks_msgym_pred.pkl and ranks_canopus_pred.pkl from Zenodo record 17115068. The retrieved scaffold is then fed into scaffold-conditioned generation with classifier-free spectral guidance (e.g. sample.py with --n_steps 100, --mode test, --sampling_seed 42). Scaffold correctness dramatically drives downstream accuracy: 49.0% top-1 on NIST23 with the oracle retriever versus 4.6% top-1 with the predictive retriever.

## Related tools

- **MADGEN** (Two-stage framework whose first stage is contrastive scaffold retrieval ranking; retrieved scaffold seeds the second-stage spectra-guided generation) — https://github.com/HassounLab/MADGEN
- **RDKit** (Extracts scaffolds from molecular graph representations to build the oracle lookup table (rdkit=2023.09.5 in the madgen conda env))
- **PyTorch** (Implements and trains the contrastive scaffold-ranking neural network in the scaffold_retrieval module)
- **wget** (Downloads processed dataset pickles and pre-computed rank files from Zenodo records 15036069 and 17115068)

## Examples

```
cd scaffold_retrieval/ && python train_msgym.py && python cand_rank_msg.py; cd data/msgym/raw/ && python preprocess_rank.py
```

## Evaluation signals

- Reproduced top-1 accuracy should match published values: 49.0% top-1 on NIST23 with the oracle retriever and 2.10% top-1 on CANOPUS/MassSpecGym with the predictive retriever
- Oracle setting invariant: the lookup table must always return the correct scaffold given the spectrum and chemical formula
- Frequency-based aggregation over top-k ranked candidate scaffolds should be applied consistently across data points
- Downstream evaluation_generation.py Top-K accuracy (InChI key matching) and Maximum Tanimoto Similarity (Morgan fingerprints radius 2, 2048 bits) should be computable from the assembled CSVs with true, pred, and scaffold columns
- Scaffold coverage (percentage of valid, non-disconnected scaffold predictions) should be reported alongside accuracy

## Limitations

- NIST23 is commercially available and requires separate licensing; it is not included in the Zenodo data record
- With the predictive retriever, top-1 accuracy drops dramatically (e.g. 4.6% on NIST23), so scaffold retrieval quality is the dominant bottleneck for end-to-end performance
- MSNovelist baseline results are limited to accuracy metrics; similarity and MCES measures are not available

## Evidence

- [results] oracle vs predictive accuracy: [evidence span withheld — anchor status: not_found]
