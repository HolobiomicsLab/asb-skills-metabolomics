---
name: discrete-graph-diffusion-training
description: Use when conditional molecular structure generation requires sampling
  valid, formula-constrained graphs with predefined heavy-atom composition, e.g. de
  novo generation from mass spectra on NPLIB1 or MassSpecGym benchmarks.
when_to_use_negative:
- The task is fingerprint prediction from spectra rather than graph generation — use
  the MIST spectrum-to-fingerprint encoder training instead
- The chemical formula is unknown — DiffMS is formula-restricted and relies on formula
  inference tools (SIRIUS, BUDDY, MIST-CF) upstream
- Working with graphs of more than 100-200 nodes — the DiGress-derived dense diffusion
  is not designed for large graphs (consider SparseDiff per the DiGress README)
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3474
tools:
- name: DiffMS
  role: Formula-restricted encoder-decoder diffusion model whose adjacency-matrix
    decoder is trained; codebase and configs (fp2mol.yaml, general_default.yaml)
  repo: https://github.com/coleygroup/DiffMS
- name: DiGress
  role: Base discrete denoising diffusion framework for graph generation from which
    DiffMS is adapted
  repo: https://github.com/cvignac/DiGress
- name: PyTorch
  role: Training framework (tested with PyTorch 2.3.1, cuda 11.8)
- name: torch_geometric
  role: Graph data handling for molecular graphs (tested with torch_geometrics 2.3.1)
- name: RDKit
  role: Molecule validity checking and connectivity filtering of sampled graphs during
    evaluation
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:3b8d96bd4ca740230e76f649920b2e276b62f19e3af727f7b1fac17d089384e7
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
  - torch_geometric
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# Discrete graph diffusion training for molecular generation

## Summary

Train a permutation-invariant discrete diffusion model (DiffMS, adapted from DiGress) on molecular adjacency matrices with a 5-class bond-type vocabulary (no bond, single, double, triple, aromatic), pretraining the diffusion decoder on ~2.8M fingerprint-molecule pairs and finetuning end-to-end on spectra-molecule data.

## When to use

Use when conditional molecular structure generation requires sampling valid, formula-constrained graphs with predefined heavy-atom composition, e.g. de novo generation from mass spectra on NPLIB1 or MassSpecGym benchmarks.

## When NOT to use

- The task is fingerprint prediction from spectra rather than graph generation — use the MIST spectrum-to-fingerprint encoder training instead
- The chemical formula is unknown — DiffMS is formula-restricted and relies on formula inference tools (SIRIUS, BUDDY, MIST-CF) upstream
- Working with graphs of more than 100-200 nodes — the DiGress-derived dense diffusion is not designed for large graphs (consider SparseDiff per the DiGress README)

## Inputs

- Fingerprint-molecule pretraining pairs (~2.8M from DSSTox, HMDB, COCONUT, MOSES), dataset key 'fp2mol'/'combined'
- Spectra-molecule finetuning datasets: NPLIB1 (CANOPUS subset of GNPS) or MassSpecGym, processed via data_processing/ scripts 00-03
- Diffusion decoder and MIST encoder checkpoints (Zenodo records 23542112 and 23303289)

## Outputs

- Trained discrete diffusion decoder / end-to-end DiffMS model checkpoints
- Sampled molecules per spectrum (100 samples each, invalid/disconnected removed, top-k by frequency)
- Aggregated metrics tables (e.g. prior_ablation_metrics.csv) with Top-1/Top-10 accuracy, MCES distance, and Tanimoto similarity

## How to apply

Set up the DiffMS conda environment (rdkit=2024.09.4, Python 3.9, PyTorch 2.3.1, cuda 11.8, torch_geometric 2.3.1) and process datasets by running the data_processing/ scripts (00-03) in order. Pretrain the diffusion decoder on fingerprint-molecule pairs from DSSTox, HMDB, COCONUT and MOSES via fp2mol_main.py with the dataset set to 'fp2mol' in config.yaml (primary dataset key: 'combined' in fp2mol.yaml). Then finetune the end-to-end model on spectra-molecule pairs via spec2mol_main.py, setting the dataset to 'canopus' (NPLIB1) or 'msg' (MassSpecGym) and loading decoder weights via general.load_weights in general_default.yaml. Discrete diffusion is defined directly on the adjacency matrix A, which natively enforces the heavy-atom formula constraint; ablations vary the adjacency prior (dataset marginal over edge types, empty no-bond prior, fully-connected single-bond prior) with the cosine noise schedule, the marginal prior performing best on NPLIB1 accuracy and MCES. Evaluate by sampling 100 molecules per spectrum, removing invalid or disconnected molecules, ranking top-k by frequency, and computing Top-1/Top-10 accuracy, MCES distance, and Tanimoto similarity against ground truth.

## Related tools

- **DiffMS** (Formula-restricted encoder-decoder diffusion model whose adjacency-matrix decoder is trained; codebase and configs (fp2mol.yaml, general_default.yaml)) — https://github.com/coleygroup/DiffMS
- **DiGress** (Base discrete denoising diffusion framework for graph generation from which DiffMS is adapted) — https://github.com/cvignac/DiGress
- **PyTorch** (Training framework (tested with PyTorch 2.3.1, cuda 11.8))
- **torch_geometric** (Graph data handling for molecular graphs (tested with torch_geometrics 2.3.1))
- **RDKit** (Molecule validity checking and connectivity filtering of sampled graphs during evaluation)

## Examples

```
python src/fp2mol_main.py  # with configs/config.yaml dataset set to 'fp2mol'; then finetune: python src/spec2mol_main.py with dataset 'canopus'
```

## Evaluation signals

- Top-1 accuracy on NPLIB1 de novo generation should be in the ~4-9% range depending on pretraining; the marginal edge-type prior must beat the empty and fully-connected priors on accuracy and MCES
- Encoder pretraining on spectrum-to-fingerprint nearly doubles NPLIB1 top-1 accuracy (4.36% to 8.34%) — a large drop suggests pretraining was skipped or misconfigured
- Downstream performance should scale consistently with increasing fingerprint-structure pretraining dataset size
- All sampled molecules must be chemically valid and connected after RDKit filtering; invalid/disconnected graphs must be excluded before top-k frequency ranking
- Config sanity: config.yaml dataset must be 'fp2mol' for decoder pretraining and 'canopus' or 'msg' for finetuning, with general.load_weights pointing to the decoder checkpoint

## Limitations

- The DiGress README reports a bug where dim_ffy is ignored in GraphTransformer construction, so the paper's described default (2048) differs from what runs
- The marginal prior ablation result is specific to NPLIB1; prior choice may not transfer to other datasets
- Exact molecular structure from de novo generation remains challenging; structurally close matches (MCES/Tanimoto) are the realistic target

## Evidence

- [methods] Sampling and filtering protocol: [evidence span withheld — anchor status: not_found]
