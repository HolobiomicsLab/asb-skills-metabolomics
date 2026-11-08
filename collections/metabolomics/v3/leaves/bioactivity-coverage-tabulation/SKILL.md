---
name: bioactivity-coverage-tabulation
description: Use when a MIBiG (or BGC-database) JSON export must be summarised into
  a coverage table counting compounds with bioactivity and molecular-target annotations
  per functional category.
when_to_use_negative:
- Input is raw BGC nucleotide sequence or genbank files needing cluster detection
  (use antiSMASH/GECCO instead, not annotation tabulation)
- The goal is structural cross-linking of compounds to NP Atlas/PubChem/ChemSpider/LOTUS/ChEMBL
  rather than bioactivity/target coverage counting
- The dataset is not MIBiG-standardised JSON and lacks bioactivity annotation fields
  — term normalisation against the 68 functional categories cannot be performed
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: mibig-secmet/mibig-json
  role: Source repository providing the JSON-formatted MIBiG annotation data parsed
    by this skill
  repo: https://github.com/mibig-secmet
- name: MIBiG 3.0
  role: Curated database whose entries supply the bioactivity and chemical target
    annotation fields being tabulated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:90dbe34c16faeb23e0c5cb0ad79b8920d8140a173789a8c0db0194aab681223e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG 3.0
  - mibig-secmet/mibig-json
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bioactivity-coverage-tabulation

## Summary

Tabulate per-functional-category bioactivity coverage from MIBiG 3.0 JSON data by parsing every cluster entry, normalising bioactivity terms, and flagging compounds with chemical target annotations. Reproduces the paper's headline coverage figures (1002 compounds, 51 unique bioactivities, 68 functional categories, 95 target-annotated compounds).

## When to use

Use when a MIBiG (or BGC-database) JSON export must be summarised into a coverage table counting compounds with bioactivity and molecular-target annotations per functional category.

## When NOT to use

- Input is raw BGC nucleotide sequence or genbank files needing cluster detection (use antiSMASH/GECCO instead, not annotation tabulation)
- The goal is structural cross-linking of compounds to NP Atlas/PubChem/ChemSpider/LOTUS/ChEMBL rather than bioactivity/target coverage counting
- The dataset is not MIBiG-standardised JSON and lacks bioactivity annotation fields — term normalisation against the 68 functional categories cannot be performed

## Inputs

- MIBiG 3.0 JSON-formatted cluster entries from the mibig-secmet/mibig-json `data` directory
- MIBiG 3.0 standardised functional-category mapping (68 categories)

## Outputs

- bioactivity_coverage_table.csv — per-functional-category counts of compounds with bioactivity annotations and flags for compounds with chemical target data
- Validation summary comparing tallies to reported figures (1002 compounds, 51 bioactivities, 68 categories, 95 target-annotated compounds)

## How to apply

Obtain the MIBiG 3.0 JSON-formatted data from the mibig-secmet/mibig-json repository, where datasets live in the `data` directory and entry status is tracked via the `cluster.status` field. Parse every cluster entry with Python's json/pandas libraries, extracting the bioactivity annotation fields (bioactivity terms and chemical target annotations) from each entry. Normalise the extracted activity terms into the 51 unique bioactivities and map each onto the 68 standardised functional categories used in MIBiG 3.0 curation. Tally, per functional category, the number of compounds carrying bioactivity annotations and separately flag the compounds that additionally have chemical target data. Validate the tallies against the published figures — 1002 annotated compounds, 51 unique bioactivities, 68 functional categories, 95 compounds with targets — and write the resulting table to bioactivity_coverage_table.csv.

## Related tools

- **mibig-secmet/mibig-json** (Source repository providing the JSON-formatted MIBiG annotation data parsed by this skill) — https://github.com/mibig-secmet
- **MIBiG 3.0** (Curated database whose entries supply the bioactivity and chemical target annotation fields being tabulated)

## Examples

```
import json,pandas as pd; entries=json.load(open('data/MIBiG.json')); df=pd.json_normalize(entries); df.to_csv('bioactivity_coverage_table.csv')
```

## Evaluation signals

- Total compounds with bioactivity annotations equals 1002
- Number of distinct normalised bioactivity terms equals 51
- Number of functional categories represented equals 68
- Number of compounds flagged with chemical target data equals 95
- Output CSV parses as a valid table with one row per functional category and counts summing to the expected totals

## Limitations

- Counts depend on the MIBiG 3.0 release version; the repository notes entry status is tracked via the `cluster.status` field, so status filtering may shift tallies
- Term normalisation to the 51 bioactivities and 68 functional categories assumes the paper's standardised curation mapping; other vocabularies will not reconcile
- The `genbanks` directory only holds sequence files not available from NCBI GenBank/RefSeq — it is not an input for this tabulation

## Evidence

- [readme] workflow step 1: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [intro] objective figures: "We focused on three features: the characterisation and cross-linking of 1188 chemical structures, the annotation of 1002 bioactivities of BGC products"
- [supplementary] data standard provenance: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
- [abstract] annotation emphasis: "Particular attention was paid to the annotation of compound structures and biological activities, as well as protein domain selectivities."
