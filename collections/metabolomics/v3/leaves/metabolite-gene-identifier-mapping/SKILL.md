---
name: metabolite-gene-identifier-mapping
description: Use when a user supplies a list of metabolite and/or gene identifiers
  of mixed origin (hmdb, kegg, ensembl, uniprot, CAS, pubchem, chebi...) that must
  be translated to RaMP pathway IDs before Fisher enrichment.
when_to_use_negative:
- The input is already a set of RaMP pathway IDs or an enriched pathway table — mapping
  is only needed for raw analyte lists
- Only chemical-class or ontology annotations (not pathway membership) are needed
  from RaMP-DB 3.0; use the dedicated query functions instead
- The MySQL server/connection parameters (dbname, username, conpass, host) are unavailable
  and no cached SQLite version exists — RaMP cannot be queried offline without one
  of these
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3345
tools:
- name: RaMP
  role: Provides the relational database and query layer (getPathwayFromAnalyte, getPrefixesFromAnalytes)
    used to translate analyte IDs into pathway identifiers
  repo: https://github.com/ncats/RaMP-DB/
- name: R
  role: Runtime language for the RaMP package (RaMP() constructor, getPathwayFromAnalyte,
    getPrefixesFromAnalytes)
  repo: https://github.com/Mathelab/RaMP-DB
- name: MySQL
  role: Legacy backend for the original RaMP MySQL dump queried via RMySQL/DBI
- name: BiocFileCache
  role: Manages the local cache of the RaMP-DB 3.0 SQLite database downloaded on first
    RaMP() call
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:e3118655c510c6daddd97ad31cbb5da7460729229845de23031028502f6026b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncats/RaMP-DB
  tools:
  - BiocFileCache
  - MySQL
  - R
  - RMySQL
  - RaMP
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# metabolite-gene-identifier-mapping

## Summary

Maps user-supplied gene and metabolite analyte lists (e.g. hmdb:HMDB04824, kegg:C02712, ensembl:ENSG...) onto RaMP's integrated KEGG/Reactome/WikiPathways/HMDB pathway identifiers, with optional synonym resolution, as the prerequisite step for pathway overrepresentation analysis.

## When to use

Use when a user supplies a list of metabolite and/or gene identifiers of mixed origin (hmdb, kegg, ensembl, uniprot, CAS, pubchem, chebi...) that must be translated to RaMP pathway IDs before Fisher enrichment.

## When NOT to use

- The input is already a set of RaMP pathway IDs or an enriched pathway table — mapping is only needed for raw analyte lists
- Only chemical-class or ontology annotations (not pathway membership) are needed from RaMP-DB 3.0; use the dedicated query functions instead
- The MySQL server/connection parameters (dbname, username, conpass, host) are unavailable and no cached SQLite version exists — RaMP cannot be queried offline without one of these

## Inputs

- List of metabolite and/or gene identifiers or names, with database-of-origin prefixes (e.g. hmdb:HMDB04824, kegg:C02712, CAS:2566-39-4, ensembl:…, uniprot:…)
- RaMP database connection: legacy MySQL (connectToRaPM) or RaMP-DB 3.0 SQLite object (rampDB <- RaMP())
- Pathway source set: KEGG, Reactome, WikiPathways, HMDB (RaMP-DB 3.0 also adds Lipid-MAPS, CheBI, PFOCR, RefMet, Rhea)

## Outputs

- Data frame of analyte-to-pathway mappings (RaMP pathway identifiers per analyte, with synonym-resolved hits flagged)
- Per-analyte-type counts of mapped analytes (total_metabolites, total_genes) used downstream by Fisher enrichment
- Optionally, the set of represented ID prefixes per analyte type (RaMP::getPrefixesFromAnalytes)

## How to apply

