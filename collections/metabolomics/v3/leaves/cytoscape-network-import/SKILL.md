---
name: cytoscape-network-import
description: Use when a calculated network edge list and node attribute table (source,
  target, edge-type, node ID columns) must be handed off to Cytoscape or other third-party
  network analysis/visualization software.
when_to_use_negative:
- The analysis goal is interactive in-app visualization or threshold tuning inside
  MetaMapR rather than export to external network software
- The network has not yet been calculated — no edge list or node attribute table exists
  yet (this skill is an export/handoff step, not a network construction step)
- Only metabolite identifier translation is needed (use the Chemical Translation System
  / CTSgetR path instead)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_1317
- http://edamontology.org/topic_3315
tools:
- name: MetaMapR
  role: Produces the calculated network edge list and node attributes table that are
    exported; provides the edge index option for custom node IDs
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Language in which MetaMapR is implemented; used to write the .csv exports
    and run d3Networks for static rendering
  repo: http://cran.us.r-project.org/
- name: Shiny
  role: R package providing the web UI through which the network calculation and export
    workflow is executed
  repo: https://github.com/rstudio/shiny-server
- name: Cytoscape
  role: Downstream tool that ingests the exported edge list and node attribute tables;
    empirical and domain knowledge-based variables are mapped onto node attributes
    there
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:a5a83637923b0af1ff197fa0920fb1bbc4a2c22765a6bb279842661ccfcc876c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - Cytoscape
  - MetaMapR
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_1317
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cytoscape-network-import

## Summary

Exports a MetaMapR-calculated metabolic network as Cytoscape-ready numerical tables (edge list and node attributes in .csv) and high-quality static graphics (SVG/PNG via D3.js/d3Networks), so empirical and domain-knowledge variables can be mapped onto node attributes in downstream visualization tools.

## When to use

Use when a calculated network edge list and node attribute table (source, target, edge-type, node ID columns) must be handed off to Cytoscape or other third-party network analysis/visualization software.

## When NOT to use

- The analysis goal is interactive in-app visualization or threshold tuning inside MetaMapR rather than export to external network software
- The network has not yet been calculated — no edge list or node attribute table exists yet (this skill is an export/handoff step, not a network construction step)
- Only metabolite identifier translation is needed (use the Chemical Translation System / CTSgetR path instead)

## Inputs

- MetaMapR network edge list (source, target, edge-type columns)
- Node attribute table mapping network identifiers to assigned numeric node IDs
- Calculated network (biochemical, structural similarity, mass spectral, and empirical correlation edges)

## Outputs

- Edge list .csv formatted for Cytoscape import
- Node attributes .csv preserving network identifier to numeric node ID mapping
- Static high-quality network graphics in SVG and PNG formats (via d3Networks/D3.js)

## How to apply

Load the combined edge list and node attribute table produced by the network calculation (source, target, edge-type, and node identifier columns). Write the edge list as a comma-separated .csv formatted for Cytoscape import (Shannon et al., 2003), and write the node attributes table as a separate .csv preserving the mapping between the network identifier and the assigned numeric node ID. By default numeric node IDs follow the row order of the uploaded data (row 1 = first analyte); a unique custom ID can be substituted via the edge index option under network > more options. Render the interactive network to static high-quality graphics using D3.js with the R package d3Networks, exporting SVG and PNG formats. Verify that node IDs in the edge list match the node attributes table before handoff. When multiple edge types are present, decide whether to return all edge types or only unique edges via the unique edges checkbox, which prioritizes edges by the hierarchy Biochemical > Structural > Mass Spectral > Correlation.

## Related tools

- **MetaMapR** (Produces the calculated network edge list and node attributes table that are exported; provides the edge index option for custom node IDs) — https://github.com/dgrapov/MetaMapR
- **R** (Language in which MetaMapR is implemented; used to write the .csv exports and run d3Networks for static rendering) — http://cran.us.r-project.org/
- **Shiny** (R package providing the web UI through which the network calculation and export workflow is executed) — https://github.com/rstudio/shiny-server
- **Cytoscape** (Downstream tool that ingests the exported edge list and node attribute tables; empirical and domain knowledge-based variables are mapped onto node attributes there)

## Examples

```
write.csv(edge_list, file='network_edges.csv'); write.csv(node_attributes, file='node_attributes.csv'); d3SimpleNetwork(edge_list, file='network.svg')
```

## Evaluation signals

- All output files exist and are parseable as CSV, SVG, and PNG
- Every numeric node ID appearing in the exported edge list matches a row in the node attributes table
- The node attributes table preserves the identifier-to-numeric-ID mapping (default: row order of the uploaded data; e.g. 1 = analyte in first row)
- Edge-type column retained in the exported edge list so Cytoscape can distinguish Biochemical, Structural, Mass Spectral, and Correlation edges
- Imported network in Cytoscape renders the same topology and supports mapping of empirical and domain-knowledge variables to node attributes

## Limitations

- By default numeric node IDs are assigned by row order of uploaded data; if a custom edge index is used it must be unique, otherwise mapping between edge list and attributes breaks
- Static graphic rendering relies on the R package d3Networks/D3.js; the README TODO lists 'add cytoscape.js networks' as an unimplemented feature
- Only edges are ranked when unique-edges mode is selected (Biochemical > Structural > Mass Spectral > Correlation); non-top edges are dropped, so downstream analyses see a reduced edge set

## Evidence

- [readme] Edge-type hierarchy when exporting unique edges: [evidence span withheld — anchor status: external_doc]
- [other] Static rendering library: [evidence span withheld — anchor status: not_found]
