---
name: natural-products-atlas-identifier-matching
description: Use when annotated MIBiG BGC entries need cross-references to chemical
  structure/spectral databases (Natural Products Atlas, GNPS, PubChem) based on matching
  compound names and structures.
when_to_use_negative:
- The task is de novo identification of compounds from spectra or sequences — this
  skill only cross-references already-annotated MIBiG compound names/structures against
  existing databases.
- The target clusters are unannotated candidate BGCs (e.g. from IMG-ABC or antiSMASH-DB)
  without compound name or structure fields to match on.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0157
tools:
- name: MIBiG Repository
  role: Source of annotated BGC records with compound name and structure fields to
    cross-reference
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the JSON-formatted MIBiG data standard that is parsed for compound names
    and structures
  repo: https://github.com/mibig-secmet
- name: PostgreSQL
  role: Relational database behind the mibig-api REST endpoint used to access MIBiG
    entries
  repo: https://www.postgresql.org/
- name: JSON schema
  role: Validates the MIBiG annotation records before cross-link extraction
  repo: https://json-schema.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:98ef1bf73e5eba198698f238ee8bc86120bbf31cd7a5ea147b57f43d631e134b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - MIBiG Repository
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# natural-products-atlas-identifier-matching

## Summary

Matches compound names and chemical structures annotated in MIBiG BGC records against the Natural Products Atlas (with GNPS and PubChem as companion targets) to establish cross-links and produce a per-entry linkage inventory.

## When to use

Use when annotated MIBiG BGC entries need cross-references to chemical structure/spectral databases (Natural Products Atlas, GNPS, PubChem) based on matching compound names and structures.

## When NOT to use

- The task is de novo identification of compounds from spectra or sequences — this skill only cross-references already-annotated MIBiG compound names/structures against existing databases.
- The target clusters are unannotated candidate BGCs (e.g. from IMG-ABC or antiSMASH-DB) without compound name or structure fields to match on.

## Inputs

- MIBiG 2.0 annotation records (JSON files from the GitHub mibig-secmet repository or the PostgreSQL relational export)
- Natural Products Atlas entries from https://www.npatlas.org/
- GNPS spectral library entries
- PubChem compound records

## Outputs

- crosslinks.csv: one row per MIBiG accession with fields indicating link presence for PubChem, Natural Products Atlas, and GNPS
- Counts of linked entries per external database

## How to apply

Load the MIBiG 2.0 annotation records (JSON entries from the GitHub-hosted data standard, or rows via the PostgreSQL-backed mibig-api REST endpoint), and for each BGC accession extract the compound name and chemical structure fields used for cross-referencing. Match these names/structures against the Natural Products Atlas at npatlas.org, the GNPS spectral library, and PubChem to determine which external databases carry a link for that entry. Tabulate, per MIBiG accession, link presence for each of the three databases and write the inventory as crosslinks.csv. The rationale is that MIBiG 2.0 added direct link-outs to chemical structure databases as a repository feature, with connections made according to compound name and structure matching between annotated BGCs and the chemical databases; the resulting inventory counts should be consistent with that stated feature.

## Related tools

- **MIBiG Repository** (Source of annotated BGC records with compound name and structure fields to cross-reference) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the JSON-formatted MIBiG data standard that is parsed for compound names and structures) — https://github.com/mibig-secmet
- **PostgreSQL** (Relational database behind the mibig-api REST endpoint used to access MIBiG entries) — https://www.postgresql.org/
- **JSON schema** (Validates the MIBiG annotation records before cross-link extraction) — https://json-schema.org

## Examples

```
python -c "import json,glob,csv; rows=[{'acc':p,'npa':bool(json.load(open(p)).get('compounds'))} for p in glob.glob('mibig_json/*.json')]; csv.DictWriter(open('crosslinks.csv','w'),['acc','npa']).writerows(rows)"
```

## Evaluation signals

- crosslinks.csv contains exactly one row per MIBiG entry with a link-presence field for each of PubChem, Natural Products Atlas and GNPS
- Counts of linked entries are consistent with the MIBiG 2.0 statement that direct link-outs to chemical structure databases were added as a repository feature
- Every matched Natural Products Atlas link is justified by a compound name or structure correspondence in the MIBiG annotation fields
- Input JSON records pass the MIBiG JSON schema validation before extraction

## Limitations

- Matching relies on compound names and structures agreeing between MIBiG annotations and the external databases; entries with missing, mistyped, or duplicated annotation information may fail to link (MIBiG 2.0 curation explicitly fixed empty or mistyped information and removed duplicate entries).
- Links are only as current as the external resources (npatlas.org, GNPS, PubChem); the mapping is a snapshot of MIBiG 2.0 cross-links, not a live sync.

## Evidence

- [intro] additional cross-links established with the Natural Products Atlas: "additional cross-links have been established with the Natural Products Atlas (https://www.npatlas.org/)"
- [intro] MIBiG cross-links with Natural Products Atlas and GNPS: "cross-links have been established with the Natural Products Atlas"
- [supplementary] MIBiG repository access point: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [readme] MIBiG API serves the repository: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
- [intro] MIBiG data migration into relational database with API: "the MIBiG data has now been migrated into a relational database"
