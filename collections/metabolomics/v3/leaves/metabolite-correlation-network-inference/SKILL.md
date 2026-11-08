---
name: metabolite-correlation-network-inference
description: Use when you have a samples-by-metabolites data matrix and want pairwise
  metabolite–metabolite dependency edges (including for unknowns without KEGG/PubChem
  annotations), accepting edges by a corrected significance threshold.
when_to_use_negative:
- The input is only a list of metabolite identifiers with no replicated sample-wise
  measurements — empirical correlations cannot be computed, so rely on biochemical
  (KEGG RPAIR), structural (PubChem fingerprint), or spectral similarity networks
  instead.
- The goal is to capture direct biochemical transformations rather than co-measurement
  (co-variation) associations — correlation edges do not encode reaction directionality
  and rank lowest in MetaMapR's unique-edge hierarchy (Biochemical > Structural >
  Mass Spectral > Correlation).
- The analytes are non-metabolomic and lack paired sample profiles consistent with
  the abundance matrix — the correlation step presupposes measurements across the
  same samples for every analyte.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3395
tools:
- name: MetaMapR
  role: Shiny/R application that computes the empirical dependency networks (Pearson,
    biweight, or Spearman correlations with FDR-based edge acceptance) and exports
    edge lists and node attributes
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Implementation language for the correlation computation and CSV export; MetaMapR
    and its Shiny interface are written in R
- name: CTSgetR
  role: Optional pre-step translating chemical names or identifiers to KEGG/PubChem
    CIDs (with sqlite caching) so correlation-derived nodes can later be joined to
    biochemical and structural similarity networks
  repo: https://github.com/dgrapov/CTSgetR
- name: Shiny / shiny-server
  role: Hosts MetaMapR as a standalone web application for interactive threshold tuning
    and network visualization
  repo: https://github.com/rstudio/shiny-server
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:0b8cebb2ea9cb0b8a41007907a690fe51dd0f12762ae60621bcb9db86899065a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - MetaMapR
  - R
  - Shiny / shiny-server
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3395
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolite-correlation-network-inference

## Summary

Builds empirical dependency networks between metabolites from an abundance matrix (concentrations or peak intensities across samples) using WGCNA-style pairwise correlations, so that compounds lacking biochemical or structural annotations (unknowns) can still be connected via co-measurement associations.

## When to use

Use when you have a samples-by-metabolites data matrix and want pairwise metabolite–metabolite dependency edges (including for unknowns without KEGG/PubChem annotations), accepting edges by a corrected significance threshold.

## When NOT to use

- The input is only a list of metabolite identifiers with no replicated sample-wise measurements — empirical correlations cannot be computed, so rely on biochemical (KEGG RPAIR), structural (PubChem fingerprint), or spectral similarity networks instead.
- The goal is to capture direct biochemical transformations rather than co-measurement (co-variation) associations — correlation edges do not encode reaction directionality and rank lowest in MetaMapR's unique-edge hierarchy (Biochemical > Structural > Mass Spectral > Correlation).
- The analytes are non-metabolomic and lack paired sample profiles consistent with the abundance matrix — the correlation step presupposes measurements across the same samples for every analyte.

## Inputs

- Metabolite abundance data matrix (concentrations or GC-TOF-MS peak intensities) as a delimited table (e.g. CSV) with metabolites as rows and samples as columns
- Optional: metabolite name or identifier list (KEGG, PubChem CID, HMDB, InChIKey) for identifier translation via CTSgetR
- Significance threshold parameter (FDR-adjusted p-value cutoff, e.g. 0.05)

## Outputs

- dependency_network_edges.csv — accepted edge list of metabolite pairs (FDR-adjusted p-value < 0.05)
- Node attributes table mapping numeric node IDs 1:1 to the row order of the uploaded data matrix (exportable CSV)
- Network object importable into Cytoscape for visualization with node attributes mapped

## How to apply

Load the measured metabolite concentration/peak-intensity matrix as a delimited table in R (MetaMapR context), then compute all pairwise metabolite–metabolite dependencies with non-parametric Spearman correlation (robust to outliers and monotonic relationships); MetaMapR's empirical dependency networks per Langfelder and Horvath (2008) also support parametric Pearson and biweight correlations, so choose Spearman when outliers or non-Gaussian intensity distributions are expected. Compute p-values for every pair and apply false discovery rate (FDR) correction across all pairs, then accept an edge only if its adjusted p-value passes the significance threshold of FDR p-value < 0.05. Export the retained edge list plus node attributes as CSV; in MetaMapR, numeric node IDs are assigned by row order of the uploaded matrix, and a unique custom edge index can be set under network > more options. If merging with biochemical/structural/spectral edges, note MetaMapR's unique-edge hierarchy Biochemical > Structural > Mass Spectral > Correlation (toggled via the 'unique edges' checkbox); the Chemical Translation System (CTSgetR) can optionally translate metabolite names to KEGG/PubChem CIDs first, but correlation edges require no identifiers.

## Related tools

- **MetaMapR** (Shiny/R application that computes the empirical dependency networks (Pearson, biweight, or Spearman correlations with FDR-based edge acceptance) and exports edge lists and node attributes) — https://github.com/dgrapov/MetaMapR
- **R** (Implementation language for the correlation computation and CSV export; MetaMapR and its Shiny interface are written in R)
- **CTSgetR** (Optional pre-step translating chemical names or identifiers to KEGG/PubChem CIDs (with sqlite caching) so correlation-derived nodes can later be joined to biochemical and structural similarity networks) — https://github.com/dgrapov/CTSgetR
- **Shiny / shiny-server** (Hosts MetaMapR as a standalone web application for interactive threshold tuning and network visualization) — https://github.com/rstudio/shiny-server

## Evaluation signals

- Verify every retained edge in dependency_network_edges.csv corresponds to a metabolite pair whose FDR-adjusted p-value passes the edge-acceptance criterion (< 0.05).
- Verify numeric node IDs in the edge list map 1:1 to the row order of the uploaded data matrix (node attribute table shows the identifier-to-node-ID mapping).
- Check edge-list schema: exported files are comma-separated value tables with an accepted edge list plus a node attributes mapping, importable into Cytoscape.
- Recompute a sample of Spearman correlations and FDR-adjusted p-values independently in R and confirm the accepted-edge set matches MetaMapR's output for the same threshold.
- If unique edges were requested, confirm no edge carries more than one type and that correlation-type edges were suppressed wherever a biochemical, structural, or mass spectral edge existed for the same pair.

## Limitations

- Correlation edges encode monotonic co-variation across samples, not biochemical transformation or causality; they rank lowest in MetaMapR's unique-edge hierarchy and are superseded by reaction, structural, or spectral edges.
- The choice of correlation measure matters: parametric Pearson and biweight versus non-parametric Spearman yield different dependency networks; Spearman is preferred under outliers but the article's KEGG-derived layers cannot fix directionality of transformations.
- Edges depend on adequate sample replication; unknowns without structural annotation can only connect through mass spectral similarity or empirical correlation layers, so sparse spectra yield sparse correlation-free coverage.
- Multiple edge types are returned by default (all edge types checkbox off/on) — forgetting the unique-edges setting inflates edge counts with redundant pairwise relations.

## Evidence
