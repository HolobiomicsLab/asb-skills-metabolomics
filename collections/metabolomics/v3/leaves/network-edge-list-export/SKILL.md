---
name: network-edge-list-export
description: Use when MetaMapR network calculations (KEGG RPAIR, Tanimoto >0.7 structural,
  cosine >0.7 spectral, or Spearman/FDR <0.05 correlation edges) must be persisted
  as .csv edge lists and node attributes for downstream network analysis.
when_to_use_negative:
- The metabolite identifiers have not yet been translated to KEGG compound IDs or
  PubChem CIDs via CTSgetR — run identifier translation first
- The analysis goal is a canonical pathway-based network map; MetaMapR is explicitly
  pathway independent and its biochemical layer derives only KEGG RPAIR substrate-product
  pairs
- Raw mass spectral or chromatographic data still require preprocessing — this skill
  operates on already-computed similarity/correlation edge tables
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0605
tools:
- name: MetaMapR
  role: Computes the four edge-type network and exposes the unique-edges hierarchy,
    node-ID mapping, and .csv export used by this skill
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Language in which the edge lists are assembled and the .csv edge-list/node-attribute
    files are written
- name: Shiny
  role: R package powering the MetaMapR web/desktop UI from which results are exported
  repo: https://github.com/rstudio/shiny-server
- name: Cytoscape
  role: Downstream destination into which the exported edge list and node attributes
    are imported for visualization
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:a039acf5ebb0fe58ff0783f39c7a9453221dc5bcbb186edc82df3c16af193887
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - ChemmineR
  - Cytoscape
  - Meta MapR
  - MetaMapR
  - PubChem Substructure Fingerprints
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# network-edge-list-export

## Summary

Export MetaMapR-calculated metabolic networks (biochemical, structural, spectral, empirical correlation, or combined) as comma-separated value edge lists plus node-attribute tables mapping numeric node IDs to uploaded analytes, ready for import into Cytoscape.

## When to use

Use when MetaMapR network calculations (KEGG RPAIR, Tanimoto >0.7 structural, cosine >0.7 spectral, or Spearman/FDR <0.05 correlation edges) must be persisted as .csv edge lists and node attributes for downstream network analysis.

## When NOT to use

- The metabolite identifiers have not yet been translated to KEGG compound IDs or PubChem CIDs via CTSgetR — run identifier translation first
- The analysis goal is a canonical pathway-based network map; MetaMapR is explicitly pathway independent and its biochemical layer derives only KEGG RPAIR substrate-product pairs
- Raw mass spectral or chromatographic data still require preprocessing — this skill operates on already-computed similarity/correlation edge tables

## Inputs

- Edge lists from the four MetaMapR network modules (biochemical RPAIR, PubChem Tanimoto structural, mass spectral cosine, WGCNA-style Spearman empirical correlation)
- Uploaded analyte/metabolite data matrix whose row order defines numeric node IDs
- Optional custom unique edge-index node IDs

## Outputs

- Combined network edge list as a comma-separated value (.csv) file
- Node attributes .csv mapping numeric node IDs to uploaded analytes (and optionally custom edge-index IDs)
- High-quality graphics or a Cytoscape-importable network

## How to apply

After computing the edge lists in R, assemble the substrate-product pairs (from the >14,000 KEGG RPAIR lookup), Tanimoto >0.7 structural edges, cosine correlation >0.7 spectral edges, and/or FDR-adjusted p <0.05 Spearman dependency edges into a single edge table, retaining only the highest-priority relation per node pair (Biochemical > Structural > Mass Spectral > Correlation) when the 'unique edges' option is selected. Assign numeric node IDs by the row order of the uploaded data matrix by default; a custom unique edge-index ID can be supplied under network > more options. Export the edge list and the node-attribute mapping as .csv files, then import them into Cytoscape or render them in-browser with D3.js/d3Networks via the Shiny interface.

## Related tools

- **MetaMapR** (Computes the four edge-type network and exposes the unique-edges hierarchy, node-ID mapping, and .csv export used by this skill) — https://github.com/dgrapov/MetaMapR
- **R** (Language in which the edge lists are assembled and the .csv edge-list/node-attribute files are written)
- **Shiny** (R package powering the MetaMapR web/desktop UI from which results are exported) — https://github.com/rstudio/shiny-server
- **Cytoscape** (Downstream destination into which the exported edge list and node attributes are imported for visualization)

## Evaluation signals

- The exported edge list is a well-formed .csv that parses cleanly without errors
- Every retained edge in the combined list carries the highest-priority relation type under the hierarchy Biochemical > Structural > Mass Spectral > Correlation
- Numeric node IDs map 1:1 to the row order of the uploaded data matrix (or 1:1 to the supplied custom edge-index IDs)
- Edge weights conform to module-specific bounds, e.g. Tanimoto and cosine scores in (0.7, 1] and all retained correlation edges with FDR-adjusted p <0.05
- Importing the .csv edge list plus node attributes into Cytoscape reproduces the multi-edge network shown in Figure 1 of the paper

## Limitations

- KEGG's limited description of the directionality of biochemical transformations constrains the biochemical edge layer that gets exported
- When 'unique edges' is selected, overlapping multi-type edges between the same pair are collapsed to a single highest-priority relation, so lower-priority (e.g. correlation) edges for the same pair are not exported
- Custom node IDs must be unique; duplicate edge-index values break the 1:1 mapping between the edge list and the uploaded data

## Evidence
