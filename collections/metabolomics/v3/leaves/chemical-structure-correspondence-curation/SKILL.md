---
name: chemical-structure-correspondence-curation
description: Use when a curated BGC entry carries compound name and chemical structure
  metadata and you need to inventory which external chemical resources (PubChem, Natural
  Products Atlas, GNPS) hold a corresponding cross-link for that entry.
when_to_use_negative:
- The input consists of candidate BGCs mined from genomes with unknown products (e.g.
  antiSMASH/IMG-ABC output) rather than BGCs of known function with annotated compound
  names/structures — no matching keys exist
- The goal is BGC novelty assessment or homology searching against MIBiG, not chemical
  cross-reference curation
- Only a single chemical database is of interest and no per-accession link inventory
  is needed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: MIBiG Repository
  role: Source of the annotated BGC entries and compound name/structure metadata to
    be cross-referenced
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the JSON-formatted data standard and source components from which MIBiG
    annotation records are loaded
  repo: https://github.com/mibig-secmet
- name: PostgreSQL
  role: Relational database underlying the repository access via the REST-like web
    API, used to query/export MIBiG entries
  repo: https://www.postgresql.org/
- name: JSON schema
  role: Validation technology constraining the MIBiG annotation records consumed by
    this curation step
  repo: https://json-schema.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:5038e30e57f6a23e9ded243adc542f6b5b0575edeb20d79f0dff5df93c5ee132
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - MIBiG Repository
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chemical-structure-correspondence-curation

## Summary

Curation skill that establishes and tabulates cross-links between annotated MIBiG biosynthetic gene cluster (BGC) records and external chemical/analytical resources (Natural Products Atlas, GNPS spectral library, PubChem) by matching compound names and chemical structures, mirroring the MIBiG 2.0 feature of direct link-outs to chemical structure databases.

## When to use

Use when a curated BGC entry carries compound name and chemical structure metadata and you need to inventory which external chemical resources (PubChem, Natural Products Atlas, GNPS) hold a corresponding cross-link for that entry.

## When NOT to use

- The input consists of candidate BGCs mined from genomes with unknown products (e.g. antiSMASH/IMG-ABC output) rather than BGCs of known function with annotated compound names/structures — no matching keys exist
- The goal is BGC novelty assessment or homology searching against MIBiG, not chemical cross-reference curation
- Only a single chemical database is of interest and no per-accession link inventory is needed

## Inputs

- MIBiG 2.0 annotation records (JSON-formatted data standard files from the GitHub repository)
- Compound name fields per BGC entry
- Chemical structure fields (structures) per BGC entry
- External database access points: PubChem, Natural Products Atlas (https://www.npatlas.org/), GNPS spectral library

## Outputs

- crosslinks.csv: one row per MIBiG accession with link-presence flags for PubChem, Natural Products Atlas, and GNPS

## How to apply

Load the MIBiG 2.0 annotation records from the repository data (GitHub JSON-formatted data standard, or via the relational export exposed by the mibig-api PostgreSQL backend). For each BGC accession, extract the compound name and chemical structure fields used for cross-referencing — these are the keys MIBiG 2.0 relied on, since cross-links were established by matching compound names and structures against the external resources. Match names and structures against PubChem, the Natural Products Atlas (npatlas.org), and the GNPS spectral library, and record per-accession which of the three databases carry a link. Write the linkage inventory as crosslinks.csv, one row per MIBiG entry with a link-presence field per database. Rationale: the MIBiG 2.0 paper added cross-links to the Natural Products Atlas and GNPS as a repository feature, so the output should reproduce that correspondence layer rather than infer biosynthetic assignments.

## Related tools

- **MIBiG Repository** (Source of the annotated BGC entries and compound name/structure metadata to be cross-referenced) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the JSON-formatted data standard and source components from which MIBiG annotation records are loaded) — https://github.com/mibig-secmet
- **PostgreSQL** (Relational database underlying the repository access via the REST-like web API, used to query/export MIBiG entries) — https://www.postgresql.org/
- **JSON schema** (Validation technology constraining the MIBiG annotation records consumed by this curation step) — https://json-schema.org

## Examples

```
python curate_crosslinks.py --mibig-json mibig_json/ --databases pubchem,npatlas,gnps --out crosslinks.csv
```

## Evaluation signals

- crosslinks.csv contains exactly one row per MIBiG entry with an explicit link-presence field for each of PubChem, Natural Products Atlas and GNPS
- Counts of linked entries are consistent with the MIBiG 2.0 statement that direct link-outs to chemical structure databases were added as a repository feature
- Cross-links to the Natural Products Atlas resolve to valid records at https://www.npatlas.org/
- All consumed MIBiG records validate against the adopted JSON schema description and validation technology before matching
- Entries lacking compound name/structure fields are flagged or excluded rather than silently matched

## Limitations

- Matching is performed on compound names and structures; the article does not specify an automated similarity threshold, so ambiguous name variants or homonymous compounds may require manual curation
- Links exist only where the external database already carries the compound; absence of a GNPS or Natural Products Atlas link does not imply the BGC is uncharacterized
- The repository README provides no API endpoint documentation for programmatic cross-link retrieval, and no changelog was found for the mibig-api

## Evidence

- [intro] cross-links with Natural Products Atlas and GNPS: "cross-links have been established with the Natural Products Atlas"
- [intro] Natural Products Atlas cross-link target: "additional cross-links have been established with the Natural Products Atlas (https://www.npatlas.org/)"
- [intro] relational backend for entry retrieval: "the MIBiG data has now been migrated into a relational database"
- [readme] API server for MIBiG repository: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
