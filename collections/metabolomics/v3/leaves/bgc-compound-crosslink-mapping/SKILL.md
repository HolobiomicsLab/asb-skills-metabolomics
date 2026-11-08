---
name: bgc-compound-crosslink-mapping
description: Use when a curated set of BGC entries with compound name and structure
  annotations needs to be linked out to chemical databases (Natural Products Atlas,
  GNPS, PubChem) to build a linkage inventory per accession.
when_to_use_negative:
- The input is raw genome assembly data or unannotated candidate BGCs rather than
  curated MIBiG entries with compound name/structure fields — use a genome-mining
  tool like antiSMASH first.
- The goal is de novo structure elucidation or spectral identification; this skill
  only reuses already-established name/structure-based cross-links, not novel matches.
- The BGC records lack compound names or structures, so no matching key exists for
  cross-referencing.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: MIBiG Repository
  role: Source of curated BGC annotations and the interface exposing direct link-outs
    to chemical structure databases
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the JSON-formatted MIBiG data standard and mibig-api source used to
    obtain annotation records for matching
  repo: https://github.com/mibig-secmet
- name: PostgreSQL
  role: Underlying relational database of MIBiG annotations queried via the REST-like
    web API to extract compound fields
  repo: https://www.postgresql.org/
- name: JSON schema
  role: Validation technology ensuring the MIBiG annotation records consumed for cross-linking
    comply with the data standard
  repo: https://json-schema.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:cef79b317a6d2fad0c2f82baa1ecb5d8a8db205d24ff2d869190d491f16cd50d
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
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bgc-compound-crosslink-mapping

## Summary

Maps annotated MIBiG biosynthetic gene cluster (BGC) entries to external chemical resources by matching compound names and chemical structures, establishing cross-links with databases such as the Natural Products Atlas and the GNPS spectral library as a MIBiG 2.0 repository feature.

## When to use

Use when a curated set of BGC entries with compound name and structure annotations needs to be linked out to chemical databases (Natural Products Atlas, GNPS, PubChem) to build a linkage inventory per accession.

## When NOT to use

- The input is raw genome assembly data or unannotated candidate BGCs rather than curated MIBiG entries with compound name/structure fields — use a genome-mining tool like antiSMASH first.
- The goal is de novo structure elucidation or spectral identification; this skill only reuses already-established name/structure-based cross-links, not novel matches.
- The BGC records lack compound names or structures, so no matching key exists for cross-referencing.

## Inputs

- MIBiG 2.0 annotation records (GitHub JSON files conforming to the MIBiG JSON schema)
- MIBiG relational export from the underlying PostgreSQL database via the REST-like web API
- Compound name and chemical structure fields from each BGC entry

## Outputs

- crosslinks.csv — one row per MIBiG accession with link-presence indicators for PubChem, Natural Products Atlas, and GNPS

## How to apply

Load the MIBiG 2.0 annotation records from the repository data (JSON-formatted files on GitHub or the relational PostgreSQL export exposed through the mibig-api). For each BGC entry, extract the compound name and chemical structure fields used for cross-referencing. Match these names and structures against the Natural Products Atlas (npatlas.org) and the GNPS spectral library, since MIBiG 2.0 established cross-links based on compound names and structures matching between annotated BGCs and the chemical databases. Tabulate, per MIBiG accession, which external databases carry a link for that entry, then write the linkage inventory as crosslinks.csv. Rationale: this exploits the re-annotated, quality-controlled MIBiG 2.0 data and the repository's direct link-outs to chemical structure databases.

## Related tools

- **MIBiG Repository** (Source of curated BGC annotations and the interface exposing direct link-outs to chemical structure databases) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the JSON-formatted MIBiG data standard and mibig-api source used to obtain annotation records for matching) — https://github.com/mibig-secmet
- **PostgreSQL** (Underlying relational database of MIBiG annotations queried via the REST-like web API to extract compound fields) — https://www.postgresql.org/
- **JSON schema** (Validation technology ensuring the MIBiG annotation records consumed for cross-linking comply with the data standard) — https://json-schema.org

## Examples

```
python make_crosslinks.py --mibig-json github.com/mibig-secmet --databases npatlas,gnps,pubchem --out crosslinks.csv
```

## Evaluation signals

- The output crosslinks.csv contains one row per MIBiG accession with a field indicating link presence for each of PubChem, Natural Products Atlas, and GNPS.
- Counts of linked entries are consistent with the MIBiG 2.0 statement that direct link-outs to chemical structure databases were added as a repository feature.
- Matched entries agree on compound names and chemical structures between the annotated BGCs and the external chemical databases (no name/structure mismatches among accepted links).
- Every row carries a valid MIBiG accession number obtainable from the relational/JSON repository export.
- Re-running the matching on the same MIBiG 2.0 snapshot yields a stable crosslinks.csv (deterministic, deduplicated linkage inventory).

## Limitations

- Connections are matched by compound names and structures; entries with missing, mistyped, or ambiguous compound annotations will not link, and MIBiG curation had to fix empty or mistyped information during re-annotation.
- The skill inherits the coverage limits of MIBiG 2.0 (2021 curated BGCs); BGCs outside this curated set are not covered.
- Matching depends on external databases' current state — links to the Natural Products Atlas and GNPS reflect what those resources exposed at matching time and may change.

## Evidence

- [intro] cross-links established with Natural Products Atlas and GNPS by name/structure matching: "additional cross-links have been established with the Natural Products Atlas (https://www.npatlas.org/)"
- [abstract] MIBiG 2.0 added direct link-outs to chemical structure databases as a repository feature: "we improved the user experience by adding new features such as query searches and a statistics page, and"
- [intro] MIBiG data migrated into a relational PostgreSQL database accessible via a REST-like API: "handles access to the underlying PostgreSQL (https://www.postgresql.org/) database"
- [intro] Extensive manual curation fixed mistyped/empty information, improving the data used for cross-referencing: "fixing empty or mistyped information in the data, removing duplicate entries"
- [readme] MIBiG source code, JSON data standard and SQL schema available on GitHub: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
