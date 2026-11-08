---
name: graph-diffusion-decoder-implementation
description: Use when a task requires generating molecular graphs conditioned on a
  structural input (spectrum embedding or Morgan fingerprint) while preserving a known
  heavy-atom formula — i.e., rebuilding, training, or finetuning the DiffMS discrete
  graph-diffusion decoder.
when_to_use_negative:
- The chemical formula (heavy-atom composition) is unknown — the decoder hard-conditions
  on fixed node features X, so formula inference (e.g., SIRIUS, MIST-CF) must be done
  first
- The target is text/SMILES-based autoregressive generation rather than graph-valued
  output — this skill is specifically a discrete diffusion over adjacency matrices
- Only a pretrained off-the-shelf decoder is needed without modification — use the
  Zenodo checkpoints directly instead of re-implementing
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3474
tools:
- name: DiffMS
  role: Reference implementation of the formula-constrained diffusion decoder; codebase
    adapted from DiGress
  repo: github.com/coleygroup/DiffMS
- name: DiGress
  role: Source of the discrete graph-diffusion formulation (transition matrices, cosine
    schedule, Graph Transformer, ancestral sampling) and the base codebase DiffMS
    adapts
  repo: cvignac/DiGress
- name: PyTorch 2.3.1
  role: Framework in which the decoder is implemented and trained
- name: torch_geometric 2.3.1
  role: Graph neural network library used for the Graph Transformer and molecular
    graph data handling
- name: RDKit 2024.09.4
  role: Molecule validity/connectivity checking and structure reconstruction of sampled
    graphs
- name: MIST
  role: Spectrum encoder producing the structural condition y (spectrum embedding)
    for end-to-end finetuning; fingerprint predictor used during decoder pretraining
  repo: coleygroup/MIST
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:39ab0c44df313371224f3fcacbabf32c9133018443dc12f8fb83f5b388295b08
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
  - PyTorch 2.3.1
  - RDKit 2024.09.4
  - torch_geometric 2.3.1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# graph-diffusion-decoder-implementation

## Summary

Implements DiffMS's formula-constrained molecule decoder as a DiGress-style discrete diffusion process over the molecular adjacency matrix, denoised by a Graph Transformer conditioned on spectrum embeddings (or fingerprints) with atom types fixed by the input chemical formula.

## When to use

Use when a task requires generating molecular graphs conditioned on a structural input (spectrum embedding or Morgan fingerprint) while preserving a known heavy-atom formula — i.e., rebuilding, training, or finetuning the DiffMS discrete graph-diffusion decoder.

## When NOT to use

- The chemical formula (heavy-atom composition) is unknown — the decoder hard-conditions on fixed node features X, so formula inference (e.g., SIRIUS, MIST-CF) must be done first
- The target is text/SMILES-based autoregressive generation rather than graph-valued output — this skill is specifically a discrete diffusion over adjacency matrices
- Only a pretrained off-the-shelf decoder is needed without modification — use the Zenodo checkpoints directly instead of re-implementing

## Inputs

- Molecular graph M = (A, X, y): one-hot adjacency (k=5 bond types: none, single, double, triple, aromatic), one-hot atom types from the known chemical formula, and a structural condition
- Structural condition y: spectrum-encoder embedding (MIST formula transformer output) or 4,096-bit Morgan fingerprint (radius 2, without chirality)
- Training corpora: ~2.8M fingerprint-molecule pairs from DSSTox/HMDB/COCONUT/MOSES ('combined' fp2mol dataset) and finetuning spectra from NPLIB1 (CANOPUS) or MassSpecGym
- Pretrained checkpoints from Zenodo records 23542112 (DiffMS end-to-end/decoder) and 23303289 (MIST encoder)

## Outputs

- Predicted denoised adjacency probabilities p_theta(A^0 | M^t) in R^{n x n x k}
- Generated molecular graphs/structures with fixed heavy-atom composition matching the input formula
- Per-spectrum candidate sets: 100 sampled molecules per spectrum, filtered and ranked by frequency for top-k evaluation

## How to apply

