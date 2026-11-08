---
name: json-annotation-field-extraction
description: Use when JSON-formatted biosynthetic gene cluster (BGC) annotation entries
  (e.g. MIBiG) need their per-entry structure, bioactivity, or substrate fields parsed
  and flattened into a tabular report for validation or analysis.
when_to_use_negative:
- Input data is not MIBiG JSON (e.g. raw GenBank/RefSeq FASTA without JSON annotations)
  — there are no `cluster.status` or structure fields to extract.
- The goal is de novo BGC detection from sequence (use antiSMASH/fungiSMASH/plantiSMASH/GECCO
  etc.), not field extraction from curated annotations.
- Entries have already been extracted and flattened into a per-entry table; re-parsing
  the raw JSON would duplicate work rather than add information.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0780
tools:
- name: PIKAChU
  role: 'Downstream consumer: validates the correctness of SMILES syntax of the extracted
    structure fields from MIBiG 3.0 JSON entries'
- name: mibig-secmet/mibig-json
  role: Source repository providing the JSON-formatted MIBiG annotations that are
    parsed and field-extracted
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:fd6c2ea30e54087f21b205f9fd06efc4bdecf8ad5865f7ddcea057a086852a1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  - mibig-secmet/mibig-json
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# json-annotation-field-extraction

## Summary

Parse MIBiG 3.0 JSON-format annotations from the mibig-secmet/mibig-json repository to extract structure (SMILES), bioactivity, and protein-domain substrate fields, enabling downstream validation such as SMILES syntax checking with PIKAChU.

## When to use

Use when JSON-formatted biosynthetic gene cluster (BGC) annotation entries (e.g. MIBiG) need their per-entry structure, bioactivity, or substrate fields parsed and flattened into a tabular report for validation or analysis.

## When NOT to use

- Input data is not MIBiG JSON (e.g. raw GenBank/RefSeq FASTA without JSON annotations) — there are no `cluster.status` or structure fields to extract.
- The goal is de novo BGC detection from sequence (use antiSMASH/fungiSMASH/plantiSMASH/GECCO etc.), not field extraction from curated annotations.
- Entries have already been extracted and flattened into a per-entry table; re-parsing the raw JSON would duplicate work rather than add information.

## Inputs

- MIBiG 3.0 JSON annotation entries from the `data` directory of mibig-secmet/mibig-json
- Optionally, sequence files from the `genbanks` directory (for entries not available from NCBI GenBank/RefSeq)
- MIBiG 3.0 accession/status metadata (`cluster.status`)

## Outputs

- Tabular extraction report with one row per entry-field pair (entry ID, field type e.g. SMILES / bioactivity / domain substrate, value, status)
- Intermediate structures list ready for downstream SMILES syntax validation (e.g. PIKAChU parse checks)
- Flagged list of removed duplicate/low-quality entries (69 entries removed in MIBiG 3.0)

## How to apply

Retrieve the MIBiG 3.0 entries from the `data` directory of the mibig-secmet/mibig-json GitHub repository, where entry status is tracked via the `cluster.status` field. Load each JSON entry and walk the per-cluster fields to extract chemical structure SMILES strings (linked to NP Atlas, PubChem, ChemSpider, LOTUS, and ChEMBL), bioactivity annotations, and NRPS A-domain / PKS AT-domain substrate labels. Preserve the entry identifier with every extracted value so that downstream checks (e.g. PIKAChU SMILES syntax validation) can be traced back to individual BGC entries. Aggregate the extracted records into a single table (one row per entry-field pair), flagging entries whose status indicates a removed duplicate or low-quality record. The rationale is that MIBiG's 'central dogma' analysis (sequence → structure → function) requires machine-readable, per-entry field extraction before any cross-validation or model training can proceed.

## Related tools

- **PIKAChU** (Downstream consumer: validates the correctness of SMILES syntax of the extracted structure fields from MIBiG 3.0 JSON entries)
- **mibig-secmet/mibig-json** (Source repository providing the JSON-formatted MIBiG annotations that are parsed and field-extracted) — https://github.com/mibig-secmet

## Examples

```
python -c "import json, glob; entries=[json.load(open(f)) for f in glob.glob('data/*.json')]; smiles=[(e['cluster']['id'], s) for e in entries for s in e['cluster'].get('smiles',[])]; print(len(smiles))"
```

## Evaluation signals

- Number of extracted SMILES strings should reflect the 1188 chemical structures characterised in MIBiG 3.0.
- Extracted bioactivity annotations should total 1002 as reported for MIBiG 3.0.
- Extracted protein-domain substrate labels should total 2020 (NRPS A-domains) as reported.
- Every extracted value must retain its source entry identifier so downstream validation (e.g. PIKAChU) can be traced per entry.
- Entries flagged as removed duplicates/low-quality (69 in MIBiG 3.0) must not appear in the final extraction output.

## Limitations

- The `genbanks` directory only contains sequence files not available from NCBI GenBank/RefSeq, so some entries cannot be cross-checked against NCBI records.
- A-domain substrate prediction requires substantially more data (over 500 known substrates) than AT-domain prediction, so extracted substrate fields may be incomplete for NRPS systems.
- SMILES syntax validity is not guaranteed by extraction alone; a separate PIKAChU validation pass is required, and any corrected entries from the paper's curation should not appear as unresolved failures.

## Evidence

- [readme] Repository structure: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [intro] Cross-linked chemical databases: "cross-link them to five databases of chemical structures: NP Atlas (9), PubChem, ChemSpider (17), LOTUS (18), and ChEMBL (19)"
- [intro] PIKAChU validation of SMILES: "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [intro] Three feature objectives: "the characterisation and cross-linking of 1188 chemical structures, the annotation of 1002 bioactivities of BGC products, and the validation and annotation of 2020"
- [intro] A-domain data challenge: "for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge, which will require substantially more data"
- [intro] Removed entries: "we added 661 novel BGCs to the MIBiG database which were published since the last database update and removed 69 duplicate and low-quality entries"
