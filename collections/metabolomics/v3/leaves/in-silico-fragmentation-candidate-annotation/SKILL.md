---
name: in-silico-fragmentation-candidate-annotation
description: Use when candidate structures for spectral-library-unmatched mass signals
  must be annotated in silico before propagation, e.g. to build MetFrag 2.3 CL fragment
  results feeding a ChemWalker/MetFusion weighted graph.
when_to_use_negative:
- When the query signal already has a spectral-library match — those nodes serve as
  seed nodes for ChemWalker and do not need de novo in silico annotation
- When candidates cannot be enumerated from a structure database, since MetFrag requires
  explicit candidate structures to fragment
- When the goal is propagation of already-annotated identities across a network (use
  ChemWalker's random-walk step on the existing weighted graph instead)
edam_topics:
- http://edamontology.org/topic_3335
tools:
- name: MetFrag
  role: Performs the combinatorial in silico fragmentation of candidate structures
    that provides the annotation scores consumed downstream
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Scoring function that fuses MetFrag fragmentation results with the spectral
    network to weight the candidate graph
- name: ChemWalker
  role: Python package that consumes the MetFrag-annotated candidates and propagates
    spectral-library-match identities via random walk
  repo: https://github.com/computational-chemical-biology/ChemWalker
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:e5d7ed06c3a33018b30798b8724d67aef0139fafde53dabecc3f51d0daf4de88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - Network Annotation Propagation (NAP)
  - RDKit
  - pyrwr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# in-silico-fragmentation-candidate-annotation

## Summary

Annotate candidate structures for mass signals by combinatorial in silico fragmentation with MetFrag against biologically relevant databases, producing the fragment/candidate scores that ChemWalker converts into a MetFusion-weighted graph for random-walk annotation propagation.

## When to use

Use when candidate structures for spectral-library-unmatched mass signals must be annotated in silico before propagation, e.g. to build MetFrag 2.3 CL fragment results feeding a ChemWalker/MetFusion weighted graph.

## When NOT to use

- When the query signal already has a spectral-library match — those nodes serve as seed nodes for ChemWalker and do not need de novo in silico annotation
- When candidates cannot be enumerated from a structure database, since MetFrag requires explicit candidate structures to fragment
- When the goal is propagation of already-annotated identities across a network (use ChemWalker's random-walk step on the existing weighted graph instead)

## Inputs

- Query mass signals / peak lists (e.g. MS/MS spectra from a spectral-network or mzML-derived table)
- Candidate structure lists per query (SMILES/SDF from biologically relevant structure databases)
- MetFrag2.3-CL.jar command-line jar (tested version for ChemWalker)
- Java runtime for the MetFrag CL jar

## Outputs

- MetFrag candidate rankings with fragment-to-peak assignments per query signal
- Per-candidate in silico fragmentation scores used as MetFusion/ChemWalker graph weights
- Annotated candidate sets that feed the weighted graph for random-walk propagation

## How to apply

Obtain the exact command-line jar the method was validated with, MetFrag2.3-CL.jar, or an older release from the ipb-halle/MetFragRelaunched releases page, and run it on the candidate structure lists (e.g. from biologically relevant databases) for each query mass signal. MetFrag performs combinatorial in silico fragmentation, scoring each candidate structure by how well its predicted fragments explain the query spectrum's peaks. The per-candidate MetFrag scores are then combined with the spectral network and pairwise structural similarity among candidates via the MetFusion scoring function, producing a weighted graph whose nodes are candidate structures. This is the mandatory in silico annotation layer upstream of ChemWalker's random-walk propagation from spectral-library-match seed nodes, so annotation quality directly conditions the downstream ranking.

## Related tools

- **MetFrag** (Performs the combinatorial in silico fragmentation of candidate structures that provides the annotation scores consumed downstream) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Scoring function that fuses MetFrag fragmentation results with the spectral network to weight the candidate graph)
- **ChemWalker** (Python package that consumes the MetFrag-annotated candidates and propagates spectral-library-match identities via random walk) — https://github.com/computational-chemical-biology/ChemWalker

## Examples

```
wget http://ccbl.fcfrp.usp.br/ccbl/MetFrag2.3-CL.jar && conda env create -f environment.yml && conda activate chemwalker && pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- The jar used is the tested MetFrag2.3-CL.jar (or a documented release from MetFragRelaunched), matching the version ChemWalker was validated on
- Every query signal in the benchmark has a non-empty MetFrag candidate list with fragment-peak assignments
- Resulting candidate scores combine with spectral-network edges through the MetFusion weighting without unmatched or null weights
- Downstream ChemWalker run from seed nodes (spectral library matches) yields per-candidate probabilities that reproduce the NIST 555-spectra benchmark behavior (top-10 classification comparable to NAP)

## Limitations

- ChemWalker was only tested on MetFrag2.3-CL.jar; other MetFrag versions/fragments may not reproduce the published behavior
- Annotation of mass signals remains the biggest bottleneck for untargeted mass spectrometry of complex mixtures, so in silico candidates alone cannot be treated as confident identifications
- NAP-style propagation only spreads information to the first neighbor of a spectral match; the fragmentation layer alone does not reach nodes not directly connected to matches — the random-walk step is required for that

## Evidence

- [abstract] MetFrag dependency in the ChemWalker pipeline: "ChemWalker relies on combinatorial in silico fragmentation results, performed by MetFrag, searching biologically relevant databases"
- [methods] MetFrag role as annotation engine: "ChemWalker uses MetFrag for in silico annotation"
- [readme] Tested MetFrag CL jar version: "ChemWalker was tested on [MetFrag2.3.-CL.jar](http://ccbl.fcfrp.usp.br/ccbl/MetFrag2.3-CL.jar), download MetFrag CL"
- [abstract] Weighting of the candidate graph via MetFusion: "we have used MetFusion Scoring function to create a weight function, producing a weighted graph"
