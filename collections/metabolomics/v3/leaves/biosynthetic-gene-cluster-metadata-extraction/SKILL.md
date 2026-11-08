---
name: biosynthetic-gene-cluster-metadata-extraction
description: Use when you need to enumerate or audit evidence codes and annotations
  (e.g. 13 NRPS substrate-selectivity evidence codes) directly from the MIBiG JSON
  schema or its entry JSON files.
when_to_use_negative:
- The goal is BGC detection from raw DNA sequence — use a detection tool such as antiSMASH,
  fungiSMASH, GECCO or DeepBGC instead; this skill only extracts annotations already
  curated in MIBiG.
- The input is a PKS AT-domain selectivity problem or generic chemical-structure cross-linking
  (NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL) rather than NRPS A-domain substrate-selectivity
  evidence codes from the MIBiG schema.
- Entries lack curated status in `cluster.status` (e.g. low-quality or duplicate entries
  removed in MIBiG 3.0) — verify entry quality before treating extracted annotations
  as validated.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3174
tools:
- name: mibig-json
  role: Source repository hosting the MIBiG JSON-formatted data standard whose schema
    enumerations supply the evidence codes; parsed with Python's json module.
  repo: https://github.com/mibig-secmet
- name: Python json module
  role: Loads the JSON schema definition files and traverses enumerated definitions
    to collect evidence codes.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:38837e8d85d2b4585fc9bff3a3b5c4233594afeebcc7d8fc81c6b9004ceb600e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python json module
  - mibig-json
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# biosynthetic-gene-cluster-metadata-extraction

## Summary

Extracts structured biosynthetic gene cluster (BGC) metadata — including NRPS A-domain substrate-selectivity evidence codes and their literature references — from the MIBiG JSON-formatted data standard hosted in the mibig-secmet/mibig-json repository, converting schema enumerations into tabular summaries such as a Table 1 mirror.

## When to use

Use when you need to enumerate or audit evidence codes and annotations (e.g. 13 NRPS substrate-selectivity evidence codes) directly from the MIBiG JSON schema or its entry JSON files.

## When NOT to use

- The goal is BGC detection from raw DNA sequence — use a detection tool such as antiSMASH, fungiSMASH, GECCO or DeepBGC instead; this skill only extracts annotations already curated in MIBiG.
- The input is a PKS AT-domain selectivity problem or generic chemical-structure cross-linking (NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL) rather than NRPS A-domain substrate-selectivity evidence codes from the MIBiG schema.
- Entries lack curated status in `cluster.status` (e.g. low-quality or duplicate entries removed in MIBiG 3.0) — verify entry quality before treating extracted annotations as validated.

## Inputs

- mibig-json repository JSON schema definition file(s) (cloned from github.com/mibig-secmet/mibig-json)
- MIBiG entry JSON files in the repository's `data` directory (with `cluster.status` tracking entry status)

## Outputs

- table1_evidence_codes.csv — one row per substrate-selectivity evidence code, mirroring Table 1 of the MIBiG 3.0 paper
- Validated count of exactly 13 NRPS substrate-selectivity evidence codes (or a flagged discrepancy)

## How to apply

Clone github.com/mibig-secmet/mibig-json, which tracks MIBiG curation data in JSON format (current entries live in the `data` directory, with entry status tracked via the `cluster.status` field, and a `genbanks` directory holding sequence files unavailable from NCBI GenBank/RefSeq). Load the JSON schema definition files with Python's json module and traverse the enumerated definitions, collecting every evidence code that governs substrate-selectivity annotations of protein domains. For MIBiG 3.0 the expected yield for NRPS A-domain substrate selectivity is exactly 13 evidence codes, each paired with a literature reference for the A-domain substrate annotation (2020 domain substrates were validated/annotated in this release). Write the codes one row per code to a CSV (e.g. table1_evidence_codes.csv) mirroring the paper's Table 1, and validate the count against the published figure of 13, flagging any discrepancy. Rationale: A-domain substrate prediction in NRPSs recognizes over 500 known substrates, so substrate-selectivity evidence must be captured with explicit provenance codes rather than free-text annotations.

## Related tools

- **mibig-json** (Source repository hosting the MIBiG JSON-formatted data standard whose schema enumerations supply the evidence codes; parsed with Python's json module.) — https://github.com/mibig-secmet
- **Python json module** (Loads the JSON schema definition files and traverses enumerated definitions to collect evidence codes.)

## Examples

```
git clone https://github.com/mibig-secmet/mibig-json && python -c "import json; s=json.load(open('mibig-json/schema/mibig_json_3.0.schema.json')); print(sum(1 for _ in s.get('definitions',{})))"
```

## Evaluation signals

- The output CSV contains exactly 13 evidence codes for NRPS substrate selectivity, matching the count the MIBiG 3.0 paper reports adding (Table 1); any deviation is flagged.
- Each collected evidence code corresponds to an enumerated definition in the MIBiG JSON schema (no fabricated codes).
- Each A-domain substrate-selectivity annotation is paired with a literature reference, consistent with MIBiG's annotation convention.
- Extracted entries carry a `cluster.status` field, confirming they are current MIBiG curation records from the `data` directory.

## Limitations

- The extracted codes reflect only what the MIBiG JSON schema enumerates; annotations absent from the schema (e.g. predicted rather than validated selectivities) will not be captured.
- The `genbanks` directory covers only 'a handful of sequence files that are not available from NCBI's GenBank/RefSeq databases', so sequence provenance for other entries must come from NCBI.
- NRPS A-domain substrate prediction is harder than PKS AT-domain prediction because A-domains recognize over 500 known substrates, so evidence-code coverage alone does not guarantee complete selectivity characterization.

## Evidence

- [other] Clone the mibig-json repository from github.com/mibig-secmet/mibig-json, which hosts the MIBiG JSON-formatted data standard.: [evidence span withheld — anchor status: not_found]
- [readme] This repository tracks the current [MIBiG] annotations in JSON format.: "This repository tracks the current [MIBiG] annotations in JSON format."
- [readme] The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field.: "The current datasets for [MIBiG] live in the `data` directory, entry status is
now tracked via the `cluster.status` field."
- [other] 13 evidence codes governing substrate-selectivity annotations of protein domains.: [evidence span withheld — anchor status: not_found]
- [intro] the annotation of 1002 bioactivities of BGC products, and the validation and annotation of 2020 protein domain substrates of NRPSs: [evidence span withheld — anchor status: not_found]
- [intro] for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge: "for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge"
- [intro] The `genbanks` directory contains a handful of sequence files that are not available from NCBI's GenBank/RefSeq databases.: "The `genbanks` directory contains a handful of sequence files that are not available from NCBI's GenBank/RefSeq databases."
