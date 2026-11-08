---
name: chemical-similarity-network-edge-filtering
description: 'Use when you need to connect metabolites lacking known biochemical intermediates
  via structural similarity: identifiers (names, KEGG, HMDB, InChIKey) must be converted
  to PubChem CIDs, fingerprints compared by Tanimoto coefficient, and edges filtered
  at a specified cutoff.'
when_to_use_negative:
- When the goal is biochemical transformation networks based on KEGG RPAIR substrate-product
  pairs rather than structural similarity
- When input compounds have no structural annotation and no retrievable PubChem fingerprints
  (mass spectral similarity may be the only extension for unknowns)
- When edges should derive from empirical associations (Pearson, biweight, or Spearman
  correlations) rather than structural similarity
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: MetaMapR
  role: Integrates enzymatic, structural similarity, mass spectral and empirical associations
    into the network; applies the >0.7 Tanimoto threshold and exports edge lists
  repo: https://github.com/dgrapov/MetaMapR
- name: CTSgetR
  role: Translates user metabolite identifiers (chemical names, KEGG, HMDB, InChIKey)
    to PubChem CIDs required for the structural similarity calculation
  repo: https://github.com/dgrapov/CTSgetR
- name: ChemmineR
  role: R package used with the PubChem Power User Gateway (PUG) to retrieve PubChem
    Substructure Fingerprints for each CID
- name: Shiny
  role: R package providing the MetaMapR web/desktop user interface for threshold
    tuning and edge export
  repo: https://github.com/rstudio/shiny-server
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:015ab2e3f314a8dc70051f7b0acef5a3c147f95869d28b230514d65904a4b814
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - Chemical Translation System
  - ChemmineR
  - MetaMapR
  - PubChem Substructure Fingerprints
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chemical-similarity-network-edge-filtering

## Summary

Constructs a metabolite structural similarity network by translating identifiers to PubChem CIDs, retrieving PubChem Substructure Fingerprints, computing pairwise Tanimoto coefficients, and retaining only edges whose Tanimoto score exceeds the >0.7 threshold used by MetaMapR.

## When to use

Use when you need to connect metabolites lacking known biochemical intermediates via structural similarity: identifiers (names, KEGG, HMDB, InChIKey) must be converted to PubChem CIDs, fingerprints compared by Tanimoto coefficient, and edges filtered at a specified cutoff.

## When NOT to use

- When the goal is biochemical transformation networks based on KEGG RPAIR substrate-product pairs rather than structural similarity
- When input compounds have no structural annotation and no retrievable PubChem fingerprints (mass spectral similarity may be the only extension for unknowns)
- When edges should derive from empirical associations (Pearson, biweight, or Spearman correlations) rather than structural similarity

## Inputs

- Metabolite identifier list (CSV or pasted names/IDs such as KEGG, HMDB, or chemical names)
- PubChem CIDs obtained via identifier translation
- PubChem Substructure Fingerprints per CID
- Structural similarity threshold (Tanimoto score >0.7)

## Outputs

- Pairwise Tanimoto coefficient matrix between all compound pairs
- Filtered network edge list (CSV) containing only edges with Tanimoto score in (0.7, 1]
- Cytoscape-importable network with numeric node IDs and node attribute mapping

## How to apply

First translate user metabolite names or identifiers to PubChem CIDs using CTSgetR, the R interface to the Chemical Translation System, which supports >200 biochemical databases. Retrieve PubChem Substructure Fingerprints for each CID (e.g., via ChemmineR with the PubChem PUG) and compute pairwise Tanimoto coefficients between the binary fingerprint bit vectors of all compound pairs, with scores bounded between 0 and 1. Retain only edges whose Tanimoto similarity score exceeds the threshold of >0.7, the value MetaMapR applied for the T1D NOD mouse network. Export the retained edge list as a .csv suitable for import into Cytoscape, noting that numeric node IDs map to rows of the uploaded data by default. Rationale: structural similarity fills gaps between missing biochemical intermediates, yielding richly connected networks even for unknowns.

## Related tools

- **MetaMapR** (Integrates enzymatic, structural similarity, mass spectral and empirical associations into the network; applies the >0.7 Tanimoto threshold and exports edge lists) — https://github.com/dgrapov/MetaMapR
- **CTSgetR** (Translates user metabolite identifiers (chemical names, KEGG, HMDB, InChIKey) to PubChem CIDs required for the structural similarity calculation) — https://github.com/dgrapov/CTSgetR
- **ChemmineR** (R package used with the PubChem Power User Gateway (PUG) to retrieve PubChem Substructure Fingerprints for each CID)
- **Shiny** (R package providing the MetaMapR web/desktop user interface for threshold tuning and edge export) — https://github.com/rstudio/shiny-server

## Examples

```
CTSgetR(id = c("alanine", "lactic acid"), from = "Chemical Name", to = "PubChem CID", db_name = db_name)
```

## Evaluation signals

- Every edge weight in the exported CSV is a Tanimoto score in the range (0.7, 1], consistent with the 0-1 similarity score bounds
- No edge is retained with Tanimoto score <= 0.7, matching the paper's threshold
- Each CIDs-fingerprint step yields a 115-bit (PubChem substructure) binary fingerprint per compound before coefficient computation
- Node IDs in the edge list map 1:1 to rows of the uploaded metabolite data (default ordering) per the MetaMapR README
- Exported edge list parses as a well-formed CSV importable into Cytoscape

## Limitations

- MetaMapR default returns only unique edges based on a hierarchy Biochemical > Structural > Mass Spectral > Correlation unless the unique-edges checkbox is deselected
- Structural similarity requires structural annotation (PubChem fingerprints); fully unknown molecules must be handled via mass spectral similarity instead
- KEGG database limitations on reaction directionality constrain the complementary biochemical transformation network

## Evidence

- [results] Structural similarity threshold: "The network structural similarity threshold was set to Tanimoto score >0.7"
- [methods] Fingerprint basis: [evidence span withheld — anchor status: not_found]
- [methods] Identifier translation: "Translations are accomplished using CTSgetR (https://github.com/dgrapov/CTSgetR)"
- [readme] Node ID mapping: "By default the numeric node IDs are assigned based on the order (row) of the uploaded data."
