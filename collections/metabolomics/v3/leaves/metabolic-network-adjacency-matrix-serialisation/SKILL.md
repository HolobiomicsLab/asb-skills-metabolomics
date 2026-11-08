---
name: metabolic-network-adjacency-matrix-serialisation
description: Use when reconstructing MetNet's structural level from KEGG KGML pathway
  data and you need a pathway-by-pathway adjacency matrix capturing shared-compound
  connections for organism comparison.
when_to_use_negative:
- The goal is the functional level of the MetNet representation (each pathway's reactions
  content), not the structural shared-compound graph — use the reactions-based comparison
  instead.
- 'The intended edges derive from KEGG maplink elements: MetNet ignores maplinks because
  they are visual-only and unreliable, so maplink-based topology is out of scope for
  this serialisation.'
- Input is already a completed adjacency matrix or the organism's metabolic data has
  not yet been retrieved from KEGG (the matrix is built only after KGML download and
  parsing).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0091
tools:
- name: MetNet
  role: Java tool that retrieves KEGG KGML data, builds the structural-level graph,
    and serialises it as an adjacency matrix for comparison
  repo: github.com/simeoni-biolab/MetNet
- name: gs-core-1.3.jar
  role: GraphStream library used to model pathways as nodes and shared-compound relations
    as edges of the structural graph
- name: gs-algo-1.3.jar
  role: GraphStream algorithm library used alongside gs-core to construct and manipulate
    the pathway graph
- name: guava-19.0.jar
  role: Google Guava dependency linked into MetNet's classpath for data structures
    used during graph construction
- name: sax2r2.jar
  role: SAX-based KGML/XML parser library MetNet uses to extract compounds and reactions
    from each downloaded KGML file
- name: poi-3.9.jar
  role: Apache POI dependency linked into MetNet's classpath for reading/writing tabular
    outputs such as the adjacency matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:1bf6a185462dad78a0583caa3b071fcedb825ad9f47970005e1e37d4ef08becf
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# metabolic-network-adjacency-matrix-serialisation

## Summary

Serialises MetNet's structural-level metabolic graph — pathways as nodes, edges denoting shared non-ubiquitous compounds between KEGG reference pathways — as a symmetric adjacency matrix for quantitative and visual comparison of two organisms. Ubiquitous compounds (H2O, phosphate, ATP, ADP) must be excluded before edges are created so the matrix reflects meaningful metabolic connectivity.

## When to use

Use when reconstructing MetNet's structural level from KEGG KGML pathway data and you need a pathway-by-pathway adjacency matrix capturing shared-compound connections for organism comparison.

## When NOT to use

- The goal is the functional level of the MetNet representation (each pathway's reactions content), not the structural shared-compound graph — use the reactions-based comparison instead.
- The intended edges derive from KEGG maplink elements: MetNet ignores maplinks because they are visual-only and unreliable, so maplink-based topology is out of scope for this serialisation.
- Input is already a completed adjacency matrix or the organism's metabolic data has not yet been retrieved from KEGG (the matrix is built only after KGML download and parsing).

## Inputs

- KEGG organism codes for two organisms (e.g. hsa, ptr) supplied as command-line parameters
- pathwayList.txt configuration file listing the KEGG reference pathways to consider
- organismList.txt configuration file listing KEGG organisms
- KGML files of each pathway of the organism downloaded via the public KEGG APIs
- comparison method parameter: 'set' or 'multiset'

## Outputs

- Symmetric adjacency matrix over all considered KEGG reference pathways whose cells record the number of compounds shared between two pathways (structural level)
- GraphStream graph object with pathways as nodes and shared-compound relations as edges
- Two-level metabolic network representation for comparison with similarity indexes at structural and functional levels

## How to apply

Download the KGML files of each pathway listed in pathwayList.txt for the selected KEGG organism codes via the public KEGG APIs, and parse each file (MetNet uses sax2r2 and Apache POI) to extract the compounds and reactions of each pathway. Discard the ubiquitous compounds H2O, phosphate, ATP and ADP, then pairwise-compare the compound sets of pathways: create an edge between two pathway nodes whenever they share at least one non-ubiquitous compound, ignoring KEGG maplinks because they are visual-only and unreliable. Model the graph with the GraphStream Java library (gs-core/gs-algo, guava for data structures) with pathways as nodes and shared-compound relations as edges. Serialise the graph as a symmetric adjacency matrix in which rows and columns represent all the KEGG reference pathways considered for the comparison (those listed in the configuration file), off-diagonal cells encode absence/presence of a shared-compound edge, and diagonal entries store pathway status or connection count. The organism pair and comparison method ('set' or 'multiset') are supplied on the command line, and configuration files organismList.txt and pathwayList.txt must sit in the tool's directory.

## Related tools

- **MetNet** (Java tool that retrieves KEGG KGML data, builds the structural-level graph, and serialises it as an adjacency matrix for comparison) — github.com/simeoni-biolab/MetNet
- **gs-core-1.3.jar** (GraphStream library used to model pathways as nodes and shared-compound relations as edges of the structural graph)
- **gs-algo-1.3.jar** (GraphStream algorithm library used alongside gs-core to construct and manipulate the pathway graph)
- **guava-19.0.jar** (Google Guava dependency linked into MetNet's classpath for data structures used during graph construction)
- **sax2r2.jar** (SAX-based KGML/XML parser library MetNet uses to extract compounds and reactions from each downloaded KGML file)
- **poi-3.9.jar** (Apache POI dependency linked into MetNet's classpath for reading/writing tabular outputs such as the adjacency matrix)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- The matrix is symmetric: the entry for pathway i,j equals the entry for j,i for all pairs.
- Rows and columns cover exactly the pathways listed in pathwayList.txt that the organism actually possesses; no missing or extra pathway nodes.
- Every non-zero off-diagonal cell corresponds to at least one compound shared between the two pathways after removal of H2O, phosphate, ATP and ADP.
- Removing ubiquitous compounds (H2O, phosphate, ATP, ADP) changes the edge set — edges attributed only to those compounds must not appear.
- Edges derived from KEGG maplink elements are absent; only shared-compound edges are encoded.

## Limitations

- The reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information.
- Ubiquitous compounds (H2O, phosphate, ATP, ADP) must be discarded or they would create spurious edges between essentially all pathways.
- KEGG maplinks are visual-only and unreliable and are therefore ignored when building the structural graph.
- The structural matrix only encodes pathway-pair connectivity via shared compounds; it does not represent reaction-level functional detail.
- The executable must be run with organismList.txt and pathwayList.txt present in the tool's directory.

## Evidence

- [methods] Structural adjacency built from shared non-ubiquitous compounds: [evidence span withheld — anchor status: not_found]
- [methods] KGML retrieval and parsing as the pipeline's first stage: [evidence span withheld — anchor status: not_found]
- [results] Adjacency matrix dimensions defined by the configured pathway list: "whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file)"
- [readme] Tool scope and workflow selection of two organisms: "MetNet is a Java tool that makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually."
- [readme] Required configuration files that must accompany execution: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."
- [intro] Maplinks excluded from structural graph construction: [evidence span withheld — anchor status: not_found]
- [abstract] Two-level representation and similarity indexes: [evidence span withheld — anchor status: not_found]
- [readme] Classpath of library jars used to build/compile MetNet: "javac -cp ./lib/gs-algo-1.3.jar:./lib/gs-core-1.3.jar:./lib/gs-ui-1.3.jar:./lib/guava-19.0.jar:./lib/poi-3.9.jar:./lib/sax2r2.jar"
