---
name: network-graphics-export
description: Use when a combined metabolic network edge list (source, target, edge-type)
  and node attribute table exist and must be handed off to Cytoscape or exported as
  publication-quality SVG/PNG network graphics.
when_to_use_negative:
- Input is raw metabolite measurement data that has not yet been converted into a
  calculated network edge list by MetaMapR
- The goal is to compute new network edges (biochemical, structural similarity, mass
  spectral or correlation) rather than export an already calculated network
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: MetaMapR
  role: Source of the calculated network edge list and node attributes and the application
    from which export is performed
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Programming language in which MetaMapR and the export/rendering pipeline (d3Networks,
    Shiny) are implemented
- name: Shiny
  role: R package providing the web/desktop user interface through which network export
    options are accessed
  repo: https://github.com/rstudio/shiny-server
- name: Cytoscape
  role: Downstream network analysis and visualization tool into which the exported
    CSV tables are imported and node attributes mapped
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:7ad2736078aa9b7cdd5aeb51a0a22376de9891a8c1fefbce4e0e408a4d8dc5eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - Cytoscape
  - MetaMapR
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# network-graphics-export

## Summary

Export a calculated MetaMapR metabolic network edge list and node attributes as CSV numerical tables for ingestion into Cytoscape, and render the interactive D3.js network to high-quality static SVG and PNG graphics.

## When to use

Use when a combined metabolic network edge list (source, target, edge-type) and node attribute table exist and must be handed off to Cytoscape or exported as publication-quality SVG/PNG network graphics.

## When NOT to use

- Input is raw metabolite measurement data that has not yet been converted into a calculated network edge list by MetaMapR
- The goal is to compute new network edges (biochemical, structural similarity, mass spectral or correlation) rather than export an already calculated network

## Inputs

- MetaMapR network edge list (source, target, edge-type columns)
- Node attributes table mapping network identifier to assigned numeric node ID

## Outputs

- Edge list CSV (.csv) formatted for Cytoscape import
- Node attributes CSV (.csv) preserving numeric node ID mapping
- High-quality network graphics in SVG format
- High-quality network graphics in PNG format

## How to apply

Load the combined edge list and node attribute table produced by the network calculation (columns: source, target, edge-type, node identifier). Write the edge list as a comma-separated value (.csv) file formatted for import into third-party network software such as Cytoscape (Shannon et al., 2003), and write the node attributes table as a separate .csv preserving the mapping between the network identifier and the assigned numeric node ID — by default numeric IDs follow the row order of the uploaded data, and a unique custom ID can be substituted via the network > more options > edge index setting. Render the interactive network (built with the R package d3Networks using the D3.js library) to static scalable vector graphic (SVG) and portable network graphic (PNG) formats. Finally, validate that all output files exist and are parseable CSV/SVG/PNG, and that node IDs in the edge list match the node attributes table.

## Related tools

- **MetaMapR** (Source of the calculated network edge list and node attributes and the application from which export is performed) — https://github.com/dgrapov/MetaMapR
- **R** (Programming language in which MetaMapR and the export/rendering pipeline (d3Networks, Shiny) are implemented)
- **Shiny** (R package providing the web/desktop user interface through which network export options are accessed) — https://github.com/rstudio/shiny-server
- **Cytoscape** (Downstream network analysis and visualization tool into which the exported CSV tables are imported and node attributes mapped)

## Examples

```
write.csv(edge_list, 'network_edges.csv', row.names=FALSE); write.csv(node_attributes, 'node_attributes.csv', row.names=FALSE)
```

## Evaluation signals

- All output files (edge list CSV, node attributes CSV, SVG, PNG) exist and are parseable
- Node IDs in the exported edge list match the node attributes table
- The mapping between network identifier and numeric node ID is preserved (default: ID equals row order of uploaded data, e.g. 1 = first-row analyte)
- Edge-type column is retained so Cytoscape users can distinguish Biochemical > Structural > Mass Spectral > Correlation edges or the unique-edges hierarchy
- Imported network renders in Cytoscape with empirical and domain knowledge-based variables mappable to node attributes

## Limitations

- KEGG database limitations on directionality of biochemical transformations carry into the exported network
- Custom node IDs used via the edge index option must be unique; non-unique IDs break the mapping
- Rendering depends on d3Networks/D3.js; MetaMapR's TODO list includes adding cytoscape.js networks, so interactive web rendering options are limited in v1.4.0

## Evidence
