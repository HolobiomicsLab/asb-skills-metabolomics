---
name: tabular-data-format-conversion
description: Use when a calculated MetaMapR network (edge list with source, target,
  edge-type columns plus a node attribute table) must be handed off to downstream
  tools like Cytoscape or archived as numerical tables and high-quality graphics.
when_to_use_negative:
- The network itself has not yet been calculated (biochemical, structural, spectral
  or empirical networks still need to be computed first) — this skill only converts
  and exports existing tables
- The input is already a Cytoscape-ready edge/attribute table or the goal is network
  visualization within Cytoscape rather than file format conversion
- The task is metabolite identifier translation between biochemical databases (e.g.
  chemical names to KEGG or PubChem CIDs) — that is handled by the Chemical Translation
  System/CTSgetR, not table export
tools:
- name: MetaMapR
  role: Source of the calculated edge list and node attribute tables; exposes the
    node attributes tab and the edge index option for custom node IDs
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Programming language in which MetaMapR is implemented and in which the edge
    list and node attribute tables are written out as .csv files
  repo: http://cran.us.r-project.org/
- name: Shiny
  role: R package powering the MetaMapR interface from which networks and node attributes
    are exported
  repo: https://github.com/rstudio/shiny-server
- name: Cytoscape
  role: Downstream network analysis and visualization tool that ingests the exported
    .csv edge list, with empirical and domain knowledge-based variables mapped to
    node attributes
- name: d3Networks
  role: R package using D3.js to render the interactive network to static high-quality
    SVG and PNG graphics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:09eac941f36582df99241e676a41df2e106c7e71516bfbc5d46f00d2a8a5dddb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - Cytoscape
  - MetaMapR
  - R
  - Shiny
  - d3Networks
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tabular-data-format-conversion

## Summary

Exports MetaMapR network outputs — the edge list and node attribute tables — into comma-separated value (.csv) files formatted for import into Cytoscape and other third-party network analysis/visualization software, alongside static SVG/PNG graphics.

## When to use

Use when a calculated MetaMapR network (edge list with source, target, edge-type columns plus a node attribute table) must be handed off to downstream tools like Cytoscape or archived as numerical tables and high-quality graphics.

## When NOT to use

- The network itself has not yet been calculated (biochemical, structural, spectral or empirical networks still need to be computed first) — this skill only converts and exports existing tables
- The input is already a Cytoscape-ready edge/attribute table or the goal is network visualization within Cytoscape rather than file format conversion
- The task is metabolite identifier translation between biochemical databases (e.g. chemical names to KEGG or PubChem CIDs) — that is handled by the Chemical Translation System/CTSgetR, not table export

## Inputs

- MetaMapR network edge list (source, target, edge-type columns)
- MetaMapR node attribute table with identifier-to-numeric-node-ID mapping
- Empirical and domain knowledge-based variables to map onto node attributes

## Outputs

- Edge list .csv file formatted for Cytoscape import
- Node attributes .csv file keyed by network identifier and numeric node ID
- Static network graphics in SVG format
- Static network graphics in PNG format

## How to apply

Load the combined network edge list and node attribute table produced by the MetaMapR network calculation (source, target, edge-type, and node identifier columns). Write the edge list as a comma-separated value (.csv) file formatted for import into third-party network software such as Cytoscape (Shannon et al., 2003), keeping the edge-type column so Biochemical, Structural, Mass Spectral and Correlation edges remain distinguishable (or use the 'unique edges' hierarchy Biochemical > Structural > Mass Spectral > Correlation if only unique edges are wanted). Write the node attributes table as a separate .csv preserving the mapping between the network identifier and the assigned numeric node ID — by default numeric IDs follow the row order of the uploaded data (row 1 = first analyte), and a unique custom ID can be substituted via the edge index option under network > more options. Render the interactive network to static high-quality graphics using D3.js via the R package d3Networks, exporting in SVG and PNG formats. Validate that all output files exist, parse as CSV/SVG/PNG, and that node IDs in the edge list match those in the node attributes table.

## Related tools

- **MetaMapR** (Source of the calculated edge list and node attribute tables; exposes the node attributes tab and the edge index option for custom node IDs) — https://github.com/dgrapov/MetaMapR
- **R** (Programming language in which MetaMapR is implemented and in which the edge list and node attribute tables are written out as .csv files) — http://cran.us.r-project.org/
- **Shiny** (R package powering the MetaMapR interface from which networks and node attributes are exported) — https://github.com/rstudio/shiny-server
- **Cytoscape** (Downstream network analysis and visualization tool that ingests the exported .csv edge list, with empirical and domain knowledge-based variables mapped to node attributes)
- **d3Networks** (R package using D3.js to render the interactive network to static high-quality SVG and PNG graphics)

## Evaluation signals

- All expected output files exist and are parseable as CSV, SVG and PNG
- Node IDs appearing in the exported edge list match the node IDs in the node attributes table
- Node attribute table preserves the identifier-to-numeric-ID mapping consistent with the uploaded data row order (row 1 = first analyte) or the custom edge index ID
- Edge-type column is retained so the Biochemical, Structural, Mass Spectral and Correlation edge classes (or the unique-edge hierarchy) are distinguishable after import into Cytoscape

## Limitations

- By default numeric node IDs are assigned by row order of the uploaded data, so reordering rows changes IDs; a unique custom ID must be supplied via the edge index option to keep mappings stable
- Custom node IDs used through the edge index option must be unique, otherwise the mapping is ambiguous
- Only unique edges (hierarchical: Biochemical > Structural > Mass Spectral > Correlation) or all edges can be exported — mixed granularity must be decided before export
- The exported numerical tables are only useful downstream if the network was previously thresholded (e.g. Tanimoto score >0.7, cosine correlation >0.7); export does not itself apply filters

## Evidence
