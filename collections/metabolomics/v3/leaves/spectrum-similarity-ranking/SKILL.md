---
name: spectrum-similarity-ranking
description: Use when you have a real MS/MS test spectrum plus candidate molecules
  (e.g. a 50-compound PubChem set built by Tanimoto similarity) and must rank candidates
  by spectral similarity to compute Top-1/3/5 identification accuracy.
when_to_use_negative:
- You are doing formula annotation rather than retrieval-based compound identification
  — the paper poses formula annotation as a classification problem, a separate task
- The unknown spectrum can be matched directly against a real spectral library and
  library coverage is sufficient; C2MS-augmented retrieval targets molecules missing
  from libraries
- Your spectra come from instruments or precursor adducts outside the training scope
  (the models were trained/evaluated on Orbitrap HCD [M+H]+ data and performance under
  broader conditions is unvalidated)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: FraGNNet
  role: C2MS model whose predicted spectra are scored against real MS/MS spectra to
    rank the 50-compound candidate sets; the best-performing retriever
  repo: github.com/FraGNNet/fragnnet
- name: RDKit
  role: Computes the chemical fingerprints whose Tanimoto similarity selects the 49
    decoy PubChem molecules for each candidate set
- name: PyTorch
  role: Implementation framework for loading the models and running batch spectrum
    prediction over candidate sets
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:485e087f357db7b2799af16d75657401a947c2aaefee9c1612ba6a04f0fd04f1
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - PyTorch
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# spectrum-similarity-ranking

## Summary

Ranks candidate compounds against an unknown MS/MS spectrum by scoring predicted or reference spectra with spectral-similarity metrics (binned cosine, Hungarian cosine at 10 ppm tolerance), enabling retrieval-based compound identification (MS2C) with Top-k accuracy.

## When to use

Use when you have a real MS/MS test spectrum plus candidate molecules (e.g. a 50-compound PubChem set built by Tanimoto similarity) and must rank candidates by spectral similarity to compute Top-1/3/5 identification accuracy.

## When NOT to use

- You are doing formula annotation rather than retrieval-based compound identification — the paper poses formula annotation as a classification problem, a separate task
- The unknown spectrum can be matched directly against a real spectral library and library coverage is sufficient; C2MS-augmented retrieval targets molecules missing from libraries
- Your spectra come from instruments or precursor adducts outside the training scope (the models were trained/evaluated on Orbitrap HCD [M+H]+ data and performance under broader conditions is unvalidated)

## Inputs

- ~4,000 held-out NIST20 MS/MS test spectrum-molecule pairs
- PubChem molecules for building 50-compound candidate sets (1 true compound + 49 Tanimoto-nearest neighbors)
- Predicted MS/MS spectra from C2MS models (FraGNNet-D4, ICEBERG, MassFormer, NEIMS)
- RDKit chemical fingerprints for candidate-set construction
- Spectral-similarity metrics: binned cosine, Hungarian cosine (10 ppm tolerance), with/without square-root intensity transformation

## Outputs

- Ranking of the 50 candidates per test molecule per model, with the rank of the true compound
- Top-1/Top-3/Top-5 accuracy percentages (mean ± std over 5 random seeds) per model on InChIKey and Scaffold splits
- Per-model record of which spectral-similarity ranking metric yielded the best result

## How to apply

For each held-out test molecule X_i from the ~4,000 NIST20 MS/MS pairs, construct a candidate set C_i = {X_i} ∪ 49 PubChem molecules chosen by highest Tanimoto similarity on RDKit chemical fingerprints. Predict an MS/MS spectrum for every candidate with each C2MS model (FraGNNet-D4, ICEBERG, MassFormer, NEIMS), then score each predicted spectrum against the real spectrum Y_i using multiple similarity metrics — binned cosine and Hungarian cosine at 10 ppm tolerance, each with and without a square-root intensity transformation — and keep the best-ranking metric per model. Rank the 50 candidates, record the rank of the true compound, and report Top-1/Top-3/Top-5 accuracy as mean ± standard deviation over 5 random seeds on both InChIKey and Scaffold splits. Rationale: augmenting real spectral libraries with predicted spectra lifts retrieval accuracy; FraGNNet-D4 reaches Top-1 = 36.0 ± 0.4 (InChIKey split) and 30.3 ± 0.6 (Scaffold split), the reference values for validating a reproduction.

## Related tools

- **FraGNNet** (C2MS model whose predicted spectra are scored against real MS/MS spectra to rank the 50-compound candidate sets; the best-performing retriever) — github.com/FraGNNet/fragnnet
- **RDKit** (Computes the chemical fingerprints whose Tanimoto similarity selects the 49 decoy PubChem molecules for each candidate set)
- **PyTorch** (Implementation framework for loading the models and running batch spectrum prediction over candidate sets)

## Evaluation signals

- FraGNNet-D4 reproduces Table 2 values: Top-1 = 36.0 ± 0.4 on the InChIKey split and 30.3 ± 0.6 on the Scaffold split
- Every candidate set contains exactly 50 compounds: the true molecule plus its 49 highest-Tanimoto PubChem neighbors
- Accuracy is reported as mean ± standard deviation over 5 random seeds (candidate-set resampling), not a single draw
- The best spectral-similarity metric per model is recorded and compared across binned cosine vs Hungarian cosine at 10 ppm tolerance, with and without sqrt intensity transformation
- Ranking of FraGNNet-D4 relative to ICEBERG, MassFormer and NEIMS is consistent on both the InChIKey and Scaffold splits

## Limitations

- The fragmentation algorithm fails to account for some fragments, evidenced by unexplained Outside-of-Support (OS) peaks, which can depress similarity scores
- Unexplained peaks may arise from complex chemistry such as cyclizations that dramatically expand the fragment search space and are challenging to model
- Performance has not been evaluated on unmerged spectra or broader experimental conditions (different instrument platforms and precursor adducts)
- The recursive edge-removal fragmentation algorithm is not parallelized, limiting throughput when predicting spectra for large candidate libraries

## Evidence

- [methods] Each C_i in C_i - {X_i} is selected to have high chemical similarity with X_i as measured by Tanimoto similarity between chemical fingerprints: [evidence span withheld — anchor status: not_found]
- [methods] a candidate set C_i is constructed from X_i and 49 other molecules sampled from PubChem: [evidence span withheld — anchor status: not_found]
- [readme] This repository contains the code and models for the paper FraGNNet: A Deep Probabilistic Model for Mass Spectrum Prediction.: [evidence span withheld — anchor status: external_doc]
