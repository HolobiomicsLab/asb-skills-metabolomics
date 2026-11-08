---
name: heavy-atom-formula-constrained-generation
description: Use when generating candidate molecular structures from a tandem mass
  spectrum where the chemical formula (heavy-atom composition) is already known or
  inferred (e.g., via SIRIUS or MIST), and atom counts must be preserved exactly in
  every generated candidate.
when_to_use_negative:
- The chemical formula is unknown or unreliable — the diffusion is formula-restricted,
  so it cannot fix node counts/types without the heavy-atom composition (use SIRIUS/MIST
  first or a different approach)
- The goal is to predict the chemical formula from the spectrum — formula inference
  is a separate upstream step performed by tools like SIRIUS or MIST, not this decoder
- The target is ranking or matching against a known structure database rather than
  de novo structure generation — this skill generates novel candidate graphs, not
  identifications against a library
edam_operation: http://edamontology.org/operation_3095
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3473
tools:
- name: DiffMS
  role: End-to-end encoder-decoder system that hosts the formula-restricted graph
    diffusion decoder; trained via fp2mol_main.py (pretraining) and spec2mol_main.py
    (finetuning)
  repo: github.com/coleygroup/DiffMS
- name: DiGress
  role: Provides the discrete graph diffusion formulation, noise schedules, and Graph
    Transformer denoiser adapted by DiffMS
  repo: cvignac/DiGress
- name: MIST
  role: Formula transformer producing spectrum-to-fingerprint predictions and embeddings
    used as the structural condition y for the decoder
  repo: coleygroup/MIST
- name: RDKit
  role: Molecule construction and post-sampling validation (validity, connectivity)
    of generated graphs from the denoised adjacency matrices
- name: PyTorch / torch_geometric
  role: Implementation framework for the diffusion process and Graph Transformer (tested
    with PyTorch 2.3.1, cuda 11.8, torch_geometric 2.3.1)
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:117e646d004f6f260adb4f2b7983f943441ddfcb05fd1ae208458d36f25946a0
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DiGress
  - DiffMS
  - MIST
  - PyTorch
  - PyTorch / torch_geometric
  - RDKit
  edam_operation: http://edamontology.org/operation_3095
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5806451612903226
n_claims: 31
---

# heavy-atom-formula-constrained-generation

## Summary

Generate molecular structures conditioned on mass spectra by running a DiGress-style discrete graph diffusion process over the adjacency matrix while fixing node (heavy-atom) counts and types from a known chemical formula, so the formula constraint is enforced natively by the diffusion design rather than by post-hoc filtering.

## When to use

Use when generating candidate molecular structures from a tandem mass spectrum where the chemical formula (heavy-atom composition) is already known or inferred (e.g., via SIRIUS or MIST), and atom counts must be preserved exactly in every generated candidate.

## When NOT to use

- The chemical formula is unknown or unreliable — the diffusion is formula-restricted, so it cannot fix node counts/types without the heavy-atom composition (use SIRIUS/MIST first or a different approach)
- The goal is to predict the chemical formula from the spectrum — formula inference is a separate upstream step performed by tools like SIRIUS or MIST, not this decoder
- The target is ranking or matching against a known structure database rather than de novo structure generation — this skill generates novel candidate graphs, not identifications against a library

## Inputs

- Tandem mass spectrum (or a spectrum-encoder embedding from the MIST formula transformer) serving as the structural condition y
- Known chemical formula for the target molecule, giving the fixed heavy-atom composition (node count and atom types X)
- Molecular fingerprint (e.g., 4096-bit Morgan fingerprint, radius 2, no chirality) as the condition y during fingerprint-molecule pretraining
- Training dataset of molecule graphs (e.g., 2.8M fingerprint-molecule pairs from DSSTox/HMDB/COCONUT/MOSES) from which the edge-type marginal m is computed

## Outputs

- Sampled denoised adjacency matrices in R^{n x n x k} decoded into molecular graphs with the formula-fixed atom composition
- Set of 100 generated molecules per spectrum, filtered to valid, connected structures and ranked by frequency (top-1 / top-10 candidates)
- Trained denoising network phi_theta (Graph Transformer) with per-entry cross-entropy training loss over upper-triangle adjacency entries

