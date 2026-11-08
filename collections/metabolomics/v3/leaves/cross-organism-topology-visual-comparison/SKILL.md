---
name: cross-organism-topology-visual-comparison
description: Use when two (or a set of) KEGG organisms' metabolisms must be compared
  at the structural/topological level and a qualitative visual judgement of network
  shape difference (e.g. complex host vs. simple symbiont) is the deliverable.
when_to_use_negative:
- The analysis goal is a quantitative, pathway-by-pathway or global similarity score
  rather than a qualitative visual topology comparison (use MetNet's similarity measures
  instead).
- 'The input is raw genomic data rather than KEGG organisms: MetNet''s reconstruction
  is based uniquely on the KEGG database and does not currently support input of genomic
  data.'
- Only a single organism's network needs to be reconstructed or visualized with no
  cross-organism comparison.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3314
tools:
- name: MetNet
  role: Reconstructs the structural-level metabolic networks of the two KEGG organisms
    and renders their graph topologies for visual comparison
  repo: https://github.com/simeoni-biolab/MetNet
- name: gs-core-1.3.jar
  role: GraphStream library used by MetNet to build and render the structural graphs
- name: gs-ui-1.3.jar
  role: GraphStream UI library providing the visual layout engine for exporting one
    image per organism
- name: KEGG
  role: Source database from which each organism's pathway KGML data is retrieved
    to build the structural graphs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:f0e57569100c1d9aa60b6df77e73a30f62939aa834670219d44546c7db181038
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - KEGG
  - MetNet
  - gs-core-1.3.jar
  - gs-ui-1.3.jar
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-organism-topology-visual-comparison

## Summary

Renders and visually compares the structural-level (graph-based) metabolic network topologies of two KEGG organisms reconstructed by MetNet, where pathways are nodes and edges connect pathways sharing at least one non-ubiquitous compound. This qualitative visual inspection complements MetNet's quantitative similarity indexes.

## When to use

Use when two (or a set of) KEGG organisms' metabolisms must be compared at the structural/topological level and a qualitative visual judgement of network shape difference (e.g. complex host vs. simple symbiont) is the deliverable.

## When NOT to use

- The analysis goal is a quantitative, pathway-by-pathway or global similarity score rather than a qualitative visual topology comparison (use MetNet's similarity measures instead).
- The input is raw genomic data rather than KEGG organisms: MetNet's reconstruction is based uniquely on the KEGG database and does not currently support input of genomic data.
- Only a single organism's network needs to be reconstructed or visualized with no cross-organism comparison.

## Inputs

- KEGG organism codes for the two organisms to compare (e.g. 'api', 'bap')
- KEGG-derived KGML files of each organism's pathways (downloaded via KEGG's public APIs)
- MetNet configuration files organismList.txt and pathwayList.txt listing available KEGG organisms and the pathways to consider

## Outputs

- One rendered structural-level metabolic network image per organism (GraphStream layout)
- Qualitative record of topology difference/similarity between the two organisms' structural graphs

## How to apply

Select the pair of organisms by their KEGG codes (e.g. 'api' and 'bap') and run MetNet to automatically reconstruct each structural-level graph: pathways are nodes, and edges connect pathways sharing at least one non-ubiquitous compound (H2O, phosphate, ATP, ADP are excluded as ubiquitous). MetNet renders each structural graph using the GraphStream libraries (gs-core-1.3.jar, gs-ui-1.3.jar, gs-algo-1.3.jar) bundled in lib/, exporting one rendered topology image per organism. Place the two rendered topologies side by side and inspect them qualitatively, recording whether the network shapes differ — e.g. a large-genome insect (464 Mb) shows a complex network while a 640 Kb-genome endosymbiont shows a much simpler one. Interpret divergence in topology as consistent with complementary rather than similar host-symbiont metabolisms, and pair the visual finding with MetNet's structural similarity index for a quantitative counterpart.

## Related tools

- **MetNet** (Reconstructs the structural-level metabolic networks of the two KEGG organisms and renders their graph topologies for visual comparison) — https://github.com/simeoni-biolab/MetNet
- **gs-core-1.3.jar** (GraphStream library used by MetNet to build and render the structural graphs)
- **gs-ui-1.3.jar** (GraphStream UI library providing the visual layout engine for exporting one image per organism)
- **KEGG** (Source database from which each organism's pathway KGML data is retrieved to build the structural graphs)

## Examples

```
java -jar MetNet.jar api bap set
```

## Evaluation signals

- Each rendered structural graph must have pathways as nodes and edges only between pathways sharing at least one non-ubiquitous compound, with H2O, phosphate, ATP and ADP excluded as connectors.
- One distinct topology image must be exported per organism, generated from the same adjacency information used for the quantitative comparison.
- The visual inspection must record a qualitative shape judgement (e.g. 'complex' vs 'much simpler' network) that can be checked against genome-size expectations and against MetNet's structural similarity index.
- The correct KEGG organism codes must appear in the organismList.txt-derived selection and in the command-line invocation (e.g. 'api', 'bap').
- Re-running MetNet on the same organism pair with the same comparison method ('set' or 'multiset') should reproduce consistent structural adjacency and a visually equivalent rendering.

## Limitations

- The comparison is purely visual/qualitative at this level; it should be complemented with MetNet's structural similarity indexes for quantitative support.
- Reconstruction is limited to the KEGG database, which can be a limit with respect to the represented information.
- MetNet does not currently support input of genomic data; a significant extension would be required to reconstruct networks from newly annotated genomes.
- Data repositories such as KEGG are incomplete, heterogeneous and incoherent, so reconstruction generally reflects those limitations.

## Evidence

- [abstract] the first level is graph-based and depicts pathways as nodes and relations between pathways as edges: "Our proposal relies on a two-level representation of the huge metabolic network: the first level is graph-based and depicts pathways as nodes and relations between pathways as edges"
- [readme] MetNet visual/quantitative comparison scope: "MetNet is a Java tool that makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually."
- [intro] structural graph adjacency definition: "representing a metabolic network in two distinct levels: the structural level and the functional level."
- [methods] rendering library dependency: "javac -cp ./lib/gs-algo-1.3.jar:./lib/gs-core-1.3.jar:./lib/gs-ui-1.3.jar:./lib/guava-19.0.jar:./lib/poi-3.9.jar:./lib/sax2r2.jar"
- [discussion] MetNet limitation to KEGG: "is based uniquely on the KEGG database, which can be a limit wrt. the represented information"
