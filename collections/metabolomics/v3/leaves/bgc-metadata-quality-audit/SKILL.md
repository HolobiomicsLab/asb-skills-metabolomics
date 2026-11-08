---
name: bgc-metadata-quality-audit
description: Use when auditing a versioned repository of biosynthetic gene cluster
  annotations — typically MIBiG JSON entries — to quantify missing chemical structures,
  incomplete property fields, or annotation deltas between releases.
when_to_use_negative:
- The goal is predicting or discovering BGCs from genome mining (antiSMASH/ClusterFinder
  use-cases) rather than auditing an existing curated repository's annotation completeness.
- Input records are not MIBiG-JSON schema-conformant and the schema itself is the
  thing being designed, not audited.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: JSON schema
  role: Defines field presence/absence rules used to detect entries lacking chemical
    structure or with incomplete chemical property annotations
  repo: https://json-schema.org
- name: PostgreSQL
  role: Relational database into which MIBiG data is migrated; supports programmatic
    field-completeness queries across all 2021 entries
  repo: https://www.postgresql.org/
- name: GitHub
  role: Source of the JSON-formatted data standard, entries, and SQL schema used as
    audit inputs
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:30561fd33732550e28891600ce6d0768378c1080330502cd2b6875ac9a414a06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bgc-metadata-quality-audit

## Summary

A reproducible audit method that programmatically counts metadata gaps across a curated BGC repository (e.g. MIBiG 2.0's 2021 entries), quantifying how many entries lack product chemical structures (770 pre-restructuring), have missing/incomplete chemical properties (500), and how many received new structures during re-curation (220).

## When to use

Use when auditing a versioned repository of biosynthetic gene cluster annotations — typically MIBiG JSON entries — to quantify missing chemical structures, incomplete property fields, or annotation deltas between releases.

## When NOT to use

- The goal is predicting or discovering BGCs from genome mining (antiSMASH/ClusterFinder use-cases) rather than auditing an existing curated repository's annotation completeness.
- Input records are not MIBiG-JSON schema-conformant and the schema itself is the thing being designed, not audited.

## Inputs

- MIBiG 2.0 JSON-formatted entries for 2021 BGCs (from GitHub mibig-secmet)
- Pre-2.0 baseline listing of 1170 BGC entries
- JSON schema (data standard) defining required/optional fields per entry

## Outputs

- Structured summary file with counts: BGCs lacking product structures (770), BGCs with missing/incomplete properties (500), BGCs with newly added structures (220), and total BGC count (2021)
- Per-record flags for empty structure fields and incomplete property annotations (formula, mass, chirality)

## How to apply

Obtain the MIBiG 2.0 JSON-formatted data standard and entries from the mibig-secmet GitHub repository, plus the pre-2.0 baseline listing of 1170 BGC entries. Load all 2021 BGC records and programmatically count entries whose chemical structure field is empty, comparing against the 1170-entry baseline to isolate previously uncharacterized structures. Scan all records for missing or incomplete chemical property annotations — absent molecular formula, mass, or chirality fields — and tally affected entries. Diff chemical structure annotations before and after re-curation to count BGCs to which new structure information was incorporated (the article reports 220 such BGCs). Write the three audit counts (770, 500, 220) alongside the total 2021 BGC count to a structured summary file. This mirrors MIBiG 2.0's stated curation goal: 'fixing empty or mistyped information in the data' and improving overall data quality through extensive re-annotation.

## Related tools

- **JSON schema** (Defines field presence/absence rules used to detect entries lacking chemical structure or with incomplete chemical property annotations) — https://json-schema.org
- **PostgreSQL** (Relational database into which MIBiG data is migrated; supports programmatic field-completeness queries across all 2021 entries) — https://www.postgresql.org/
- **GitHub** (Source of the JSON-formatted data standard, entries, and SQL schema used as audit inputs) — https://github.com/mibig-secmet

## Examples

```
python -c "import json; d=json.load(open('mibig_json/all.json')); print(sum(1 for e in d if not e.get('chem_struct')), len(d))"
```

## Evaluation signals

- Count of BGCs lacking product chemical structures equals 770 (paper-reported pre-restructuring gap).
- Count of BGCs with missing or incomplete chemical properties (formula/mass/chirality) equals 500.
- Count of BGCs to which new structure information was incorporated during re-curation equals 220.
- Total audited record count equals 2021 BGCs, consistent with expansion from the 1170-entry pre-2.0 baseline.
- Every entry checked conforms to the MIBiG JSON schema, so absence is a data gap rather than a parsing error.

## Limitations

- Audit counts are relative to MIBiG 2.0's schema; the schema restructuring means pre-2.0 baseline comparisons require the 1170-entry snapshot, not the restructured data.
- Mistyped or duplicate entries can distort gap counts; the article notes curation had to 'fix empty or mistyped information in the data, removing duplicate entries' — no changelog was available to trace per-entry changes.
- Structure completeness depends on fields the schema defines; properties outside the schema (e.g. spectral links to GNPS/NPAtlas) are cross-links, not audited quality signals.

## Evidence

- [intro] Fixing empty or mistyped information in the data: "fixing empty or mistyped information in the data, removing duplicate entries"
- [abstract] 851 new BGCs and manual curation: "Over the past five years, 851 new BGCs have been added. Additionally, we performed extensive manual data curation of all entries to improve the annotation quality"
- [readme] Repository and data standard availability: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
