---
name: biochemical-reaction-network-construction
description: Use when metabolite identifiers (names or database IDs such as KEGG,
  PubChem CID, HMDB) must be translated to KEGG compounds and edges are needed between
  metabolites linked by known biochemical transformations in KEGG RPAIR.
when_to_use_negative:
- Measured compounds lack KEGG identifiers or known biochemistry (unknowns) — biochemical
  edges will be absent; combine with structural similarity (Tanimoto >0.7), spectral
  similarity (cosine >0.7), or empirical Spearman correlation edges (FDR <0.05) instead
- The goal is pathway-database-driven mapping of known biochemical domain knowledge
  rather than pathway-independent associations — a tool like Metscape may be more
  appropriate
- Input data are non-metabolomic features (genomic, proteomic, clinical) with no chemical
  identity that can be translated to KEGG compounds
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetaMapR
  role: Shiny/R application that computes the Biochemical Reaction Network from the
    KEGG RPAIR substrate-product lookup table and exports the edge list and node attributes
  repo: https://github.com/dgrapov/MetaMapR
- name: CTSgetR
  role: R interface to the Chemical Translation Service used to translate metabolite
    identifiers to the KEGG compound IDs needed for network calculation
  repo: https://github.com/dgrapov/CTSgetR
- name: Cytoscape
  role: 'Downstream visualization: the exported MetaMapR network is imported into
    Cytoscape and enhanced by mapping empirical and domain-knowledge variables to
    node attributes'
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:a6f1472a31a12c165423239aca4ea1874f3f293996a553a28ec79fc3f14591cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - Cytoscape
  - MetaMapR
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# biochemical-reaction-network-construction

## Summary

Constructs enzymatic (biochemical) reaction networks by mapping metabolites to KEGG compound IDs and connecting those joined by KEGG RPAIR substrate-product pairs, using MetaMapR's curated lookup table built from over 14,000 KEGG RPAIR reactions.

## When to use

Use when metabolite identifiers (names or database IDs such as KEGG, PubChem CID, HMDB) must be translated to KEGG compounds and edges are needed between metabolites linked by known biochemical transformations in KEGG RPAIR.

## When NOT to use

- Measured compounds lack KEGG identifiers or known biochemistry (unknowns) — biochemical edges will be absent; combine with structural similarity (Tanimoto >0.7), spectral similarity (cosine >0.7), or empirical Spearman correlation edges (FDR <0.05) instead
- The goal is pathway-database-driven mapping of known biochemical domain knowledge rather than pathway-independent associations — a tool like Metscape may be more appropriate
- Input data are non-metabolomic features (genomic, proteomic, clinical) with no chemical identity that can be translated to KEGG compounds

## Inputs

- Metabolite identifier list (synonyms or common database identifiers) as a delimited/pasted table (e.g. GC-TOF-MS plasma metabolites with BinBase identifiers for unknowns)
- KEGG RPAIR substrate-product pair reaction database (curated lookup table from over 14,000 reactions)

## Outputs

- Biochemical reaction edge list (substrate-product pairs) as .csv
- Node attributes table (.csv) mapping numeric node IDs to uploaded metabolite rows
- Network graphic exportable as SVG for Cytoscape import

## How to apply

First run Identifier Translation with CTSgetR (the R interface to the Chemical Translation System) to map user metabolite names or identifiers to KEGG compound IDs required for network calculation. Then load the metabolite identifier list as a delimited table into MetaMapR and generate the Biochemical Reaction Network from the KEGG RPAIR substrate-product pair database via the curated lookup table of over 14,000 reactions. Assemble substrate-product relationships into an edge list, assigning numeric node IDs by the row order of the uploaded data by default. Optionally enable the 'unique edges' checkbox so that, where multiple edge types exist, the biochemical edge wins under the hierarchy Biochemical > Structural > Mass Spectral > Correlation. Export the edge list and node attributes as .csv for downstream import into Cytoscape.

## Related tools

- **MetaMapR** (Shiny/R application that computes the Biochemical Reaction Network from the KEGG RPAIR substrate-product lookup table and exports the edge list and node attributes) — https://github.com/dgrapov/MetaMapR
- **CTSgetR** (R interface to the Chemical Translation Service used to translate metabolite identifiers to the KEGG compound IDs needed for network calculation) — https://github.com/dgrapov/CTSgetR
- **Cytoscape** (Downstream visualization: the exported MetaMapR network is imported into Cytoscape and enhanced by mapping empirical and domain-knowledge variables to node attributes)

## Examples

```
library(CTSgetR); init_CTSgetR_db('ctsgetr.sqlite'); CTSgetR(id=c('alanine','lactic acid'), from='Chemical Name', to='KEGG', db_name='ctsgetr.sqlite')
```

## Evaluation signals

- Every emitted edge corresponds to a KEGG RPAIR substrate-product pair retrieved via the curated lookup table
- Each translated metabolite resolves to a valid KEGG identifier through CTS (CTSgetR returns non-NA KEGG IDs, e.g. alanine → C00041)
- Numeric node IDs in the edge list map to the row order of the uploaded data; verify via the node attributes tab
- Exported edge-list and node-attribute files parse as delimited .csv and node identifiers correspond to the uploaded metabolites
- With the unique-edges checkbox enabled, conflicting edge types are resolved by the hierarchy Biochemical > Structural > Mass Spectral > Correlation

## Limitations

- The KEGG database's description of the directionality of biochemical transformations is limited
- Associations cannot be derived biochemically for metabolites with unknown annotations — the tool mitigates this by adding structural, mass spectral, and empirical correlation edges
- Pathway/domain-knowledge-only network mapping yields sparse disconnected graphs; biochemical edges alone may not produce a richly connected representation

## Evidence

- [methods] KEGG RPAIR substrate-product reactions: [evidence span withheld — anchor status: not_found]
- [abstract] associations for unknowns: "leverages KEGG and PubChem databases to derive associations between metabolites even in cases where biochemical domain or molecular annotations are unknown."
- [methods] identifier translation prerequisite: [evidence span withheld — anchor status: not_found]
- [readme] edge-type hierarchy option: "The default setting for returning all edge types or only unique edges can be found under the network tab using the unique edges checkbox."
- [results] KEGG directionality caveat: "Due to the limitation of the KEGG database description of the directionality of biochemical transformations"
