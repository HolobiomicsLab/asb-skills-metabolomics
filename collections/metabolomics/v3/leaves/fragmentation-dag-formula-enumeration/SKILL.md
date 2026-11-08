---
name: fragmentation-dag-formula-enumeration
description: Use when building the formula support underlying a predicted MS/MS spectrum
  from a molecular structure, or when comparing a fragment-derived formula distribution
  against fixed-vocabulary C2MS models (e.g. top-10,000 GrAFF-MS formulae).
when_to_use_negative:
- The goal is a fixed formula-vocabulary prediction where the support is pre-defined
  (e.g. GrAFF-MS top 10,000 product and neutral loss formulae by total intensity)
  rather than derived from the molecule's structure
- The spectra are glycan or peptide spectra, which the authors excluded from formula
  annotation analysis, or annotations cannot satisfy the 10 ppm formula-mass tolerance
- Only a single fragment is possible for the molecule (degenerate case), which the
  authors exclude from agreement analysis
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: FraGNNet
  role: Deep probabilistic model whose recursive bond-breaking algorithm generates
    the fragmentation DAG and whose GNNs predict the fragment/formula distribution
  repo: github.com/FraGNNet/fragnnet
- name: Cython
  role: Implements the recursive fragmentation algorithm for the DAG
- name: RDKit
  role: Data preprocessing and molecule featurization of the input molecular graph
- name: Pytorch Geometric
  role: Implements the Molecule GNN and Fragment GNN modules that predict the distribution
    over DAG fragments
  repo: Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:ed9d59e2e581fb6426f4a7f0572bd767137c17b50037db906fc3c3b688dbc95d
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - FraGNNet
  - GrAFF-MS
  - Python
  - Pytorch Geometric
  - RDKit
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# fragmentation-dag-formula-enumeration

## Summary

Enumerate plausible molecule fragments via a recursive bond-breaking algorithm organized as an approximate fragmentation DAG, then derive the induced formula distribution P_theta(f) used to generate predicted MS/MS peaks. This replaces a fixed formula vocabulary with a structure-derived formula support.

## When to use

Use when building the formula support underlying a predicted MS/MS spectrum from a molecular structure, or when comparing a fragment-derived formula distribution against fixed-vocabulary C2MS models (e.g. top-10,000 GrAFF-MS formulae).

## When NOT to use

- The goal is a fixed formula-vocabulary prediction where the support is pre-defined (e.g. GrAFF-MS top 10,000 product and neutral loss formulae by total intensity) rather than derived from the molecule's structure
- The spectra are glycan or peptide spectra, which the authors excluded from formula annotation analysis, or annotations cannot satisfy the 10 ppm formula-mass tolerance
- Only a single fragment is possible for the molecule (degenerate case), which the authors exclude from agreement analysis

## Inputs

- Input molecular graph G (e.g. SMILES featurized with RDKit)
- Approximate fragmentation DAG parameters: depth d in {3,4}, hydrogen tolerance j=4
- Peak mass annotations with formula assignments from NIST20 MS/MS expert-curated formula annotations

## Outputs

- Approximate fragmentation DAG M_hat(G,d,j) of plausible molecule fragments
- Marginalized formula distribution P_theta(f) per test molecule
- Per-molecule formula support-size statistics (e.g. formula_support_stats.csv with median and quartiles)

## How to apply

First reduce the input molecule G to its heavy atom skeleton H(G) (the largest connected subgraph with non-hydrogen atoms), then run the recursive bond-breaking algorithm implemented in Cython to generate the approximate fragmentation DAG M_hat(G,d,j). Use DAG depth d in {3,4} and hydrogen tolerance j=4, which the authors show captures most total peak intensity in real MS/MS data. The probabilistic model (Molecule GNN g_theta^MOL followed by Fragment GNN g_theta^FRAG, built with PyTorch Geometric) predicts a distribution over fragments in the DAG, and this fragment distribution induces the formula distribution. Convert formulae to peak masses using formula masses, excluding annotations whose formula masses deviate more than 10 ppm from the associated peak mass, and exclude degenerate cases where only a single fragment is possible. Per-molecule support size (number of formulae with nonzero probability) can then be summarized (median, quartiles) with pandas/numpy and compared against baseline supports such as GrAFF-MS's top 10,000 product and neutral-loss formulae.

## Related tools

- **FraGNNet** (Deep probabilistic model whose recursive bond-breaking algorithm generates the fragmentation DAG and whose GNNs predict the fragment/formula distribution) — github.com/FraGNNet/fragnnet
- **Cython** (Implements the recursive fragmentation algorithm for the DAG)
- **RDKit** (Data preprocessing and molecule featurization of the input molecular graph)
- **Pytorch Geometric** (Implements the Molecule GNN and Fragment GNN modules that predict the distribution over DAG fragments) — Lightning-AI/pytorch-lightning

## Evaluation signals

- The fragmentation DAG uses depth d in {3,4} and hydrogen tolerance j=4; settings outside these ranges are not validated by the paper
- All retained formula annotations must lie within 10 ppm of their associated peak mass
- The induced formula support is structure-dependent (median support size on the order of hundreds of formulae per molecule for FraGNNet-D4), not a fixed vocabulary size such as GrAFF-MS's 10,000 formulae
- Captured peak intensity should be high: d in {3,4}, j=4 'can effectively capture most of the total peak intensity in real MS/MS data'
- Degenerate single-fragment cases must be excluded before comparing formula agreement

## Limitations

- The recursive edge-removal fragmentation algorithm is not parallelized, which limits runtime
- The algorithm fails to account for some fragments, evidenced by unexplained Outside-of-the-Support (OS) peaks in the spectra
- Complex chemistry such as cyclizations dramatically expands the fragment search space and is challenging to model

## Evidence

- [methods] recursive bond-breaking generates fragments: [evidence span withheld — anchor status: not_found]
- [methods] DAG parameters capture intensity: [evidence span withheld — anchor status: not_found]
- [methods] Cython implementation: [evidence span withheld — anchor status: not_found]
