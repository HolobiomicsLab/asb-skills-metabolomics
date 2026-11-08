---
name: molecular-structural-similarity-networking
description: Use when a metabolomic dataset (e.g. GC-TOF-MS feature table with BinBase
  identifiers) contains metabolites with no known biochemical intermediates or missing
  KEGG annotations, producing sparse disconnected biochemical networks that need structural
  edges.
when_to_use_negative:
- All analytes lack structural annotations (no translatable PubChem CID), e.g. fully
  unknown features — use mass spectral (cosine) similarity to relate them instead
- The goal is evidence of direct enzymatic transformation between metabolites — use
  KEGG RPAIR Biochemical Reaction Networks, since structural similarity does not imply
  a substrate-product relationship
- A precomputed structural similarity matrix already exists — recomputing fingerprints
  and Tanimoto scores in MetaMapR is redundant
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaMapR
  role: Computes the Structural Similarity Network from PubChem Substructure Fingerprints
    with Tanimoto thresholding and exports the edge list and node attributes
  repo: https://github.com/dgrapov/MetaMapR
- name: CTSgetR
  role: R interface to the Chemical Translation Service used to map metabolite names/IDs
    to the PubChem CIDs required for structural similarity calculations
  repo: https://github.com/dgrapov/CTSgetR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:ddbbeb37d1a330ca3b2dd8e1a5ae31844eea1e6cbee238c8abf7c92805b3f3d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - MetaMapR
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Molecular structural similarity networking

## Summary

Builds pathway-independent molecular networks by linking metabolites via pairwise Tanimoto similarity of PubChem Substructure Fingerprints, filling gaps between metabolites that lack direct KEGG RPAIR biochemical transformations.

## When to use

Use when a metabolomic dataset (e.g. GC-TOF-MS feature table with BinBase identifiers) contains metabolites with no known biochemical intermediates or missing KEGG annotations, producing sparse disconnected biochemical networks that need structural edges.

## When NOT to use

- All analytes lack structural annotations (no translatable PubChem CID), e.g. fully unknown features — use mass spectral (cosine) similarity to relate them instead
- The goal is evidence of direct enzymatic transformation between metabolites — use KEGG RPAIR Biochemical Reaction Networks, since structural similarity does not imply a substrate-product relationship
- A precomputed structural similarity matrix already exists — recomputing fingerprints and Tanimoto scores in MetaMapR is redundant

## Inputs

- Metabolite identifier table (chemical names or BinBase/database IDs) from a metabolomic experiment, e.g. plasma GC-TOF-MS measurements
- PubChem CIDs obtained by translating identifiers via CTSgetR / the Chemical Translation Service
- PubChem Substructure Fingerprints (bit vectors) for each metabolite with a resolvable CID

## Outputs

- Structural similarity network edge list (pairs of metabolite node IDs with Tanimoto similarity score >0.7), exportable as .csv
- Node attribute table mapping numeric node IDs to uploaded data rows and identifiers, exportable as .csv
- Network graphic exportable as SVG for import into Cytoscape

## How to apply

First translate user metabolite names or identifiers (e.g. BinBase IDs, chemical names) to PubChem CIDs using CTSgetR, the R interface to the Chemical Translation Service, since CIDs are required for chemical similarity calculations. Then compute pairwise Tanimoto similarity between PubChem Substructure Fingerprints (bit vectors) for all metabolite pairs, as implemented in MetaMapR's Structural Similarity Networks module. Accept edges only above the Tanimoto score threshold >0.7, as used in the published T1D NOD mouse analysis. When combining with biochemical, mass spectral, and correlation edges, note that MetaMapR's unique-edges option applies the hierarchy Biochemical > Structural > Mass Spectral > Correlation, so structural edges fill gaps where RPAIR substrate-product pairs are absent. The rationale is that structurally similar metabolites likely share biochemical domain, which enriches networks even for molecules with unknown biochemistry.

## Related tools

- **MetaMapR** (Computes the Structural Similarity Network from PubChem Substructure Fingerprints with Tanimoto thresholding and exports the edge list and node attributes) — https://github.com/dgrapov/MetaMapR
- **CTSgetR** (R interface to the Chemical Translation Service used to map metabolite names/IDs to the PubChem CIDs required for structural similarity calculations) — https://github.com/dgrapov/CTSgetR

## Examples

```
CTSgetR(id = c("alanine","lactic acid"), from = "Chemical Name", to = "PubChem CID", db_name = "ctsgetr.sqlite")
```

## Evaluation signals

- Every retained structural edge has a Tanimoto similarity score above the 0.7 threshold
- Exported edge-list and node-attribute CSVs parse as delimited files and their node identifiers correspond to uploaded metabolite rows (numeric ID 1 = first data row per the README)
- Metabolites lacking KEGG annotations appear in the network only via structural (or spectral/correlation) edges, confirming the unknowns are incorporated
- With the 'unique edges' option enabled, structural edges are reported only where no higher-priority biochemical edge exists, per the Biochemical > Structural > Mass Spectral > Correlation hierarchy
- Identifier translation coverage can be audited: any analyte with no PubChem CID from CTSgetR must show no structural edges

## Limitations

- Structural similarity edges are not evidence of biochemical transformations; they fill gaps where RPAIR reactions are missing but should be interpreted as domain-level association
- Metabolites with unknown structures cannot receive structural edges — the paper extends analysis to unknowns via mass spectral cosine similarity instead
- Structure-based connectivity is limited by PubChem fingerprint/CID resolution and by identifier translation failures in the Chemical Translation Service

## Evidence

- [methods] Structural Similarity Networks are determined based on similarities between Pub Chem Substructure Fingerprints: [evidence span withheld — anchor status: not_found]
