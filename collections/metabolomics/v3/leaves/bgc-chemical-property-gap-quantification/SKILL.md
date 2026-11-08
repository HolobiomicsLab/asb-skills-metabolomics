---
name: bgc-chemical-property-gap-quantification
description: Use when auditing annotation completeness of a known-function BGC database
  — e.g. quantifying how many entries lack a chemical structure or have empty molecular
  formula/mass/chirality fields before and after a re-curation release.
when_to_use_negative:
- Input is a set of predicted BGCs of unknown function (MIBiG curates only BGCs of
  known function, so completeness audits against chemical structure fields are not
  meaningful)
- The goal is cataloguing newly added entries (851 new BGCs) rather than auditing
  annotation gaps in existing entries
- The database schema has no chemical-structure or property fields to audit — e.g.
  a sequence-only BGC table
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3272
tools:
- name: JSON schema
  role: Defines the schema against which chemical property fields are validated and
    embedded with dependency rules, enabling programmatic detection of empty/incomplete
    fields
  repo: https://json-schema.org
- name: PostgreSQL
  role: Relational database holding the migrated MIBiG data, allowing SQL queries
    to tally entries by completeness of chemical annotation fields
  repo: https://www.postgresql.org
- name: GitHub
  role: Hosts the JSON-formatted data standard and entry records that are downloaded
    and audited for missing structures and properties
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:bbf15dd48f9bb33c784594e738f6a0d38bc6e334d3205349c95c67115adb1936
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bgc-chemical-property-gap-quantification

## Summary

Audits a curated BGC repository (MIBiG) for missing chemical annotations: programmatically counting entries lacking product structures, entries with missing/incomplete chemical property fields, and entries to which new structure information was added by re-curation.

## When to use

Use when auditing annotation completeness of a known-function BGC database — e.g. quantifying how many entries lack a chemical structure or have empty molecular formula/mass/chirality fields before and after a re-curation release.

## When NOT to use

- Input is a set of predicted BGCs of unknown function (MIBiG curates only BGCs of known function, so completeness audits against chemical structure fields are not meaningful)
- The goal is cataloguing newly added entries (851 new BGCs) rather than auditing annotation gaps in existing entries
- The database schema has no chemical-structure or property fields to audit — e.g. a sequence-only BGC table

## Inputs

- MIBiG 2.0 JSON-formatted data standard and entry records from GitHub (mibig-secmet)
- Pre-2.0 baseline listing of 1170 BGC entries
- List of chemical property fields defined by the schema (molecular formula, mass, chirality, structure)

## Outputs

- Structured summary file of audit counts: BGCs lacking product structures (770), BGCs with missing or incomplete properties (500), BGCs with newly added structures (220), and total BGC count (2021)
- Per-entry completeness report of which chemical property fields are empty or incomplete

## How to apply

Obtain the JSON-formatted MIBiG data standard and entries from the mibig-secmet GitHub repository, plus the pre-update baseline entry count (1170 BGCs). Load all 2021 BGC records and programmatically count entries whose chemical structure field is empty, comparing against the pre-update set to identify previously uncharacterized structures. Scan all records for missing or incomplete chemical property annotations (absent molecular formula, mass, or chirality fields) and tally the affected entries. Compare chemical structure annotations before and after re-curation to count BGCs to which new structure information was incorporated. Validate the audit by requiring each computed count to match the reported values: 770 entries lacking structures, 500 with missing or incomplete properties, and 220 BGCs receiving new structures, against the 2021-entry total.

## Related tools

- **JSON schema** (Defines the schema against which chemical property fields are validated and embedded with dependency rules, enabling programmatic detection of empty/incomplete fields) — https://json-schema.org
- **PostgreSQL** (Relational database holding the migrated MIBiG data, allowing SQL queries to tally entries by completeness of chemical annotation fields) — https://www.postgresql.org
- **GitHub** (Hosts the JSON-formatted data standard and entry records that are downloaded and audited for missing structures and properties) — https://github.com/mibig-secmet

## Examples

```
python -c "import json,glob; e=[json.load(open(f)) for f in glob.glob('mibig_json/*.json')]; print(len(e), sum(1 for x in e if not x.get('chemical_structure')))"
```

## Evaluation signals

- Count of BGCs lacking product structures equals the reported 770
- Count of BGCs with missing or incomplete chemical properties equals the reported 500
- Count of BGCs to which new structure information was incorporated equals the reported 220
- Total audited BGC entries equal 2021 (1170 pre-update baseline + 851 new entries)
- All records pass the JSON schema validation with embedded dependency rules before field-empty tallies are computed

## Limitations

- Counts depend on the 2.0 schema restructuring: property fields (molecular formula, mass, chirality) were added in the redesign, so comparisons against pre-2.0 data rely on the 1170-entry baseline listing rather than schema-validated records
- Reliance on manual re-annotation quality: the article notes fixing empty or mistyped information and removing duplicate entries, so residual data-entry errors may still affect tallies
- Data quality issues are addressed in aggregate (extensive re-annotation of the entire database), not per-field, so no per-entry confidence measure accompanies the gap counts

## Evidence

- [abstract] extensive manual curation counts: "Over the past five years, 851 new BGCs have been added. Additionally, we performed extensive manual data curation of all entries to improve the annotation quality"
- [intro] database expansion baseline: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [intro] data-quality fixes audited: "fixing empty or mistyped information in the data, removing duplicate entries"
- [intro] schema validation technology: "embed validation and dependency rules into the schema"
- [readme] data access channel: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
