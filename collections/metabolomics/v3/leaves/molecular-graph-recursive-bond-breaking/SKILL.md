---
name: molecular-graph-recursive-bond-breaking
description: Use when building the fragment vocabulary underlying predicted MS/MS
  spectra for a known molecular structure — e.g. generating fragmentation DAGs for
  molecules from NIST20 so a probabilistic model can distribute intensity over formulae
  and peaks.
when_to_use_negative:
- The input is already a peak list or spectrum with no molecular structure — this
  skill needs the full molecular graph G, not spectral data alone.
- 'High-mass molecules where combinatorial fragment enumeration explodes: the paper
  notes the algorithm can fail to account for fragments (unexplained OS peaks) and
  that cyclizations dramatically expand the search space.'
- Peptide/glycan spectra — the paper excluded these from its small-molecule formula-annotation
  analysis.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3520
tools:
- name: RDKit
  role: Data preprocessing and molecule featurization; builds the molecular graph
    G=(V,E) and computes the heavy-atom skeleton H(G) by removing hydrogen atoms
- name: Cython
  role: Implements the recursive fragmentation (edge-removal) algorithm for generating
    the fragmentation DAG
- name: Python
  role: Host language for the FraGNNet model and the fragmentation pipeline
- name: NumPy
  role: Numerical computation over the generated fragment formulae and monoisotopic
    masses
- name: FraGNNet
  role: The model that consumes the generated fragmentation DAG to parameterize the
    fragment distribution; code at github.com/FraGNNet/fragnnet
  repo: github.com/FraGNNet/fragnnet
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:430aadb55ec63de33e4ed9813559aec45d93de88e33a80bf35af952357068b92
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# molecular-graph-recursive-bond-breaking

## Summary

Generates a molecule's fragmentation DAG by recursively removing single bonds from the heavy-atom skeleton up to depth d and recording each fragment's subgraph, heavy-atom formula, and attached-hydrogen count with tolerance j, the combinatorial core of FraGNNet's C2MS spectral prediction.

## When to use

Use when building the fragment vocabulary underlying predicted MS/MS spectra for a known molecular structure — e.g. generating fragmentation DAGs for molecules from NIST20 so a probabilistic model can distribute intensity over formulae and peaks.

## When NOT to use

- The input is already a peak list or spectrum with no molecular structure — this skill needs the full molecular graph G, not spectral data alone.
- High-mass molecules where combinatorial fragment enumeration explodes: the paper notes the algorithm can fail to account for fragments (unexplained OS peaks) and that cyclizations dramatically expand the search space.
- Peptide/glycan spectra — the paper excluded these from its small-molecule formula-annotation analysis.

## Inputs

- Molecular structure parseable by RDKit into a molecular graph G=(V,E) (e.g. SMILES/MolBlock from the NIST20 compound set)
- Depth parameter d (d=4 standard; d=3 as the cheaper variant)
- Hydrogen tolerance j (j=4 per the paper)

## Outputs

- Fragmentation DAG G_F^d serialized as a JSON graph file; nodes carry fragment subgraph G_n, heavy-atom molecular formula f_n, attached hydrogen count h_n, and enumerated formulae/masses {f_n^{-j},…,f_n^{j}} / {m_n^{-j},…,m_n^{j}}; edges are single-bond-removal parent–child relations
- Derived mass set M_hat(G,d,j) of candidate peak masses

## How to apply

Parse the input molecule with RDKit into a graph G=(V,E), strip hydrogens to obtain the heavy-atom skeleton H(G) — |H2(G)| averages ~43% smaller than |E| on NIST20 — then run the recursive edge-removal bond-breaking algorithm (implemented in Cython) that expands nodes at most d hops from the root, using d=4 (or the cheaper d=3 variant) to yield the approximate fragmentation DAG G_F^d. For each DAG node n, record the fragment subgraph G_n, its heavy-atom molecular formula f_n (e.g. mass(C2O1H5)=45.03404 Da), the attached hydrogen count h_n, and enumerate the hydrogen-tolerant formulae {f_n^{-j},…,f_n^{j}} and their monoisotopic masses with tolerance j=4. Serialize the DAG as a JSON graph whose nodes carry subgraph, formula, and mass annotations and whose edges connect each parent fragment to a child obtained by removing exactly one bond. Validate by checking structural invariants (root = original graph, leaves = individual atoms, every edge a single-edge-removal relation) and confirm that M_hat(G,d,j) with d∈{3,4}, j=4 captures most of the total peak intensity in real MS/MS data.

