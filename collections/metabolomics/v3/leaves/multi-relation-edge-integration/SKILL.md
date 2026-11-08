---
name: multi-relation-edge-integration
description: Use when four metabolite association edge lists (biochemical, structural,
  spectral, correlation) must be combined into a single network and overlapping edges
  between the same metabolite pairs need type-resolved deduplication.
when_to_use_negative:
- You need all relation types retained per metabolite pair — leave the 'unique edges'
  option unchecked instead of applying the deduplication hierarchy
- Your networks are pathway-based metabolic maps rather than pathway-independent association
  networks — MetaMapR was built for pathway-independent analysis where biochemical
  intermediates may be unknown
- Only a single edge type (e.g. biochemical alone) is available — there are no overlapping
  multi-type edges to resolve
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaMapR
  role: Aggregates the four edge lists and applies the unique-edges priority hierarchy
    to the combined network
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Programming language in which MetaMapR is implemented and in which the edge
    lists are loaded and merged
- name: CTSgetR
  role: Translates metabolite identifiers (via Chemical Translation Service) to KEGG
    or PubChem CIDs required before edge lists can be aligned across sources
  repo: https://github.com/dgrapov/CTSgetR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:684bfaa5216e90a735b6e9bd69d25fcaf9bf754f19841a879c35575bdaff07b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - Meta MapR
  - MetaMapR
  - R
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multi-relation-edge-integration

## Summary

Merge MetaMapR's four orthogonal association edge lists — biochemical (KEGG RPAIR), structural (PubChem Substructure Fingerprint Tanimoto >0.7), mass spectral (cosine correlation >0.7), and empirical correlation — into one network edge list, resolving duplicate node-pair edges with the documented priority hierarchy Biochemical > Structural > Mass Spectral > Correlation.

## When to use

Use when four metabolite association edge lists (biochemical, structural, spectral, correlation) must be combined into a single network and overlapping edges between the same metabolite pairs need type-resolved deduplication.

## When NOT to use

- You need all relation types retained per metabolite pair — leave the 'unique edges' option unchecked instead of applying the deduplication hierarchy
- Your networks are pathway-based metabolic maps rather than pathway-independent association networks — MetaMapR was built for pathway-independent analysis where biochemical intermediates may be unknown
- Only a single edge type (e.g. biochemical alone) is available — there are no overlapping multi-type edges to resolve

## Inputs

- Biochemical edge list from KEGG RPAIR substrate-product pairs
- Structural similarity edge list (PubChem Substructure Fingerprints, Tanimoto >0.7)
- Mass spectral similarity edge list (cosine correlation >0.7)
- Empirical dependency edge list (Spearman correlation, FDR-adjusted p <0.05)
- Uploaded analyte data matrix defining node row order

## Outputs

- Combined network edge list (.csv) with one highest-priority relation per node pair
- Node attribute table (.csv) mapping numeric node IDs to uploaded identifiers
- Rendered network (d3.js/d3Networks) or Cytoscape-compatible export

## How to apply

Load the four upstream edge lists produced in R: biochemical substrate-product pairs from KEGG RPAIR (>14,000 curated reactions), structural similarity edges from PubChem Substructure Fingerprints at Tanimoto score >0.7, spectral similarity edges at cosine correlation >0.7, and empirical dependency edges from Spearman correlations with FDR-adjusted p <0.05. Apply the MetaMapR 'unique edges' checkbox so that each node pair retains only the highest-priority relation per the hierarchy Biochemical > Structural > Mass Spectral > Correlation; leaving it unchecked returns all edge types instead. Map numeric node IDs to the row order of the uploaded analyte data (node 1 = first row), or supply a custom unique edge-index ID under network > more options. Export the combined edge list and node attributes as .csv files, optionally rendering with d3.js/d3Networks or exporting to Cytoscape for visualization with variables mapped to node attributes.

## Related tools

- **MetaMapR** (Aggregates the four edge lists and applies the unique-edges priority hierarchy to the combined network) — https://github.com/dgrapov/MetaMapR
- **R** (Programming language in which MetaMapR is implemented and in which the edge lists are loaded and merged)
- **CTSgetR** (Translates metabolite identifiers (via Chemical Translation Service) to KEGG or PubChem CIDs required before edge lists can be aligned across sources) — https://github.com/dgrapov/CTSgetR

## Evaluation signals

- Every retained edge carries the highest-priority relation type per the hierarchy Biochemical > Structural > Mass Spectral > Correlation
- No duplicate node-pair edges remain in the combined edge list when 'unique edges' is selected
- The combined edge list and node attribute CSVs parse cleanly as comma-separated files
- The merged network reproduces the multi-edge combination shown in Figure 1 of the paper
- Numeric node IDs correspond to the row order of the uploaded analyte data (node 1 = first row analyte)

## Limitations

- KEGG RPAIR does not fully describe the directionality of biochemical transformations
- Node IDs are assigned by row order of uploaded data by default; custom IDs must be unique (edge index option)
- The correlation and similarity thresholds (Tanimoto >0.7, cosine >0.7) are tunable; different thresholds change which edges enter the merge

## Evidence

- [readme] edges are returned based on a hierarchy Biochemical > Structural > Mass Spectral > Correlation: [evidence span withheld — anchor status: external_doc]
- [methods] Biochemical Reaction Networks are generated based on the KEGG RPAIR substrate-product pair reaction database: [evidence span withheld — anchor status: not_found]
