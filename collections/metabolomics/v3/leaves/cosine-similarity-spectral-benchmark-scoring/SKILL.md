---
name: cosine-similarity-spectral-benchmark-scoring
description: Use when comparing predicted spectra from C2MS models (FraGNNet, ICEBERG,
  GrAFF-MS, MassFormer, NEIMS, Precursor-Only) against real [M+H]+ Orbitrap HCD MS/MS
  test spectra to fill a Table 1-style prediction-error benchmark.
when_to_use_negative:
- When the task is retrieval-based MS2C compound identification (Top-k candidate ranking)
  rather than direct spectrum-prediction error scoring
- When spectra come from unmerged acquisitions, different instrument platforms, or
  other precursor adducts — the paper states performance is not yet evaluated under
  broader experimental conditions
- When evaluating formula annotation as a classification problem; that task excludes
  glycan/peptide spectra and uses a different accuracy metric, not C_BIN/C_HUN
tools:
- name: FraGNNet
  role: Source of predicted spectra (fragment distribution converted to spectrum via
    formula masses) that are scored against real spectra with C_BIN/C_HUN
  repo: github.com/FraGNNet/fragnnet
- name: PyTorch
  role: Implementation framework (version 2.1 with CUDA 11.8) in which FraGNNet and
    the baselines being scored are run
- name: PyTorch Lightning
  role: Training/experiment framework for the checkpoints that produce the predicted
    spectra under evaluation
  repo: Lightning-AI/pytorch-lightning
- name: RDKit
  role: Data preprocessing and molecule featurization used to prepare the test molecules
    and split assignments (InChIKey, scaffold, Tanimoto-based candidate sets)
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:70d5bbd127c1eef9737ded5b91b16c043aed6235fd921acc9d9742122e25c24b
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# cosine-similarity-spectral-benchmark-scoring

## Summary

Benchmark C2MS spectrum-prediction models by scoring each predicted spectrum against the real NIST20 MS/MS test spectrum using binned cosine similarity (C_BIN, 0.01 Da bins) and Hungarian-matched cosine similarity (C_HUN, 10 ppm peak-matching tolerance), reported as mean ± standard deviation over 5 seeds on InChIKey and scaffold splits.

## When to use

Use when comparing predicted spectra from C2MS models (FraGNNet, ICEBERG, GrAFF-MS, MassFormer, NEIMS, Precursor-Only) against real [M+H]+ Orbitrap HCD MS/MS test spectra to fill a Table 1-style prediction-error benchmark.

## When NOT to use

- When the task is retrieval-based MS2C compound identification (Top-k candidate ranking) rather than direct spectrum-prediction error scoring
- When spectra come from unmerged acquisitions, different instrument platforms, or other precursor adducts — the paper states performance is not yet evaluated under broader experimental conditions
- When evaluating formula annotation as a classification problem; that task excludes glycan/peptide spectra and uses a different accuracy metric, not C_BIN/C_HUN

## Inputs

- Held-out NIST20 MS/MS test spectra ([M+H]+ adduct, Orbitrap HCD instrument)
- Trained model checkpoints for FraGNNet-D3, FraGNNet-D4, and baselines (ICEBERG, GrAFF-MS, MassFormer, NEIMS, Precursor-Only)
- Predicted spectra per test molecule from each model
- Split assignments (InChIKey split and scaffold split) for the test set

## Outputs

- spectrum_prediction_benchmark.csv with per-model, per-split C_BIN and C_HUN scores as mean ± standard deviation over 5 seeds
- Table 1-equivalent benchmark table with best-per-column values bolded

## How to apply

Load the held-out NIST20 MS/MS test spectra ([M+H]+ Orbitrap HCD) and trained checkpoints from the github.com/FraGNNet/fragnnet codebase, then generate a predicted spectrum for every test molecule with each model (FraGNNet-D3/D4 via its Cython fragmentation DAG with depth d in {3,4} and hydrogen tolerance j=4, ICEBERG via its fragment-generator/intensity-predictor, GrAFF-MS over its static top-10,000 formula library, MassFormer/NEIMS via binned-output networks, and Precursor-Only as a single precursor peak). Score each predicted-vs-real pair with C_BIN using 0.01 Da binning and with C_HUN using 10 ppm peak-matching tolerance, mirroring the paper's 10 ppm cutoff that excluded annotations with formula masses more than 10 ppm from the peak mass. Compute C_BIN and C_HUN separately for the InChIKey split and the more challenging scaffold split. Repeat the full evaluation over 5 random seeds and report mean ± standard deviation per model per split, bolding the best score. Validate against the reported anchors, e.g. FraGNNet-D4 C_BIN of 0.736 ± 0.002 (InChIKey) and 0.709 ± 0.002 (scaffold), with FraGNNet-D4 outperforming all baselines.

## Related tools

- **FraGNNet** (Source of predicted spectra (fragment distribution converted to spectrum via formula masses) that are scored against real spectra with C_BIN/C_HUN) — github.com/FraGNNet/fragnnet
- **PyTorch** (Implementation framework (version 2.1 with CUDA 11.8) in which FraGNNet and the baselines being scored are run)
- **PyTorch Lightning** (Training/experiment framework for the checkpoints that produce the predicted spectra under evaluation) — Lightning-AI/pytorch-lightning
- **RDKit** (Data preprocessing and molecule featurization used to prepare the test molecules and split assignments (InChIKey, scaffold, Tanimoto-based candidate sets))

## Evaluation signals

- FraGNNet-D4 reaches C_BIN of 0.736 ± 0.002 on the InChIKey split and 0.709 ± 0.002 on the scaffold split, matching the reported validation anchors
- FraGNNet-D4 outperforms all baselines (ICEBERG, GrAFF-MS, MassFormer, NEIMS, Precursor-Only) on both C_BIN and C_HUN under both splits
- Scores are reported as mean ± standard deviation across 5 random seeds with the best value per column bolded
- C_BIN uses 0.01 Da bins and C_HUN uses 10 ppm peak-matching tolerance, consistent with the paper's 10 ppm annotation-exclusion cutoff
- Metrics are computed separately per split, with the scaffold split expected to yield lower scores than the InChIKey split

## Limitations

- The fragmentation algorithm fails to account for some fragments, evidenced by unexplained Outside-of-Support (OS) peaks in the spectra, which depresses similarity scores
- Unexplained peaks may arise from complex chemistry such as cyclizations that dramatically expand the fragment search space and are challenging to model
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts)

## Evidence

- [methods] C_BIN 0.01 Da binning and C_HUN tolerance: [evidence span withheld — anchor status: not_found]
- [methods] NIST20 held-out evaluation set: [evidence span withheld — anchor status: not_found]
- [abstract] state-of-the-art claim to validate against: "FraGNNet achieves state-of-the-art performance in terms of prediction error and surpasses existing C2MS models as a tool for retrieval-based MS2C."
- [supplementary] instrument/adduct scope of the scored spectra: "Orbitrap instrument with higher-energy collisional dissociation (HCD)"
