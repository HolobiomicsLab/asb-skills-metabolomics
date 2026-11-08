---
name: msms-spectrum-prediction-inference
description: Use when a trained C2MS model must convert a molecular structure into
  a predicted tandem MS/MS spectrum — e.g. to predict spectra for PubChem molecules
  absent from NIST20 so they can be ranked against an unknown spectrum.
when_to_use_negative:
- When an experimental MS/MS spectrum is already available in a spectral library such
  as NIST20 — retrieval against real spectra is the baseline and predicted spectra
  should augment, not replace, it
- When the target chemistry involves complex rearrangements (e.g. cyclizations) that
  the recursive edge-removal algorithm cannot account for, as these produce unexplained
  Outside-of-Support (OS) peaks
- When evaluating spectra from unmerged data or experimental conditions beyond the
  training scope (different instrument platforms, non-[M+H]+ precursor adducts), where
  performance has not been established
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3172
tools:
- name: FraGNNet
  role: Two-stage probabilistic GNN performing the C2MS prediction from the fragmentation
    DAG
  repo: github.com/FraGNNet/fragnnet
- name: Pytorch
  role: Framework implementing the FraGNNet model and inference (version 2.1, CUDA
    11.8; repo README pins torch==2.4.1 with cu121)
- name: Pytorch Lightning
  role: Training/inference module wrapping FraGNNet with checkpoint building and config-driven
    runners
  repo: https://github.com/Lightning-AI/pytorch-lightning
- name: Pytorch Geometric
  role: Implements the GINE Molecule GNN graph-neural-network modules
- name: Cython
  role: Implements the recursive bond-breaking/edge-removal fragmentation algorithm
    generating the DAG
- name: RDKit
  role: Data preprocessing and molecule/atom/bond featurization of the input graph
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_content_hash: sha256:7f4bcd57aafe17ca7d6c66022ead871497a2dcd6ea7847841b7f594ef5347d28
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - FraGNNet
  - Python
  - Pytorch
  - Pytorch Geometric
  - Pytorch Lightning
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# msms-spectrum-prediction-inference

## Summary

Runs the FraGNNet two-stage probabilistic GNN (Molecule GNN + Fragment GNN) to predict an MS/MS spectrum from a molecular structure via its fragmentation DAG, producing a table of monoisotopic-mass peaks with intensities and formula/fragment annotations. This is the C2MS inference step used both for prediction-error benchmarking and to generate candidate spectra for retrieval-based MS2C.

## When to use

Use when a trained C2MS model must convert a molecular structure into a predicted tandem MS/MS spectrum — e.g. to predict spectra for PubChem molecules absent from NIST20 so they can be ranked against an unknown spectrum.

## When NOT to use

- When an experimental MS/MS spectrum is already available in a spectral library such as NIST20 — retrieval against real spectra is the baseline and predicted spectra should augment, not replace, it
- When the target chemistry involves complex rearrangements (e.g. cyclizations) that the recursive edge-removal algorithm cannot account for, as these produce unexplained Outside-of-Support (OS) peaks
- When evaluating spectra from unmerged data or experimental conditions beyond the training scope (different instrument platforms, non-[M+H]+ precursor adducts), where performance has not been established

## Inputs

- Molecular structure (SMILES/MOL file) of the query compound, featurized with RDKit
- Hydrogen-stripped heavy-atom skeleton graph H(G) of the input molecule
- Trained FraGNNet checkpoint (FraGNNet-D4, fragmentation depth d=4, hydrogen tolerance j=4)
- Model/config file modeled on config/template.yml for the PyTorch Lightning module

## Outputs

- Predicted MS/MS spectrum as a table of monoisotopic-mass peaks with intensities
- Formula/fragment annotations per peak from the latent fragment distribution
- Latent distributions P(n), P(f|n), P(f), and mass distribution P(m) for interpretability

## How to apply

