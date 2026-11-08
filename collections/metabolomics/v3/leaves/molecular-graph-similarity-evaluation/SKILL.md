---
name: molecular-graph-similarity-evaluation
description: Use when scoring a set of candidate molecules generated per spectrum
  (or per query) against a known reference molecule, e.g. after sampling 100 structures
  per spectrum from a generative model and ranking by frequency.
when_to_use_negative:
- Input is already an aggregated metrics table rather than per-spectrum ranked candidate
  lists — no re-ranking or metric computation is possible
- The task is spectral matching against a spectral library (retrieval/identification)
  rather than de novo structure generation, where MCES/Tanimoto against a single ground-truth
  structure is not defined
- Candidate molecules cannot be sanitized by RDKit (invalid or disconnected) and have
  not yet passed the invalid/disconnected filter step — including them corrupts the
  top-k ranking
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Molecule sanitization (removing invalid/disconnected candidates), Morgan fingerprint
    computation, and Tanimoto similarity calculation
  repo: https://github.com/coleygroup/DiffMS
- name: DiffMS
  role: Source of generated candidate molecules per spectrum and of the metric protocol
    (Top-k accuracy, min MCES, max Tanimoto)
  repo: https://github.com/coleygroup/DiffMS
- name: DiGress
  role: Provides the orca.cpp MCES computation used for graph-based molecular similarity,
    compiled to the orca binary
  repo: https://github.com/cvignac/DiGress
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:70e8d63bdf87ed5a3c3a7680cf0f098f731be006a131ca08283c3cf490b28686
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - DiGress
  - DiffMS
  - MIST
  - PyTorch
  - RDKit
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# molecular-graph-similarity-evaluation

## Summary

Evaluates de novo generated molecular structures against ground-truth molecules using Top-1/Top-10 accuracy plus two structure-aware graph similarity metrics: minimum MCES distance and maximum Tanimoto similarity. This is the standard evaluation protocol DiffMS applies after frequency-ranking sampled molecules on NPLIB1 and MassSpecGym.

## When to use

Use when scoring a set of candidate molecules generated per spectrum (or per query) against a known reference molecule, e.g. after sampling 100 structures per spectrum from a generative model and ranking by frequency.

## When NOT to use

- Input is already an aggregated metrics table rather than per-spectrum ranked candidate lists — no re-ranking or metric computation is possible
- The task is spectral matching against a spectral library (retrieval/identification) rather than de novo structure generation, where MCES/Tanimoto against a single ground-truth structure is not defined
- Candidate molecules cannot be sanitized by RDKit (invalid or disconnected) and have not yet passed the invalid/disconnected filter step — including them corrupts the top-k ranking

## Inputs

- Frequency-ranked top-k candidate molecules (SMILES) per spectrum, sampled from a generative model (e.g. 100 samples per spectrum with invalid/disconnected molecules removed)
- Ground-truth reference molecules for each spectrum from the benchmark dataset (NPLIB1/CANOPUS or MassSpecGym)
- Compiled orca binary for MCES distance computation

## Outputs

- Top-1 and Top-10 accuracy per model variant
- Top-1 and Top-10 minimum MCES distance per model variant
- Top-1 and Top-10 maximum Tanimoto similarity per model variant
- Aggregated metrics comparison table (e.g. prior_ablation_metrics.csv)

## How to apply

For each spectrum, take the frequency-ranked top-k candidate molecules (after removing invalid or disconnected molecules via RDKit sanitization) and compute exact-match Top-1 and Top-10 accuracy against the ground-truth molecule. For ranked candidates that are not exact matches, compute the minimum MCES distance to the reference (top-1 and top-10), which quantifies the smallest bond-number difference between molecular graphs, and the maximum Tanimoto similarity (top-1 and top-10) over Morgan fingerprints. MCES computation requires the compiled orca binary from the DiGress codebase (g++ compile of orca.cpp); Tanimoto is computed with RDKit. Aggregate all six metrics (Top-1/Top-10 accuracy, Top-1/Top-10 min MCES, Top-1/Top-10 max Tanimoto) into a comparison table, e.g. mirroring DiffMS Table 6 for the adjacency-prior ablation where the dataset-marginal prior should give the best accuracy and lowest MCES.

## Related tools

- **RDKit** (Molecule sanitization (removing invalid/disconnected candidates), Morgan fingerprint computation, and Tanimoto similarity calculation) — https://github.com/coleygroup/DiffMS
- **DiffMS** (Source of generated candidate molecules per spectrum and of the metric protocol (Top-k accuracy, min MCES, max Tanimoto)) — https://github.com/coleygroup/DiffMS
- **DiGress** (Provides the orca.cpp MCES computation used for graph-based molecular similarity, compiled to the orca binary) — https://github.com/cvignac/DiGress

## Examples

```
g++ -O2 -std=c++11 -o orca orca.cpp; from rdkit import Chem, DataStructs; sim = DataStructs.TanimotoSimilarity(fp_gen, fp_true)
```

## Evaluation signals

- All six metrics present per model variant: Top-1/Top-10 accuracy, Top-1/Top-10 min MCES distance, Top-1/Top-10 max Tanimoto similarity
- Accuracy increases monotonically with k (Top-10 accuracy >= Top-1 accuracy) and min MCES decreases with k (Top-10 min MCES <= Top-1 min MCES)
- Invalid or disconnected molecules are excluded before frequency-based top-k selection — candidate counts per spectrum reflect only RDKit-valid, connected structures
- In the prior-distribution ablation, the dataset-marginal prior variant should yield the best accuracy and lowest MCES relative to the empty and fully-connected single-bond priors
- Tanimoto similarity values fall in [0, 1]; MCES distances are non-negative integers counting bond differences

## Limitations

- Exact top-1 structural identity is rarely achieved de novo; structurally close matches (low MCES, high Tanimoto) are the realistic outcome, so metrics must be read jointly rather than accuracy alone
- MCES depends on a correctly compiled orca binary (g++ -O2 -std=c++11 -o orca orca.cpp); failure to compile yields missing distances
- Results depend on the pre-sampling filter (removal of invalid/disconnected molecules) and on the sampling count (100 per spectrum), so changes to those upstream parameters change the metrics non-comparably

## Evidence

- [methods] Sampling and filtering protocol: [evidence span withheld — anchor status: not_found]
