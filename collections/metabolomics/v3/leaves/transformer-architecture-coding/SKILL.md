---
name: transformer-architecture-coding
description: Use when implementing or modifying the denoising network of a discrete
  graph diffusion model that must output per-edge bond-type probabilities (k=5 classes)
  while respecting fixed heavy-atom composition from a chemical formula.
when_to_use_negative:
- The target is a continuous (non-categorical) graph representation — DiGress/DiffMS
  use discrete diffusion on one-hot adjacency, not continuous node-coordinate diffusion
- Node counts or atom types are unknown and must be generated — this decoder conditions
  on fixed formula-derived node features X, so it cannot produce structures violating
  the given heavy-atom composition
- You only need generic sequence transformers (e.g., for spectra encoding) — this
  skill covers the graph denoising network, not the MIST spectrum-transformer encoder
tools:
- name: DiffMS
  role: Host codebase whose diffusion decoder implements the Graph Transformer phi_theta;
    decoder pretraining and end-to-end finetuning entry points
  repo: github.com/coleygroup/DiffMS
- name: DiGress
  role: Source implementation of the discrete graph diffusion Graph Transformer that
    DiffMS adapts; provides GraphTransformer construction and dataset/marginal utilities
  repo: https://github.com/cvignac/DiGress
- name: PyTorch 2.3.1
  role: Framework used to implement and train the transformer denoiser (tested with
    cuda 11.8)
- name: torch_geometric 2.3.1
  role: Graph tensor/data utilities underlying the transformer's message passing over
    molecular graphs
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:2506b237ca1a3c75875181085f9c402fd9c7fe704dea1c47f721ccc7c85ff71b
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DiGress
  - DiffMS
  - PyTorch
  - PyTorch 2.3.1
  - torch_geometric 2.3.1
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# transformer-architecture-coding

## Summary

Implement the Graph Transformer denoising network phi_theta at the heart of DiffMS's discrete diffusion decoder, which takes noised adjacency A^t, formula-fixed node features X, and a structural condition y and predicts denoised adjacency probabilities p_theta(A^0 | M^t) in R^{n x n x k}. The architecture follows Dwivedi & Bresson (2021) as coded in DiGress, which DiffMS adapts.

## When to use

Use when implementing or modifying the denoising network of a discrete graph diffusion model that must output per-edge bond-type probabilities (k=5 classes) while respecting fixed heavy-atom composition from a chemical formula.

## When NOT to use

- The target is a continuous (non-categorical) graph representation — DiGress/DiffMS use discrete diffusion on one-hot adjacency, not continuous node-coordinate diffusion
- Node counts or atom types are unknown and must be generated — this decoder conditions on fixed formula-derived node features X, so it cannot produce structures violating the given heavy-atom composition
- You only need generic sequence transformers (e.g., for spectra encoding) — this skill covers the graph denoising network, not the MIST spectrum-transformer encoder

## Inputs

- Noised adjacency matrix A^t (upper-triangle one-hot, k=5 bond types: none, single, double, triple, aromatic)
- Fixed one-hot node features X of atom types derived from the given chemical formula (heavy-atom composition)
- Structural condition y: a MIST spectrum-encoder embedding (finetuning) or a molecular fingerprint (fp2mol pretraining on 2.8M fingerprint-structure pairs)

## Outputs

- Predicted denoised adjacency probabilities p_theta(A^0 | M^t) as an n x n x k tensor (k=5 edge classes)
- Trained decoder checkpoint usable via general.load_weights in general_default.yaml, and finally sampled molecular graphs after symmetrization

## How to apply

Build phi_theta as a Graph Transformer (Dwivedi & Bresson, 2021) with three separate MLP embeddings: one for edge features of the noised adjacency A^t, one for node (atom-type) features X taken from the given formula, and one for the structural condition y (either a spectrum-encoder embedding or a 4096-bit Morgan fingerprint radius 2 without chirality for fp2mol pretraining). Cap the transformer with an MLP head that outputs predicted denoised adjacency probabilities p_theta(A^0 | M^t) shaped n x n x 5, where the 5 edge classes are none, single, double, triple, aromatic. Train with cross-entropy summed over upper-triangle entries, L(A, A_hat) = sum_{1<=i<j<=n} CE(a_ij, a_hat_ij), and symmetrize the adjacency matrix during ancestral sampling. Reuse the DiGress implementation rather than coding from scratch: the DiffMS codebase is adapted from cvignac/DiGress and was tested with PyTorch 2.3.1, cuda 11.8 and torch_geometric 2.3.1. When constructing the Graph Transformer, check the dim_ffy feed-forward width explicitly, since a DiGress bug causes it to be ignored and the default 2048 to be used instead of the paper's value.

## Related tools

- **DiffMS** (Host codebase whose diffusion decoder implements the Graph Transformer phi_theta; decoder pretraining and end-to-end finetuning entry points) — github.com/coleygroup/DiffMS
- **DiGress** (Source implementation of the discrete graph diffusion Graph Transformer that DiffMS adapts; provides GraphTransformer construction and dataset/marginal utilities) — https://github.com/cvignac/DiGress
- **PyTorch 2.3.1** (Framework used to implement and train the transformer denoiser (tested with cuda 11.8))
- **torch_geometric 2.3.1** (Graph tensor/data utilities underlying the transformer's message passing over molecular graphs)

## Examples

```
python src/fp2mol_main.py  # pretrain the decoder; set dataset in configs/config.yaml to 'fp2mol'
```

## Evaluation signals

- Head output has shape n x n x 5 with probabilities summing to 1 over the 5 edge classes per upper-triangle entry
- Training loss equals cross-entropy summed over upper-triangle entries only; loss decreases stably over fp2mol pretraining
- Network is permutation-invariant with respect to atom ordering, matching DiffMS's permutation-invariant design claim
- Generated molecules always satisfy the formula-fixed node composition (atom counts/types from X) after sampling
- Sampled graphs remain valid and connected after removing invalid/disconnected molecules, and top-k frequency ranking reproduces reported benchmarks (NPLIB1 top-1 ~8.34%, top-10 ~15.44%; MassSpecGym top-1 ~2.30%, top-10 ~4.25%)

## Limitations

- Known DiGress bug: dim_ffy is ignored in GraphTransformer construction, so the default 2048 is used and the paper's reported value is incorrect; impact on performance is unquantified
- Recommended 'fixed_bug' branch fixes some neural network layers but has not been evaluated
- graph_tool and torch_geometric conflict on MacOS per the DiGress README, complicating the training environment

## Evidence

- [full_text] Denoising network architecture specification: [evidence span withheld — anchor status: not_found]
- [readme] dim_ffy hyperparameter bug affecting Graph Transformer width: [evidence span withheld — anchor status: external_doc]
