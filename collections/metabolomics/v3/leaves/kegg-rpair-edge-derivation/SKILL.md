---
name: kegg-rpair-edge-derivation
description: Use when building biochemical (substrate-product) reaction networks from
  a metabolite identifier list and the identifiers may be common database names or
  synonyms rather than KEGG compound IDs, with no pathway annotation required.
when_to_use_negative:
- The goal is structural similarity edges among unknowns lacking KEGG IDs — use PubChem
  substructure fingerprint (Tanimoto >0.7) or mass spectral (cosine correlation >0.7)
  networks instead
- The input already consists of KEGG compound IDs and identifier translation is unnecessary
  overhead
- Empirical correlation-based dependency edges (Pearson, biweight, or Spearman) are
  the target association measure, not biochemical transformations
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: MetaMapR
  role: Hosts the biochemical reaction network calculation from KEGG RPAIR pairs,
    assigns numeric node IDs by uploaded data row order, and exports the edge list
    and node attributes
  repo: https://github.com/dgrapov/MetaMapR
- name: CTSgetR
  role: Translates user metabolite names/identifiers to KEGG compound IDs via the
    Chemical Translation Service prior to RPAIR edge lookup
  repo: https://github.com/dgrapov/CTSgetR
- name: Chemical Translation System
  role: Backend service enabling metabolite identifier translation between >200 common
    biochemical databases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:490fe6c9a19893f731fc8aef4e94540aedf0695641bad42573f10750afcd06fb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - Chemical Translation System
  - MetaMapR
  - R
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# kegg-rpair-edge-derivation

## Summary

Derives biochemical reaction edges between metabolites by translating user identifiers to KEGG compound IDs via CTSgetR and matching them against a curated lookup table of over 14,000 KEGG RPAIR substrate-product pairs, producing an edge list and node attribute files.

## When to use

Use when building biochemical (substrate-product) reaction networks from a metabolite identifier list and the identifiers may be common database names or synonyms rather than KEGG compound IDs, with no pathway annotation required.

## When NOT to use

- The goal is structural similarity edges among unknowns lacking KEGG IDs — use PubChem substructure fingerprint (Tanimoto >0.7) or mass spectral (cosine correlation >0.7) networks instead
- The input already consists of KEGG compound IDs and identifier translation is unnecessary overhead
- Empirical correlation-based dependency edges (Pearson, biweight, or Spearman) are the target association measure, not biochemical transformations

## Inputs

- Delimited table of metabolite identifiers (synonyms or common database identifiers, e.g. chemical names, HMDB, PubChem CID)
- Curated KEGG RPAIR substrate-product pair lookup table (>14,000 reactions)
- Local CTSgetR sqlite cache database (e.g. ctsgetr.sqlite)

## Outputs

- Biochemical reaction edge list (.csv) of KEGG RPAIR substrate-product edges
- Node attributes file (.csv) mapping identifiers to numeric node IDs
- Translated KEGG compound identifiers per input metabolite

## How to apply

Load the metabolite identifier list (synonyms or common database identifiers) as a delimited table and translate the identifiers to KEGG compound IDs using CTSgetR, the R interface to the Chemical Translation Service, which supports translation between >200 common biochemical databases and caches translations in a local sqlite database. Query the curated lookup table built from over 14,000 KEGG RPAIR substrate-product pair reactions to find biochemical relationships among the translated metabolites. Assemble the substrate-product relationships into an edge list; by default numeric node IDs are assigned by the row order of the uploaded data, with a custom unique edge index available under network > more options. Export the biochemical reaction edge list and node attributes as .csv files, optionally importing them into Cytoscape for visualization.

## Related tools

- **MetaMapR** (Hosts the biochemical reaction network calculation from KEGG RPAIR pairs, assigns numeric node IDs by uploaded data row order, and exports the edge list and node attributes) — https://github.com/dgrapov/MetaMapR
- **CTSgetR** (Translates user metabolite names/identifiers to KEGG compound IDs via the Chemical Translation Service prior to RPAIR edge lookup) — https://github.com/dgrapov/CTSgetR
- **Chemical Translation System** (Backend service enabling metabolite identifier translation between >200 common biochemical databases)

## Examples

```
db_name <- 'ctsgetr.sqlite'; init_CTSgetR_db(db_name); CTSgetR(id = c("alanine", "lactic acid"), from = "Chemical Name", to = "KEGG", db_name = db_name)
```

## Evaluation signals

- Every emitted edge corresponds to a KEGG RPAIR substrate-product pair retrieved via the curated lookup table
- Each translated metabolite resolves to a valid KEGG identifier through CTS (e.g. alanine → C00041, lactic acid → C01432)
- Numeric node IDs in the edge list match the row order of the uploaded data (or the custom edge index supplied under network > more options)
- Edge list and node attributes export cleanly as .csv files and import into Cytoscape with node attributes mapped
- Unmappable identifiers return NA rather than spurious edges (as in the CTSgetR 'foo' example)

## Limitations

- The KEGG database's description of the directionality of biochemical transformations is limited
- Biochemical edges are restricted to metabolites with known KEGG annotations; unknowns must be handled by structural or spectral similarity networks
- Identifier translation fails for names not resolvable through CTS, returning NA for all target identifiers
- CTS API availability is a hard dependency; the service at cts.fiehnlab.ucdavis.edu must be reachable

## Evidence

- [methods] Biochemical Reaction Networks are generated based on the KEGG RPAIR (http://www.genome.jp/kegg/reaction/, ftp://ftp.genome.jp/pub/db/rclass/rpair) substrate-product pair reaction database: [evidence span withheld — anchor status: not_found]
- [readme] CTSgetR provides a consitent interface to translation of chemical names and over 200 database identifiers including InChIKey, HMDB, KEGG and PubChem.: [evidence span withheld — anchor status: not_found]
- [methods] Identifier Translation can be optionally used to map user metabolite names or identifiers to KEGG or Pub Chem CIDs: [evidence span withheld — anchor status: not_found]
