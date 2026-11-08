---
name: repository-dataset-aggregation
description: Use when a paper reports aggregate statistics about a curated database
  release (total entries, newly added entries, phyla/kingdom coverage) and the underlying
  versioned data are available in a public repository, so the numbers can be recomputed
  and checked.
when_to_use_negative:
- The repository does not provide per-entry machine-readable files (e.g. only a web
  interface or PDF tables), so counts cannot be recomputed independently
- The goal is to extract or score individual cluster annotations (compounds, bioactivities,
  A-domain substrates) rather than aggregate database-level counts
- Entries from multiple releases or mixed provenance are combined in one directory,
  making the novelty delta ambiguous without per-entry version metadata
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: Python json module
  role: Parses each per-entry JSON file to extract cluster identifier, version/status
    metadata, and taxonomy fields before counting and aggregation
- name: mibig-json
  role: Source repository holding the MIBiG 3.0 curated entries as JSON files (plus
    a `genbanks` directory of sequence files not on NCBI); the data snapshot being
    aggregated
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:66676c8bff073c557d64d290e2577b76667c9fd2086b9c336df059386a7dd105
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python json module
  - mibig-json
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# repository-dataset-aggregation

## Summary

Download a released dataset from its versioned GitHub repository (e.g. mibig-secmet/mibig-json), parse every entry file, and aggregate database-level statistics (entry counts, novelty counts, taxonomy coverage) to independently verify figures reported in a publication's Results/Discussion and tables.

## When to use

Use when a paper reports aggregate statistics about a curated database release (total entries, newly added entries, phyla/kingdom coverage) and the underlying versioned data are available in a public repository, so the numbers can be recomputed and checked.

## When NOT to use

- The repository does not provide per-entry machine-readable files (e.g. only a web interface or PDF tables), so counts cannot be recomputed independently
- The goal is to extract or score individual cluster annotations (compounds, bioactivities, A-domain substrates) rather than aggregate database-level counts
- Entries from multiple releases or mixed provenance are combined in one directory, making the novelty delta ambiguous without per-entry version metadata

## Inputs

- GitHub repository snapshot of the database release (mibig-secmet/mibig-json), with per-entry JSON files in the `data` directory
- Reported publication statistics to verify (total entries, novel-entry count, phylum count, kingdom count from Table 2 / Results And Discussion)
- Prior-release baseline counts for novelty computation (e.g. MIBiG 2.0 = 2021 entries, 2019)

## Outputs

- mibig3_stats.csv with computed total entries, novel entries, distinct phyla, and distinct kingdoms
- Per-entry parsed table of cluster identifiers, version/status metadata, and taxonomy fields

## How to apply

Clone or download the repository snapshot corresponding to the release being verified — for MIBiG 3.0 this is the mibig-secmet/mibig-json repo, where curated entries live in the `data` directory as one JSON file per BGC and a `genbanks` directory holds sequence files not available from NCBI's GenBank/RefSeq. Parse each JSON entry with Python's json module, extracting the cluster identifier, version metadata, status/quality-change fields, and taxonomy fields. Count the total number of entry files for the database size, and count entries marked as new since the prior release (MIBiG 2.0 held 2021 entries in 2019) using version/quality-change metadata — the expected delta is 661 novel BGCs, with 69 duplicate and low-quality entries removed. Aggregate taxonomy fields across all entries to count distinct phyla (16) and kingdoms (5). Write computed statistics to a CSV (e.g. mibig3_stats.csv) and compare each value against the paper's Table 2 / Results And Discussion values (2502 entries, 661 new BGCs, 16 phyla, 5 kingdoms), flagging any mismatch.

## Related tools

- **Python json module** (Parses each per-entry JSON file to extract cluster identifier, version/status metadata, and taxonomy fields before counting and aggregation)
- **mibig-json** (Source repository holding the MIBiG 3.0 curated entries as JSON files (plus a `genbanks` directory of sequence files not on NCBI); the data snapshot being aggregated) — https://github.com/mibig-secmet

## Examples

```
python -c "import json,glob; d=[json.load(open(f)) for f in glob.glob('mibig-json/data/*.json')]; print(len(d), len({e['cluster']['taxon']['phylum'] for e in d}))"
```

## Evaluation signals

- Recomputed total entry-file count equals 2502 for the MIBiG 3.0 release snapshot
- Count of entries flagged as new since the 2021-entry MIBiG 2.0 baseline equals 661, and 69 entries are identified as removed duplicate/low-quality entries
- Aggregated taxonomy fields yield exactly 16 distinct phyla across 5 kingdoms of life
- Every field of mibig3_stats.csv matches the values stated in the paper's Results And Discussion / Table 2 with zero discrepancies
- Every file in the `data` directory parses as valid JSON and yields a non-empty cluster identifier

## Limitations

- Counts depend on using the repository snapshot that exactly corresponds to the release; mixing entries from different versions will break the novelty delta
- Novelty classification relies on entry version/quality-change metadata fields being present and correctly populated in the JSON entries
- 69 duplicate and low-quality entries were removed in MIBiG 3.0, so raw file counts from earlier snapshots will not reconcile with the 2502 total without accounting for removals

## Evidence

- [readme] Data directory layout: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [intro] Novel entries and removals: "In addition, we added 661 novel BGCs to the MIBiG database which were published since the last database update and removed 69 duplicate and low-quality entries"
- [abstract] Prior baseline: "was expanded to 2021 entries in 2019 (MIBiG 2.0)"
- [results] Total and taxonomy coverage: [evidence span withheld — anchor status: not_found]
- [supplementary] Sequence files source: "The `genbanks` directory contains a handful of sequence files that are not available from NCBI's GenBank/RefSeq databases."
