---
name: discrete-diffusion-process-design
description: Use when a generative model must output molecular graphs with a prescribed
  atom count/type composition (formula-constrained generation), by diffusing and denoising
  bond-type entries of the adjacency matrix.
when_to_use_negative:
- The number or type of atoms is unknown a priori — this formulation fixes node features
  X from the given formula and cannot vary heavy-atom composition
- Generation is over continuous node/edge attributes or non-graph outputs; discrete
  categorical diffusion over adjacency entries is unnecessary there (DiGress offers
  a separate continuous model mode)
- Working with very large graphs (more than 100-200 nodes) where the dense DiGress-style
  diffusion is impractical and a sparse variant is recommended
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DiGress
  role: Source formulation and codebase for the discrete denoising diffusion over
    graphs; DiffMS is adapted from it
  repo: https://github.com/cvignac/DiGress
- name: DiffMS
  role: Implements the formula-constrained discrete graph diffusion decoder conditioned
    on spectrum embeddings or fingerprints
  repo: https://github.com/coleygroup/DiffMS
- name: PyTorch
  role: Framework used to implement and train the diffusion decoder (tested with PyTorch
    2.3.1, cuda 11.8)
- name: torch_geometric
  role: Graph data handling for the Graph Transformer denoiser (tested at version
    2.3.1)
- name: RDKit
  role: Validity checks on sampled molecules (installed as rdkit=2024.09.4 in the
    diffms conda environment)
- name: MIST
  role: Spectrum transformer encoder producing the structural condition y (formula-aware
    embeddings) for end-to-end finetuning
  repo: https://github.com/coleygroup/MIST
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:aba73b1603451c4179c39445bb3a02bdc6a76a95db89643eb2036797e6b3b7db
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cvignac/DiGress
  tools:
  - DiGress
  - DiffMS
  - MIST
  - PyTorch
  - RDKit
  - torch_geometric
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# discrete-diffusion-process-design

## Summary

Design a discrete (categorical) diffusion process over molecular graph adjacency matrices, following the DiGress formulation, so that generation can natively respect a fixed heavy-atom composition imposed by a known chemical formula.

## When to use

Use when a generative model must output molecular graphs with a prescribed atom count/type composition (formula-constrained generation), by diffusing and denoising bond-type entries of the adjacency matrix.

## When NOT to use

- The number or type of atoms is unknown a priori — this formulation fixes node features X from the given formula and cannot vary heavy-atom composition
- Generation is over continuous node/edge attributes or non-graph outputs; discrete categorical diffusion over adjacency entries is unnecessary there (DiGress offers a separate continuous model mode)
- Working with very large graphs (more than 100-200 nodes) where the dense DiGress-style diffusion is impractical and a sparse variant is recommended

## Inputs

- Molecular graphs as one-hot adjacency matrices A with k=5 bond types (none, single, double, triple, aromatic)
- Fixed node features X of atom types derived from a known chemical formula (heavy-atom composition)
- Structural condition y: either a MIST spectrum-encoder embedding or a 4,096-bit Morgan fingerprint (radius 2, no chirality)
- Training data of fingerprint-molecule pairs (e.g. 2.8M pairs from DSSTox/HMDB/COCONUT/MOSES 'combined' set) or spectra-molecule pairs (NPLIB1/CANOPUS, MassSpecGym)

## Outputs

- Predicted denoised adjacency probabilities p_theta(A^0 | M^t) in R^{n x n x k}
- Sampled molecular adjacency matrices A^0 (symmetrized, bond-typed) decodable to molecular structures
- Trained diffusion decoder checkpoint usable for fp2mol pretraining or spec2mol finetuning

## How to apply

