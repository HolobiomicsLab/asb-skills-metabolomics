---
name: cytoscape-network-visualization-styling
description: Use when a Motif_ChemicalClass_Network .graphml (or Mass2Motifs_Edges/Nodes
  .tsv pair) from pyMolNetEnhancer/RMolNetEnhancer must be visualized in Cytoscape
  with edges colored by shared Mass2Motifs and nodes colored by CF_subclass chemical
  classes.
when_to_use_negative:
- The molecular network has not yet been through Mass2Motif mapping or ClassyFire
  chemical class annotation, so the .graphml lacks 'TopSharedMotifs', 'interaction',
  and CF_-prefixed columns to map onto.
- The input is a feature table (e.g. MZmine feature quantification table) rather than
  a MolNetEnhancer graphml/tsv network output.
- Interactive per-family motif exploration in the ms2lda.org web app is the goal,
  rather than Cytoscape rendering of the enhanced network.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0602
tools:
- name: Cytoscape
  role: 'Visualization platform: import .graphml or tsv outputs and apply edge Stroke
    Color / node Fill Color discrete and continuous mappings on ''interaction'', ''TopSharedMotifs'',
    ''CF_subclass'', and ''CF_subclass_score'''
  repo: https://cytoscape.org/
- name: pyMolNetEnhancer
  role: Python module whose make_classyfire_graphml merge and networkx.write_graphml
    export produce the graphml and Mass2Motifs_Edges/Nodes tsv files styled in Cytoscape
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: R analog producing the same .graphml and Mass2Motifs_Edges_Classical.tsv /
    Mass2Motifs_Nodes_Classical.tsv outputs for Cytoscape import
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: networkx
  role: Writes the merged Motif_ChemicalClass_Network_Classical.graphml with infer_numeric_types=True
    so it parses correctly on Cytoscape import
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:260b076bda5339e5d82ba805d9f0812db819c0a9c6a1e7aa3fd74ff4b272a131
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - MolNetEnhancer
  - Network Annotation Propagation
  - RMolNetEnhancer
  - molconvert
  - networkx
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cytoscape-network-visualization-styling

## Summary

Import MolNetEnhancer-enhanced molecular networks (.graphml with integrated Mass2Motif and ClassyFire chemical class annotations) into Cytoscape and apply discrete/continuous visual mappings to edges and nodes, or load the equivalent tab-delimited edges/nodes tables separately.

## When to use

Use when a Motif_ChemicalClass_Network .graphml (or Mass2Motifs_Edges/Nodes .tsv pair) from pyMolNetEnhancer/RMolNetEnhancer must be visualized in Cytoscape with edges colored by shared Mass2Motifs and nodes colored by CF_subclass chemical classes.

## When NOT to use

- The molecular network has not yet been through Mass2Motif mapping or ClassyFire chemical class annotation, so the .graphml lacks 'TopSharedMotifs', 'interaction', and CF_-prefixed columns to map onto.
- The input is a feature table (e.g. MZmine feature quantification table) rather than a MolNetEnhancer graphml/tsv network output.
- Interactive per-family motif exploration in the ms2lda.org web app is the goal, rather than Cytoscape rendering of the enhanced network.

## Inputs

- Merged Mass2Motif + chemical class network object (MG) in pyMolNetEnhancer
- 'Motif_ChemicalClass_Network_Classical.graphml' file with CF_-prefixed columns and Mass2Motif columns
- 'Mass2Motifs_Edges_Classical.tsv' (CLUSTERID1, interact, CLUSTERID2 columns)
- 'Mass2Motifs_Nodes_Classical.tsv' node table

## Outputs

- Cytoscape network session with edge Stroke Color discrete mapping on 'interaction' and node Fill Color mapping on 'CF_subclass' (discrete) or 'CF_subclass_score' (continuous)
- Cytoscape network loaded from Mass2Motifs_Edges_Classical.tsv plus imported Mass2Motifs_Nodes_Classical.tsv node table

## How to apply

First generate the integrated network by merging the Mass2Motif-annotated network object (MG) with the ClassyFire chemical-class dataframe (final) via make_classyfire_graphml, then write it with nx.write_graphml(..., infer_numeric_types=True) to produce 'Motif_ChemicalClass_Network_Classical.graphml'. Import this .graphml into Cytoscape. For edge styling, select 'Stroke Color' in the 'Edge' tab, choose 'interaction' as Column and 'Discrete Mapping' as Mapping Type to color edges by shared Mass2Motifs between nodes. For node styling, use 'Fill Color' with a Discrete Mapping on 'CF_subclass' or a Continuous Mapping on 'CF_subclass_score'; the README also shows using 'Image/Chart' on the Node tab to display the most shared Mass2Motifs per molecular family from 'TopSharedMotifs' in the Edge Table. As an alternative to a single graphml, import 'Mass2Motifs_Edges_Classical.tsv' as a network with CLUSTERID1 as Source, 'interact' as Interaction, CLUSTERID2 as Target, then import 'Mass2Motifs_Nodes_Classical.tsv' as a node table.

## Related tools

- **Cytoscape** (Visualization platform: import .graphml or tsv outputs and apply edge Stroke Color / node Fill Color discrete and continuous mappings on 'interaction', 'TopSharedMotifs', 'CF_subclass', and 'CF_subclass_score') — https://cytoscape.org/
- **pyMolNetEnhancer** (Python module whose make_classyfire_graphml merge and networkx.write_graphml export produce the graphml and Mass2Motifs_Edges/Nodes tsv files styled in Cytoscape) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (R analog producing the same .graphml and Mass2Motifs_Edges_Classical.tsv / Mass2Motifs_Nodes_Classical.tsv outputs for Cytoscape import) — https://github.com/madeleineernst/RMolNetEnhancer
- **networkx** (Writes the merged Motif_ChemicalClass_Network_Classical.graphml with infer_numeric_types=True so it parses correctly on Cytoscape import)

## Examples

```
nx.write_graphml(MG, 'Motif_ChemicalClass_Network_Classical.graphml', infer_numeric_types=True)
```

## Evaluation signals

- The .graphml file exists, parses as valid graphml, and imports into Cytoscape without errors.
- The graphml contains CF_-prefixed chemical class columns (e.g. CF_subclass, CF_subclass_score) alongside Mass2Motif columns (TopSharedMotifs, interaction).
- Edge attributes 'TopSharedMotifs' and 'interaction' are populated for the molecular families of interest (e.g. the two Euphorbia diterpenoid families with 73 features).
- Mass2Motifs_Edges_Classical.tsv loads as a Cytoscape network with Source='CLUSTERID1', Interaction='interact', Target='CLUSTERID2', and the nodes tsv imports as a node table.
- Discrete/continuous mappings resolve visually: edges differ in stroke color by shared motif, nodes differ in fill color by CF_subclass or by CF_subclass_score gradient.

## Limitations

- Downloading the MS2LDA summary may hit a server connection timeout on ms2lda.org depending on file size; the file must then be exported manually from the web app.
- The ms2lda.org summary table contains motif-document relations already filtered by the probability/overlap thresholds set in the web app's Experimental Options tab, so thresholds set there (default prob=0.01, overlap=0.3) constrain what can be mapped and visualized downstream.
- Only the top N most shared motifs per molecular family (default top=5) are shown in TopSharedMotifs, limiting which motifs are visible in node styling.

## Evidence

- [readme] Edge coloring by shared motifs: "To color edges based on shared Mass2Motifs in between nodes select 'Stroke Color' in the 'Edge' tab to the left and choose 'interaction' as <i>Column</i> and 'Discrete Mapping'"
- [readme] Separate tsv import route: "Alternatively the edges and nodes output files can also be loaded separately into Cytoscape. To this end import the 'Mass2Motifs_Edges_Classical.tsv' output file as network into Cytoscape."
- [readme] TSV column mapping: "Select column 'CLUSTERID1' as Source Node, column 'interact' as Interaction Type and 'CLUSTERID2' as Target Node"
- [methods] Skill purpose in the workflow: "Map chemical class and MS2LDA substructural information to mass spectral molecular networks"