## How to apply

Represent each molecule as a graph M = (A, X, y) with a one-hot adjacency matrix A over k=5 bond types (none, single, double, triple, aromatic), fixed node features X obtained from the given formula, and a structural condition y (a spectrum-encoder embedding or, for pretraining-finetuning, a molecular fingerprint). Define the forward discrete diffusion on the upper triangle of A with transition matrices Q^t such that q(A^t | A^{t-1}) = A^{t-1} Q^t, using Q_bar^t = alpha_bar^t I + beta_bar^t 1_k m^T where m is the empirical marginal distribution of edge types in the training data and alpha_bar^t follows the cosine schedule of Nichol & Dhariwal (2021). Denoise with a Graph Transformer (Dwivedi & Bresson, 2021) whose separate MLPs encode A^t, X, and y, and an MLP head outputs predicted denoised adjacency probabilities in R^{n x n x k}; train with cross-entropy summed over upper-triangle entries. At inference, sample A^T from the edge-type marginal m and run ancestral sampling to A^0, marginalizing network predictions via q(a_ij^{t-1} | a_ij = a_k, a_ij^t) weighted by p_theta(a_k) where q(a_ij^t | a_ij = a_k) > 0, symmetrizing the matrix at each step. Sample 100 molecules per spectrum, discard invalid or disconnected molecules (RDKit validity/connectivity checks), and rank the remainder by frequency for top-k reporting.

## Related tools

- **DiffMS** (End-to-end encoder-decoder system that hosts the formula-restricted graph diffusion decoder; trained via fp2mol_main.py (pretraining) and spec2mol_main.py (finetuning)) — github.com/coleygroup/DiffMS
- **DiGress** (Provides the discrete graph diffusion formulation, noise schedules, and Graph Transformer denoiser adapted by DiffMS) — cvignac/DiGress
- **MIST** (Formula transformer producing spectrum-to-fingerprint predictions and embeddings used as the structural condition y for the decoder) — coleygroup/MIST
- **RDKit** (Molecule construction and post-sampling validation (validity, connectivity) of generated graphs from the denoised adjacency matrices)
- **PyTorch / torch_geometric** (Implementation framework for the diffusion process and Graph Transformer (tested with PyTorch 2.3.1, cuda 11.8, torch_geometric 2.3.1))

## Examples

```
python src/spec2mol_main.py  # with configs/config.yaml dataset set to 'msg' (MassSpecGym) or 'canopus' (NPLIB1); pretrain first via python src/fp2mol_main.py with dataset 'fp2mol'
```

## Evaluation signals

- Generated molecules must exactly reproduce the input formula's heavy-atom composition (node types/counts are fixed by X, not sampled)
- Sampled graphs must be chemically valid and connected after post-filtering; invalid/disconnected samples are discarded before top-k ranking
- End-to-end benchmark agreement: NPLIB1 top-1 accuracy ~8.34% and top-10 ~15.44%; MassSpecGym top-1 ~2.30% and top-10 ~4.25%
- Sampling diversity check: 100 samples per spectrum with top-k identified by frequency should recover consistent candidates rather than near-uniform spread
- Training loss (cross-entropy over upper-triangle adjacency entries) should decrease and the empirical edge-type marginal m used in Q_bar^t should match the training dataset's bond-type distribution

## Limitations

- The formula must be supplied: the method takes the chemical formula as given (inferred from high-resolution MS1 data and isotopic traces with tools like SIRIUS or MIST), so errors in formula inference propagate directly into node composition
- De novo generation of the exact molecular structure remains challenging; structurally close matches, not exact structures, are expected as outputs
- The DiGress-derived codebase has a known config bug (dim_ffy is ignored in GraphTransformer construction, defaulting to 2048 rather than the paper's value), so retraining from scratch may deviate from reported numbers
- Sampling is stochastic (100 molecules per spectrum with frequency-based ranking), so single-sample outputs may not reflect the best candidate

## Evidence

- [methods] sampling and filtering protocol: [evidence span withheld — anchor status: not_found]
- [readme] concrete training entry points: [evidence span withheld — anchor status: not_found]
