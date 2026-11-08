---
name: fragmentation-dag-construction
description: Use when a precursor molecular structure must be decomposed into plausible
  subgraph fragments to build a candidate fragment/formula space for MS/MS peak prediction,
  or to measure how much real peak intensity the fragment support captures.
when_to_use_negative:
- When analyzing spectra dominated by complex chemistry such as cyclizations, which
  dramatically expand the fragment search space and produce Outside-of-Support (OS)
  peaks the DAG cannot explain
- When the goal is formula annotation on glycan or peptide spectra — the paper explicitly
  excluded these and focused on small molecules
- When no molecular structure is available for the precursor (the DAG construction
  requires the input molecular graph G)
tools:
- name: RDKit
  role: Data preprocessing, molecule featurization, and hydrogen stripping to obtain
    the heavy atom skeleton H(G)
- name: Cython
  role: Implementation of the recursive bond-breaking (edge-removal) fragmentation
    algorithm (Algorithm 1)
- name: Python
  role: Host language for the fragmentation pipeline and coverage-statistic computation
provenance:
  source_task_ids:
  - task_008
  - task_001
  - task_007
  source_content_hash: sha256:24104a8eea9ea1a3c097a2de248c91fe56471e7403867a8dd9ae67c22fcdb1d0
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - FraGNNet
  - GrAFF-MS
  - PyTorch
  - PyTorch Geometric
  - PyTorch Lightning
  - Python
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# fragmentation-dag-construction

## Summary

Generates the approximate fragmentation DAG G_F^d by recursively breaking bonds on the hydrogen-stripped heavy-atom skeleton H(G) of a molecule (depth d in {3,4}, hydrogen tolerance j=4), defining the candidate fragment set that FraGNNet uses to parameterize its MS/MS spectrum distribution.

## When to use

Use when a precursor molecular structure must be decomposed into plausible subgraph fragments to build a candidate fragment/formula space for MS/MS peak prediction, or to measure how much real peak intensity the fragment support captures.

## When NOT to use

- When analyzing spectra dominated by complex chemistry such as cyclizations, which dramatically expand the fragment search space and produce Outside-of-Support (OS) peaks the DAG cannot explain
- When the goal is formula annotation on glycan or peptide spectra — the paper explicitly excluded these and focused on small molecules
- When no molecular structure is available for the precursor (the DAG construction requires the input molecular graph G)

## Inputs

- Precursor molecular structure (SMILES/MolBlock parsed into a molecular graph via RDKit)
- Hydrogen-stripped heavy-atom skeleton H(G) of the molecule
- Fragmentation depth d (3 or 4) and hydrogen tolerance j (4)
- Exported NIST20 MS/MS spectra with precursor structures (for coverage evaluation)

## Outputs

- Approximate fragmentation DAG G_F^d whose nodes are fragment subgraphs
- Per-node subgraph formulae f_n with hydrogen counts in {h_n - j, ..., h_n + j}
- Union monoisotopic mass set M̂(G, d, 4) (Definition 4.3)
- Per-(d, j) peak-intensity coverage fractions (coverage_summary.csv)

## How to apply

Parse the precursor into a molecular graph with RDKit and strip hydrogens to obtain the heavy atom skeleton H(G) (the largest connected subgraph containing no hydrogen atoms). Run the recursive edge-removal bond-breaking algorithm (Algorithm 1), implemented in Cython for speed, to depth d; use d=4 (FraGNNet-D4) or d=3 (FraGNNet-D3) per the paper's ablation. Each DAG node corresponds to a fragment subgraph; for each node n enumerate monoisotopic formula masses for hydrogen counts h_n - j to h_n + j with hydrogen tolerance j = 4 (2j+1 = 9 candidates per node), forming the union mass set M̂(G, d, 4) (Definition 4.3). Validate the DAG by matching real NIST20 peak masses against M̂(G, d, 4) and computing the fraction of total peak intensity covered — the paper found d in {3,4} with j=4 captures most total peak intensity, justifying the approximation. The DAG is consumed downstream by the Fragment GNN, whose node features include the subgraph formula f_n and DAG depth.

## Related tools

- **RDKit** (Data preprocessing, molecule featurization, and hydrogen stripping to obtain the heavy atom skeleton H(G))
- **Cython** (Implementation of the recursive bond-breaking (edge-removal) fragmentation algorithm (Algorithm 1))
- **Python** (Host language for the fragmentation pipeline and coverage-statistic computation)

## Evaluation signals

- Every DAG node is a connected subgraph of the heavy-atom skeleton H(G), and each node's enumerated formulae stay within h_n ± j (j = 4) hydrogens of the node's hydrogen count
- Matching real NIST20 peak masses against M̂(G, d, 4) yields a high fraction of total peak intensity covered for both d = 3 and d = 4 (Table 4 finding)
- All predicted spectrum peaks fall at monoisotopic fragment masses from M̂(G, d, j) — no peak outside the support
- Pipeline is deterministic for fixed (G, d, j): re-running Algorithm 1 reproduces the same DAG node and edge set

## Limitations

- The recursive edge-removal operations are not parallelized, limiting runtime on large molecules
- The algorithm fails to account for some fragments, evidenced by unexplained Outside-of-Support (OS) peaks in real spectra
- Complex chemistry such as cyclizations dramatically expands the fragment search space and is challenging to model

## Evidence

- [methods] calculating M_hat(G,d,j) (Definition 4.3) with d in {3,4} and j=4 can effectively capture most of the total peak intensity in real MS/MS data: [evidence span withheld — anchor status: not_found]
- [methods] heavy atom skeleton definition: [evidence span withheld — anchor status: not_found]
- [methods] recursive bond-breaking algorithm implementation: [evidence span withheld — anchor status: not_found]
- [readme] README fragment-generation step: [evidence span withheld — anchor status: external_doc]
