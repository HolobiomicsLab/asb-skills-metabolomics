---
name: metabolomic-network-export
description: Use when a MetaMapR network (edge list with source, target, edge-type
  columns and a node attributes table with numeric node IDs) must be handed off to
  downstream network tools such as Cytoscape.
when_to_use_negative:
- The network has not yet been calculated in MetaMapR (no edge list or node ID mapping
  exists) — identifier translation or network calculation must happen first
- The target analysis requires the interactive Shiny/D3.js visualization rather than
  a static or tabular export
- Node IDs need to be re-derived from scratch — use MetaMapR's edge index option during
  calculation instead of post-hoc reassignment
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
tools:
- name: MetaMapR
  role: Produces the calculated edge list and node attributes table being exported;
    supports custom node IDs via the edge index option
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Programming language in which MetaMapR and the export logic (CSV writing,
    d3Networks rendering) are implemented
  repo: http://cran.us.r-project.org/
- name: Shiny
  role: R package providing the web/desktop UI through which the export options (network
    tab, node attributes tab) are accessed
  repo: https://github.com/rstudio/shiny-server
- name: Cytoscape
  role: Downstream tool that ingests the exported numerical tables, where empirical
    and domain knowledge-based variables are mapped onto node attributes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:1afa8acd8132e17f83030dddf96a749a3f4c00aaaff38c42e79f8a0ae1a7e335
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomic-network-export

## Summary

Exports a MetaMapR-calculated metabolic network (edge list plus node attribute table) as CSV files formatted for Cytoscape ingestion, alongside static high-quality graphics (SVG/PNG) rendered from the interactive D3.js visualization.

## When to use

Use when a MetaMapR network (edge list with source, target, edge-type columns and a node attributes table with numeric node IDs) must be handed off to downstream network tools such as Cytoscape.

## When NOT to use

- The network has not yet been calculated in MetaMapR (no edge list or node ID mapping exists) — identifier translation or network calculation must happen first
- The target analysis requires the interactive Shiny/D3.js visualization rather than a static or tabular export
- Node IDs need to be re-derived from scratch — use MetaMapR's edge index option during calculation instead of post-hoc reassignment

## Inputs

- MetaMapR network edge list (source, target, edge-type columns)
- MetaMapR node attributes table mapping network identifiers to numeric node IDs
- Interactive D3.js network visualization rendered by the MetaMapR Shiny app

## Outputs

- Edge list .csv file formatted for Cytoscape import
- Node attributes .csv file preserving identifier-to-numeric-node-ID mapping
- Static network graphics in SVG format
- Static network graphics in PNG format

## How to apply

Load the combined edge list and node attribute table produced by the network calculation step, containing source, target, edge-type and node identifier columns. Write the edge list as a comma-separated (.csv) file formatted for import into Cytoscape (Shannon et al., 2003), and write the node attributes table as a separate .csv that preserves the mapping between the network identifier and the assigned numeric node ID. By default numeric node IDs follow the row order of the uploaded data (row 1 = first analyte); a unique custom ID can be substituted during calculation via network > more options > edge index. If multiple edge types exist, decide whether to return all edges or only unique ones — the unique-edges checkbox in the network tab returns edges by the hierarchy Biochemical > Structural > Mass Spectral > Correlation. Finally, render the interactive D3.js network to static high-quality graphics in SVG and PNG formats using the R package d3Networks.

## Related tools

- **MetaMapR** (Produces the calculated edge list and node attributes table being exported; supports custom node IDs via the edge index option) — https://github.com/dgrapov/MetaMapR
- **R** (Programming language in which MetaMapR and the export logic (CSV writing, d3Networks rendering) are implemented) — http://cran.us.r-project.org/
- **Shiny** (R package providing the web/desktop UI through which the export options (network tab, node attributes tab) are accessed) — https://github.com/rstudio/shiny-server
- **Cytoscape** (Downstream tool that ingests the exported numerical tables, where empirical and domain knowledge-based variables are mapped onto node attributes)

## Evaluation signals

- All output files exist and are parseable as CSV, SVG, or PNG
- Every node ID appearing in the exported edge list has a matching entry in the exported node attributes table
- The edge list contains source, target, and edge-type columns for every exported edge
- Numeric node IDs correspond to the row order of the uploaded data (or the unique custom ID given via edge index)
- Exported network re-imported into Cytoscape reproduces the MetaMapR connectivity for the chosen edge-type setting (all edges vs. unique edges hierarchy)

## Limitations

- Default numeric node IDs are assigned by row order of the uploaded data, so any reordering of the input table breaks the ID-to-analyte mapping unless a custom edge index was used
- The custom node ID substituted via the edge index option must be unique; duplicate IDs invalidate the mapping
- If unique edges are requested, edge information is collapsed by the hierarchy Biochemical > Structural > Mass Spectral > Correlation, so lower-priority edge types are lost
- Static SVG/PNG rendering loses the interactivity of the D3.js visualization

## Evidence

- [readme] README FAQ on unique edge hierarchy: [evidence span withheld — anchor status: external_doc]
