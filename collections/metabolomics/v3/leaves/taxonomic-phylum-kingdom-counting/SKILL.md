---
name: taxonomic-phylum-kingdom-counting
description: Use when you have a directory of structured JSON database entries with
  taxonomy fields and need to compute diversity counts (distinct phyla/kingdoms) to
  validate reported database statistics.
when_to_use_negative:
- The input is raw DNA/GenBank sequence files (e.g. the `genbanks` directory) rather
  than structured JSON annotation entries — taxonomy fields are only present in the
  JSON standard
- The database version is not MIBiG 3.0 (or a release with matching entry set), since
  the expected counts (16 phyla, 5 kingdoms) are version-specific and will not reconcile
  against older releases
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: mibig-secmet/mibig-json
  role: Source repository holding the JSON-formatted MIBiG annotation standard whose
    per-entry taxonomy fields are aggregated for the phylum/kingdom counts
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:222e2675a6d95870eab7f59092be09a34a629d61fd68d1ab4eb7813d08a11dc0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - mibig-secmet/mibig-json
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# taxonomic-phylum-kingdom-counting

## Summary

Aggregates taxonomy fields across all MIBiG JSON entry files to count the distinct phyla and kingdoms spanned by the database, enabling reproduction of the MIBiG 3.0 reported coverage (16 phyla across 5 kingdoms of life).

## When to use

Use when you have a directory of structured JSON database entries with taxonomy fields and need to compute diversity counts (distinct phyla/kingdoms) to validate reported database statistics.

## When NOT to use

- The input is raw DNA/GenBank sequence files (e.g. the `genbanks` directory) rather than structured JSON annotation entries — taxonomy fields are only present in the JSON standard
- The database version is not MIBiG 3.0 (or a release with matching entry set), since the expected counts (16 phyla, 5 kingdoms) are version-specific and will not reconcile against older releases

## Inputs

- MIBiG 3.0 JSON entry files from mibig-secmet/mibig-json (data directory, one JSON file per BGC entry with cluster.taxonomy fields)

## Outputs

- Count of distinct phyla across all entries (expected: 16)
- Count of distinct kingdoms across all entries (expected: 5)
- mibig3_stats.csv containing total entries, novel BGC count, phylum count, and kingdom count for comparison against Table 2

## How to apply

Download the MIBiG 3.0 JSON annotations from the mibig-secmet/mibig-json GitHub repository, where the current datasets live in the `data` directory. Parse every JSON entry with Python's json module, extracting the cluster identifier and taxonomy fields from each file, ignoring files in the `genbanks` directory which contain sequence data rather than annotation records. Aggregate the taxonomy fields across all entries and count the distinct phyla and kingdoms represented. Compare the computed counts against the values stated in the Results And Discussion / Table 2 for MIBiG 3.0: 16 phyla across 5 kingdoms. Note that per-entry status is tracked via the `cluster.status` field, which can be used to confirm which entries are active when deduplicating taxonomy contributions.

## Related tools

- **mibig-secmet/mibig-json** (Source repository holding the JSON-formatted MIBiG annotation standard whose per-entry taxonomy fields are aggregated for the phylum/kingdom counts) — https://github.com/mibig-secmet

## Examples

```
import json, glob; entries=[json.load(open(f)) for f in glob.glob('mibig-json/data/*.json')]; print(len(entries), {e['cluster']['status'] for e in entries})
```

## Evaluation signals

- Computed distinct phylum count equals 16 and distinct kingdom count equals 5, matching the MIBiG 3.0 Results And Discussion / Table 2 values
- Total parsed JSON entry files equals 2502, the reported MIBiG 3.0 database size
- Entries flagged as new since MIBiG 2.0 sum to 661, consistent with the 2021-entry 2019 release plus new additions minus 69 removed duplicate/low-quality entries
- Every parsed entry yields a non-empty taxonomy field; entries lacking taxonomy data are logged rather than silently dropped from the phylum/kingdom tallies

## Limitations

- Counts depend on the release snapshot; MIBiG was expanded to 2021 entries in 2019 (MIBiG 2.0) and 69 duplicate and low-quality entries were removed for 3.0, so older or unofficial JSON dumps will not reproduce the 16 phyla / 5 kingdoms figures
- Taxonomy aggregation relies on the JSON standard's taxonomy annotations being complete and consistently curated; minimal or partially curated entries may under-represent an organism's full lineage

## Evidence

- [readme] JSON data directory location: "The current datasets for [MIBiG] live in the `data` directory, entry status is
now tracked via the `cluster.status` field."
- [abstract] MIBiG 3.0 update composition: "we describe MIBiG 3.0, a database update comprising large-scale validation and re-annotation of existing entries and 661 new entries."
- [intro] Entry removal in 3.0: "In addition, we added 661 novel BGCs to the MIBiG database which were published since the last database update and removed 69 duplicate and low-quality entries"
- [readme] Repository layout caveat: "The `genbanks` directory contains a handful of sequence
files that are not available from NCBI's GenBank/RefSeq databases."
