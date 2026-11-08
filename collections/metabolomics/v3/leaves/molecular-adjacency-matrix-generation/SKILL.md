---
name: molecular-adjacency-matrix-generation
description: Use when a chemical formula (heavy-atom composition) is known and a molecular
  graph must be generated conditionally from a latent structural signal such as a
  spectrum embedding or 4,096-bit Morgan fingerprint.
when_to_use_negative:
- The chemical formula is unknown or unreliable — the pipeline takes the formula as
  given and cannot recover node counts/types; use formula-inference tools (SIRIUS,
  BUDDY, MIST-CF) first.
- The task is unconditional molecule generation or graph sizes exceed ~100-200 nodes,
  where the dense Graph Transformer diffusion formulation becomes impractical.
- The goal is property regression or spectrum fingerprint prediction rather than generating
  a molecular structure graph.
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: DiffMS
  role: Formula-restricted encoder-decoder framework hosting the discrete graph diffusion
    decoder; code adapted from DiGress
  repo: https://github.com/coleygroup/DiffMS
- name: DiGress
  role: Source discrete denoising diffusion formulation for graph generation that
    the decoder follows (Q^t transitions, cosine schedule, Graph Transformer)
  repo: https://github.com/cvignac/DiGress
- name: PyTorch
  role: Training/inference framework (tested with PyTorch 2.3.1, cuda 11.8)
  repo: https://github.com/coleygroup/DiffMS
- name: torch_geometric
  role: Graph data handling for the Graph Transformer denoiser (version 2.3.1)
  repo: https://github.com/coleygroup/DiffMS
- name: RDKit
  role: Post-sampling validity and connectivity checks of decoded molecules (rdkit=2024.09.4
    conda env)
  repo: https://github.com/coleygroup/DiffMS
- name: MIST
  role: Supplies the spectrum-encoder structural condition y used to condition generation
    during finetuning
  repo: https://github.com/coleygroup/MIST
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:e37f87113c9661713d54947119e4af2bffde5e4213369e88d9a62938aeab0938
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
  - torch_geometric
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# molecular-adjacency-matrix-generation

## Summary

Generate a molecular adjacency matrix (bond-type graph) via discrete denoising diffusion conditioned on a fixed heavy-atom composition and a structural condition (spectrum embedding or molecular fingerprint), following the DiGress formulation as adapted by DiffMS.

## When to use

Use when a chemical formula (heavy-atom composition) is known and a molecular graph must be generated conditionally from a latent structural signal such as a spectrum embedding or 4,096-bit Morgan fingerprint.

## When NOT to use

- The chemical formula is unknown or unreliable — the pipeline takes the formula as given and cannot recover node counts/types; use formula-inference tools (SIRIUS, BUDDY, MIST-CF) first.
- The task is unconditional molecule generation or graph sizes exceed ~100-200 nodes, where the dense Graph Transformer diffusion formulation becomes impractical.
- The goal is property regression or spectrum fingerprint prediction rather than generating a molecular structure graph.

## Inputs

- Fixed one-hot node features X of atom types derived from an input chemical formula (heavy-atom composition)
- Structural condition y: either a spectrum-encoder embedding or a 4,096-bit Morgan fingerprint (radius 2, no chirality)
- Empirical edge-type marginal distribution m computed from the training dataset adjacency matrices
- Number of atoms n and bond-type vocabulary (k=5: none, single, double, triple, aromatic)

## Outputs

- Predicted denoised adjacency probabilities p_theta(A^0 | M^t) in R^{n x n x k}
- Sampled symmetric molecular adjacency matrix A^0 with bond-type labels convertible to an RDKit molecule
- Post-filtered set of valid, connected candidate molecules per condition (100 samples, ranked by frequency)

## How to apply

Represent the molecule as M = (A, X, y): a one-hot adjacency matrix A over k=5 bond types (none, single, double, triple, aromatic) restricted to the upper triangle, fixed one-hot node features X derived from the given formula, and a condition y. Run a forward discrete diffusion with transition matrices Q^t on the upper triangle of A, using the marginal-based schedule Q_bar^t = alpha_bar^t I + beta_bar^t 1_k m^T where m is the empirical edge-type marginal, and a cosine noise schedule alpha_bar^t = cos(pi (t/T + eps) / (2(1+eps)))^2. Train a Graph Transformer denoiser phi_theta (separate MLPs encode A^t, X, and y; an MLP head outputs p_theta(A^0 | M^t) in R^{n x n x k}) with cross-entropy summed over upper-triangle entries. At sampling time, draw A^T from the edge-type marginal m, then iterate ancestral sampling that marginalizes network predictions through q(a_ij^{t-1} | a_ij = a_k, a_ij^t) weighted by p_theta(a_k), symmetrizing A at every step while keeping node types X fixed so the formula constraint is preserved. Validate generated molecules with RDKit (validity, connectivity) before ranking candidates by frequency.

## Related tools

- **DiffMS** (Formula-restricted encoder-decoder framework hosting the discrete graph diffusion decoder; code adapted from DiGress) — https://github.com/coleygroup/DiffMS
- **DiGress** (Source discrete denoising diffusion formulation for graph generation that the decoder follows (Q^t transitions, cosine schedule, Graph Transformer)) — https://github.com/cvignac/DiGress
- **PyTorch** (Training/inference framework (tested with PyTorch 2.3.1, cuda 11.8)) — https://github.com/coleygroup/DiffMS
- **torch_geometric** (Graph data handling for the Graph Transformer denoiser (version 2.3.1)) — https://github.com/coleygroup/DiffMS
- **RDKit** (Post-sampling validity and connectivity checks of decoded molecules (rdkit=2024.09.4 conda env)) — https://github.com/coleygroup/DiffMS
- **MIST** (Supplies the spectrum-encoder structural condition y used to condition generation during finetuning) — https://github.com/coleygroup/MIST

## Examples

```
python src/spec2mol_main.py  # with configs/config.yaml dataset set to 'msg' or 'canopus'
```

## Evaluation signals

- Generated molecules preserve the formula-fixed node composition (atom types/counts match the input heavy-atom list exactly).
- Sampled adjacency matrices are symmetric and contain only the 5 allowed edge classes after symmetrization at each denoising step.
- After post-filtering, sampled graphs decode to RDKit-valid and connected molecules at the expected rate (invalid/disconnected removed from the 100 samples per spectrum).
- End-to-end accuracy matches reported benchmarks: NPLIB1 top-1 ~8.34% and top-10 ~15.44%; MassSpecGym top-1 ~2.30% and top-10 ~4.25%.
- Training cross-entropy loss over upper-triangle entries decreases and predicted p_theta(A^0) rows sum to 1 across the k edge classes.

## Limitations

- The denoising network phi_theta is a dense Graph Transformer, making large graphs (more than ~100-200 nodes) impractical, as noted in the upstream DiGress README.
- Generation is formula-constrained: it cannot propose structures with a different heavy-atom composition than the supplied formula, and de novo recovery of the exact structure remains challenging even when structurally close matches are produced.
- A DiGress README update notes a configuration bug where dim_ffy is ignored in GraphTransformer construction (default 2048 used), so described hyperparameters may not match executed behavior.

## Evidence

- [methods] Define the forward discrete diffusion process: [evidence span withheld — anchor status: not_found]
- [methods] Cosine noise schedule: [evidence span withheld — anchor status: not_found]
- [methods] Graph Transformer denoiser: [evidence span withheld — anchor status: not_found]
- [methods] Sampling and filtering: [evidence span withheld — anchor status: not_found]
- [readme] Codebase origin: [evidence span withheld — anchor status: not_found]
