---
name: molecular-graph-sampling-and-validation
description: Use when sampling candidate molecular structures from a conditional graph
  diffusion decoder and you must filter invalid/disconnected molecules and select
  top-k hits by frequency.
when_to_use_negative:
- When the chemical formula is unknown — DiffMS is formula-restricted (heavy atom-constrained)
  and requires composition to be given by tools like SIRIUS or MIST-CF
- When the goal is de novo generation of the exact molecular structure with guaranteed
  correctness — the paper notes structurally close matches, not exact structures,
  are what the method delivers
- When ranking MSNovelist-style outputs — that baseline uses beam search (width 100)
  with log-likelihood ranking, not diffusion sampling with frequency-based top-k selection
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DiffMS
  role: Formula-restricted encoder-decoder diffusion model whose decoder produces
    the candidate molecular graphs that are sampled and validated
  repo: https://github.com/coleygroup/DiffMS
- name: DiGress
  role: Discrete denoising diffusion framework for graph generation that DiffMS adapts;
    defines ancestral sampling and the Graph Transformer denoiser
  repo: https://github.com/cvignac/DiGress
- name: RDKit
  role: Post-filtering of decoded molecular graphs for chemical validity and connectedness;
    installed as rdkit=2024.09.4 in the diffms conda environment
- name: PyTorch
  role: Runtime for the diffusion sampling loop and ancestral denoising steps (tested
    with PyTorch 2.3.1, cuda 11.8)
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:6df19d3e2fd9c4dd013589aad1aed7cbcde91aba27f34660ea4a976888eb3d55
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - DiGress
  - DiffMS
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# molecular-graph-sampling-and-validation

## Summary

Generates candidate molecular graphs from a trained diffusion decoder (DiffMS/DiGress formulation), then post-filters them with RDKit for chemical validity, connectedness, and formula-consistent atom composition before ranking candidates by frequency.

## When to use

Use when sampling candidate molecular structures from a conditional graph diffusion decoder and you must filter invalid/disconnected molecules and select top-k hits by frequency.

## When NOT to use

- When the chemical formula is unknown — DiffMS is formula-restricted (heavy atom-constrained) and requires composition to be given by tools like SIRIUS or MIST-CF
- When the goal is de novo generation of the exact molecular structure with guaranteed correctness — the paper notes structurally close matches, not exact structures, are what the method delivers
- When ranking MSNovelist-style outputs — that baseline uses beam search (width 100) with log-likelihood ranking, not diffusion sampling with frequency-based top-k selection

## Inputs

- Trained DiffMS decoder checkpoint (Zenodo record 23542112) or MIST spectrum-encoder embedding
- Tandem mass spectrum with chemical formula inferred by SIRIUS/MIST-CF or the MIST formula transformer
- Fixed one-hot node features X (heavy-atom composition from the formula)
- Empirical edge-type marginal m over k=5 bond types (none, single, double, triple, aromatic)

## Outputs

- 100 sampled candidate molecular graphs per spectrum (symmetric adjacency A + node features)
- Filtered set of valid, connected molecules (RDKit Mol objects / SMILES)
- Top-k molecules ranked by generation frequency
- Top-1 and top-10 accuracy metrics on NPLIB1 or MassSpecGym

## How to apply

Run ancestral discrete diffusion sampling from A^T drawn from the empirical edge-type marginal m, iteratively denoising to A^0 while symmetrizing the adjacency matrix at each step; atom/node features X stay fixed to the heavy-atom composition implied by the known chemical formula. Sample 100 molecules per spectrum, then post-filter each decoded graph with RDKit to remove chemically invalid or disconnected molecules. Rank surviving candidates by generation frequency and report top-k molecules. Validate end-to-end on NPLIB1 and MassSpecGym, where the reported operating points are top-1 accuracy ~8.34% (top-10 ~15.44%) on NPLIB1 and top-1 ~2.30% (top-10 ~4.25%) on MassSpecGym. For the MSNovelist baseline comparison, use beam search with width 100 and log-likelihood-based ranking instead of diffusion sampling.

## Related tools

- **DiffMS** (Formula-restricted encoder-decoder diffusion model whose decoder produces the candidate molecular graphs that are sampled and validated) — https://github.com/coleygroup/DiffMS
- **DiGress** (Discrete denoising diffusion framework for graph generation that DiffMS adapts; defines ancestral sampling and the Graph Transformer denoiser) — https://github.com/cvignac/DiGress
- **RDKit** (Post-filtering of decoded molecular graphs for chemical validity and connectedness; installed as rdkit=2024.09.4 in the diffms conda environment)
- **PyTorch** (Runtime for the diffusion sampling loop and ancestral denoising steps (tested with PyTorch 2.3.1, cuda 11.8))

## Examples

```
conda activate diffms; python src/spec2mol_main.py  # with dataset set to 'canopus' (NPLIB1) or 'msg' (MassSpecGym) in configs/config.yaml
```

## Evaluation signals

- Every generated molecule must satisfy the formula-fixed node composition: the decoded graph's atom counts/types equal the heavy-atom composition specified by the input chemical formula
- Adjacency matrices must be symmetric at every sampling step; molecules failing RDKit validity or connectedness checks are removed before ranking
- Exactly 100 samples are drawn per spectrum, and top-k is computed from the frequency distribution over the surviving valid candidates
- End-to-end accuracy should reproduce the reported benchmarks: NPLIB1 top-1 ~8.34% / top-10 ~15.44%; MassSpecGym top-1 ~2.30% / top-10 ~4.25%
- Downstream performance must scale consistently with the size of the fingerprint-structure pretraining set (2.8M DSSTox/HMDB/COCONUT/MOSES pairs)

## Limitations

- De novo generation of the exact molecular structure remains challenging; structurally close matches are the realistic outcome
- Downstream accuracy is low in absolute terms (top-1 in the 2–8% range on NPLIB1/MassSpecGym), so candidates should be treated as hypotheses for domain experts
- The DiGress codebase DiffMS adapts carries known issues (e.g., dim_ffy ignored in GraphTransformer construction; a 'fixed_bug' branch exists but is unevaluated); graph_tool and torch_geometric conflict on MacOS

## Evidence

- [methods] Sampling with post-filtering and top-k frequency ranking: [evidence span withheld — anchor status: not_found]
- [readme] Entry point for end-to-end finetuning and sampling runs: [evidence span withheld — anchor status: not_found]
