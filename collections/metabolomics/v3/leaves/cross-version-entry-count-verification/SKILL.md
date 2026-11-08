---
name: cross-version-entry-count-verification
description: Use when auditing a MIBiG JSON release to verify that an update met its
  stated goal of increasing non-minimal entries, or when comparing entry-status composition
  across MIBiG versions.
when_to_use_negative:
- The dataset is not the MIBiG JSON data standard (e.g. antiSMASH GenBank output or
  NCBI GenBank/RefSeq records), which lacks a cluster.status field.
- The analysis goal is entry-level curation quality (structures, bioactivities, domain
  substrates) rather than version-level status composition.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3489
tools:
- name: mibig-json
  role: Source of the JSON-formatted MIBiG curation data whose cluster.status field
    encodes minimal vs non-minimal entry classification
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:369667008b358bad3116551a43e10013451301c50dd5e305ce8b343a18e5559a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - mibig-json
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cross-version-entry-count-verification

## Summary

Classifies MIBiG JSON entries as minimal vs non-minimal via the cluster.status field and tallies each class, enabling verification of database-update goals such as MIBiG 3.0's increase of non-minimal entries from 486 (v2.0) to 928.

## When to use

Use when auditing a MIBiG JSON release to verify that an update met its stated goal of increasing non-minimal entries, or when comparing entry-status composition across MIBiG versions.

## When NOT to use

- The dataset is not the MIBiG JSON data standard (e.g. antiSMASH GenBank output or NCBI GenBank/RefSeq records), which lacks a cluster.status field.
- The analysis goal is entry-level curation quality (structures, bioactivities, domain substrates) rather than version-level status composition.

## Inputs

- MIBiG JSON entry files from the data directory of the mibig-secmet/mibig-json repository (MIBiG 3.0 release)
- Reference counts for prior versions (e.g. 486 non-minimal entries in MIBiG 2.0)

## Outputs

- Two-row CSV count table with one row per cluster.status class (minimal, non-minimal)
- Verified non-minimal entry count for the release (expected: 928 for MIBiG 3.0)

## How to apply

Load the MIBiG 3.0 JSON entry files from the mibig-secmet/mibig-json repository, where entry status is tracked via the cluster.status field. Read cluster.status for every entry and bin each entry into exactly two classes: minimal and non-minimal. Tally the counts per class and write a two-row count table (one row per status class) as a CSV file. Validate by checking that the non-minimal count equals the reported 928 for MIBiG 3.0, up from 486 in MIBiG 2.0; MIBiG 3.0 added 661 novel BGCs and removed 69 duplicate and low-quality entries, so discrepancies may indicate incomplete file loading or untracked status fields.

## Related tools

- **mibig-json** (Source of the JSON-formatted MIBiG curation data whose cluster.status field encodes minimal vs non-minimal entry classification) — https://github.com/mibig-secmet

## Examples

```
python -c "import json,glob,collections; c=collections.Counter(json.load(open(f))['cluster']['status'] for f in glob.glob('data/*.json')); print(c)"
```

## Evaluation signals

- Non-minimal count for MIBiG 3.0 equals the reported 928 entries (up from 486 in MIBiG 2.0).
- Count table contains exactly two rows, one per status class (minimal, non-minimal).
- Sum of minimal + non-minimal rows matches the number of JSON files loaded from the data directory.
- Every entry parsed yields a defined cluster.status value; no null or unclassified entries remain.

## Limitations

- The README notes entry status is 'now tracked via the cluster.status field', so older schema versions or external copies may not carry this field.
- Version comparison assumes consistent status semantics between MIBiG 2.0 and 3.0; removal of 69 duplicate and low-quality entries in 3.0 can shift totals independent of status promotion.

## Evidence

- [readme] status field location: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [abstract] non-minimal increase: "Here, we present MIBiG 3.0: an update designed to increase the number of non-minimal entries in our database and adding new data entries through a large-scale community annotation effort."
- [intro] entries added and removed: "we added 661 novel BGCs to the MIBiG database which were published since the last database update and removed 69 duplicate and low-quality entries"
- [supplementary] repo location: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
