---
name: graph-layout-visualization
description: Use when a reconstructed structural-level metabolic network (pathways
  as nodes, edges between pathways sharing at least one non-ubiquitous compound) needs
  to be rendered as an image for qualitative topology comparison between organisms.
when_to_use_negative:
- The analysis target is the functional level (reaction content of each pathway) rather
  than the structural-level graph topology
- Input metabolic data is not from KEGG — MetNet's reconstruction and rendering are
  based uniquely on the KEGG database, which limits the represented information
- Only a quantitative similarity index is needed and no visual inspection of network
  shape is required
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetNet
  role: Reconstructs the structural-level metabolic network from KEGG data and drives
    GraphStream-based rendering of the graph for visual comparison
  repo: github.com/simeoni-biolab/MetNet
- name: gs-core-1.3.jar
  role: GraphStream core library used by MetNet to build and display the structural
    graph layout
- name: gs-ui-1.3.jar
  role: GraphStream UI library providing the visual display of the rendered metabolic
    network topology
- name: KEGG
  role: Source database whose KGML files and reference pathways feed the network reconstruction
    that is then rendered
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:08fa52745c035434f1ef28dcaecb20f910fcba7ec22e9878660fb43df92f014b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - MetNet
  - gs-core-1.3.jar
  - gs-ui-1.3.jar
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# graph-layout-visualization

## Summary

Renders metabolic network topology as a graph image by laying out nodes and edges with GraphStream (gs-core/gs-ui 1.3), the visualization engine bundled with MetNet. Used to visually compare structural-level metabolic networks of two KEGG organisms.

## When to use

Use when a reconstructed structural-level metabolic network (pathways as nodes, edges between pathways sharing at least one non-ubiquitous compound) needs to be rendered as an image for qualitative topology comparison between organisms.

## When NOT to use

- The analysis target is the functional level (reaction content of each pathway) rather than the structural-level graph topology
- Input metabolic data is not from KEGG — MetNet's reconstruction and rendering are based uniquely on the KEGG database, which limits the represented information
- Only a quantitative similarity index is needed and no visual inspection of network shape is required

## Inputs

- KEGG organism codes for two organisms (e.g. 'api', 'bap')
- KGML pathway files downloaded from KEGG for each organism
- Structural-level adjacency information (pathways as nodes, shared non-ubiquitous compounds as edges)
- Configuration files organismList.txt and pathwayList.txt
- GraphStream jars: gs-core-1.3.jar, gs-ui-1.3.jar, gs-algo-1.3.jar

## Outputs

- One rendered structural-level network image per organism (graph layout of pathways and inter-pathway relations)
- Qualitative topology comparison of the two rendered networks

## How to apply

First reconstruct the structural-level graph with MetNet from the organism's KEGG-derived data: KGML files are retrieved via KEGG's public APIs and parsed for compounds and reactions, then each pathway becomes a node and edges connect pathways sharing at least one non-ubiquitous compound (H2O, phosphate, ATP and ADP excluded). Render the resulting adjacency structure using the GraphStream libraries (gs-core-1.3.jar, gs-ui-1.3.jar, plus gs-algo-1.3.jar, guava-19.0.jar, poi-3.9.jar, sax2r2.jar on the classpath) that MetNet ships in its lib folder, exporting one image per organism. Run MetNet with two KEGG organism codes and the comparison method ('set' or 'multiset'); MetNet's visual interface guides the rendering steps, ensuring organismList.txt and pathwayList.txt are present. The rationale is that MetNet's first level is graph-based — pathways as nodes and relations between pathways as edges — so a layout of this graph exposes network topology differences (e.g. a 464 Mb-genome insect's complex network vs a 640 Kb-genome symbiont's much simpler one) that similarity indexes alone do not show visually.

## Related tools

- **MetNet** (Reconstructs the structural-level metabolic network from KEGG data and drives GraphStream-based rendering of the graph for visual comparison) — github.com/simeoni-biolab/MetNet
- **gs-core-1.3.jar** (GraphStream core library used by MetNet to build and display the structural graph layout)
- **gs-ui-1.3.jar** (GraphStream UI library providing the visual display of the rendered metabolic network topology)
- **KEGG** (Source database whose KGML files and reference pathways feed the network reconstruction that is then rendered)

## Examples

```
java -jar MetNet.jar api bap set
```

## Evaluation signals

- The rendered image shows one node per KEGG reference pathway listed in pathwayList.txt, with edges only between pathways sharing a non-ubiquitous compound (H2O, phosphate, ATP, ADP excluded)
- Exactly one structural-graph image is produced per organism, and the two images can be inspected side by side for topology differences
- For organisms with very different genome sizes (e.g. api, 464 Mb, vs bap, 640 Kb), the larger-genome organism's rendered network should appear visibly more complex than the smaller-genome symbiont's simpler network
- The run completes without missing configuration files — organismList.txt and pathwayList.txt must be in the tool's directory
- The same adjacency matrices fed to an equivalent layout engine should reproduce a structurally consistent rendering of the MetNet graph

## Limitations

- Reconstruction (and hence the rendered graph) is based uniquely on the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data; significant extension is required to reconstruct networks from newly annotated genomes
- The comparison is qualitative at this level: topology differences are recorded by visual inspection of the rendered images rather than by a numeric layout metric

## Evidence

- [abstract] the first level is graph-based and depicts pathways as nodes and relations between pathways as edges: "Our proposal relies on a two-level representation of the huge metabolic network: the first level is graph-based and depicts pathways as nodes and relations between pathways as edges"
- [methods] GraphStream jars on compile classpath: "javac -cp ./lib/gs-algo-1.3.jar:./lib/gs-core-1.3.jar:./lib/gs-ui-1.3.jar:./lib/guava-19.0.jar:./lib/poi-3.9.jar:./lib/sax2r2.jar"
- [readme] MetNet CLI parameters for organism comparison: [evidence span withheld — anchor status: not_found]
- [methods] executable availability: "executable version of MetNet is available at www.dsi.unive.it/~biolab"
- [readme] visual interface guidance: "MetNet will start the execution and the user will be guided through the various steps of the comparison by the visual interface."
