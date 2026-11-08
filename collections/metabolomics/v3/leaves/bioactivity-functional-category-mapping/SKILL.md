---
name: bioactivity-functional-category-mapping
description: Use when you need to quantify how many compounds in a biosynthetic gene
  cluster database carry bioactivity or molecular-target annotations, or when building
  a per-functional-category coverage table from MIBiG JSON data.
when_to_use_negative:
- You need bioactivity annotations for BGCs that lack experimentally validated characterisation
  — MIBiG only contains experimentally validated BGC entries, so the table cannot
  generalise to predicted/unvalidated clusters
- You want automated activity prediction from sequence — this skill only tallies manually
  curated annotations, it does not predict bioactivity (see e.g. the Walker and Clardy
  ML model for that)
- Your input is raw GenBank/RefSeq sequence rather than curated MIBiG JSON entries
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0780
tools:
- name: Python json/pandas
  role: Parse every MIBiG 3.0 cluster JSON entry and extract/tabulate compound bioactivity
    and chemical-target annotation fields
  repo: https://github.com/mibig-secmet
- name: manual expert review
  role: Community curation step that assigned bioactivity terms to the 68 standardised
    functional categories underlying the coverage table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:34d07ed43e54695925078e0d0a7ff75dcb476f1857c9fce9b9df3843bef4daa7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python json/pandas
  - manual expert review
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bioactivity-functional-category-mapping

## Summary

Parses MIBiG 3.0 JSON cluster entries to tally compounds with bioactivity annotations (1002 compounds, 51 unique bioactivities) and chemical target annotations (95 compounds), then normalises activity terms onto the 68 standardised functional categories to build a per-category coverage table.

## When to use

Use when you need to quantify how many compounds in a biosynthetic gene cluster database carry bioactivity or molecular-target annotations, or when building a per-functional-category coverage table from MIBiG JSON data.

## When NOT to use

- You need bioactivity annotations for BGCs that lack experimentally validated characterisation — MIBiG only contains experimentally validated BGC entries, so the table cannot generalise to predicted/unvalidated clusters
- You want automated activity prediction from sequence — this skill only tallies manually curated annotations, it does not predict bioactivity (see e.g. the Walker and Clardy ML model for that)
- Your input is raw GenBank/RefSeq sequence rather than curated MIBiG JSON entries

## Inputs

- MIBiG 3.0 JSON-formatted data files from the mibig-secmet/mibig-json repository `data` directory (one JSON entry per BGC, with compound bioactivity and chemical target annotation fields)
- MIBiG 3.0 standardised functional-category scheme (68 categories) used in the community curation

## Outputs

- bioactivity_coverage_table.csv — per-functional-category counts of compounds with bioactivity annotations, plus flags for the 95 compounds annotated to molecular targets
- Validation report confirming tallies match 1002 annotated compounds, 51 unique bioactivities, 68 functional categories, 95 compounds with chemical targets

## How to apply

Obtain the MIBiG 3.0 JSON-formatted data from the mibig-secmet/mibig-json GitHub repository, where the current datasets live in the `data` directory and entry status is tracked via the `cluster.status` field. Parse every cluster entry with Python's json/pandas libraries, extracting the compound bioactivity annotation fields (bioactivity terms and chemical target annotations) from each entry. Normalise the extracted activity terms into the 51 unique bioactivities and map each onto the 68 standardised functional categories used in the MIBiG 3.0 curation. Tally, per functional category, the number of compounds carrying bioactivity annotations and separately flag the compounds that additionally have chemical target data. Validate the tallies against the reported figures (1002 annotated compounds, 51 unique bioactivities, 68 functional categories, 95 compounds with targets) and write the result to bioactivity_coverage_table.csv.

## Related tools

- **Python json/pandas** (Parse every MIBiG 3.0 cluster JSON entry and extract/tabulate compound bioactivity and chemical-target annotation fields) — https://github.com/mibig-secmet
- **manual expert review** (Community curation step that assigned bioactivity terms to the 68 standardised functional categories underlying the coverage table)

## Examples

```
import json, pandas as pd; entries = [json.load(open(f)) for f in glob.glob('mibig-json/data/*.json')]; df = pd.DataFrame([c['compounds'] for c in entries])
```

## Evaluation signals

- Compound count with bioactivity annotations equals 1002
- Number of unique normalised bioactivity terms equals 51
- Number of functional categories in the coverage table equals 68
- Number of compounds flagged with chemical target data equals 95
- Output CSV row counts and per-category sums reconcile with the compound-level annotation tallies (no double-counting across categories)

## Limitations

- Coverage reflects only curated MIBiG 3.0 entries; 69 duplicate and low-quality entries were removed in the update, so counts depend on the curation snapshot
- The mapping relies on manual expert curation of activity terms into the 68 functional categories, so newly published BGCs (e.g. the 661 added since the last update) may shift the tallies in later database versions
- Compounds without bioactivity or target annotations are invisible to this tally — coverage is a subset of the 1188 characterised chemical structures

## Evidence

- [readme] workflow step 1: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [intro] objective with 1002 bioactivities: "the annotation of 1002 bioactivities of BGC products, and the validation and annotation of 2020"
- [abstract] attention to bioactivity annotation: "Particular attention was paid to the annotation of compound structures and biological activities, as well as protein domain selectivities."
- [intro] database curation scope: "we added 661 novel BGCs to the MIBiG database which were published since the last database update and removed 69 duplicate and low-quality entries"
