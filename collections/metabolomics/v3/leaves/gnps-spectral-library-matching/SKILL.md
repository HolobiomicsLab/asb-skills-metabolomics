---
name: gnps-spectral-library-matching
description: Use when curating a BGC repository of known-function entries and you
  need to establish link-outs to chemical structure or spectral databases (PubChem,
  Natural Products Atlas, GNPS) based on compound name/structure correspondence.
when_to_use_negative:
- The BGC entries lack compound names or chemical structure annotations — matching
  is name/structure-based, so nothing can be cross-referenced
- The goal is de novo identification of metabolites from spectra rather than establishing
  repository link-outs for entries of known function
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: MIBiG Repository
  role: Source of the annotated BGC entries whose compound names and structures are
    cross-referenced against external databases
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the JSON-formatted MIBiG data standard and SQL schema used as input
    for cross-link extraction
  repo: https://github.com/mibig-secmet
- name: PostgreSQL
  role: Relational database underlying the MIBiG API from which the relational export
    of annotations can be loaded
  repo: https://www.postgresql.org/
- name: JSON schema
  role: Validation technology governing the MIBiG JSON annotation records that supply
    the compound name and structure fields
  repo: https://json-schema.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:af4fd40d607ee26728fde868343716988b7d8e853c7905518a7f424a22d48fc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - MIBiG Repository
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gnps-spectral-library-matching

## Summary

Cross-link annotated MIBiG BGC entries to external chemical databases (Natural Products Atlas, GNPS spectral library) by matching compound names and chemical structures, yielding per-accession linkage inventory.

## When to use

Use when curating a BGC repository of known-function entries and you need to establish link-outs to chemical structure or spectral databases (PubChem, Natural Products Atlas, GNPS) based on compound name/structure correspondence.

## When NOT to use

- The BGC entries lack compound names or chemical structure annotations — matching is name/structure-based, so nothing can be cross-referenced
- The goal is de novo identification of metabolites from spectra rather than establishing repository link-outs for entries of known function

## Inputs

- MIBiG 2.0 annotation records (JSON-formatted data standard from GitHub)
- MIBiG relational database export (PostgreSQL)
- Compound name and chemical structure fields per BGC entry

## Outputs

- crosslinks.csv — one row per MIBiG accession with link-presence flags for PubChem, Natural Products Atlas, and GNPS

## How to apply

Load the MIBiG 2.0 annotation records (JSON files from GitHub or the relational export from the underlying PostgreSQL database). For each BGC entry, extract the compound name and chemical structure fields used for cross-referencing. Match compound names and structures against external resources — the Natural Products Atlas (https://www.npatlas.org/), the GNPS spectral library, and PubChem — recording which databases carry a link for each MIBiG accession. Tabulate the result as one row per entry with link-presence flags per database (crosslinks.csv). Rationale: MIBiG 2.0 added direct link-outs to chemical structure databases as a repository feature, and connections are made according to compound names and structures matching between the annotated BGCs and the chemical databases.

## Related tools

- **MIBiG Repository** (Source of the annotated BGC entries whose compound names and structures are cross-referenced against external databases) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the JSON-formatted MIBiG data standard and SQL schema used as input for cross-link extraction) — https://github.com/mibig-secmet
- **PostgreSQL** (Relational database underlying the MIBiG API from which the relational export of annotations can be loaded) — https://www.postgresql.org/
- **JSON schema** (Validation technology governing the MIBiG JSON annotation records that supply the compound name and structure fields) — https://json-schema.org

## Examples

```
psql -d mibig -c "SELECT bgc_id, compound_name FROM mibig_annotations" > compounds.tsv && python scripts/match_crosslinks.py --input compounds.tsv --databases pubchem,npatlas,gnps --output crosslinks.csv
```

## Evaluation signals

- Output table lists one row per MIBiG entry with a field indicating link presence for each of PubChem, Natural Products Atlas, and GNPS
- Counts of linked entries are consistent with the MIBiG 2.0 statement that direct link-outs to chemical structure databases were added as a repository feature
- Matched links correspond to compound names and structures that actually agree between the annotated BGCs and the external databases
- Cross-link target for Natural Products Atlas resolves to https://www.npatlas.org/ as cited in MIBiG 2.0

## Limitations

- Matching relies on compound names and structure agreement, so entries with missing, mistyped, or inconsistent compound annotations will not link correctly
- Only databases with established cross-links in MIBiG 2.0 (Natural Products Atlas, GNPS, and chemical structure databases) are covered; other resources require new curation

## Evidence

- [intro] cross-links to Natural Products Atlas: "additional cross-links have been established with the Natural Products Atlas (https://www.npatlas.org/)"
- [abstract] link-outs as repository feature: [evidence span withheld — anchor status: not_found]
- [intro] data source for matching: "we performed extensive re-annotation of the entire database, increasing the overall data quality"
- [supplementary] data availability: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
- [intro] database backend: "handles access to the underlying PostgreSQL (https://www.postgresql.org/) database"
- [supplementary] repository availability: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
