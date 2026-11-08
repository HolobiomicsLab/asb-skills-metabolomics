---
name: metabolomic-network-construction
description: Use when you have a metabolite abundance table (e.g., >470 plasma metabolites
  from GC-TOF-MS) and need to connect analytes — including unknowns — via biochemical,
  structural, spectral, and empirical edges rather than pathway maps.
when_to_use_negative:
- The goal is pathway-map-based visualization (e.g., KEGG pathway overlays) rather
  than pathway-independent association networks.
- Analytes lack KEGG/PubChem identifiers, computable structures, and mass spectra,
  so no biochemical, structural, or spectral edge can be derived and the network would
  be empty or disconnected.
- Input is raw chromatography–mass spectrometry data not yet processed into a peak/abundance
  table with metabolite-level identifiers.
edam_operation: http://edamontology.org/operation_0272
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetaMapR
  role: Primary tool that computes the four edge lists, applies the unique-edges hierarchy,
    assigns numeric node IDs, and exports the merged network
  repo: https://github.com/dgrapov/MetaMapR
- name: CTSgetR
  role: R interface to the Chemical Translation Service used to translate metabolite
    names/identifiers to KEGG or PubChem CIDs before edge calculation
  repo: https://github.com/dgrapov/CTSgetR
- name: R
  role: Implementation language and runtime for MetaMapR (Shiny app) and CTSgetR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:32f749e8edbfdad6da27bc13205e1e64372ee92cbb5fd763013a63d950c608fa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - Meta MapR
  - MetaMapR
  - R
  edam_operation: http://edamontology.org/operation_0272
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# metabolomic-network-construction

## Summary

Construct a pathway-independent metabolomic network by integrating four orthogonal edge types — biochemical (KEGG RPAIR), structural similarity (PubChem Substructure Fingerprints, Tanimoto >0.7), mass spectral similarity (cosine >0.7), and empirical dependency (Spearman correlation, FDR p<0.05) — using MetaMapR, which connects metabolites even when biochemical annotations or structures are unknown.

## When to use

Use when you have a metabolite abundance table (e.g., >470 plasma metabolites from GC-TOF-MS) and need to connect analytes — including unknowns — via biochemical, structural, spectral, and empirical edges rather than pathway maps.

## When NOT to use

- The goal is pathway-map-based visualization (e.g., KEGG pathway overlays) rather than pathway-independent association networks.
- Analytes lack KEGG/PubChem identifiers, computable structures, and mass spectra, so no biochemical, structural, or spectral edge can be derived and the network would be empty or disconnected.
- Input is raw chromatography–mass spectrometry data not yet processed into a peak/abundance table with metabolite-level identifiers.

## Inputs

- metabolite abundance table (rows = analytes, e.g., GC-TOF-MS plasma metabolites)
- metabolite identifiers or chemical names translatable to KEGG/PubChem CIDs
- mass spectral peak lists for spectral similarity calculation
- sample metadata/condition matrix for empirical correlation networks

## Outputs

- combined network edge list (.csv) with typed edges
- node attribute table (.csv) mapping numeric node IDs to identifiers
- interactive network visualization or export for Cytoscape

## How to apply

In MetaMapR (an R/Shiny application), optionally run Identifier Translation via the Chemical Translation System (CTSgetR) to map user metabolite names to KEGG or PubChem CIDs required for biochemical and chemical similarity calculations. Build the four edge lists: biochemical substrate-product pairs from the KEGG RPAIR database; structural similarity from Tanimoto scores on PubChem Substructure Fingerprints thresholded at >0.7; spectral similarity from pairwise mass spectral cosine correlations thresholded at >0.7; and empirical dependency edges from Pearson, biweight, or non-parametric Spearman correlations with FDR-adjusted p<0.05. Under the network tab, choose whether to return all edge types or apply the unique-edges checkbox, which resolves duplicate node pairs by the precedence hierarchy Biochemical > Structural > Mass Spectral > Correlation. Numeric node IDs are assigned by the row order of the uploaded data (a custom unique edge-index ID can be set under network > more options), and results are exported as CSV edge lists and node attributes for rendering with D3.js/d3Networks or import into Cytoscape for attribute mapping.

## Related tools

- **MetaMapR** (Primary tool that computes the four edge lists, applies the unique-edges hierarchy, assigns numeric node IDs, and exports the merged network) — https://github.com/dgrapov/MetaMapR
- **CTSgetR** (R interface to the Chemical Translation Service used to translate metabolite names/identifiers to KEGG or PubChem CIDs before edge calculation) — https://github.com/dgrapov/CTSgetR
- **R** (Implementation language and runtime for MetaMapR (Shiny app) and CTSgetR)

## Evaluation signals

- Every retained edge carries the highest-priority relation type per the Biochemical > Structural > Mass Spectral > Correlation hierarchy when unique edges is selected.
- No structural edge has Tanimoto score ≤0.7 and no spectral edge has cosine correlation ≤0.7, matching the Figures 1B & 1C thresholds.
- Combined edge-list and node-attribute CSVs parse cleanly and numeric node IDs map 1:1 to the row order of the uploaded data.
- The merged multi-edge network reproduces the combination shown in Figure 1 of the paper when the same thresholds are used.
- Identifier translation round-trips a test compound (e.g., alanine → InChIKey QNAYBMKLOCPYGJ-REOHCLBHSA-N) through CTS before network calculation.

## Limitations

- KEGG has limited description of the directionality of biochemical transformations, so biochemical edges may be undirected.
- The unique-edges option discards lower-priority overlapping edge types between the same pair (e.g., a pair with both biochemical and structural edges keeps only the biochemical edge); deselect the checkbox to retain all edges.
- Without identifier translation to KEGG/PubChem CIDs, biochemical and structural similarity edges cannot be computed; mass spectral similarity is the fallback for structurally unknown metabolites.
- Custom edge-index IDs must be unique, otherwise node mapping in the output edge list is ambiguous.

## Evidence

- [readme] edges are returned based on a hierarchy Biochemical > Structural > Mass Spectral > Correlation, otherwise all edges are returned: [evidence span withheld — anchor status: external_doc]
- [methods] KEGG RPAIR biochemical network source: [evidence span withheld — anchor status: not_found]
