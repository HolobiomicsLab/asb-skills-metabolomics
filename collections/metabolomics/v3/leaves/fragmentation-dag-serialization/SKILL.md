---
name: fragmentation-dag-serialization
description: 'Use when building the fragment set for a combinatorial bond-breaking
  MS/MS peak predictor: molecules must be converted from structures into a deterministic
  fragmentation DAG with formula/mass annotations before probabilistic fragment-distribution
  modelling.'
when_to_use_negative:
- The input is already a serialized fragment DAG / JSON graph file — no regeneration
  is needed (e.g., resuming from preprocessed data).
- The target is a formula-only predictor that does not use structural bond-breaking
  fragments (e.g., the GrAFF-MS top-10,000 product and neutral loss formulae selection).
- The chemistry of interest involves rearrangements such as cyclizations that dramatically
  expand the fragment search space — the deterministic edge-removal algorithm fails
  to account for such fragments (unexplained OS peaks).
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RDKit
  role: 'Data preprocessing and molecule featurization: parses input structures into
    molecular graphs and supports hydrogen removal to build the heavy-atom skeleton
    H(G)'
- name: Cython
  role: Implementation language for the recursive (edge-removal) fragmentation algorithm
    for performance
- name: FraGNNet
  role: The model whose fragment vocabulary is the DAG produced by this skill; the
    serialized DAGs feed its Fragment GNN stage
  repo: github.com/FraGNNet/fragnnet
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:72d7120d5848737d7cafc79050f15d4f472ed64b7fdc944c9e5d0558f217acfc
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - FraGNNet
  - NumPy
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# fragmentation-dag-serialization

## Summary

Generates and serializes the approximate fragmentation DAG G_F^d that underlies FraGNNet's fragment vocabulary: a recursive bond-breaking (edge-removal) algorithm over the heavy-atom skeleton, whose nodes carry fragment subgraphs, heavy-atom formulae, hydrogen counts, hydrogen-tolerant formulae (tolerance j=4), and monoisotopic masses, all exported as a JSON graph per molecule.

## When to use

Use when building the fragment set for a combinatorial bond-breaking MS/MS peak predictor: molecules must be converted from structures into a deterministic fragmentation DAG with formula/mass annotations before probabilistic fragment-distribution modelling.

## When NOT to use

- The input is already a serialized fragment DAG / JSON graph file — no regeneration is needed (e.g., resuming from preprocessed data).
- The target is a formula-only predictor that does not use structural bond-breaking fragments (e.g., the GrAFF-MS top-10,000 product and neutral loss formulae selection).
- The chemistry of interest involves rearrangements such as cyclizations that dramatically expand the fragment search space — the deterministic edge-removal algorithm fails to account for such fragments (unexplained OS peaks).

## Inputs

- Molecular structures parsed by RDKit into molecular graphs G=(V,E) (e.g., NIST20 compounds)
- Depth parameter d (d=4 standard; d=3 as cheaper variant)
- Hydrogen tolerance j (j=4 as used in the paper)

## Outputs

- Per-molecule JSON graph file encoding the approximate fragmentation DAG G_F^d
- Per-node annotations: fragment subgraph G_n, heavy-atom formula f_n, attached hydrogen count h_n
- Hydrogen-tolerant formulae {f_n^{-j},...,f_n^{j}} and their monoisotopic masses {m_n^{-j},...,m_n^{j}}
- Derived mass set M_hat(G,d,j) usable for peak-intensity coverage in MS/MS spectra

## How to apply

Parse each input molecular structure with RDKit into a molecular graph G=(V,E), then strip hydrogens to obtain the heavy-atom skeleton H(G) (justified because |H2(G)| is on average ~43% smaller than |E| on NIST20). Run the recursive edge-removal (bond-breaking) algorithm — implemented in Cython for speed — which expands nodes at most d hops from the root, using d=4 (d=3 as the cheaper variant), to produce G_F^d. For each DAG node n, record the fragment subgraph G_n, its heavy-atom molecular formula f_n, and its attached hydrogen count h_n; then enumerate the hydrogen-tolerant formulae {f_n^{-j},...,f_n^{j}} with tolerance j=4 and their monoisotopic masses (e.g., mass(C2O1H5)=45.03404 Da). Serialize each molecule's DAG as a JSON graph file whose nodes carry subgraph, formula, and mass annotations and whose edges connect a parent fragment to a child fragment obtained by removing a single bond. The resulting mass set M_hat(G,d,j), with d in {3,4} and j=4, is then usable to explain peak intensities in MS/MS spectra from sources such as NIST20.

## Related tools

- **RDKit** (Data preprocessing and molecule featurization: parses input structures into molecular graphs and supports hydrogen removal to build the heavy-atom skeleton H(G))
- **Cython** (Implementation language for the recursive (edge-removal) fragmentation algorithm for performance)
- **FraGNNet** (The model whose fragment vocabulary is the DAG produced by this skill; the serialized DAGs feed its Fragment GNN stage) — github.com/FraGNNet/fragnnet

## Evaluation signals

- Structural invariant: the DAG root corresponds to the original molecular graph G
- Structural invariant: DAG leaves correspond to individual atoms of the heavy-atom skeleton
- Every DAG edge is a single-edge-removal (single-bond) parent-child relation
- Each node carries the triple (fragment subgraph G_n, heavy-atom formula f_n, hydrogen count h_n) plus formulae and monoisotopic masses for tolerance j=4 (spot-check e.g. mass(C2O1H5)=45.03404 Da)
- The mass set M_hat(G,d,j) with d in {3,4} and j=4 captures most of the total peak intensity on real MS/MS data (NIST20)

## Limitations

- The recursive edge-removal fragmentation algorithm is not parallelized, limiting runtime on large compounds.
- The algorithm fails to account for some fragments, evidenced by unexplained Outside-of-Support (OS) peaks in spectra.
- Complex chemistry such as cyclizations dramatically expands the fragment search space and is challenging to model; performance has not been evaluated on unmerged spectra or across broader instrument platforms and precursor adducts (only Orbitrap HCD, [M+H]+ adduct in the paper's data).

## Evidence

- [methods] calculating M_hat(G,d,j) (Definition 4.3) with d in {3,4} and j=4 can effectively capture most of the total peak intensity in real MS/MS data: [evidence span withheld — anchor status: not_found]
- [methods] The recursive fragmentation algorithm was implemented in Cython (Behnel et al., 2011): [evidence span withheld — anchor status: not_found]
- [methods] The data preprocessing and molecule featurization used RDKit (Landrum, 2022): [evidence span withheld — anchor status: not_found]
- [readme] A **recursive bond-breaking algorithm** for fragment generation.: [evidence span withheld — anchor status: external_doc]
- [discussion] the fragmentation algorithm currently depends on recursive edge-removal operations, which could be parallelized to accelerate runtime.: [evidence span withheld — anchor status: not_found]
