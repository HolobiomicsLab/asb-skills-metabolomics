---
name: pubchem-structure-linkage
description: Use when a curated biosynthetic gene cluster repository or compound annotation
  table needs link-outs to chemical structure or spectral databases, and compound
  name/structure fields are available for matching.
when_to_use_negative:
- The dataset contains only predicted/novel BGCs without compound names or chemical
  structures — MIBiG cross-links require molecules of known chemical structure
- The goal is de novo structure elucidation rather than linking already-annotated
  compounds to external databases
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: MIBiG Repository
  role: Source of annotated BGC entries with compound names, structures, and the cross-link
    feature being inventoried
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the JSON-formatted MIBiG data standard and SQL schema from which annotation
    records are loaded
  repo: https://github.com/mibig-secmet
- name: PostgreSQL
  role: Relational database backend of the MIBiG repository accessed via the REST-like
    web API to extract entry-level compound fields
  repo: https://www.postgresql.org/
- name: JSON schema
  role: Validation technology ensuring MIBiG annotation entries conform to the schema
    before cross-referencing
  repo: https://json-schema.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:6fcfefffec3abafafb1ddfef63f1cd8540dc4a55452aa622f2f18fba1f64a1c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - MIBiG Repository
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pubchem-structure-linkage

## Summary

Method for establishing cross-links between annotated MIBiG BGC entries and external chemical/analytical databases (Natural Products Atlas, GNPS spectral library, PubChem) by matching compound names and chemical structures, producing a per-accession linkage inventory.

## When to use

Use when a curated biosynthetic gene cluster repository or compound annotation table needs link-outs to chemical structure or spectral databases, and compound name/structure fields are available for matching.

## When NOT to use

- The dataset contains only predicted/novel BGCs without compound names or chemical structures — MIBiG cross-links require molecules of known chemical structure
- The goal is de novo structure elucidation rather than linking already-annotated compounds to external databases

## Inputs

- MIBiG 2.0 annotation records (JSON schema-compliant entries from GitHub or PostgreSQL relational export)
- Compound name and chemical structure fields per BGC entry
- External database lookup endpoints: Natural Products Atlas (https://www.npatlas.org/) and GNPS spectral library

## Outputs

- crosslinks.csv — one row per MIBiG accession with per-database link presence flags (PubChem, Natural Products Atlas, GNPS)
- Counts of linked entries per external database

## How to apply

Load the MIBiG 2.0 annotation records from the repository data (JSON-formatted data standard on GitHub, or the PostgreSQL backend via the REST-like mibig-api). For each BGC accession, extract the compound name and chemical structure fields used for cross-referencing — these were populated during MIBiG 2.0's extensive manual re-annotation that added chemical structures to entries. Match compound names and structures against the Natural Products Atlas (npatlas.org) and the GNPS spectral library, as MIBiG 2.0 established cross-links based on compound names and structures matching between annotated BGCs and the chemical databases. Tabulate, per MIBiG accession, which external databases carry a link for that entry, and write the inventory (e.g. crosslinks.csv) with one row per entry and a field indicating link presence per database. Validate by checking that linked-entry counts are consistent with MIBiG 2.0's feature statement that direct link-outs to chemical structure databases were added to the repository.

## Related tools

- **MIBiG Repository** (Source of annotated BGC entries with compound names, structures, and the cross-link feature being inventoried) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the JSON-formatted MIBiG data standard and SQL schema from which annotation records are loaded) — https://github.com/mibig-secmet
- **PostgreSQL** (Relational database backend of the MIBiG repository accessed via the REST-like web API to extract entry-level compound fields) — https://www.postgresql.org/
- **JSON schema** (Validation technology ensuring MIBiG annotation entries conform to the schema before cross-referencing) — https://json-schema.org

## Examples

```
python scripts/build_crosslinks.py --mibig-json data/mibig_json/ --databases npatlas,gnps --output crosslinks.csv
```

## Evaluation signals

- crosslinks.csv has exactly one row per MIBiG entry with a link-presence field for each of PubChem, Natural Products Atlas and GNPS
- Counts of linked entries per database are consistent with the MIBiG 2.0 statement that direct link-outs to chemical structure databases were added as a repository feature
- Every claimed cross-link resolves to a matching compound name or structure in the target database (Natural Products Atlas / GNPS)
- Input MIBiG entries pass JSON schema validation before matching (schema compliance was a MIBiG 2.0 design goal)
- BGC accessions in the output table are a subset of the loaded MIBiG 2.0 entry set (2021 curated BGCs at release)

## Limitations

- Cross-links depend on exact compound name/structure matching; entries with empty or mistyped information (a problem MIBiG 2.0 re-annotation had to fix) will fail to link
- Cross-link coverage is limited to databases MIBiG 2.0 explicitly established links with (Natural Products Atlas, GNPS); PubChem coverage must be assessed separately and may not exist per entry
- Duplicate entries or retired low-quality entries can distort linkage counts if not removed first

## Evidence

- [intro] Cross-links established via compound name/structure matching: "additional cross-links have been established with the Natural Products Atlas (https://www.npatlas.org/)"
- [intro] Cross-links also to GNPS: "cross-links have been established with the Natural Products Atlas"
- [abstract] Link-outs as a repository feature: "we improved the user experience by adding new features such as query searches and a statistics page, and"
- [abstract] Manual curation added structures: "Over the past five years, 851 new BGCs have been added. Additionally, we performed extensive manual data curation of all entries to improve the annotation quality"
- [intro] API access to PostgreSQL backend: "A REST-like web API (https://github.com/mibig-secmet/mibig-api/) handles access to the underlying PostgreSQL"
