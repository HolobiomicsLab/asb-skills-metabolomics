---
name: hydrogen-count-tolerance-formula-enumeration
description: Use when a recursive bond-breaking fragmentation DAG has nodes annotated
  with heavy-atom formulae f_n and attached hydrogen counts h_n, and you must map
  predicted fragments onto observed MS/MS peaks via formula masses (e.g. on NIST20
  spectra).
when_to_use_negative:
- Formula annotation analysis of glycan or peptide spectra — the paper explicitly
  excluded these and focused on small molecules
- Explaining OS (Outside of the Support) peaks arising from complex chemistry such
  as cyclizations — the algorithm fails to account for these fragments, so hydrogen-tolerant
  enumeration will not cover them
- Cases where only a single fragment is possible for a formula — these degenerate
  single-fragment formulae are excluded from the agreement analysis
tools:
- name: RDKit
  role: Parses input molecular structures into molecular graphs and supports construction
    of the heavy-atom skeleton H(G) by removing hydrogens
  repo: github.com/FraGNNet/fragnnet
- name: Cython
  role: Implements the recursive fragmentation (edge-removal) algorithm whose DAG
    node formulae are enumerated with hydrogen tolerance j=4
  repo: github.com/FraGNNet/fragnnet
- name: Python
  role: Language for the FraGNNet model and preprocessing pipeline that serializes
    DAG nodes with formula and mass annotations
  repo: github.com/FraGNNet/fragnnet
- name: NumPy
  role: Numerical computation of monoisotopic masses {m_n^{-j},...,m_n^{j}} for enumerated
    formulae
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:b332a8fc0d1cc21b0ed6af5f1d2486fdb738e93a7c59a66e1237ca1fc7a5095e
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - NumPy
  - Python
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Hydrogen-count tolerance formula enumeration

## Summary

For each node n of the approximate fragmentation DAG G_F^d, enumerate the hydrogen-tolerant formula set {f_n^{-j},...,f_n^{j}} with tolerance j=4 around the node's heavy-atom formula f_n, and compute each variant's monoisotopic mass. This bridges the fragment distribution predicted over the DAG to concrete MS/MS peak masses.

## When to use

Use when a recursive bond-breaking fragmentation DAG has nodes annotated with heavy-atom formulae f_n and attached hydrogen counts h_n, and you must map predicted fragments onto observed MS/MS peaks via formula masses (e.g. on NIST20 spectra).

## When NOT to use

- Formula annotation analysis of glycan or peptide spectra — the paper explicitly excluded these and focused on small molecules
- Explaining OS (Outside of the Support) peaks arising from complex chemistry such as cyclizations — the algorithm fails to account for these fragments, so hydrogen-tolerant enumeration will not cover them
- Cases where only a single fragment is possible for a formula — these degenerate single-fragment formulae are excluded from the agreement analysis

## Inputs

- Input molecular graph G parsed from a molecular structure (SMILES/compound record) via RDKit
- Heavy-atom skeleton H(G) with all hydrogen atoms removed
- Approximate fragmentation DAG G_F^d produced by recursive bond-breaking with depth d in {3,4}
- Per-node annotations: fragment subgraph G_n, heavy-atom formula f_n, attached hydrogen count h_n
- Hydrogen tolerance parameter j (j=4 in the paper) and DAG depth d

## Outputs

- Per-node hydrogen-tolerant formula sets {f_n^{-j},...,f_n^{j}}
- Per-node monoisotopic mass lists {m_n^{-j},...,m_n^{j}}
- Derived mass set M_hat(G,d,j) supporting spectrum prediction over the DAG
- Serialized JSON graph file per molecule whose nodes carry subgraph, formula, and mass annotations and whose edges are single-bond-removal parent-child relations

## How to apply

First parse the input structure with RDKit and strip hydrogens to obtain the heavy-atom skeleton H(G), since |H2(G)| is on average ~43% smaller than |E| on NIST20; hydrogen bookkeeping is then handled by tolerance enumeration rather than explicit atoms. Run the recursive edge-removal (bond-breaking) algorithm — implemented in Cython in the paper — expanding nodes at most d hops from the root (d=4, or d=3 as the cheaper variant), producing the approximate fragmentation DAG G_F^d. For each DAG node n, record the fragment subgraph G_n, its heavy-atom molecular formula f_n, and the attached hydrogen count h_n, then enumerate the hydrogen-tolerant formulae {f_n^{-j},...,f_n^{j}} with j=4 and their monoisotopic masses {m_n^{-j},...,m_n^{j}} (e.g. mass(C2O1H5)=45.03404 Da). The resulting derived mass set M_hat(G,d,j) with d in {3,4} and j=4 is what converts the model's fragment distribution into a predicted spectrum; a candidate annotation is discarded if its formula mass lies more than 10 ppm from the peak mass.

## Related tools

- **RDKit** (Parses input molecular structures into molecular graphs and supports construction of the heavy-atom skeleton H(G) by removing hydrogens) — github.com/FraGNNet/fragnnet
- **Cython** (Implements the recursive fragmentation (edge-removal) algorithm whose DAG node formulae are enumerated with hydrogen tolerance j=4) — github.com/FraGNNet/fragnnet
- **Python** (Language for the FraGNNet model and preprocessing pipeline that serializes DAG nodes with formula and mass annotations) — github.com/FraGNNet/fragnnet
- **NumPy** (Numerical computation of monoisotopic masses {m_n^{-j},...,m_n^{j}} for enumerated formulae)

## Evaluation signals

- Sanity-check monoisotopic mass computation against the paper's reference value: mass(C2O1H5) = 45.03404 Da
- Verify DAG structural invariants: root corresponds to the original graph, leaves correspond to individual atoms, and every edge is a single-edge-removal parent-child relation
- Confirm that the derived mass set M_hat(G,d,j) with d in {3,4} and j=4 captures most of the total peak intensity in real MS/MS data, as reported for NIST20
- Check that annotations with formula masses more than 10 ppm from their associated peak mass are excluded downstream

## Limitations

- The recursive edge-removal fragmentation algorithm is not parallelized, which limits runtime when regenerating DAGs at scale
- The algorithm fails to account for some fragments, evidenced by unexplained OS peaks in the spectra
- Unexplained peaks may arise from complex chemistry such as cyclizations, which dramatically expand the fragment search space and are challenging to model

## Evidence

- [methods] calculating M_hat(G,d,j) (Definition 4.3) with d in {3,4} and j=4 can effectively capture most of the total peak intensity in real MS/MS data: [evidence span withheld — anchor status: not_found]
- [methods] The graph H(G) is called a heavy atom skeleton of G if H(G) is the largest connected subgraph of G such that omega_a != H (hydrogen): [evidence span withheld — anchor status: not_found]
- [readme] A **recursive bond-breaking algorithm** for fragment generation.: [evidence span withheld — anchor status: external_doc]
