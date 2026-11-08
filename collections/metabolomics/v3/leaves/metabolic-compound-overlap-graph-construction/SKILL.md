---
name: metabolic-compound-overlap-graph-construction
description: Use when building the structural (graph-based) level of a KEGG organism's
  metabolic network from pathway KGML files, where pathways must be connected via
  shared non-ubiquitous compounds rather than reactions.
when_to_use_negative:
- The goal is reaction-level (functional-level) comparison — this skill only builds
  the structural compound-overlap graph, not the reactions-content representation.
- KEGG maplinks are the desired connection basis — the article explicitly treats maplinks
  as visual-only and unreliable, so maplink-based graphs are out of scope.
- Ubiquitous compounds should connect pathways — the method deliberately discards
  H2O, phosphate, ATP and ADP before edge construction.
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0091
tools:
- name: MetNet
  role: 'Implements the whole structural-level reconstruction: KGML retrieval, parsing,
    ubiquitous-compound filtering, graph construction and CSV serialisation'
  repo: github.com/simeoni-biolab/MetNet
- name: sax2r2.jar
  role: Java XML/KGML parsing library used by MetNet to extract compounds and reactions
    from each KGML file
- name: gs-core-1.3.jar
  role: GraphStream core library used to represent pathways as nodes and shared-compound
    relations as edges
- name: gs-algo-1.3.jar
  role: GraphStream algorithms library supporting the graph representation used for
    the structural level
- name: poi-3.9.jar
  role: Apache POI library used by MetNet for KGML/data parsing during reconstruction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:3ec126a9dc1459b69a7d7b54f369084f932452ce0ab503b9712fd30712513e51
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MetNet
  - gs-algo-1.3.jar
  - gs-core-1.3.jar
  - guava-19.0.jar
  - poi-3.9.jar
  - sax2r2.jar
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# metabolic-compound-overlap-graph-construction

## Summary

Reconstructs a structural-level metabolic network by treating KEGG reference pathways as nodes and drawing edges between pathways that share at least one non-ubiquitous compound, then serialises the result as an adjacency matrix for cross-organism comparison.

## When to use

Use when building the structural (graph-based) level of a KEGG organism's metabolic network from pathway KGML files, where pathways must be connected via shared non-ubiquitous compounds rather than reactions.

## When NOT to use

- The goal is reaction-level (functional-level) comparison — this skill only builds the structural compound-overlap graph, not the reactions-content representation.
- KEGG maplinks are the desired connection basis — the article explicitly treats maplinks as visual-only and unreliable, so maplink-based graphs are out of scope.
- Ubiquitous compounds should connect pathways — the method deliberately discards H2O, phosphate, ATP and ADP before edge construction.

## Inputs

- KEGG organism code for the organism under reconstruction (e.g. hsa)
- pathwayList.txt — list of KEGG reference pathways to consider
- KGML files for each pathway of the organism, downloaded via KEGG public APIs

## Outputs

- structural_graph.csv — symmetric adjacency matrix (pathways x pathways) with 0/1 entries marking shared non-ubiquitous compounds
- Graph object (GraphStream) with pathways as nodes and shared-compound relations as edges

## How to apply

Retrieve the KGML files for each pathway the organism possesses using KEGG's public HTTP APIs, then parse each KGML to extract its compound and reaction sets (MetNet does this with sax2r2 XML parsing and Apache POI). Remove ubiquitous compounds (H2O, phosphate, ATP, ADP) from every pathway's compound set, since they would trivially connect nearly all pathways. Pairwise-compare the filtered compound sets and create an undirected edge between two pathway nodes whenever they share at least one non-ubiquitous compound; ignore KEGG maplinks, which are visual-only and unreliable. Build the graph with GraphStream (gs-core/gs-algo) so pathways are nodes and shared-compound relations are edges, then serialise it as a CSV adjacency matrix whose rows/columns are the pathways from pathwayList.txt and whose cells record absence/presence (0/1) of a shared-compound edge.

## Related tools

- **MetNet** (Implements the whole structural-level reconstruction: KGML retrieval, parsing, ubiquitous-compound filtering, graph construction and CSV serialisation) — github.com/simeoni-biolab/MetNet
- **sax2r2.jar** (Java XML/KGML parsing library used by MetNet to extract compounds and reactions from each KGML file)
- **gs-core-1.3.jar** (GraphStream core library used to represent pathways as nodes and shared-compound relations as edges)
- **gs-algo-1.3.jar** (GraphStream algorithms library supporting the graph representation used for the structural level)
- **poi-3.9.jar** (Apache POI library used by MetNet for KGML/data parsing during reconstruction)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- The adjacency matrix contains exactly the pathways the organism actually possesses (from organismList.txt / KGML retrieval), with no missing rows or columns.
- Every off-diagonal 1 entry corresponds to at least one shared non-ubiquitous compound between the two pathways; spot-check a few edges by re-parsing the KGML.
- No edge was created solely because of H2O, phosphate, ATP or ADP — filter those compounds before comparison and verify they contributed no links.
- The matrix is symmetric (undirected graph) and diagonal entries store pathway status/connection count as per MetNet's convention.
- GraphStream graph node count equals the number of pathways in pathwayList.txt that the organism possesses.

## Limitations

- Reconstruction is based uniquely on the KEGG database, which limits the represented information.
- KEGG maplinks are ignored because they are visual-only and unreliable, so cross-pathway hints from map00780 etc. are not encoded.
- Discarding ubiquitous compounds means pathways whose only shared metabolites are H2O, phosphate, ATP or ADP will appear disconnected.
- MetNet does not currently accept genomic data input; it can only use what KEGG already annotates for the organism.

## Evidence

- [methods] structural graph construction from KGML and shared compounds: [evidence span withheld — anchor status: not_found]
- [methods] KGML parsing extracts compounds and reactions: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [abstract] two-level representation: structural level is graph-based with pathways as nodes and relations as edges: "the first level is graph-based and depicts pathways as nodes and relations between pathways as edges"
- [readme] pathwayList.txt is the configuration file listing pathways considered for the comparison: "pathwayList.txt : configuration file containing the list of patways to be considered for the comparison"
- [abstract] MetNet automatically reconstructs the metabolic network of two KEGG organisms and compares them: "MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually."
