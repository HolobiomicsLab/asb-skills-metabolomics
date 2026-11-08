---
name: kegg-pathway-adjacency-construction
description: Use when a KEGG organism code is available and the goal is a graph-based
  (topological) representation of its metabolism — pathways as nodes, shared-compound
  relations as edges — for later comparison or visualization.
when_to_use_negative:
- The input is raw genomic data rather than a KEGG organism code — MetNet reconstructs
  uniquely from the KEGG database and does not yet accept genomic data as input.
- The analysis target is the functional level (pathway reactions content and reaction-based
  similarity) rather than the structural/topological level.
- The organism is not present in the KEGG organism list (organismList.txt), since
  reconstruction depends entirely on KEGG reference pathway decomposition.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetNet
  role: Downloads KGML via KEGG APIs, parses compounds/reactions per pathway, and
    builds the structural-level adjacency with pathways as nodes and shared-non-ubiquitous-compound
    edges
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG
  role: Source database supplying the KGML files, organism list, and standardized
    reference pathways used for reconstruction
- name: gs-core-1.3.jar
  role: GraphStream library on MetNet's classpath used to represent and (with gs-ui)
    render the constructed structural graph
  repo: github.com/simeoni-biolab/MetNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:2f971f52942146a8cc629310efa3f1f6b0a03005582317a7c43d3a1336437c58
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - MetNet
  - gs-core-1.3.jar
  - gs-ui-1.3.jar
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# kegg-pathway-adjacency-construction

## Summary

Builds the structural-level metabolic network adjacency for a KEGG organism: pathways become nodes and edges connect pathways sharing at least one non-ubiquitous compound, reconstructed automatically from KEGG KGML data.

## When to use

Use when a KEGG organism code is available and the goal is a graph-based (topological) representation of its metabolism — pathways as nodes, shared-compound relations as edges — for later comparison or visualization.

## When NOT to use

- The input is raw genomic data rather than a KEGG organism code — MetNet reconstructs uniquely from the KEGG database and does not yet accept genomic data as input.
- The analysis target is the functional level (pathway reactions content and reaction-based similarity) rather than the structural/topological level.
- The organism is not present in the KEGG organism list (organismList.txt), since reconstruction depends entirely on KEGG reference pathway decomposition.

## Inputs

- KEGG organism code (e.g. 'api', 'bap', 'hsa')
- organismList.txt configuration file listing available KEGG organisms
- pathwayList.txt configuration file listing KEGG reference pathways to consider
- KGML pathway files downloaded via public KEGG APIs

## Outputs

- Structural-level metabolic network adjacency (pathways as nodes, edges between pathways sharing at least one non-ubiquitous compound)
- Renderable graph for layout/visualization (via GraphStream gs-core/gs-ui 1.3)

## How to apply

First run automatic data retrieval: download the KGML files of each pathway of the organism through the public KEGG's APIs, then parse each KGML file to extract the relevant information — the compounds and the reactions of each pathway. Assemble the structural-level graph where the first level is graph-based: pathways are nodes and relations between pathways are edges, defined by pathways sharing compounds; ubiquitous compounds such as H2O, phosphate, ATP and ADP are excluded so that edges reflect meaningful metabolite exchange rather than universal currency metabolites. Ensure the configuration files organismList.txt (KEGG organism codes) and pathwayList.txt (pathways to consider) are in the tool's directory, since they determine which organisms and which KEGG reference pathways enter the reconstruction. Run MetNet, which guides reconstruction through its visual interface or accepts the two organism KEGG codes and a comparison method ('set' or 'multiset') on the command line; the resulting adjacency defines the network topology that can then be rendered with the bundled GraphStream libraries (gs-core-1.3.jar, gs-ui-1.3.jar). The reconstruction is fully automatic and based uniquely on the KEGG database, which standardizes metabolism into reference pathways.

## Related tools

- **MetNet** (Downloads KGML via KEGG APIs, parses compounds/reactions per pathway, and builds the structural-level adjacency with pathways as nodes and shared-non-ubiquitous-compound edges) — github.com/simeoni-biolab/MetNet
- **KEGG** (Source database supplying the KGML files, organism list, and standardized reference pathways used for reconstruction)
- **gs-core-1.3.jar** (GraphStream library on MetNet's classpath used to represent and (with gs-ui) render the constructed structural graph) — github.com/simeoni-biolab/MetNet

## Examples

```
java -jar MetNet.jar api bap set
```

## Evaluation signals

- The structural graph's node set corresponds to the pathways listed in pathwayList.txt for the selected organism.
- Edges exist only between pathway pairs sharing at least one non-ubiquitous compound; pairs connected solely via H2O, phosphate, ATP, or ADP must not be linked.
- An organism with a small genome (e.g. a ~640 Kb bacterium such as bap) should yield a visibly simpler structural topology than a large-genome organism (e.g. a ~464 Mb insect such as api).
- Compilation succeeds with the specified classpath (gs-algo/gs-core/gs-ui 1.3, guava-19.0, poi-3.9, sax2r2) and execution requires organismList.txt and pathwayList.txt present in the tool directory.
- Running with two different organism codes produces two distinct adjacency structures consistent with each organism's KEGG pathway content.

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information.
- MetNet does not currently support input of genomic data; significant extension is required to reconstruct from newly annotated genomes.
- Data repositories are incomplete, heterogeneous and incoherent, which historically makes metabolism reconstruction difficult without human intervention — this approach limits itself to KEGG's standardized reference pathways.

## Evidence

- [abstract] structural level: "the first level is graph-based and depicts pathways as nodes and relations between pathways as edges"
- [methods] data retrieval: [evidence span withheld — anchor status: not_found]
- [methods] KGML parsing: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [readme] config files: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."
- [discussion] KEGG-only basis: "is based uniquely on the KEGG database, which can be a limit wrt. the represented information"