## Related tools

- **RDKit** (Data preprocessing and molecule featurization; builds the molecular graph G=(V,E) and computes the heavy-atom skeleton H(G) by removing hydrogen atoms)
- **Cython** (Implements the recursive fragmentation (edge-removal) algorithm for generating the fragmentation DAG)
- **Python** (Host language for the FraGNNet model and the fragmentation pipeline)
- **NumPy** (Numerical computation over the generated fragment formulae and monoisotopic masses)
- **FraGNNet** (The model that consumes the generated fragmentation DAG to parameterize the fragment distribution; code at github.com/FraGNNet/fragnnet) — github.com/FraGNNet/fragnnet

## Examples

```
python preproc_scripts/generate_fragdag.py --input molecule.smi --depth 4 --hydrogen-tolerance 4 --out fragdag.json  # see https://github.com/FraGNNet/fragnnet preproc_scripts/README.md
```

## Evaluation signals

- Structural invariants: the DAG root corresponds to the original molecular graph, leaves correspond to individual atoms, and every edge is a single-edge-removal parent–child relation.
- Per-node annotation completeness: each node carries a fragment subgraph, heavy-atom formula f_n, attached-hydrogen count h_n, and enumerated formula/mass lists up to tolerance j=4.
- Node-expansion budget: no node is expanded more than d hops from the root (d=4, or d=3 for the cheaper variant).
- Coverage check on real data: M_hat(G,d,j) with d∈{3,4}, j=4 captures most of the total peak intensity in real MS/MS spectra (OS/IS split — peaks far from the predicted mass set are Outside of the Support).
- JSON serialization round-trips: DAG files load as graphs whose nodes carry subgraph/formula/mass annotations.

## Limitations

- Recursive edge-removal operations are not parallelized, limiting runtime on large compounds.
- The algorithm fails to account for some fragments, evidenced by unexplained OS peaks in the spectra.
- Unexplained peaks may arise from complex chemistry such as cyclizations, which dramatically expand the fragment search space and are challenging to model.
- The exact (non-relaxed) fragmentation DAG is only feasible for small compounds; chemical constraints are relaxed for greater efficiency and scale, following the ICEBERG-style approach.

## Evidence

- [methods] The graph H(G) is called a heavy atom skeleton of G if H(G) is the largest connected subgraph of G such that omega_a != H (hydrogen): [evidence span withheld — anchor status: not_found]
- [methods] calculating M_hat(G,d,j) (Definition 4.3) with d in {3,4} and j=4 can effectively capture most of the total peak intensity in real MS/MS data: [evidence span withheld — anchor status: not_found]
- [methods] The data preprocessing and molecule featurization used RDKit (Landrum, 2022): [evidence span withheld — anchor status: not_found]
- [methods] The recursive fragmentation algorithm was implemented in Cython (Behnel et al., 2011): [evidence span withheld — anchor status: not_found]
- [discussion] the fragmentation algorithm currently depends on recursive edge-removal operations, which could be parallelized to accelerate runtime.: [evidence span withheld — anchor status: not_found]
- [methods] a recursive bond-breaking algorithm (Section 4.2) generates a set of plausible molecule fragments: [evidence span withheld — anchor status: not_found]
- [readme] This repository contains the code and models for the paper FraGNNet: A Deep Probabilistic Model for Mass Spectrum Prediction: [evidence span withheld — anchor status: not_found]
- [methods] The code is available at github.com/FraGNNet/fragnnet.: [evidence span withheld — anchor status: not_found]