Represent each molecule as M = (A, X, y): a one-hot adjacency matrix A over k=5 bond types (none, single, double, triple, aromatic), fixed node features X (atom types taken directly from the known formula), and a structural condition y (spectrum-encoder embedding or molecular fingerprint). Define the forward process only on the upper triangle of A with transition matrices Q^t, q(A^t | A^{t-1}) = A^{t-1} Q^t, using the marginal-based schedule Q_bar^t = alpha_bar^t I + beta_bar^t 1_k m^T where m is the empirical edge-type marginal of the training data (e.g. the 2.8M fingerprint-molecule pretraining pairs from DSSTox, HMDB, COCONUT, MOSES). Use the cosine noise schedule alpha_bar^t = cos(pi (t/T + eps) / (2(1+eps)))^2 with beta_bar^t = 1 - alpha_bar^t, sampling noised graphs directly as A * Q_bar^t. Train the denoiser phi_theta — a Graph Transformer with separate MLP encoders for edge features A^t, node features X, and condition y, plus an MLP head outputting p_theta(A^0 | M^t) in R^{n x n x k} — using cross-entropy summed over upper-triangle entries. At inference, run ancestral sampling: marginalize network predictions as p_theta(a_ij^{t-1} | M^t) via q(a_ij^{t-1} | a_ij = a_k, a_ij^t) weighted by p_theta(a_k) where q(a_ij^t | a_ij = a_k) > 0, start A^T from the edge-type marginal m, and symmetrize the matrix at every step while holding X fixed.

## Related tools

- **DiGress** (Source formulation and codebase for the discrete denoising diffusion over graphs; DiffMS is adapted from it) — https://github.com/cvignac/DiGress
- **DiffMS** (Implements the formula-constrained discrete graph diffusion decoder conditioned on spectrum embeddings or fingerprints) — https://github.com/coleygroup/DiffMS
- **PyTorch** (Framework used to implement and train the diffusion decoder (tested with PyTorch 2.3.1, cuda 11.8))
- **torch_geometric** (Graph data handling for the Graph Transformer denoiser (tested at version 2.3.1))
- **RDKit** (Validity checks on sampled molecules (installed as rdkit=2024.09.4 in the diffms conda environment))
- **MIST** (Spectrum transformer encoder producing the structural condition y (formula-aware embeddings) for end-to-end finetuning) — https://github.com/coleygroup/MIST

## Examples

```
python3 src/fp2mol_main.py  # with configs/config.yaml dataset set to 'fp2mol' to pretrain the diffusion decoder
```

## Evaluation signals

- Generated molecules must satisfy the formula-fixed node composition (atom counts/types match the input heavy-atom formula)
- Sampled graphs must be valid and connected after post-filtering (RDKit parse; removal of invalid or disconnected molecules)
- Training loss is cross-entropy summed over upper-triangle adjacency entries L(A, A_hat) = sum_{1<=i<j<=n} CE(a_ij, a_hat_ij) and should decrease monotonically per epoch
- End-to-end finetuned model should approach reported benchmarks: NPLIB1 top-1 accuracy ~8.34% and top-10 ~15.44%; MassSpecGym top-1 ~2.30% and top-10 ~4.25% (sampling 100 molecules per spectrum, ranked by frequency)
- Every sampled A^t must be a valid row-stochastic one-hot matrix after symmetrization, and A^T must be drawn from the empirical edge-type marginal m

## Limitations

- The DiGress README reports a bug where dim_ffy is ignored in GraphTransformer construction (default 2048 used instead of the paper value), with unknown performance impact
- graph-tool and torch_geometric currently conflict on MacOS per the DiGress README
- De novo generation of the exact molecular structure remains challenging; structurally close matches are the realistic outcome
- The DiGress checkpoint hosting was lost and the fixed_bug branch is untested, so reproducing the base model from scratch requires caution

## Evidence

- [methods] Discrete diffusion on the molecular adjacency matrix: "We define a discrete diffusion process on 𝐀 \mathbf{A}"
- [methods] Post-sampling validity filtering and frequency ranking of generated molecules: [evidence span withheld — anchor status: not_found]
