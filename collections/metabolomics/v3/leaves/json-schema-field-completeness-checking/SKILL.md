---
name: json-schema-field-completeness-checking
description: Use when auditing JSON-schema-validated repository entries for missing
  or incomplete annotations — e.g. counting MIBiG BGC records lacking product chemical
  structures or missing formula/mass/chirality properties against a pre-update baseline.
when_to_use_negative:
- Input records are uncurated candidate BGCs from genome mining tools (e.g. antiSMASH
  output) rather than curated MIBiG repository entries with schema-defined annotation
  fields
- The goal is pure structural conformance checking of a single document against a
  JSON schema (well-formedness/validation), not cross-entry completeness accounting
  against a historical baseline
- The database entries have no prior version available, so no before/after re-curation
  diff (e.g. 1170 → 2021) can be computed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: JSON schema
  role: Defines the field-completeness and dependency rules that determine which entry
    fields (chemical structure, formula, mass, chirality) must be non-empty
  repo: https://json-schema.org
- name: PostgreSQL
  role: Relational backend in which MIBiG entries are stored; SQL queries over entry
    fields drive the completeness tallies
  repo: https://www.postgresql.org/
- name: GitHub
  role: Source of the JSON-formatted data standard, SQL schema, and entry files consumed
    by the audit
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:85de9632d6ac90c1392a0fb170ead02cad5d7dffc006229051bda22215193a97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# json-schema-field-completeness-checking

## Summary

Programmatic completeness audit of schema-validated database records: loading MIBiG 2.0's JSON-formatted BGC entries and tallying empty or incomplete fields (product chemical structures, molecular formula, mass, chirality) to quantify annotation gaps before and after re-curation.

## When to use

Use when auditing JSON-schema-validated repository entries for missing or incomplete annotations — e.g. counting MIBiG BGC records lacking product chemical structures or missing formula/mass/chirality properties against a pre-update baseline.

## When NOT to use

- Input records are uncurated candidate BGCs from genome mining tools (e.g. antiSMASH output) rather than curated MIBiG repository entries with schema-defined annotation fields
- The goal is pure structural conformance checking of a single document against a JSON schema (well-formedness/validation), not cross-entry completeness accounting against a historical baseline
- The database entries have no prior version available, so no before/after re-curation diff (e.g. 1170 → 2021) can be computed

## Inputs

- MIBiG 2.0 JSON-formatted BGC entries (mibig-secmet GitHub)
- MIBiG JSON schema (data standard with validation and dependency rules)
- Pre-2.0 baseline listing of 1170 BGC entries for before/after comparison

## Outputs

- Count of BGC entries lacking product chemical structures (paper-reported: 770)
- Count of entries with missing or incomplete chemical property annotations (paper-reported: 500)
- Count of BGCs to which new chemical structure information was added in re-curation (paper-reported: 220)
- Structured summary file pairing the three audit counts with the total 2021 BGC count

## How to apply

Obtain the MIBiG 2.0 JSON-formatted data standard and entry files from the mibig-secmet GitHub organization, which also publishes the SQL schema for the underlying PostgreSQL database. Load all 2021 BGC records and, for each, test whether the chemical structure field is empty, comparing against the pre-2.0 baseline set of 1170 BGC entries so that newly characterized structures can be isolated from the diff. Separately scan each record for missing or incomplete chemical property annotations (absent molecular formula, molecular mass, or chirality fields) and tally the affected entries. Re-run the same field checks on the re-curated annotations to count how many BGCs received new structure information. Rationale: MIBiG 2.0 adopted JSON schema description and validation technology, so the schema's required/conditional fields define exactly which values must be populated for an entry to count as complete.

## Related tools

- **JSON schema** (Defines the field-completeness and dependency rules that determine which entry fields (chemical structure, formula, mass, chirality) must be non-empty) — https://json-schema.org
- **PostgreSQL** (Relational backend in which MIBiG entries are stored; SQL queries over entry fields drive the completeness tallies) — https://www.postgresql.org/
- **GitHub** (Source of the JSON-formatted data standard, SQL schema, and entry files consumed by the audit) — https://github.com/mibig-secmet

## Examples

```
import json; d=json.load(open('mibig_2.0.json')); n_struct=sum(1 for e in d if not e.get('chemical_structure')); n_props=sum(1 for e in d if not e.get('molecular_formula') or not e.get('chirality'))
```

## Evaluation signals

- Every computed count must match the paper-reported values: 770 entries lacking product structures, 500 with missing/incomplete properties, 220 newly added structures, out of 2021 total BGCs
- All loaded entries must validate against the MIBiG JSON schema before completeness tallies are trusted
- Baseline diff must reconcile: 2021 total − 1170 pre-2.0 baseline = 851 new entries, matching the abstract's '851 new BGCs have been added'
- Audit counts are reproducible from the GitHub JSON data standard and SQL schema alone (no hidden state), so a re-run yields identical tallies

## Limitations

- Completeness judgments depend entirely on the MIBiG 2.0 schema's required/conditional fields; restructuring the schema changes which fields count as incomplete
- The audit requires the pre-2.0 baseline listing to attribute structure additions to re-curation versus new entries
- The mibig-api repository README is minimal — no changelog and only build prerequisites (Go 1.11.x, Apache-2.0 license) are documented, so audit tooling must rely on the article and data standard

## Evidence

- [intro] JSON schema adopted for validation and dependency rules: "we also adopted the JSON schema description and validation technology"
- [intro] Extensive re-annotation improved data quality: "we performed extensive re-annotation of the entire database, increasing the overall data quality"
- [supplementary] Data standard and SQL schema available on GitHub: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
- [intro] PostgreSQL backend accessed via the mibig-api server: "A REST-like web API (https://github.com/mibig-secmet/mibig-api/) handles access to the underlying PostgreSQL"
- [readme] API server implementation context: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
