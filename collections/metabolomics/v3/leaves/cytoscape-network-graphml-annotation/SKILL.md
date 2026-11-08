---
name: cytoscape-network-graphml-annotation
description: Use when a Mass2Motif-annotated molecular network (.graphml or edges/nodes
  TSV from pyMolNetEnhancer/RMolNetEnhancer) must be inspected in Cytoscape with edge
  colors for shared motifs and node colors for most-shared motifs per family.
when_to_use_negative:
- The network has not yet been annotated with Mass2Motifs via pyMolNetEnhancer/RMolNetEnhancer
  — Cytoscape styling alone will not add motif-document relations.
- Only chemical class annotation is needed without MS2LDA substructure mapping — use
  the class-only workflow instead.
- Motif–document relations need to be re-filtered at different prob/overlap thresholds
  — regenerate the summary/graphml first, since the ms2lda.org summary table already
  contains relations filtered at the web-app thresholds.
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: pyMolNetEnhancer
  role: Generates the .graphml and edges/nodes TSV files that are imported into Cytoscape
    for motif visualization
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: R equivalent of pyMolNetEnhancer; produces the same .graphml/TSV outputs for
    Cytoscape import
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Mass2Motif source whose substructure assignments populate the network attributes
    visualized in Cytoscape
- name: GNPS
  role: Platform that created the molecular network (classical or feature-based) being
    visualized
- name: Cytoscape
  role: Visualization target for the annotated network; edge/node styling applied
    here
  repo: https://cytoscape.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:afab725dac873a4719bc9bd0a965be0e17699f4869976efe8bab488f5872f636
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - GNPS
  - MS2LDA
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cytoscape-network-graphml-annotation

## Summary

Visual annotation of motif-mapped molecular networks by importing the pyMolNetEnhancer .graphml output (or the separate edges/nodes TSV files) into Cytoscape and styling edges by shared Mass2Motifs and nodes by TopSharedMotifs per molecular family.

## When to use

Use when a Mass2Motif-annotated molecular network (.graphml or edges/nodes TSV from pyMolNetEnhancer/RMolNetEnhancer) must be inspected in Cytoscape with edge colors for shared motifs and node colors for most-shared motifs per family.

## When NOT to use

- The network has not yet been annotated with Mass2Motifs via pyMolNetEnhancer/RMolNetEnhancer — Cytoscape styling alone will not add motif-document relations.
- Only chemical class annotation is needed without MS2LDA substructure mapping — use the class-only workflow instead.
- Motif–document relations need to be re-filtered at different prob/overlap thresholds — regenerate the summary/graphml first, since the ms2lda.org summary table already contains relations filtered at the web-app thresholds.

## Inputs

- .graphml network file with Mass2Motif presence per node/edge (pyMolNetEnhancer output)
- Mass2Motifs_Edges_Classical.tsv (source=CLUSTERID1, interaction=interact, target=CLUSTERID2)
- Mass2Motifs_Nodes_Classical.tsv node attribute table
- GNPS molecular network job output (MGF clustered spectra / classical or feature-based workflow)

## Outputs

- Cytoscape session with edges colored by discrete mapping on shared Mass2Motifs (interaction column)
- Cytoscape session with nodes colored by Image/Chart via the TopSharedMotifs column of the Edge Table
- Annotated network visualization revealing subtle substructural diversity within molecular families

## How to apply

After running the Mass2Motifs-2-Network workflow (prob=0.01, overlap=0.3, top=5), import the .graphml file into Cytoscape. To color edges based on shared Mass2Motifs between nodes, go to the Edge tab, select 'Stroke Color', choose 'interaction' as Column and 'Discrete Mapping' as Mapping Type. To color nodes by the most shared Mass2Motifs per molecular family (network component index), select 'Image/Chart' in the Node tab and pick Mass2Motifs listed in the 'TopSharedMotifs' column of the Edge Table. Alternatively, import 'Mass2Motifs_Edges_Classical.tsv' as a network (Source=CLUSTERID1, Interaction=interact, Target=CLUSTERID2) and load 'Mass2Motifs_Nodes_Classical.tsv' as a node table, which decouples network construction from table import.

## Related tools

- **pyMolNetEnhancer** (Generates the .graphml and edges/nodes TSV files that are imported into Cytoscape for motif visualization) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (R equivalent of pyMolNetEnhancer; produces the same .graphml/TSV outputs for Cytoscape import) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Mass2Motif source whose substructure assignments populate the network attributes visualized in Cytoscape)
- **GNPS** (Platform that created the molecular network (classical or feature-based) being visualized)
- **Cytoscape** (Visualization target for the annotated network; edge/node styling applied here) — https://cytoscape.org/

## Examples

```
pip install pyMolNetEnhancer; then run Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb to emit Mass2Motifs_Network.graphml for import into Cytoscape.
```

## Evaluation signals

- Edges in the imported network must show 'interaction' values from shared Mass2Motifs and discrete color mapping must resolve every distinct motif to a color.
- The 'TopSharedMotifs' Edge Table column must exist and contain at most 5 motifs per molecular family (network component index), matching the top=5 setting.
- Node Image/Chart rendering must reflect the motif lists in TopSharedMotifs for each network component.
- When loading TSVs separately, the source/interaction/target columns must map exactly to CLUSTERID1/interact/CLUSTERID2 or the network import will fail.
- Motif-document relations in the visualized graph must match the prob=0.01 / overlap=0.3 filtered relations from the ms2lda.org summary table.

## Limitations

- The ms2lda.org summary table already contains motif–document relations filtered by the thresholds set in the web app's Experimental Options tab; different thresholds there change what the graphml contains.
- Depending on the MS2LDA job file size, a server connection timeout may occur; the summary must then be downloaded manually from ms2lda.org.
- Coloring by Image/Chart requires selecting motifs from the TopSharedMotifs column in the Edge Table — custom motif sets per node are not directly supported through this workflow.
- Mass2Motif mapping was demonstrated for both classical and feature-based networks, but the .graphml must be regenerated per workflow type.

## Evidence

- [readme] Edge coloring via discrete mapping: "To color edges based on shared Mass2Motifs in between nodes select 'Stroke Color' in the 'Edge' tab to the left and choose 'interaction' as <i>Column</i> and 'Discrete Mapping' as <i>Mapping Type</i>:"
- [readme] Node coloring via TopSharedMotifs: "To color nodes by the most shared Mass2Motifs per molecular family (network component index) select 'Image/Chart' in the 'Node' tab to the left and select Mass2Motifs shown in 'TopSharedMotifs' in"
- [readme] Alternative TSV import path: "import the 'Mass2Motifs_Edges_Classical.tsv' output file as network into Cytoscape. Select column 'CLUSTERID1' as Source Node, column 'interact' as Interaction Type and 'CLUSTERID2' as Target Node"
- [methods] Threshold semantics: "<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] Summary table pre-filtering: "Importantly, the summary table contains filtered motif-document relations using the set thresholds in the web app."
