---
name: heavy-atom-skeleton-extraction
description: Use when you have a molecular structure (e.g. SDF/MolBlock from a spectral
  library entry such as NIST20) and need the heavy-atom-only graph as input to combinatorial
  bond-breaking or fragmentation-DAG generation.
when_to_use_negative:
- The graph is already a heavy-atom skeleton or has no hydrogens (re-running is a
  no-op and adds no information)
- The analysis requires explicit hydrogen placement or connectivity (e.g. adduct/protonation
  site assignment); the skeleton deliberately discards hydrogens, which the paper
  handles only via hydrogen tolerance j=4 on fragment formulae
- The task operates on spectra or peak lists rather than molecular structures (e.g.
  formula annotation or peak-intensity coverage scoring) — those consume the DAG,
  not the skeleton
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: RDKit
  role: Parses the input molecular structure into a molecular graph and performs hydrogen
    removal to produce the heavy atom skeleton H(G)
- name: Cython
  role: Hosts the downstream recursive bond-breaking/fragmentation algorithm (Algorithm
    1) that consumes H(G) to build the fragmentation DAG
- name: Python
  role: Implementation language for the preprocessing pipeline wrapping RDKit featurization
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_content_hash: sha256:774f8a40dde718619542d0387fbbaf49f7696d615a943a91a723f003ad9428c9
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
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# heavy-atom-skeleton-extraction

## Summary

Derives the heavy atom skeleton H(G) — the largest connected subgraph of a parsed molecular graph with all hydrogen atoms removed — so that recursive bond-breaking fragmentation operates on heavy-atom edges only. On NIST20 this shrinks the edge set by ~43% on average, making the downstream fragmentation DAG cheaper to build.

## When to use

Use when you have a molecular structure (e.g. SDF/MolBlock from a spectral library entry such as NIST20) and need the heavy-atom-only graph as input to combinatorial bond-breaking or fragmentation-DAG generation.

## When NOT to use

- The graph is already a heavy-atom skeleton or has no hydrogens (re-running is a no-op and adds no information)
- The analysis requires explicit hydrogen placement or connectivity (e.g. adduct/protonation site assignment); the skeleton deliberately discards hydrogens, which the paper handles only via hydrogen tolerance j=4 on fragment formulae
- The task operates on spectra or peak lists rather than molecular structures (e.g. formula annotation or peak-intensity coverage scoring) — those consume the DAG, not the skeleton

## Inputs

- Molecular structure record for a compound from the NIST20 MS/MS library (SDF/MolBlock/MolFile parseable by RDKit)
- Molecular graph G=(V,E) with explicit or implicit hydrogens

## Outputs

- Heavy atom skeleton H(G): the hydrogen-free connected subgraph of G (Definition 4.2)
- Input to the recursive bond-breaking fragment generator (Algorithm 1) that yields the approximate fragmentation DAG G_F^d

## How to apply

Parse each molecule with RDKit into a molecular graph G=(V,E), then strip all hydrogen atoms to obtain the heavy atom skeleton H(G) as defined in Definition 4.2 (the largest connected subgraph in which no atom is hydrogen). This is the first preprocessing step before running the Cython-implemented recursive edge-removal (bond-breaking) algorithm that builds the approximate fragmentation DAG G_F^d with depth d in {3,4}. Hydrogens removed at this stage are recovered downstream by enumerating hydrogen-tolerant formulae {f_n^{-j},...,f_n^{j}} around each DAG node's heavy-atom formula f_n with tolerance j=4, so exact hydrogen placement does not need to be retained. Keep H(G) as a connected subgraph so that the DAG root corresponds to the original molecule graph and leaves correspond to individual heavy atoms.

## Related tools

- **RDKit** (Parses the input molecular structure into a molecular graph and performs hydrogen removal to produce the heavy atom skeleton H(G))
- **Cython** (Hosts the downstream recursive bond-breaking/fragmentation algorithm (Algorithm 1) that consumes H(G) to build the fragmentation DAG)
- **Python** (Implementation language for the preprocessing pipeline wrapping RDKit featurization)

## Examples

```
mol = Chem.RemoveHs(Chem.MolFromMolBlock(sdf_block))  # heavy atom skeleton H(G) per Definition 4.2
```

## Evaluation signals

- H(G) contains no hydrogen-labeled atoms (all omega_a != H per Definition 4.2)
- H(G) is connected and node/edge sets are a subset of the original G=(V,E)
- Edge count drops substantially relative to E — on NIST20 the heavy-atom graph |H2(G)| is on average ~43% smaller than |E|
- After downstream DAG construction, the root of G_F^d corresponds to the original (heavy-atom) graph and leaves correspond to individual atoms, with every edge a single-bond-removal parent-child relation

## Limitations

- Hydrogen placement is lost; the framework only recovers hydrogen counts via hydrogen-tolerant formula enumeration with tolerance j=4 around each node's heavy-atom formula
- The downstream recursive edge-removal fragmentation is not parallelized, so skeleton construction for large candidate sets can be a runtime bottleneck
- Even with a correct skeleton, the fragmentation algorithm fails to account for some fragments, evidenced by unexplained Outside-of-the-Support (OS) peaks possibly arising from complex chemistry such as cyclizations

## Evidence

- [methods] The graph H(G) is called a heavy atom skeleton of G if H(G) is the largest connected subgraph of G such that omega_a != H (hydrogen): [evidence span withheld — anchor status: not_found]
- [methods] The data preprocessing and molecule featurization used RDKit (Landrum, 2022): [evidence span withheld — anchor status: not_found]
- [methods] The recursive fragmentation algorithm was implemented in Cython (Behnel et al., 2011): [evidence span withheld — anchor status: not_found]
- [readme] A recursive bond-breaking algorithm for fragment generation.: [evidence span withheld — anchor status: not_found]
- [methods] The recursive bond-breaking algorithm was implemented in Cython: [evidence span withheld — anchor status: not_found]
