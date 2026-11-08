---
name: tanimoto-similarity-computation
description: Use when comparing de novo EI-MS spectral translation outputs against
  database-search candidates on the same test spectra, using Morgan-fingerprint Tanimoto
  similarity as the ranking criterion.
when_to_use_negative:
- The input is raw spectra (e.g. an msp file) rather than already-decoded SMILES candidates
  — use the inference/translation skill first
- The task is MS/MS de novo annotation — the article notes all existing de novo comparison
  approaches were designed and trained for MS/MS spectra, not GC-EI-MS, so the comparison
  is not straightforward
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: SpecTUS
  role: Repository containing the evaluation notebooks, deposited predictions, NIST
    splits, and ground-truth SMILES needed to compute Morgan-fingerprint Tanimoto
    comparisons
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Runs the repository's model-evaluation and model-comparison notebooks that
    implement the top-k accuracy and Tanimoto win-rate calculations
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Creates the Python environment from trainSpectus_build.yaml before running
    the evaluation notebooks
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:9919fc401096786243fa058d13c5220731feaffa02485645165cd34c11f48bd7
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# tanimoto-similarity-computation

## Summary

Computes Tanimoto similarity of Morgan fingerprints between a ground-truth molecular structure (SMILES) and candidate structures predicted de novo or retrieved by database search (HSS/SSS/BDC), and aggregates these into top-k accuracy and win-rate metrics for SpecTUS-style model evaluation.

## When to use

Use when comparing de novo EI-MS spectral translation outputs against database-search candidates on the same test spectra, using Morgan-fingerprint Tanimoto similarity as the ranking criterion.

## When NOT to use

- The input is raw spectra (e.g. an msp file) rather than already-decoded SMILES candidates — use the inference/translation skill first
- The task is MS/MS de novo annotation — the article notes all existing de novo comparison approaches were designed and trained for MS/MS spectra, not GC-EI-MS, so the comparison is not straightforward

## Inputs

- Ground-truth SMILES strings for the 28,267-spectrum NIST 20 held-out test split
- Deposited predictions from predictions/ for SpecTUS models and database search methods (HSS, SSS, BDC)
- NIST 20 test split assignment files from data/

## Outputs

- spectus_vs_hss_sss_metrics.csv with Acc_k and average top-k Tanimoto similarity for k = 1, 10, 50
- SpecTUS-vs-HSS win rate and at-least-as-good rate at k = 1 and k = 10

## How to apply

Set up the SpecTUS environment with `conda env create -f trainSpectus_build.yaml`, then load the deposited predictions in the repository's `predictions/` directory together with the NIST 20 test split (28,267 spectra) and ground-truth SMILES from `data/`. For each test spectrum, parse SMILES into RDKit molecules, compute Morgan fingerprints, and derive the Tanimoto similarity between ground truth and each candidate. For each k (1, 10, 50), compute Acc_k (fraction of spectra whose exact structure appears in the top-k candidates) and average top-k similarity (highest Tanimoto similarity among the first k candidates). Treat an exact structural match as the best possible outcome; otherwise compare SpecTUS's best candidate against the HSS result by Tanimoto similarity to determine win and at-least-as-good rates. Write all recomputed metrics to `spectus_vs_hss_sss_metrics.csv` and check them against the reported headline numbers.

## Related tools

- **SpecTUS** (Repository containing the evaluation notebooks, deposited predictions, NIST splits, and ground-truth SMILES needed to compute Morgan-fingerprint Tanimoto comparisons) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Runs the repository's model-evaluation and model-comparison notebooks that implement the top-k accuracy and Tanimoto win-rate calculations) — https://github.com/hejjack/SpecTUS
- **conda** (Creates the Python environment from trainSpectus_build.yaml before running the evaluation notebooks)

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && cd SpecTUS && conda env create -f trainSpectus_build.yaml && jupyter notebook notebooks/
```

## Evaluation signals

- Acc1 on the NIST 20 test split reproduces at approximately 0.43
- Acc10 reproduces at approximately 0.65
- SpecTUS-vs-HSS win rate at k = 1 is approximately 0.76
- SpecTUS-vs-HSS win rate at k = 10 is approximately 0.84
- HSS single-candidate success lands around 20%

## Limitations

- Tanimoto of Morgan fingerprints is a structural-similarity proxy; only exact structural match counts as perfect reconstruction, so high similarity does not guarantee correct identification
- Reproducing training or re-running predictions requires a license for the proprietary NIST20 dataset; only deposited predictions, splits, and synthetic data are freely available
- Model performance is sensitive to input spectrum quality — performance drops on the less rigorously curated Cayman and MONA test sets compared to NIST or SWGDRUG

## Evidence
