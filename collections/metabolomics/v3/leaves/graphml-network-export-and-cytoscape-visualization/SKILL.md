---
name: graphml-network-export-and-cytoscape-visualization
description: Use when a motif-to-network mapping step (e.g. pyMolNetEnhancer Mass2Motifs_2_Network)
  has produced an annotated network that must be exported as .graphml and rendered
  in Cytoscape with node pie charts and edge discrete mappings.
when_to_use_negative:
- Input is a raw GNPS job output or MS2LDA summary that has not yet passed through
  the Mass2Motifs_2_Network mapping — no motif attributes exist to visualize yet
- You need chemical class annotations rather than motif substructure annotations;
  the ClassyFire-based mapping step is a separate notebook
- The output target is not Cytoscape or a graphml consumer; the node pie-chart rendering
  depends on Cytoscape's Image/Chart feature
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: pyMolNetEnhancer
  role: Produces the motif-annotated network and writes the .graphml and TSV outputs
    via networkx
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: networkx
  role: Serializes the annotated network object to .graphml format
- name: Cytoscape
  role: Visualization platform where the .graphml is imported; node pie charts via
    Image/Chart on TopSharedMotifs and edge colors via Discrete Mapping on interaction
  repo: https://cytoscape.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:63a36b5ae0716725c042be1e7e6de482eba58761ac8f36eb0e8234ac3b15c708
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - MolNetEnhancer
  - networkx
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# graphml-network-export-and-cytoscape-visualization

## Summary

Exports the motif-annotated GNPS molecular network as a .graphml file (via networkx.write_graphml) plus Mass2Motifs_Edges and Mass2Motifs_Nodes TSVs, then loads these into Cytoscape to visualize Mass2Motif distributions on nodes as pie charts and shared-motif edges by discrete color mapping.

## When to use

Use when a motif-to-network mapping step (e.g. pyMolNetEnhancer Mass2Motifs_2_Network) has produced an annotated network that must be exported as .graphml and rendered in Cytoscape with node pie charts and edge discrete mappings.

## When NOT to use

- Input is a raw GNPS job output or MS2LDA summary that has not yet passed through the Mass2Motifs_2_Network mapping — no motif attributes exist to visualize yet
- You need chemical class annotations rather than motif substructure annotations; the ClassyFire-based mapping step is a separate notebook
- The output target is not Cytoscape or a graphml consumer; the node pie-chart rendering depends on Cytoscape's Image/Chart feature

## Inputs

- Annotated network object from pyMolNetEnhancer Mass2Motifs_2_Network mapping (classical or feature-based)
- MS2LDA motif-document summary filtered at prob = 0.01 and overlap = 0.3

## Outputs

- Motif_ChemicalClass_Network_Classical.graphml (annotated network with TopSharedMotifs and interaction attributes)
- Mass2Motifs_Edges_Classical.tsv (CLUSTERID1, interact, CLUSTERID2)
- Mass2Motifs_Nodes_Classical.tsv (per-node motif annotations)
- Cytoscape visualization with pie-chart nodes and discrete-mapping edge colors

## How to apply

After running the Mass2Motifs_2_Network mapping (with prob = 0.01, overlap = 0.3, top = 5) in the Mass2Motifs_2_Network_Classical notebook, export the annotated network as Motif_ChemicalClass_Network_Classical.graphml using networkx.write_graphml, alongside the Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv outputs. Import the .graphml into Cytoscape. For edges, select 'Stroke Color' in the 'Edge' tab, choose 'interaction' as Column and 'Discrete Mapping' as Mapping Type to color edges by shared Mass2Motifs between nodes. For nodes, select 'Image/Chart' in the 'Node' tab and choose Mass2Motifs shown in 'TopSharedMotifs' so each node displays a pie chart whose slice sizes reflect the relative abundance (overlap score) of each motif. Alternatively, load the edges TSV as a network (CLUSTERID1 as Source, interact as Interaction Type, CLUSTERID2 as Target) and the nodes TSV as a table into Cytoscape separately.

## Related tools

- **pyMolNetEnhancer** (Produces the motif-annotated network and writes the .graphml and TSV outputs via networkx) — https://github.com/madeleineernst/pyMolNetEnhancer
- **networkx** (Serializes the annotated network object to .graphml format)
- **Cytoscape** (Visualization platform where the .graphml is imported; node pie charts via Image/Chart on TopSharedMotifs and edge colors via Discrete Mapping on interaction) — https://cytoscape.org/

## Examples

```
networkx.write_graphml(annotated_network, 'Motif_ChemicalClass_Network_Classical.graphml')
```

## Evaluation signals

- Motif_ChemicalClass_Network_Classical.graphml loads into Cytoscape without import errors and preserves TopSharedMotifs and interaction attributes
- Node fill color can be mapped discretely by shared Mass2Motifs (Image/Chart on TopSharedMotifs) — pie slices proportional to motif overlap scores
- Edge stroke color maps via Discrete Mapping on the interaction column
- The alternative path (Mass2Motifs_Edges_Classical.tsv as network + Mass2Motifs_Nodes_Classical.tsv as table) yields an equivalent rendering in Cytoscape

## Limitations

- Cytoscape-specific rendering: the pie-chart node style relies on Cytoscape's Image/Chart feature, so outputs are not directly visualizable in other graph tools
- The Mass2Motifs_2_Network mapping may encounter a server connection timeout when fetching the MS2LDA summary; download the file manually from ms2lda.org instead
- The MS2LDA summary table already contains motif-document relations filtered by the thresholds set in the web app; setting thresholds only in the notebook will not unfilter them

## Evidence

- [methods] graphml export as main output: [evidence span withheld — anchor status: not_found]
- [readme] Cytoscape import and edge mapping instructions: "To visualize results import the .graphml output file into [Cytoscape](https://cytoscape.org/). To color edges based on shared Mass2Motifs in between nodes select 'Stroke Color' in the 'Edge' tab to"
- [readme] Cytoscape node pie-chart rendering via TopSharedMotifs: "To color nodes by the most shared Mass2Motifs per molecular family (network component index) select 'Image/Chart' in the 'Node' tab to the left and select Mass2Motifs shown in 'TopSharedMotifs' in"
- [full_text] Motifs rendered as pie charts on network nodes: [evidence span withheld — anchor status: not_found]
- [readme] MS2LDA server timeout and manual download workaround: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/"
