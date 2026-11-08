---
name: chemical-target-annotation-extraction
description: Use when you need compound-level bioactivity and molecular-target coverage
  statistics from MIBiG JSON entries, e.g. to reproduce the 1002-compound / 51-bioactivity
  / 95-target annotation table.
when_to_use_negative:
- Input is antiSMASH-style predicted BGC output rather than manually curated, experimentally
  validated MIBiG entries — MIBiG annotations come from community curation, not automated
  prediction.
- You need sequence-structure-function training data (e.g. A-domain substrates) rather
  than compound-level bioactivity/target coverage counts.
- The MIBiG version predates 3.0 (e.g. MIBiG 2.0, 2021 entries), so the 1002-compound
  / 51-bioactivity / 95-target tallies will not validate.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3407
tools:
- name: PIKAChU
  role: Used in the same curation effort to validate SMILES syntax of the annotated
    compound structures that the bioactivity/target annotations attach to.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:3ac45637da94879c3e04ac26a6618639f8e404924ccfb39de5b5433b0acdf190
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# chemical-target-annotation-extraction

## Summary

Parses MIBiG 3.0 JSON-formatted cluster entries to extract compound bioactivity annotations and the subset carrying chemical (molecular) target data, then normalises bioactivity terms into the 68 standardised functional categories and tallies per-category compound coverage.

## When to use

Use when you need compound-level bioactivity and molecular-target coverage statistics from MIBiG JSON entries, e.g. to reproduce the 1002-compound / 51-bioactivity / 95-target annotation table.

## When NOT to use

- Input is antiSMASH-style predicted BGC output rather than manually curated, experimentally validated MIBiG entries — MIBiG annotations come from community curation, not automated prediction.
- You need sequence-structure-function training data (e.g. A-domain substrates) rather than compound-level bioactivity/target coverage counts.
- The MIBiG version predates 3.0 (e.g. MIBiG 2.0, 2021 entries), so the 1002-compound / 51-bioactivity / 95-target tallies will not validate.

## Inputs

- MIBiG 3.0 JSON-formatted cluster entries (data directory of mibig-secmet/mibig-json)
- Bioactivity annotation fields and chemical target annotation fields per cluster entry

## Outputs

- bioactivity_coverage_table.csv — per-functional-category counts of compounds with bioactivity annotations
- Set of 51 normalised unique bioactivity terms mapped onto 68 standardised functional categories
- Flagged list of 95 compounds annotated with molecular targets

## How to apply

Obtain the MIBiG 3.0 JSON data from the mibig-secmet/mibig-json GitHub repository, where datasets live in the `data` directory and entry status is tracked via the `cluster.status` field. Parse every cluster entry with Python's json/pandas libraries, extracting the bioactivity annotation fields (bioactivity terms and chemical target annotations) from each entry. Normalise the extracted activity terms into the 51 unique bioactivities and map each onto the 68 standardised functional categories used in MIBiG 3.0 curation. Tally, per functional category, the number of compounds carrying bioactivity annotations and flag the 95 compounds that additionally have chemical target data. Validate the tallies against the reported figures (1002 annotated compounds, 51 unique bioactivities, 68 functional categories, 95 compounds with targets) and write the result to bioactivity_coverage_table.csv.

## Related tools

- **PIKAChU** (Used in the same curation effort to validate SMILES syntax of the annotated compound structures that the bioactivity/target annotations attach to.)

## Examples

```
python -c "import json,glob; entries=[json.load(open(f)) for f in glob.glob('data/*.json')]; print(sum(1 for e in entries if e['cluster'].get('bioactivity')))"
```

## Evaluation signals

- Total compounds with bioactivity annotations equals 1002.
- Number of unique normalised bioactivity terms equals 51.
- Number of standardised functional categories in the mapping equals 68.
- Number of compounds flagged with chemical target data equals 95.
- Every entry processed has a cluster.status field; output CSV rows sum to the per-category compound counts.

## Limitations

- Only experimentally validated BGCs with curated bioactivity fields are covered; 69 duplicate and low-quality entries were removed in MIBiG 3.0, so counts depend on the exact release.
- Counts are release-specific: 1002 annotated compounds and 95 compounds with molecular targets reflect MIBiG 3.0, not earlier versions (MIBiG 2.0 had 2021 entries).
- Entry status must be respected via the cluster.status field when parsing the data directory.

## Evidence

- [intro] annotation of 1002 bioactivities: "We focused on three features: the characterisation and cross-linking of 1188 chemical structures, the annotation of 1002 bioactivities of BGC products, and the validation and annotation of 2020"
- [readme] mibig-json repo README: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [supplementary] JSON data standard availability: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