Represent each molecule as M = (A, X, y): a one-hot adjacency matrix A with k=5 bond types (none, single, double, triple, aromatic), fixed node features X of atom types taken from the given formula, and a structural condition y (spectrum-encoder embedding or 4096-bit Morgan fingerprint radius 2 for pretraining). Define the forward diffusion on the upper triangle of A with Q_bar^t = alpha_bar^t I + beta_bar^t 1_k m^T, where m is the empirical edge-type marginal of the training set and alpha_bar^t follows the cosine schedule alpha_bar^t = cos(pi (t/T + eps) / (2(1+eps)))^2; sample noised graphs directly as A * Q_bar^t. Implement the denoiser phi_theta as a Graph Transformer with separate MLP encoders for A^t edge features, node features X, and condition y, ending in an MLP head that outputs p_theta(A^0 | M^t) in R^{n x n x k}; train with cross-entropy summed over upper-triangle pairs sum_{1<=i<j<=n} CE(a_ij, a_hat_ij). Sample ancestrally: initialize A^T from the edge-type marginal m, then at each step marginalize network predictions over q(a_ij^{t-1} | a_ij = a_k, a_ij^t) weighted by p_theta(a_k) for edge classes with q(a_ij^t | a_ij = a_k) > 0, iterating to A^0 while symmetrizing the matrix at every step. The decoder is pretrained on ~2.8M fingerprint-structure pairs (DSSTox, HMDB, COCONUT, MOSES) via fp2mol_main.py, then finetuned end-to-end on spectra-molecule pairs (NPLIB1/CANOPUS or MassSpecGym) via spec2mol_main.py; the condition y is swapped between fingerprint (pretraining) and MIST spectrum embedding (finetuning).

## Related tools

- **DiffMS** (Reference implementation of the formula-constrained diffusion decoder; codebase adapted from DiGress) — github.com/coleygroup/DiffMS
- **DiGress** (Source of the discrete graph-diffusion formulation (transition matrices, cosine schedule, Graph Transformer, ancestral sampling) and the base codebase DiffMS adapts) — cvignac/DiGress
- **PyTorch 2.3.1** (Framework in which the decoder is implemented and trained)
- **torch_geometric 2.3.1** (Graph neural network library used for the Graph Transformer and molecular graph data handling)
- **RDKit 2024.09.4** (Molecule validity/connectivity checking and structure reconstruction of sampled graphs)
- **MIST** (Spectrum encoder producing the structural condition y (spectrum embedding) for end-to-end finetuning; fingerprint predictor used during decoder pretraining) — coleygroup/MIST

## Examples

```
python src/spec2mol_main.py  # with configs/config.yaml dataset set to 'canopus' (NPLIB1) or 'msg' (MassSpecGym)
```

## Evaluation signals

- Generated molecules must satisfy the formula-fixed heavy-atom composition (node counts/types preserved exactly, since X is conditioned from the formula)
- Sampled adjacency matrices are symmetric (symmetrized at every ancestral-sampling step) and contain only the 5 edge classes
- End-to-end accuracy matches reported benchmarks: NPLIB1 top-1 ~8.34%, top-10 ~15.44%; MassSpecGym top-1 ~2.30%, top-10 ~4.25%
- After sampling 100 molecules per spectrum, filtering removes invalid or disconnected structures and the top-k ranking is by sample frequency
- Training loss is cross-entropy restricted to upper-triangle adjacency entries sum_{1<=i<j<=n} CE(a_ij, a_hat_ij), and the noise schedule follows alpha_bar^t = cos(pi (t/T + eps) / (2(1+eps)))^2

## Limitations

- The decoder requires the chemical formula as given; heavy-atom composition must come from external tools (SIRIUS, BUDDY, MIST-CF/MIST), and MIST-CF does not consider isotope distributions
- De novo generation of the exact molecular structure remains challenging; structurally close matches are the realistic outcome
- The upstream DiGress codebase notes a bug where dim_ffy is ignored in GraphTransformer construction (default 2048 is used instead of the paper value), with unknown performance impact

## Evidence

- [methods] sampling and filtering procedure: [evidence span withheld — anchor status: not_found]