1. Instantiate the database connection (legacy: connectToRaPM(dbname, username, conpass, host) backed by RMySQL/DBI against the local RaMP MySQL dump; RaMP-DB 3.0: rampDB <- RaMP(), which downloads and caches the SQLite database via BiocFileCache). 2. Ensure every input ID is prepended with its database of origin and the colon is included (e.g. hmdb:HMDB04824, kegg:C02712, CAS:2566-39-4, ensembl:ENSG…, uniprot:P…); mixed-source lists are allowed. 3. Call RaMP::getPrefixesFromAnalytes(db = rampDB, analyteType = 'metabolite' or 'gene') to enumerate the currently represented prefixes (metabolites: hmdb, pubchem, chebi, chemspider, kegg, CAS, LIPIDMAPS, swisslipids, lipidbank, wikidata, plantfa, kegg_glycan, refmet, rhea-comp; genes: ensembl, entrez, gene_symbol, uniprot, hmdb, ncbiprotein, EN, wikidata, chebi, brenda). 4. Map analytes to pathways with getPathwayFromAnalyte(analytes, find_synonym, conpass, host, dbname, username, NameOrIds), setting find_synonym to enable synonym resolution so near-miss or alternate names are still matched; NameOrIds controls whether inputs are names or identifiers. 5. Inspect the returned pathwaydf for unmapped analytes and confirm analyte_type counts (total_metabolites/total_genes) before feeding the mapping into runFisherTest/runCombinedFisherTest, since pathway coverage directly determines the 2x2 contingency tables and downstream p-values.

## Related tools

- **RaMP** (Provides the relational database and query layer (getPathwayFromAnalyte, getPrefixesFromAnalytes) used to translate analyte IDs into pathway identifiers) — https://github.com/ncats/RaMP-DB/
- **R** (Runtime language for the RaMP package (RaMP() constructor, getPathwayFromAnalyte, getPrefixesFromAnalytes)) — https://github.com/Mathelab/RaMP-DB
- **MySQL** (Legacy backend for the original RaMP MySQL dump queried via RMySQL/DBI)
- **BiocFileCache** (Manages the local cache of the RaMP-DB 3.0 SQLite database downloaded on first RaMP() call)

## Examples

```
rampDB <- RaMP(); mapped <- RaMP::getPathwayFromAnalyte(analytes=c('hmdb:HMDB04824','kegg:C02712'), find_synonym=TRUE, db=rampDB)
```

## Evaluation signals

- Every input ID carries a recognized database-of-origin prefix (verified via RaMP::getPrefixesFromAnalytes) and includes the colon separator
- The returned pathwaydf contains at least one pathway hit for the majority of submitted analytes; unmapped analytes are explicitly identified and excluded from total_metabolites/total_genes counts
- Synonym resolution (find_synonym = TRUE) recovers additional hits without introducing false-positive cross-species or cross-class matches
- Downstream Fisher tables use the same analyte counts as the mapping step — total_metabolites/total_genes match the number of uniquely mapped analytes of each type
- RaMP-DB 3.0 lists >250,000 metabolites and ~34,000 genes/proteins with >122,000 pathways; a mapping covering an order of magnitude fewer unique analytes than expected from the input list signals ID-format or prefix errors

## Limitations

- The legacy Mathelab/RaMP-DB repo is no longer maintained — users must migrate to ncats/RaMP-DB
- In legacy RaMP the MySQL dump must be imported manually (empty MySQL DB, e.g. 'ramp', populated via the RaMP-BackEnd scripts and Program.cs bulk import) before any query works
- RaMP-DB 3.0 requires BiocFileCache; if it is not pre-installed, RaMP-DB installation fails
- Annotate IDs must carry their source prefix — unprefixed IDs are not recognized, and the list of supported prefixes differs between metabolites and genes/proteins
- RaMP's annotation sources are updated multiple times a year, so mapping results may differ between database versions (e.g. specify version = '2.5.4' for reproducibility)

## Evidence

- [readme] RaMP-DB 3.0 prefix requirements: [evidence span withheld — anchor status: external_doc]
- [methods] getPathwayFromAnalyte signature with synonym resolution: "getPathwayFromAnalyte(analytes, find_synonym, conpass, host, dbname, username, NameOrIds)"
- [readme] RaMP-DB 3.0 SQLite caching via BiocFileCache: "rampDB <- RaMP()

# initializes the RaMP database object, downloading and caching the latest SQLite database"
- [readme] RaMP-DB 3.0 annotation coverage: [evidence span withheld — anchor status: external_doc]
- [abstract] Database update cadence affecting mapping reproducibility: "Updates for databases in RaMP will be checked multiple times a year and RaMP will be updated accordingly."
- [readme] Legacy repo deprecation notice: "This repo is no longer maintained. Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
