---
name: network-edge-priority-resolution
description: Use when merging multiple MetaMapR edge lists (KEGG RPAIR biochemical,
  PubChem Tanimoto structural, mass spectral cosine, empirical Spearman) into one
  network and duplicate node pairs carry more than one edge type.
when_to_use_negative:
- The analysis needs all association types represented per metabolite pair (e.g. to
  study edge-type overlap itself) — leave the 'unique edges' checkbox unchecked instead.
- Edge lists come from a single association type only, so no duplicate node-pair edges
  exist to resolve.
- Edges lack the four MetaMapR association types (e.g. arbitrary custom relations
  with no defined priority), making the Biochemical > Structural > Mass Spectral >
  Correlation hierarchy inapplicable.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0605
tools:
- name: MetaMapR
  role: Provides the 'unique edges' option that applies the edge-type priority hierarchy
    to the merged network edge list
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Implementation language in which the network calculation and unique-edge resolution
    run
  repo: https://github.com/dgrapov/MetaMapR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:83b8dea87e0b52abb25d7a315b8cdbde28e2fe95a790fb05faed223bd5b0e0a4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - Meta MapR
  - MetaMapR
  - R
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# network-edge-priority-resolution

## Summary

Resolves overlapping multi-type edges (biochemical, structural, mass spectral, correlation) between the same metabolite pairs in MetaMapR by returning only the edge from the highest-priority association type, using the fixed hierarchy Biochemical > Structural > Mass Spectral > Correlation.

## When to use

Use when merging multiple MetaMapR edge lists (KEGG RPAIR biochemical, PubChem Tanimoto structural, mass spectral cosine, empirical Spearman) into one network and duplicate node pairs carry more than one edge type.

## When NOT to use

- The analysis needs all association types represented per metabolite pair (e.g. to study edge-type overlap itself) — leave the 'unique edges' checkbox unchecked instead.
- Edge lists come from a single association type only, so no duplicate node-pair edges exist to resolve.
- Edges lack the four MetaMapR association types (e.g. arbitrary custom relations with no defined priority), making the Biochemical > Structural > Mass Spectral > Correlation hierarchy inapplicable.

## Inputs

- Biochemical edge list (KEGG RPAIR substrate-product pairs)
- Structural similarity edge list (Tanimoto score >0.7 on PubChem Substructure Fingerprints)
- Mass spectral similarity edge list (cosine correlation >0.7)
- Empirical dependency edge list (Spearman correlations, FDR-adjusted p <0.05)
- Uploaded analyte data table (row order defines numeric node IDs, or custom edge-index IDs)

## Outputs

- Combined network edge list where each node pair carries only the highest-priority edge type
- Node attribute CSV mapping numeric node IDs to analyte identifiers
- Optional D3.js/d3Networks or Cytoscape rendering of the merged network

## How to apply

Load the four upstream edge lists — biochemical substrate-product pairs from KEGG RPAIR, structural similarity edges (Tanimoto score >0.7 on PubChem Substructure Fingerprints), spectral similarity edges (cosine correlation >0.7), and empirical dependency edges (e.g. Spearman correlations with FDR-adjusted p <0.05) — into MetaMapR or its R implementation. In the network tab, enable the 'unique edges' checkbox so that, per the documented precedence hierarchy Biochemical > Structural > Mass Spectral > Correlation, only the highest-priority relation is retained for each metabolite pair; leaving it unchecked returns all edge types. Node IDs are numeric and assigned by row order of the uploaded analyte data (node attributes tab shows the mapping), or a custom unique edge-index ID can be supplied under network > more options. Export the resulting edge list and node attributes as .csv, optionally rendering with D3.js/d3Networks or exporting to Cytoscape for further attribute mapping.

## Related tools

- **MetaMapR** (Provides the 'unique edges' option that applies the edge-type priority hierarchy to the merged network edge list) — https://github.com/dgrapov/MetaMapR
- **R** (Implementation language in which the network calculation and unique-edge resolution run) — https://github.com/dgrapov/MetaMapR

## Evaluation signals

- Every retained edge in the output carries the relation type expected from the hierarchy Biochemical > Structural > Mass Spectral > Correlation (no lower-priority duplicate survives when a higher-priority edge exists for the same pair).
- No node pair in the merged edge list appears more than once.
- The combined edge list and node attribute CSVs parse cleanly and node IDs map to the row order of the uploaded analyte data.
- The merged network reproduces the multi-edge combination shown in Figure 1 of the paper.

## Limitations

- The priority hierarchy is fixed (Biochemical > Structural > Mass Spectral > Correlation); it cannot be re-ordered by the user.
- Unique-edge resolution discards the lower-priority edges entirely, losing information about multi-type associations.
- Custom edge-index IDs must be unique, and KEGG's limited description of transformation directionality constrains the biochemical edges that feed this resolution.

## Evidence
