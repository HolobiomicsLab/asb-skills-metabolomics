---
name: inchikey-scaffold-dataset-split-evaluation
description: Use when benchmarking C2MS (compound-to-spectrum) models like FraGNNet,
  ICEBERG, GrAFF-MS, MassFormer, NEIMS, or Precursor-Only on MS/MS prediction or retrieval
  tasks and you need generalization-controlled splits with spectral similarity metrics.
when_to_use_negative:
- Evaluating formula annotation as a classification task rather than spectrum prediction
  — the paper poses formula annotation separately with its own 10 ppm annotation-exclusion
  filter
- Test spectra from instrument platforms, precursor adducts, or unmerged spectra outside
  [M+H]+ Orbitrap HCD — the authors note performance is not yet evaluated under broader
  experimental conditions
- Glycan or peptide spectra — these were excluded from the paper's small-molecule
  analyses
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: Python
  role: Implementation language for the evaluation pipeline
- name: PyTorch
  role: Framework in which FraGNNet and baselines were implemented and from which
    predictions are generated
  repo: https://github.com/Lightning-AI/pytorch-lightning
- name: RDKit
  role: Data preprocessing, molecule featurization, and scaffold computation for splits
- name: FraGNNet
  role: Primary model under evaluation (recursive bond-breaking fragmentation DAG
    + two-stage GNN)
  repo: https://github.com/FraGNNet/fragnnet
- name: GrAFF-MS
  role: Baseline C2MS model evaluated over its static formula library
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:65614f3dcbc0602908a0244c5a578098d8b3ed77442ca5d7a1ed6563d2bacbe5
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Lightning-AI/pytorch-lightning
  tools:
  - Cython
  - FraGNNet
  - GrAFF-MS
  - PyTorch
  - PyTorch Geometric
  - PyTorch Lightning
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# inchikey-scaffold-dataset-split-evaluation

## Summary

Evaluate MS/MS spectrum prediction models on held-out NIST20 test spectra under two complementary splits — an InChIKey split (blocking whole compounds) and a harder scaffold split — scoring binned cosine (C_BIN, 0.01 Da bins) and Hungarian cosine (C_HUN, 10 ppm matching tolerance) over 5 random seeds.

## When to use

Use when benchmarking C2MS (compound-to-spectrum) models like FraGNNet, ICEBERG, GrAFF-MS, MassFormer, NEIMS, or Precursor-Only on MS/MS prediction or retrieval tasks and you need generalization-controlled splits with spectral similarity metrics.

## When NOT to use

- Evaluating formula annotation as a classification task rather than spectrum prediction — the paper poses formula annotation separately with its own 10 ppm annotation-exclusion filter
- Test spectra from instrument platforms, precursor adducts, or unmerged spectra outside [M+H]+ Orbitrap HCD — the authors note performance is not yet evaluated under broader experimental conditions
- Glycan or peptide spectra — these were excluded from the paper's small-molecule analyses

## Inputs

- Held-out NIST20 MS/MS test spectra ([M+H]+ Orbitrap HCD)
- Trained model checkpoints: FraGNNet-D4, FraGNNet-D3, ICEBERG, GrAFF-MS, MassFormer, NEIMS, Precursor-Only
- Molecule structures/identifiers supporting InChIKey-based and scaffold-based splits (RDKit-featurized)
- Scaffold assignment per test compound (e.g. via RDKit)

## Outputs

- spectrum_prediction_benchmark.csv with mean ± std similarity scores per model per split
- Per-model, per-split C_BIN (0.01 Da binned cosine) and C_HUN (10 ppm Hungarian cosine) scores
- Table 1-equivalent comparison with best score bolded

## How to apply

Load the held-out NIST20 MS/MS test spectra ([M+H]+ adduct, Orbitrap HCD) and trained checkpoints for FraGNNet-D3/D4 and all baselines from the github.com/FraGNNet/fragnnet codebase, then predict spectra for every test molecule under the InChIKey split and the more challenging scaffold split. Score each predicted spectrum against the real spectrum with binned cosine similarity C_BIN using 0.01 Da bins, and Hungarian cosine similarity C_HUN using a 10 ppm peak-matching tolerance, computing each metric separately per split. Report mean ± standard deviation over 5 random seeds per model per split, and bold the best score in the resulting Table 1-equivalent (expected FraGNNet-D4 reference: C_BIN 0.736 ± 0.002 on the InChIKey split and 0.709 ± 0.002 on the scaffold split, outperforming all baselines). Write the results to spectrum_prediction_benchmark.csv for validation against reported values.

## Related tools

- **Python** (Implementation language for the evaluation pipeline)
- **PyTorch** (Framework in which FraGNNet and baselines were implemented and from which predictions are generated) — https://github.com/Lightning-AI/pytorch-lightning
- **RDKit** (Data preprocessing, molecule featurization, and scaffold computation for splits)
- **FraGNNet** (Primary model under evaluation (recursive bond-breaking fragmentation DAG + two-stage GNN)) — https://github.com/FraGNNet/fragnnet
- **GrAFF-MS** (Baseline C2MS model evaluated over its static formula library)

## Examples

```
bash env/env_setup_gpu_py311_cu121_250602.sh && conda activate FRAGNNET-GPU && python scripts/evaluate.py --checkpoint fragnnet_d4.pt --split inchikey,scaffold
```

## Evaluation signals

- FraGNNet-D4 reproduces C_BIN of 0.736 ± 0.002 on the InChIKey split and 0.709 ± 0.002 on the scaffold split
- FraGNNet-D4 outperforms all baseline models on both metrics and both splits
- Metrics are computed separately under InChIKey and scaffold splits, with mean ± standard deviation across 5 random seeds
- spectrum_prediction_benchmark.csv is produced and matches the reported table values
- C_HUN peak matching uses a 10 ppm tolerance and C_BIN uses 0.01 Da bins consistently across models

## Limitations

- Scaffold split is substantially more challenging than the InChIKey split (scores drop, e.g. 0.736 to 0.709 for FraGNNet-D4), so single-split results overstate generalization
- The fragmentation algorithm fails to account for some fragments, as evidenced by unexplained Outside-of-Support (OS) peaks, possibly from complex chemistry such as cyclizations that expand the fragment search space
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts)

## Evidence

- [other] Load the held-out NIST20 MS/MS test spectra ([M+H]+ Orbitrap HCD) and the trained checkpoints for FraGNNet-D4, FraGNNet-D3, and the baselines from the github.com/FraGNNet/fragnnet codebase.: [evidence span withheld — anchor status: not_found]
- [readme] This repository contains the code and models for the paper FraGNNet: A Deep Probabilistic Model for Mass Spectrum Prediction.: [evidence span withheld — anchor status: external_doc]
