---
name: molecular-network-annotation-propagation
description: Use when you have a GNPS-style molecular network with per-spectrum candidate
  annotations and sparse spectral library matches (e.g. only ~10% seed nodes) and
  need annotations on nodes without any library match.
when_to_use_negative:
- When the goal is direct spectral library matching of individual spectra rather than
  propagation of library identities across a network.
- When NAP's local first-neighbour propagation is sufficient and a standalone Java/NAP
  pipeline is preferred over a Python package — though ChemWalker is the better choice
  for running time, scalability and maintainability.
- 'When the graph contains non-positive edge weights: pyrwr''s RWR is defined only
  on positively weighted networks.'
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ChemWalker
  role: Python package performing the random-walk propagation of spectral library
    match identities through MetFrag candidate structures
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Random walk with restart solver (power iteration) used to compute candidate
    probabilities from seed nodes on the weighted graph
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Provides combinatorial in silico fragmentation results for candidate structures
    searched in biologically relevant databases
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Scoring function used to create the weight function combining spectral network
    edges and structural similarity among candidates
- name: Network Annotation Propagation (NAP)
  role: Baseline annotation-propagation method whose first-neighbour limitation ChemWalker
    extends; used for top-10 classification comparison
- name: GNPS (Global Natural Products Social Molecular Networking)
  role: Source of the molecular network and spectral library match context for the
    benchmark
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:f76e17dfcb4230ec97800f4f20d091917d052c2ef3adc613369b0fc3d7fb1e9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - GNPS (Global Natural Products Social Molecular Networking)
  - MetFrag
  - MetFusion
  - Network Annotation Propagation (NAP)
  - pyrwr
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# molecular-network-annotation-propagation

## Summary

Propagate spectral library match identities through candidate structures on a molecular network using a random walk (ChemWalker), overcoming NAP's limitation of propagating only to the first neighbour of a spectral match.

## When to use

Use when you have a GNPS-style molecular network with per-spectrum candidate annotations and sparse spectral library matches (e.g. only ~10% seed nodes) and need annotations on nodes without any library match.

## When NOT to use

- When the goal is direct spectral library matching of individual spectra rather than propagation of library identities across a network.
- When NAP's local first-neighbour propagation is sufficient and a standalone Java/NAP pipeline is preferred over a Python package — though ChemWalker is the better choice for running time, scalability and maintainability.
- When the graph contains non-positive edge weights: pyrwr's RWR is defined only on positively weighted networks.

## Inputs

- GNPS-derived molecular network (e.g. NIST 555-spectra benchmark network)
- Per-spectrum candidate annotation tables from MetFrag in silico fragmentation
- Spectral library matches designated as seed nodes (~10% of nodes)
- MetFusion-weighted graph combining spectral-network edges and structural similarity among candidate structures

## Outputs

- Per-node candidate probabilities from the random walk (pyrwr RWR score vectors)
- Ranked candidate annotations per spectrum, including annotations on nodes not directly connected to any spectral library match
- ChemWalker vs NAP top-10 candidate classification comparison (results.csv)
- Aggregated non-neighbor annotation counts (annotation_propagation_nonneighbor_summary.csv)

## How to apply

First run MetFrag CL (tested on MetFrag2.3.-CL.jar) for in silico combinatorial fragmentation of candidate structures searched in biologically relevant databases. Combine the spectral network with structural similarity among candidate structures using the MetFusion scoring function as a weight function to produce a weighted graph. Designate spectral library matches as seed nodes (at most ~10% of nodes in the NIST 555-spectra benchmark regime) and run the random walk via pyrwr from these seeds to compute the probability of 'walking' through the candidate set; pyrwr's RWR implementation supports positively weighted networks with power-iteration parameters c (restart probability, default 0.15), epsilon (1e-9) and max_iters (100). Extract propagated candidate probabilities/rankings per spectrum, including for nodes with no direct edge to a library match. Set up a dedicated conda environment from environment.yml and install ChemWalker from its GitHub repository.

## Related tools

- **ChemWalker** (Python package performing the random-walk propagation of spectral library match identities through MetFrag candidate structures) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Random walk with restart solver (power iteration) used to compute candidate probabilities from seed nodes on the weighted graph) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Provides combinatorial in silico fragmentation results for candidate structures searched in biologically relevant databases) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Scoring function used to create the weight function combining spectral network edges and structural similarity among candidates)
- **Network Annotation Propagation (NAP)** (Baseline annotation-propagation method whose first-neighbour limitation ChemWalker extends; used for top-10 classification comparison)
- **GNPS (Global Natural Products Social Molecular Networking)** (Source of the molecular network and spectral library match context for the benchmark)

## Examples

```
conda env create -f environment.yml; conda activate chemwalker; pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- Statistical comparison of top-10 candidate classifications against NAP on the 555-spectra NIST network shows no statistical differentiation (reproduces the paper's finding).
- Nonzero count of annotations recorded on nodes with no direct edge to any spectral-library-match node, demonstrating propagation beyond first neighbours.
- RWR score vectors are well-formed: one score per node (n, 1 ndarray) with convergence within epsilon=1e-9 and max 100 power-iteration steps.
- Redundant weighted edges are summed correctly in the adjacency matrix (e.g. edges 1-2 with weights 3 and 5 yield weight 8).

## Limitations

- pyrwr's random walk with restart requires positively weighted networks; only positive weights are allowed.
- NAP's limitation, which motivated ChemWalker, is that information from spectral matches is only propagated locally to the first neighbour of a spectral match — naive local propagation cannot annotate distant nodes.
- Benchmark performance was validated with at most ~10% of nodes as seed nodes; behaviour outside this regime was not characterized here.
- ChemWalker was tested specifically on MetFrag2.3.-CL.jar; other MetFrag versions are untested in this context.

## Evidence

- [abstract] Here, we show that annotation propagation can be expanded to nodes not directly connected to spectral matches using random walks on graphs: "Here, we show that annotation propagation can be expanded to nodes not directly connected to spectral matches using random walks on graphs"
- [abstract] One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match: "One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match"
- [intro] ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by in silico fragmentation, using random walk: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by in silico fragmentation, using"
- [readme] RWR is defined on positively weighted networks; thus, only positive weights are allowed.: [evidence span withheld — anchor status: external_doc]
- [readme] ChemWalker was tested on MetFrag2.3.-CL.jar: "ChemWalker was tested on MetFrag2.3.-CL.jar"
