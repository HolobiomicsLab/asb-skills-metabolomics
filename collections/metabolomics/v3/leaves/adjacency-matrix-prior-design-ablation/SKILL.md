---
name: adjacency-matrix-prior-design-ablation
description: Use when validating design choices of a discrete graph diffusion model
  for molecular generation — specifically whether the prior distribution over edge-type
  categories in the adjacency matrix materially affects top-k accuracy, MCES distance,
  or Tanimoto similarity.
when_to_use_negative:
- Evaluating pretraining design choices (encoder fingerprint pretraining or decoder
  fingerprint-structure pretraining dataset size) — those require separate scaling/encoder
  ablations, not prior variants
- Benchmarking DiffMS against baselines like MSNovelist or ICEBERG on MassSpecGym
  — this skill only isolates the adjacency prior within DiffMS itself
- Generation tasks with continuous diffusion on atomic coordinates rather than a discrete,
  categorical bond-type adjacency matrix
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: DiffMS
  role: Discrete graph diffusion model whose adjacency-matrix prior is being ablated;
    fine-tuned end-to-end on NPLIB1 via spec2mol_main.py
  repo: github.com/coleygroup/DiffMS
- name: MIST
  role: Pretrained spectrum encoder (spectrum-to-fingerprint) whose weights are loaded
    before fine-tuning each prior variant
  repo: github.com/coleygroup/MIST
- name: RDKit
  role: Used for molecule validity checking and Tanimoto similarity computation when
    filtering sampled molecules
- name: PyTorch
  role: Training framework for fine-tuning the three prior variants
- name: torch_geometric
  role: Graph data handling for the molecular adjacency tensors used in discrete diffusion
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:aeaccec8bbc7b56a5566e0ef50c55d20b64321bc4540abcf137432e2b7afda64
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DiffMS
  - MIST
  - PyTorch
  - RDKit
  - SIRIUS
  - torch_geometric
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# Adjacency-matrix prior design ablation

## Summary

An ablation protocol for validating the choice of prior distribution over the molecular adjacency matrix in DiffMS's discrete graph diffusion decoder, comparing the dataset marginal edge-type prior against empty and fully-connected single-bond priors on the NPLIB1 de novo generation benchmark.

## When to use

Use when validating design choices of a discrete graph diffusion model for molecular generation — specifically whether the prior distribution over edge-type categories in the adjacency matrix materially affects top-k accuracy, MCES distance, or Tanimoto similarity.

## When NOT to use

- Evaluating pretraining design choices (encoder fingerprint pretraining or decoder fingerprint-structure pretraining dataset size) — those require separate scaling/encoder ablations, not prior variants
- Benchmarking DiffMS against baselines like MSNovelist or ICEBERG on MassSpecGym — this skill only isolates the adjacency prior within DiffMS itself
- Generation tasks with continuous diffusion on atomic coordinates rather than a discrete, categorical bond-type adjacency matrix

## Inputs

- NPLIB1 (CANOPUS) spectra-molecule pairs processed via data_processing scripts 00-03 from the DiffMS repository
- MIST spectrum encoder pretrained on spectrum-to-fingerprint prediction (checkpoint from zenodo.org/records/23303289)
- Diffusion decoder pretrained on 2.8M fingerprint-molecule pairs from DSSTox, HMDB, COCONUT, and MOSES via fp2mol_main.py
- Adjacency-matrix prior configuration (marginal, empty, or fully-connected single-bond) in the DiffMS config

## Outputs

- Three fine-tuned DiffMS end-to-end checkpoints differing only in adjacency prior
- Per-variant metrics: Top-1/Top-10 accuracy, Top-1/Top-10 minimum MCES distance, Top-1/Top-10 maximum Tanimoto similarity
- Aggregated prior ablation comparison table saved as prior_ablation_metrics.csv (mirroring Table 6)

## How to apply

Fine-tune the end-to-end DiffMS model on NPLIB1 spectra-molecule pairs via spec2mol_main.py three times, varying only the adjacency-matrix prior: (1) the dataset marginal distribution over the 5 bond-type vocabulary (no bond, single, double, triple, aromatic), (2) an empty no-bond prior, and (3) a fully-connected single-bond prior, keeping the cosine noise schedule fixed. For each model, sample 100 molecules per spectrum, remove invalid or disconnected molecules, and identify top-k molecules by frequency. Compute Top-1/Top-10 accuracy, minimum MCES distance, and maximum Tanimoto similarity against ground truth, aggregate into a comparison table mirroring Table 6, and verify the marginal prior yields the best accuracy and lowest MCES. The rationale is that a structurally informative prior (matching the empirical edge-type marginal) should ease denoising relative to uninformative priors like all-no-bond or all-single-bond.

## Related tools

- **DiffMS** (Discrete graph diffusion model whose adjacency-matrix prior is being ablated; fine-tuned end-to-end on NPLIB1 via spec2mol_main.py) — github.com/coleygroup/DiffMS
- **MIST** (Pretrained spectrum encoder (spectrum-to-fingerprint) whose weights are loaded before fine-tuning each prior variant) — github.com/coleygroup/MIST
- **RDKit** (Used for molecule validity checking and Tanimoto similarity computation when filtering sampled molecules)
- **PyTorch** (Training framework for fine-tuning the three prior variants)
- **torch_geometric** (Graph data handling for the molecular adjacency tensors used in discrete diffusion)

## Examples

```
python src/spec2mol_main.py  # with dataset='canopus' in configs/config.yaml and the adjacency prior variant set, repeated for marginal / empty / fully-connected priors
```

## Evaluation signals

- The marginal (dataset edge-type distribution) prior variant should achieve the highest Top-1 and Top-10 accuracy and the lowest MCES distance among the three priors, mirroring Table 6
- All three variants must use identical settings elsewhere: cosine noise schedule, the 5 bond-type vocabulary (no bond, single, double, triple, aromatic), 100 samples per spectrum, and top-k frequency ranking
- Invalid or disconnected molecules must be filtered before top-k selection for every variant, so metrics are comparable
- Each fine-tuned checkpoint must load the same pretrained MIST encoder and fp2mol-pretrained decoder weights (via general.load_weights in general_default.yaml) so that the only difference is the prior
- Metrics table must contain all three prior rows with all six metrics (Top-1/Top-10 for accuracy, MCES, Tanimoto) and be saved as prior_ablation_metrics.csv

## Limitations

- Ablation is performed only on NPLIB1; conclusions about prior design may not transfer to MassSpecGym or other spectra-molecule benchmarks
- Results depend on downstream sampling — performance is measured after filtering invalid/disconnected molecules and frequency ranking, so prior effects on raw sampling diversity are not directly quantified
- Known DiGress codebase bug report (dim_ffy ignored in GraphTransformer, default 2048 used) means reported architectural parameters may not match the code, adding uncertainty to reproducibility

## Evidence

- [methods] Discrete diffusion on the molecular adjacency matrix: "We define a discrete diffusion process on 𝐀 \mathbf{A}"
- [methods] Sampling protocol for evaluation: [evidence span withheld — anchor status: not_found]
- [methods] Fine-tuning entry point for each prior variant: "To finetune the end-to-end model on spectra-molecule generation, run spec2mol_main.py"
- [methods] Pretrained weight loading configuration: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark dataset used for the ablation: [evidence span withheld — anchor status: not_found]
