---
name: mass-spectral-molecular-network-annotation-merging
description: Use when a GNPS molecular network already has mapped Mass2Motifs and
  a chemical class annotation dataframe is available, and the two annotation layers
  must be combined into one Cytoscape-ready network artifact.
when_to_use_negative:
- The network has not yet had Mass2Motifs mapped to it or the chemical class annotation
  dataframe has not yet been produced — run the motif-mapping and chemical-class mapping
  steps first.
- Only in silico structure annotation (Network Annotation Propagation or DEREPLICATOR)
  output is needed — those tools are complementary to this workflow and are not merged
  by this skill.
- A standalone chemical classification (ClassyFire) result is wanted without integration
  into a molecular network.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pyMolNetEnhancer
  role: Python module that performs the merge via make_classyfire_graphml and exports
    the integrated network; installed with pip install pyMolNetEnhancer
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: networkx
  role: Writes the merged network object to .graphml with infer_numeric_types=True
- name: Cytoscape
  role: Imports the .graphml (or the separate edge/node TSV files) for visualization
    and discrete/continuous color mapping of edges and nodes
  repo: https://cytoscape.org/
- name: RMolNetEnhancer
  role: Analogous R package providing the same network annotation integration workflow
  repo: https://github.com/madeleineernst/RMolNetEnhancer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:90cb2df18ac91de9610c1ea4d55c45f652e2c9f919a2e979137ea22a94abf638
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - MolNetEnhancer
  - RMolNetEnhancer
  - networkx
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mass-spectral-molecular-network-annotation-merging

## Summary

Merges Mass2Motif substructure annotations and ClassyFire chemical class annotations into a single GNPS molecular network object using pyMolNetEnhancer, then exports the integrated network as a .graphml file plus tab-delimited edge/node tables for Cytoscape visualization.

## When to use

Use when a GNPS molecular network already has mapped Mass2Motifs and a chemical class annotation dataframe is available, and the two annotation layers must be combined into one Cytoscape-ready network artifact.

## When NOT to use

- The network has not yet had Mass2Motifs mapped to it or the chemical class annotation dataframe has not yet been produced — run the motif-mapping and chemical-class mapping steps first.
- Only in silico structure annotation (Network Annotation Propagation or DEREPLICATOR) output is needed — those tools are complementary to this workflow and are not merged by this skill.
- A standalone chemical classification (ClassyFire) result is wanted without integration into a molecular network.

## Inputs

- GNPS molecular network object with mapped Mass2Motifs (from the MS2LDA motif-mapping step, classical or feature-based)
- Chemical class annotation dataframe (ClassyFire-based, from the chemical-class mapping step)

## Outputs

- 'Motif_ChemicalClass_Network_Classical.graphml' containing both Mass2Motif columns (TopSharedMotifs, interaction) and CF_-prefixed chemical class columns (e.g. CF_subclass, CF_subclass_score)
- 'Mass2Motifs_Edges_Classical.tsv' tab-delimited edge table
- 'Mass2Motifs_Nodes_Classical.tsv' tab-delimited node table
- Cytoscape visualization with edges colored by shared Mass2Motifs and nodes colored by chemical class

## How to apply

Load the network object with mapped Mass2Motifs (produced by the motif-mapping step, filtered by the user-defined prob (default 0.01), overlap (default 0.3), and top (default 5) parameters) together with the chemical class annotation dataframe into pyMolNetEnhancer. Call make_classyfire_graphml(MG, final) to merge the Mass2Motif-annotated network with the ClassyFire-derived chemical class information into a single network object. Write the merged network with networkx.write_graphml using infer_numeric_types=True to produce 'Motif_ChemicalClass_Network_Classical.graphml'. The same structural information is additionally exported as tab-delimited text files ('Mass2Motifs_Edges_Classical.tsv' and 'Mass2Motifs_Nodes_Classical.tsv'). In Cytoscape, import the .graphml and color edges by 'interaction' (Stroke Color, Discrete Mapping) and nodes by 'CF_subclass' (Fill Color, Discrete Mapping) or 'CF_subclass_score' (Continuous Mapping); alternatively load the edge TSV as a network with Source: CLUSTERID1, Interaction: interact, Target: CLUSTERID2 and import the node TSV as a node table.

## Related tools

- **pyMolNetEnhancer** (Python module that performs the merge via make_classyfire_graphml and exports the integrated network; installed with pip install pyMolNetEnhancer) — https://github.com/madeleineernst/pyMolNetEnhancer
- **networkx** (Writes the merged network object to .graphml with infer_numeric_types=True)
- **Cytoscape** (Imports the .graphml (or the separate edge/node TSV files) for visualization and discrete/continuous color mapping of edges and nodes) — https://cytoscape.org/
- **RMolNetEnhancer** (Analogous R package providing the same network annotation integration workflow) — https://github.com/madeleineernst/RMolNetEnhancer

## Examples

```
import networkx as nx; G = make_classyfire_graphml(MG, final); nx.write_graphml(G, 'Motif_ChemicalClass_Network_Classical.graphml', infer_numeric_types=True)
```

## Evaluation signals

- The 'Motif_ChemicalClass_Network_Classical.graphml' file exists and parses as valid graphml.
- The .graphml contains CF_-prefixed chemical class columns (e.g. CF_subclass, CF_subclass_score) alongside the Mass2Motif columns (TopSharedMotifs, interaction).
- 'Mass2Motifs_Edges_Classical.tsv' is loadable in Cytoscape as a network with Source: CLUSTERID1, Interaction: interact, Target: CLUSTERID2.
- 'Mass2Motifs_Nodes_Classical.tsv' is importable as a Cytoscape node table.
- In Cytoscape, edge Stroke Color can be mapped discretely on 'interaction' and node Fill Color on 'CF_subclass' (or CF_subclass_score continuously).

## Limitations

- The MS2LDA summary file download may hit a server connection timeout depending on file size; it can alternatively be downloaded manually at http://ms2lda.org/.
- The probability and overlap thresholds can also be set within the ms2lda.org app under the Experimental Options tab, and the summary table then contains filtered motif-document relations using those web-app thresholds, so thresholds must be consistent between the web app and the mapping parameters.
- The top parameter (default 5) restricts how many most shared motifs per molecular family (network component index) are shown.

## Evidence

- [intro] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
- [methods] Map combined chemical class and MS2LDA substructural information to mass spectral molecular networks: "Map chemical class and MS2LDA substructural information to mass spectral molecular networks"
- [readme] Color edges by shared Mass2Motifs in Cytoscape: "To color edges based on shared Mass2Motifs in between nodes select 'Stroke Color' in the 'Edge' tab to the left and choose 'interaction' as <i>Column</i>"
- [readme] Import edges TSV into Cytoscape with CLUSTERID1/interact/CLUSTERID2 columns: "import the 'Mass2Motifs_Edges_Classical.tsv' output file as network into Cytoscape. Select column 'CLUSTERID1' as Source Node, column 'interact' as Interaction Type"
