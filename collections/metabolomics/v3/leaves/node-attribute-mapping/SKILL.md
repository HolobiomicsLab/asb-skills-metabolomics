---
name: node-attribute-mapping
description: Use when a calculated metabolic network edge list must be linked back
  to per-metabolite measurements or annotations before export to Cytoscape, especially
  when numeric node IDs were assigned by row order of the uploaded data.
when_to_use_negative:
- The network has not yet been calculated in MetaMapR — identifier translation and
  network construction (KEGG RPAIR, PubChem fingerprint, spectral, correlation) must
  precede attribute mapping
- The goal is identifier translation itself (e.g. chemical names to PubChem CIDs)
  rather than attaching measured variables to network nodes
- Input is a raw metabolomics feature table not yet matched to the network's numeric
  node IDs and lacking a unique ID per node
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: MetaMapR
  role: Produces the calculated edge list and node attributes tab showing the mapping
    between network identifier and assigned numeric node ID
  repo: https://github.com/dgrapov/MetaMapR
- name: Cytoscape
  role: Downstream tool into which the exported tables are imported and empirical/domain
    variables are mapped onto node attributes
- name: Shiny
  role: R package underpinning the MetaMapR interface used to generate and export
    the network tables
  repo: https://github.com/rstudio/shiny-server
- name: R
  role: Language in which the export and attribute-mapping workflow is implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:6d209cdf4783d3718f546df5a3d0ef456c214eef3880fd5f4d92ddab4f355b73
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - Cytoscape
  - MetaMapR
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# node-attribute-mapping

## Summary

Maps empirical and domain knowledge-based variables (e.g. fold-changes, annotations) onto metabolite nodes of a MetaMapR-derived network, preserving the mapping between network numeric node IDs and uploaded data rows so the enriched network can be analyzed and visualized in Cytoscape.

## When to use

Use when a calculated metabolic network edge list must be linked back to per-metabolite measurements or annotations before export to Cytoscape, especially when numeric node IDs were assigned by row order of the uploaded data.

## When NOT to use

- The network has not yet been calculated in MetaMapR — identifier translation and network construction (KEGG RPAIR, PubChem fingerprint, spectral, correlation) must precede attribute mapping
- The goal is identifier translation itself (e.g. chemical names to PubChem CIDs) rather than attaching measured variables to network nodes
- Input is a raw metabolomics feature table not yet matched to the network's numeric node IDs and lacking a unique ID per node

## Inputs

- MetaMapR network edge list (source, target, edge-type columns)
- Node attribute table with mapping between network identifier and assigned numeric node ID
- Uploaded metabolite data table (row order determines default numeric node IDs)

## Outputs

- Edge list .csv formatted for Cytoscape import
- Node attributes .csv mapping network identifiers to numeric node IDs and empirical/domain variables
- Static high-quality network graphics in SVG and PNG formats (via d3Networks)

## How to apply

Load the combined edge list and node attribute table produced by the network calculation (source, target, edge-type columns plus node identifier columns). Because MetaMapR assigns numeric node IDs by default according to the row order of the uploaded data (row 1 = analyte in the first row), build the node attributes table so each network identifier is paired with its assigned numeric ID; a unique custom ID can be substituted during network calculation under network > more options > edge index. Write the edge list as a .csv formatted for Cytoscape import and the node attributes as a separate .csv, then in Cytoscape map the empirical and domain knowledge-based variables (e.g. T1D-associated metabolite changes from the GC-TOF-MS plasma study) onto the network as node attributes. Render the interactive network to static high-quality graphics (SVG/PNG) via D3.js with the R package d3Networks if static figures are also needed.

## Related tools

- **MetaMapR** (Produces the calculated edge list and node attributes tab showing the mapping between network identifier and assigned numeric node ID) — https://github.com/dgrapov/MetaMapR
- **Cytoscape** (Downstream tool into which the exported tables are imported and empirical/domain variables are mapped onto node attributes)
- **Shiny** (R package underpinning the MetaMapR interface used to generate and export the network tables) — https://github.com/rstudio/shiny-server
- **R** (Language in which the export and attribute-mapping workflow is implemented)

## Examples

```
write.csv(edge_list, 'network_edges.csv', row.names=FALSE); write.csv(node_attributes, 'network_node_attributes.csv', row.names=FALSE)
```

## Evaluation signals

- All output files exist and are parseable CSV/SVG/PNG
- Node IDs in the edge list match the node attributes table one-to-one
- Mapping between network identifier and assigned numeric node ID preserved (default: ID follows row order of uploaded data)
- If a custom edge index was used, every custom ID is unique across the network
- Empirical and domain knowledge-based variables appear as node attributes after import into Cytoscape

## Limitations

- Default numeric node IDs depend on the row order of the uploaded data; reordering the input table changes the ID mapping unless a unique custom edge index is supplied
- KeggR database limitation on directionality of biochemical transformations means the exported network may not encode reaction direction as a node/edge attribute
- Custom node IDs must be unique; duplicate IDs break the attribute mapping

## Evidence
