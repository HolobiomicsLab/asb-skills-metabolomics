---
name: metfrag-annotation-score-averaging
description: Use when you have MetFrag candidate scores for each node in a spectral
  molecular network plus spectral cosine similarity between connected nodes, and you
  need to build a weighted graph for ChemWalker random-walk propagation.
when_to_use_negative:
- You only have a single node's MetFrag candidates and no molecular network edges
  — MetFusion edge weighting requires pairs of connected nodes
- Spectral library has already confidently annotated the nodes and no in silico candidate
  expansion is needed — direct annotation needs no edge weighting
- Your graph edges are negative or zero-weighted — pyrwr's RWR only accepts positively
  weighted networks, so MetFusion-style weights must be rescaled first
edam_operation: http://edamontology.org/operation_3763
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MetFrag
  role: Provides the per-candidate in silico combinatorial fragmentation annotation
    scores that are averaged into each edge weight
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Defines the Fusion scoring concept and the alpha-weighted combination rule
    used to fuse MetFrag scores with spectral and structural similarity into edge
    weights
- name: ChemWalker
  role: Python package that implements the MetFusion-style weight function and writes
    the resulting weighted graphml for random-walk propagation
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Consumes the weighted graph produced by this skill to run random walk with
    restart over the edge weights
  repo: https://github.com/jinhongjung/pyrwr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:09cc974fd1ced3edaab4bb7d2f5a06531415f9727a0512de51a970ef117b1ddf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - pyrwr
  edam_operation: http://edamontology.org/operation_3763
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metfrag-annotation-score-averaging

## Summary

Aggregate per-candidate MetFrag in silico fragmentation scores across connected molecular-network nodes into edge weights using the MetFusion-style alpha-weighted fusion of averaged MetFrag scores, spectral cosine similarity, and structural (chemical) similarity, producing a weighted graph for downstream random-walk annotation propagation.

## When to use

Use when you have MetFrag candidate scores for each node in a spectral molecular network plus spectral cosine similarity between connected nodes, and you need to build a weighted graph for ChemWalker random-walk propagation.

## When NOT to use

- You only have a single node's MetFrag candidates and no molecular network edges — MetFusion edge weighting requires pairs of connected nodes
- Spectral library has already confidently annotated the nodes and no in silico candidate expansion is needed — direct annotation needs no edge weighting
- Your graph edges are negative or zero-weighted — pyrwr's RWR only accepts positively weighted networks, so MetFusion-style weights must be rescaled first

## Inputs

- MetFrag in silico fragmentation results (per-candidate annotation scores per molecular-network node)
- Spectral cosine similarity scores between connected molecular-network nodes
- Chemical/structural similarity among candidate structures (e.g. computed from candidate SMILES/InChI)
- Molecular network graph (nodes = mass signals, edges = spectral cosine links)
- alpha fusion weight parameter

## Outputs

- Weighted graph in graphml format with MetFusion-style edge weights w_ij
- Edge weight values (finite, non-negative) ready for random-walk propagation

## How to apply

Parse the MetFrag CL output (produced with MetFrag2.3-CL.jar) to obtain per-candidate annotation scores for each node's candidate structures. For each edge between connected nodes in the spectral network, compute the alpha-weighted average of the two nodes' MetFrag scores, then combine it (weight 1-alpha) with the sigmoid of the spectral cosine score multiplied by the chemical similarity of the corresponding candidate structures, following the MetFusion Fusion scoring concept. Tune the alpha parameter to control the relative contribution of MetFrag scores versus spectral/structural similarity; a low alpha biases weights toward spectral similarity and structural similarity, a high alpha toward MetFrag annotation quality. Assemble the nodes and computed edge weights w_ij into a weighted graph in graphml format. The resulting graph must have finite, non-negative edge weights on every edge, since it is consumed by pyrwr's random walk with restart, which is defined only on positively weighted networks.

## Related tools

- **MetFrag** (Provides the per-candidate in silico combinatorial fragmentation annotation scores that are averaged into each edge weight) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Defines the Fusion scoring concept and the alpha-weighted combination rule used to fuse MetFrag scores with spectral and structural similarity into edge weights)
- **ChemWalker** (Python package that implements the MetFusion-style weight function and writes the resulting weighted graphml for random-walk propagation) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Consumes the weighted graph produced by this skill to run random walk with restart over the edge weights) — https://github.com/jinhongjung/pyrwr

## Examples

```
conda env create -f environment.yml; conda activate chemwalker; pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- The emitted graphml file parses as a valid, readable weighted graph with no NaN or negative edge weights on any edge
- Edge weights w_ij fall in a finite positive range consistent with sigmoid output combined with averaged MetFrag scores (no zero or infinite values)
- Changing alpha monotonically shifts relative contributions: increasing alpha should raise the weight contribution of MetFrag annotation scores versus spectral cosine similarity
- Edges between nodes with high spectral cosine similarity and highly structurally similar candidate structures should receive higher weights than edges between dissimilar nodes at fixed alpha
- The weighted graph supports the downstream ChemWalker random walk (pyrwr accepts it and returns finite RWR scores from spectral-match seed nodes)

## Limitations

- The MetFusion-style weight function is evaluated only as part of the end-to-end random-walk propagation; the paper does not provide a standalone quantitative criterion for edge-weight quality
- MetFrag scores depend on the candidate databases searched; poor candidate sets propagate low-quality weights into the graph
- The method was tested with MetFrag2.3-CL.jar; other MetFrag versions may require adjustments
- pyrwr requires positively weighted networks, so weights must remain strictly positive

## Evidence

- [methods] Edge weights w_ij follow a MetFusion-style formulation: an alpha-weighted average of the two nodes' MetFrag scores is combined (weight 1-alpha) with the sigmoid of the spectral cosine score multiplied by the chemical similarity of their candidate structures.: [evidence span withheld — anchor status: not_found]
- [abstract] MetFusion scoring creates the weight function from the combination of spectral network and structural similarity among candidate structures.: "Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function, producing a"
- [abstract] ChemWalker relies on MetFrag combinatorial in silico fragmentation against biologically relevant databases.: "ChemWalker relies on combinatorial in silico fragmentation results, performed by MetFrag, searching biologically relevant databases"
- [readme] ChemWalker was tested on MetFrag2.3.-CL.jar (README).: "ChemWalker was tested on MetFrag2.3.-CL.jar"
- [readme] RWR is defined on positively weighted networks; only positive weights are allowed (pyrwr README).: [evidence span withheld — anchor status: external_doc]
