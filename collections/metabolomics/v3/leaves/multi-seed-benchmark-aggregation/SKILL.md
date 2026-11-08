---
name: multi-seed-benchmark-aggregation
description: Use when a benchmark comparison (e.g., spectrum-prediction cosine scores
  across multiple models and data splits) must be repeated over multiple random seeds
  and reported as mean ± std to demonstrate stability of a claim.
when_to_use_negative:
- The goal is model training or hyperparameter sweeps rather than aggregating evaluation
  scores across seeds
- Only a single deterministic run exists and per-seed variance cannot be estimated
- The evaluation target is a different task output (e.g., retrieval-based MS2C top-k
  ranking or formula annotation classification) with its own metric rather than spectrum-prediction
  cosine scores
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FraGNNet
  role: Model under evaluation whose per-seed prediction error and retrieval performance
    are aggregated; state-of-the-art reference for validation
  repo: github.com/FraGNNet/fragnnet
- name: GrAFF-MS
  role: C2MS baseline over a static formula library whose per-seed scores are aggregated
    for comparison
- name: PyTorch
  role: Framework in which FraGNNet and the baselines are implemented for seed-repeat
    inference runs
- name: PyTorch Lightning
  role: Training/experiment framework for the models whose checkpoints are evaluated
    across seeds
  repo: Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:61b4b3c6867eae0489ab3ba167b970216c8d2f7a0ccecd54af7029113d9badb8
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - FraGNNet
  - GrAFF-MS
  - PyTorch
  - PyTorch Geometric
  - PyTorch Lightning
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# multi-seed-benchmark-aggregation

## Summary

Aggregates benchmark scores computed across 5 random seeds into mean ± standard deviation per model per split, as used to compare FraGNNet against C2MS baselines (ICEBERG, GrAFF-MS, MassFormer, NEIMS, Precursor-Only) on held-out NIST20 MS/MS spectra.

## When to use

Use when a benchmark comparison (e.g., spectrum-prediction cosine scores across multiple models and data splits) must be repeated over multiple random seeds and reported as mean ± std to demonstrate stability of a state-of-the-art claim.

## When NOT to use

- The goal is model training or hyperparameter sweeps rather than aggregating evaluation scores across seeds
- Only a single deterministic run exists and per-seed variance cannot be estimated
- The evaluation target is a different task output (e.g., retrieval-based MS2C top-k ranking or formula annotation classification) with its own metric rather than spectrum-prediction cosine scores

## Inputs

- Held-out NIST20 MS/MS test spectra ([M+H]+ Orbitrap HCD)
- Trained model checkpoints for FraGNNet-D4, FraGNNet-D3, and baselines (ICEBERG, GrAFF-MS, MassFormer, NEIMS, Precursor-Only) from github.com/FraGNNet/fragnnet
- InChIKey split and scaffold split definitions of the test set
- Per-seed binned cosine (C_BIN) and Hungarian cosine (C_HUN) scores

## Outputs

- spectrum_prediction_benchmark.csv with mean ± standard deviation per model per split
- Bolded best score per metric/split (Table 1-equivalent)

## How to apply

Run spectrum prediction for every test molecule with each model — FraGNNet-D4, FraGNNet-D3, and baselines — on the held-out NIST20 MS/MS test spectra ([M+H]+, Orbitrap HCD), repeating the full prediction-and-scoring procedure over 5 random seeds. For each seed, score each predicted spectrum against the real spectrum using binned cosine similarity C_BIN (0.01 Da bins) and Hungarian cosine similarity C_HUN (10 ppm peak-matching tolerance). Compute the metrics separately under the InChIKey split and the more challenging scaffold split rather than pooling them. Aggregate per-model per-split values as mean ± standard deviation across the 5 seeds, flagging the best score for bolding. Write the Table 1-equivalent results to a CSV (e.g., spectrum_prediction_benchmark.csv) and validate against reported values, e.g. FraGNNet-D4 C_BIN of 0.736 ± 0.002 (InChIKey) and 0.709 ± 0.002 (scaffold), with FraGNNet-D4 outperforming all baselines.

## Related tools

- **FraGNNet** (Model under evaluation whose per-seed prediction error and retrieval performance are aggregated; state-of-the-art reference for validation) — github.com/FraGNNet/fragnnet
- **GrAFF-MS** (C2MS baseline over a static formula library whose per-seed scores are aggregated for comparison)
- **PyTorch** (Framework in which FraGNNet and the baselines are implemented for seed-repeat inference runs)
- **PyTorch Lightning** (Training/experiment framework for the models whose checkpoints are evaluated across seeds) — Lightning-AI/pytorch-lightning

## Evaluation signals

- FraGNNet-D4 C_BIN equals 0.736 ± 0.002 on the InChIKey split and 0.709 ± 0.002 on the scaffold split
- Standard deviations across the 5 seeds are small (±0.002 scale), indicating stable per-seed scores
- FraGNNet-D4 outperforms all baselines on both C_BIN and C_HUN in both splits (bolded best score)
- Metrics are reported separately for InChIKey and scaffold splits, never pooled
- Output table matches the Table 1-equivalent structure when written to spectrum_prediction_benchmark.csv

## Limitations

- Evaluation is limited to merged spectra under a single experimental condition ([M+H]+ adduct, Orbitrap HCD); performance under broader instrument platforms and precursor adducts was not assessed
- The scaffold split is more challenging than the InChIKey split, so cross-split comparisons should not be conflated
- Aggregated cosine scores can hide per-seed failure modes such as unexplained Outside-of-Support (OS) peaks arising from complex chemistry like cyclizations

## Evidence

- [readme] Code availability: [evidence span withheld — anchor status: external_doc]