Set up the fragnnet environment (CUDA 11.8 per the paper's experiments) and load a trained checkpoint through the PyTorch Lightning module built on config/template.yml. Featurize the input molecular graph with RDKit, strip hydrogens to obtain the heavy-atom skeleton H(G), and run the Cython recursive edge-removal fragmentation algorithm with depth d=4 (FraGNNet-D4) and hydrogen tolerance j=4 to build the approximate fragmentation DAG. Run the Molecule GNN (GINE via PyTorch Geometric) over the molecular graph, then the ablated node-wise Fragment GNN over DAG nodes featurized by average atom embedding, subgraph formula f_n, and DAG depth. Each DAG node's output MLP emits a 2j+1 = 9-dimensional logit vector for P(n, f_n^i), i in {-j,...,j}; normalize, marginalize and apply Bayes' theorem to derive P(n), P(f|n), P(f), and the mass distribution P(m) mapping each formula to its monoisotopic mass (Dirac delta at inference). Emit the spectrum by summing per-node mass distributions, yielding peaks at monoisotopic fragment masses from M_hat(G, d, j) — with d in {3,4} and j=4 capturing most total peak intensity in real MS/MS data.

## Related tools

- **FraGNNet** (Two-stage probabilistic GNN performing the C2MS prediction from the fragmentation DAG) — github.com/FraGNNet/fragnnet
- **Pytorch** (Framework implementing the FraGNNet model and inference (version 2.1, CUDA 11.8; repo README pins torch==2.4.1 with cu121))
- **Pytorch Lightning** (Training/inference module wrapping FraGNNet with checkpoint building and config-driven runners) — https://github.com/Lightning-AI/pytorch-lightning
- **Pytorch Geometric** (Implements the GINE Molecule GNN graph-neural-network modules)
- **Cython** (Implements the recursive bond-breaking/edge-removal fragmentation algorithm generating the DAG)
- **RDKit** (Data preprocessing and molecule/atom/bond featurization of the input graph)

## Examples

```
bash env/env_setup_gpu_py311_cu121_250602.sh; conda activate FRAGNNET-GPU  # then run inference with a FraGNNet-D4 checkpoint from config/template.yml
```

## Evaluation signals

- The predicted spectrum contains peaks only at monoisotopic fragment masses from M_hat(G, d, j) — no peaks outside the support of the formula library
- On NIST20 held-out molecules, binned cosine similarity C_BIN reaches the reported FraGNNet-D4 level (~0.736 on the InChIKey split)
- In the 50-compound PubChem candidate retrieval task, Top-1 accuracy reproduces Table 2 values (36.0 ± 0.4 InChIKey split, 30.3 ± 0.6 Scaffold split, mean ± sd over 5 random seeds)
- Each DAG node's output MLP logits have dimension 2j+1 = 9 (for j=4) and the derived distributions P(n), P(f|n), P(f), P(m) are properly normalized
- Fraction of peak intensity Inside the Support (IS) is high; OS (Outside of Support) peaks should be a small residual — large OS mass fraction signals fragmentation-algorithm failure

## Limitations

- The recursive edge-removal fragmentation algorithm is not parallelized, limiting runtime on large candidate libraries
- The algorithm fails to account for some fragments, as evidenced by unexplained OS peaks in predicted spectra
- Complex chemistry such as cyclizations dramatically expands the fragment search space and is challenging to model
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms, precursor adducts beyond [M+H]+ HCD Orbitrap data)

## Evidence

- [methods] Recursive bond-breaking to generate plausible molecule fragments: [evidence span withheld — anchor status: not_found]
- [methods] Fragmentation DAG depth and hydrogen tolerance: [evidence span withheld — anchor status: not_found]
- [methods] Two-stage architecture: [evidence span withheld — anchor status: not_found]
- [readme] Repository provides code and trained models for spectrum prediction: [evidence span withheld — anchor status: external_doc]
