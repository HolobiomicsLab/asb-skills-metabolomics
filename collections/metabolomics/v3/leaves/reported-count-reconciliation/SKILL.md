---
name: reported-count-reconciliation
description: Use when a paper reports aggregate counts for a curated resource (e.g.
  total entries, novel additions, taxonomic coverage) and you must verify them directly
  from the released data files rather than trusting the text.
when_to_use_negative:
- The paper's claims are about per-entry annotations (e.g. 1188 chemical structures,
  1002 bioactivities, 2020 NRPS A-domain substrates) rather than whole-database entry
  counts that a file-level parse can reproduce.
- The resource data is not versioned/tracked in a public repository, so released files
  matching the paper's snapshot cannot be obtained.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3071
tools:
- name: mibig-json
  role: Source of the JSON-formatted MIBiG curation data (data directory, cluster.status
    field) that is parsed and counted
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:cd27d31fb48afcbb085761fd9cc97d6eb2b26532f5025037996056a4aa008836
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - mibig-json
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# reported-count-reconciliation

## Summary

Reproduces database-level summary statistics (total entries, newly added BGCs, phylum/kingdom counts) by parsing a database release's JSON files and reconciling computed counts against values reported in the publication (e.g. MIBiG 3.0's 2502 entries, 661 new BGCs, 16 phyla, 5 kingdoms).

## When to use

Use when a paper reports aggregate counts for a curated resource (e.g. total entries, novel additions, taxonomic coverage) and you must verify them directly from the released data files rather than trusting the text.

## When NOT to use

- The paper's claims are about per-entry annotations (e.g. 1188 chemical structures, 1002 bioactivities, 2020 NRPS A-domain substrates) rather than whole-database entry counts that a file-level parse can reproduce.
- The resource data is not versioned/tracked in a public repository, so released files matching the paper's snapshot cannot be obtained.

## Inputs

- JSON-formatted BGC entry files from the mibig-json repository `data` directory (one JSON per cluster)
- Published target statistics from the article's Results And Discussion / Table 2 (2502 entries, 661 new BGCs, 16 phyla, 5 kingdoms)

## Outputs

- mibig3_stats.csv with computed total entries, novel-entry count, phylum count, and kingdom count
- Per-field reconciliation table comparing computed counts to reported values

## How to apply

Download the released JSON entries from the paper's tracked repository — for MIBiG 3.0, the `data` directory of mibig-secmet/mibig-json, where each BGC entry is one JSON file and entry status is tracked via the `cluster.status` field. Parse every file with Python's `json` module, extracting the cluster identifier, version metadata, and taxonomy fields. Count the total number of entry files for database size; identify entries added since the previous release (MIBiG 2.0 contained 2021 entries in 2019) using the entry version/quality-change metadata to recover the 661 novel BGCs, accounting for the 69 duplicate and low-quality entries removed. Aggregate the taxonomy fields across all entries to count distinct phyla and kingdoms. Write the computed statistics (total entries, novel entries, phylum count, kingdom count) to mibig3_stats.csv and compare each value against the published figures (2502 entries, 661 new BGCs, 16 phyla, 5 kingdoms) reported in Table 2 / the Results And Discussion.

## Related tools

- **mibig-json** (Source of the JSON-formatted MIBiG curation data (data directory, cluster.status field) that is parsed and counted) — https://github.com/mibig-secmet

## Examples

```
python -c "import json,glob,csv; files=glob.glob('mibig-json/data/*.json'); entries=[json.load(open(f)) for f in files]; print(len(entries))"
```

## Evaluation signals

- Total count of parsed JSON entry files equals 2502 (MIBiG 3.0).
- Count of entries marked new since MIBiG 2.0 (2021 entries in 2019) equals 661.
- Distinct phyla aggregated across taxonomy fields equals 16; distinct kingdoms equals 5.
- Every computed value in mibig3_stats.csv matches the corresponding Table 2 / Results And Discussion value exactly.
- Parsing covers all files in the data directory, not a sample, and entries removed as duplicates/low-quality (69 in MIBiG 3.0) are not double-counted.

## Limitations

- Entry status is tracked via the `cluster.status` field, so older releases or scripts relying on directory layout rather than the status field may miscount new vs. carried-over entries.
- The `genbanks` directory contains sequence files not available from NCBI's GenBank/RefSeq — these are supplementary sequence data, not additional JSON entries, and must not be counted as database entries.
- Reconciliation depends on using the release corresponding to the paper's snapshot; the repository tracks current annotations, which may drift from the published MIBiG 3.0 state.

## Evidence

- [results] Total entries: 2502; novel BGCs: 661: [evidence span withheld — anchor status: not_found]
- [abstract] Previous release size: "was expanded to 2021 entries in 2019 (MIBiG 2.0)"
- [readme] Entry status tracking: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [readme] Genbanks caveat: "The `genbanks` directory contains a handful of sequence files that are not available from NCBI's GenBank/RefSeq databases."
- [intro] Removals: "we added 661 novel BGCs to the MIBiG database which were published since the last database update and removed 69 duplicate and low-quality entries"
