---
name: chemical-structure-annotation-coverage-analysis
description: Use when auditing a curated BGC or compound database (e.g. MIBiG JSON
  entries) before/after a re-curation release, to quantify entries lacking product
  chemical structures and entries with incomplete property annotations.
when_to_use_negative:
- The goal is BGC discovery or novelty assessment against MIBiG (e.g. antiSMASH homology
  searches), not auditing annotation completeness of the repository itself
- The dataset is a candidate-BGC prediction set from genome mining with no curated
  chemical structure fields to audit
- Entries already carry validated structures and the purpose is query/filtering of
  the repository (e.g. Boolean queries by compound name or biosynthetic class) rather
  than coverage accounting
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: JSON schema
  role: Data-standard and validation technology used to load, parse, and check MIBiG
    entry fields (including chemical structure and property fields) during the audit
  repo: https://json-schema.org
- name: PostgreSQL
  role: Underlying relational database accessed via the REST-like MIBiG web API, from
    which repository rows can be queried to verify annotation counts
  repo: https://github.com/mibig-secmet/mibig-api/
- name: GitHub
  role: Source of the JSON-formatted data standard and SQL schema used as the input
    data for the completeness audit
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:5873e240a6ba9c22836dfa14eafbdab2965d466878848047a5a78e07223b46de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - JSON schema
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# chemical-structure-annotation-coverage-analysis

## Summary

Audits a curated biosynthetic gene cluster (BGC) repository to quantify chemical-structure annotation completeness: how many entries lack product structures, how many have missing/incomplete chemical property fields (formula, mass, chirality), and how many gained structure annotations after re-curation. Modeled on the MIBiG 2.0 re-annotation audit, which reported 770 entries lacking product structures and 500 entries with missing or incomplete properties out of 2021 BGCs, with new structure information added to 220 BGCs.

## When to use

Use when auditing a curated BGC or compound database (e.g. MIBiG JSON entries) before/after a re-curation release, to quantify entries lacking product chemical structures and entries with incomplete property annotations.

## When NOT to use

- The goal is BGC discovery or novelty assessment against MIBiG (e.g. antiSMASH homology searches), not auditing annotation completeness of the repository itself
- The dataset is a candidate-BGC prediction set from genome mining with no curated chemical structure fields to audit
- Entries already carry validated structures and the purpose is query/filtering of the repository (e.g. Boolean queries by compound name or biosynthetic class) rather than coverage accounting

## Inputs

- MIBiG JSON-formatted entries (post-restructuring, 2021 BGCs)
- Pre-restructuring baseline listing of 1170 BGC entries
- MIBiG JSON schema from the mibig-secmet GitHub repository

## Outputs

- Count of BGC entries lacking an associated product chemical structure (770)
- Count of BGC entries with missing or incomplete chemical property fields such as formula, mass, or chirality (500)
- Count of BGCs to which new chemical structure information was incorporated during re-curation (220)
- Structured audit summary file containing the three counts alongside the total 2021 BGC count

## How to apply

Obtain the JSON-formatted data standard and all entries from the mibig-secmet GitHub repository, plus the pre-release baseline entry set (for MIBiG 2.0, the pre-update listing of 1170 BGCs). Load every record and programmatically tally entries whose chemical structure field is empty, comparing against the baseline set to identify previously uncharacterized structures. Scan records for missing or incomplete chemical property annotations—absent molecular formula, mass, or chirality fields—and tally those separately. Diff structure annotations before versus after re-curation to count entries to which new structure information was incorporated. Write the audit counts (770 lacking structures, 500 with incomplete properties, 220 newly structured, out of 2021 total BGCs) to a structured summary file for verification.

## Related tools

- **JSON schema** (Data-standard and validation technology used to load, parse, and check MIBiG entry fields (including chemical structure and property fields) during the audit) — https://json-schema.org
- **PostgreSQL** (Underlying relational database accessed via the REST-like MIBiG web API, from which repository rows can be queried to verify annotation counts) — https://github.com/mibig-secmet/mibig-api/
- **GitHub** (Source of the JSON-formatted data standard and SQL schema used as the input data for the completeness audit) — https://github.com/mibig-secmet

## Examples

```
python -c "import json; d=json.load(open('mibig.json')); s=sum(1 for e in d['entries'] if not e.get('chem_struct')); p=sum(1 for e in d['entries'] if not all(e.get(k) for k in ('mol_formula','mass','chirality'))); print(s, p, len(d['entries']))"
```

## Evaluation signals

- Count of entries lacking product structures must equal the reported value (770)
- Count of entries with missing or incomplete chemical property annotations must equal the reported value (500)
- Count of BGCs with newly incorporated structure information must equal the reported value (220)
- Total number of BGC records parsed must equal 2021, matching the MIBiG 2.0 entry count (1170 baseline + 851 new)
- All JSON entries must validate against the MIBiG 2.0 schema, which embeds validation and dependency rules

## Limitations

- Completeness is defined only by absence of structure or missing/incomplete fields (formula, mass, chirality); structures may still be incorrect despite being present, requiring manual re-annotation to raise overall data quality
- Comparison depends on a reliable pre-restructuring baseline entry set; duplicate entries and mistyped information in the older data must first be removed for accurate diffing
- Coverage accounting reflects the repository state at the version snapshot (MIBiG 2.0) and must be re-run for later releases

## Evidence

- [other] MIBiG 2.0 re-curation addressed 770 entries lacking product structures and 500 entries with missing or incomplete properties, adding structure information for 220 BGCs: "Over the past five years, 851 new BGCs have been added. Additionally, we performed extensive manual data curation of all entries to improve the annotation quality"
- [intro] Schema redesign and re-annotation rationale for auditing structure and property completeness: "we performed extensive re-annotation of the entire database, increasing the overall data quality"
- [abstract] MIBiG 2.0 scope: schema, data, and repository updates that the audit measures against: "Here, we present MIBiG 2.0, which encompasses major updates to the schema, the data, and the online repository itself"
- [supplementary] The audit operates on the JSON data standard and entries distributed via GitHub: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
- [readme] Repository data accessible programmatically through the API backed by PostgreSQL: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
