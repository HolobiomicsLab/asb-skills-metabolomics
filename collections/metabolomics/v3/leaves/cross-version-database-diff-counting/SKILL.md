---
name: cross-version-database-diff-counting
description: Use when auditing a curated database release against its predecessor
  to count entries with missing fields, incomplete annotations, or newly added information
  (e.g. chemical structures across MIBiG versions).
when_to_use_negative:
- Input is a raw, uncurated BGC prediction set (e.g. antiSMASH output on genomes)
  rather than a versioned, manually curated repository release
- You need novel biological findings about BGC function rather than annotation-completeness
  bookkeeping between releases
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: JSON schema
  role: Defines and validates the MIBiG data standard whose fields (chemical structure,
    molecular formula, mass, chirality) are audited for completeness
  repo: https://json-schema.org
- name: PostgreSQL
  role: Relational database backing the MIBiG repository, queried to count entries
    with empty or incomplete annotation fields across versions
  repo: https://www.postgresql.org/
- name: GitHub
  role: Hosts the JSON-formatted data standard, entries, and SQL schema needed to
    reconstruct both MIBiG versions for diffing
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:bd0af4f6749b3c446921ec4945e5851591d0fafa324d01f2a6e465a7556ed9c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cross-version-database-diff-counting

## Summary

A curation-audit method for comparing two releases of a curated scientific database (here MIBiG 1.0's 1170 entries vs MIBiG 2.0's 2021 BGCs) to quantify gaps (770 entries lacking product structures, 500 with missing/incomplete chemical properties) and curation gains (220 BGCs with newly incorporated structures).

## When to use

Use when auditing a curated database release against its predecessor to count entries with missing fields, incomplete annotations, or newly added information (e.g. chemical structures across MIBiG versions).

## When NOT to use

- Input is a raw, uncurated BGC prediction set (e.g. antiSMASH output on genomes) rather than a versioned, manually curated repository release
- You need novel biological findings about BGC function rather than annotation-completeness bookkeeping between releases

## Inputs

- MIBiG 2.0 JSON-formatted data standard and entry records from the mibig-secmet GitHub repository
- Pre-2.0 baseline listing of 1170 BGC entries
- Post-re-curation MIBiG entries (2021 BGCs) with chemical structure and chemical property fields (molecular formula, mass, chirality)

## Outputs

- Structured summary file of audit counts: 770 BGCs lacking product structures, 500 with missing/incomplete properties, 220 with newly incorporated structures, against the 2021 BGC total
- Per-field missingness diff between the two database versions

## How to apply

Obtain both database versions in a machine-readable format — for MIBiG, the JSON-formatted data standard and entries from the mibig-secmet GitHub repository, plus the pre-2.0 baseline of 1170 BGC entries. Load all 2021 post-update BGC records (e.g. into PostgreSQL via the mibig-api SQL schema) and programmatically count entries whose chemical structure field is empty, diffing against the 1170-entry baseline to isolate previously uncharacterized structures. Separately scan records for missing or incomplete chemical property annotations (absent molecular formula, mass, or chirality fields) and tally affected entries. Re-run the structure-presence check after re-curation to count BGCs that gained new structure information (220). Decision point: counts only count as 'resolved' when they match the paper-reported values (770 lacking structures, 500 with incomplete properties, 220 newly added) out of the 2021 total.

## Related tools

- **JSON schema** (Defines and validates the MIBiG data standard whose fields (chemical structure, molecular formula, mass, chirality) are audited for completeness) — https://json-schema.org
- **PostgreSQL** (Relational database backing the MIBiG repository, queried to count entries with empty or incomplete annotation fields across versions) — https://www.postgresql.org/
- **GitHub** (Hosts the JSON-formatted data standard, entries, and SQL schema needed to reconstruct both MIBiG versions for diffing) — https://github.com/mibig-secmet

## Examples

```
psql -d mibig -c "SELECT count(*) FROM bgc WHERE chemical_structure IS NULL;"
```

## Evaluation signals

- Count of BGCs lacking an associated chemical structure equals 770 in the pre-restructuring baseline
- Count of entries with missing or incomplete chemical property annotations (formula, mass, chirality) equals 500
- Count of BGCs to which new structure information was incorporated during re-curation equals 220
- Total post-update entry count equals 2021 BGCs (1170 baseline + 851 added)
- Every computed count is reproducible from the GitHub JSON data standard loaded into PostgreSQL, not hand-tallied

## Limitations

- Counts are only meaningful relative to a fixed baseline version; MIBiG's 'extensive manual data curation' also fixed mistyped info and removed duplicates, so field-level diffs may conflate additions with edits
- No changelog was found in the repository, so per-version change attribution relies on the diff itself rather than documented release notes

## Evidence

- [abstract] Pre-2.0 baseline of 1170 entries and 851 additions: "Over the past five years, 851 new BGCs have been added. Additionally, we performed extensive manual data curation of all entries to improve the annotation quality"
- [intro] Expansion from 1170 to 2021 curated BGCs with added chemical structures: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [intro] Data quality work included fixing mistyped info, removing duplicates, and adding structures: "fixing empty or mistyped information in the data, removing duplicate entries"
- [supplementary] JSON-formatted data standard and SQL schema available on GitHub: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
- [readme] README confirms the API/SQL access route used to query entries: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
