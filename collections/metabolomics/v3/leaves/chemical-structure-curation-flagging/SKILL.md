---
name: chemical-structure-curation-flagging
description: Use when a chemical structure dataset (e.g. MIBiG JSON entries with compound
  SMILES and substrate SMILES fields) needs syntax-level correctness checking before
  downstream ML training or database release.
when_to_use_negative:
- Input SMILES are already provided by a curated chemical database cross-link (NP
  Atlas, PubChem, ChemSpider, LOTUS, ChEMBL) with known-valid structures — validation
  adds little here
- The goal is validating chemical correctness (stereochemistry, valence) of a drawn
  structure rather than SMILES string syntax/parsability
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: PIKAChU
  role: Parses each extracted SMILES string to check syntax correctness; strings that
    fail to parse are flagged in the validation report
- name: mibig-secmet/mibig-json
  role: Source repository of MIBiG curation data in JSON format from which SMILES
    strings are extracted
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:84da98b8f2d1c508aa3ce07b85b82e57ab9f936c7ed9098320d0be69812db9f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  - mibig-secmet/mibig-json
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# chemical-structure-curation-flagging

## Summary

Validates the SMILES syntax of compound structures and protein-domain substrate annotations in MIBiG 3.0 JSON entries by parsing each string with the PIKAChU toolkit, and flags any string that fails syntax validation.

## When to use

Use when a chemical structure dataset (e.g. MIBiG JSON entries with compound SMILES and substrate SMILES fields) needs syntax-level correctness checking before downstream ML training or database release.

## When NOT to use

- Input SMILES are already provided by a curated chemical database cross-link (NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL) with known-valid structures — validation adds little here
- The goal is validating chemical correctness (stereochemistry, valence) of a drawn structure rather than SMILES string syntax/parsability

## Inputs

- MIBiG 3.0 JSON-formatted entries from the mibig-secmet/mibig-json GitHub repository (data directory)
- SMILES strings extracted from chemical structures fields
- SMILES strings extracted from protein-domain substrate fields

## Outputs

- smiles_validation_report.csv with one validation status per entry
- Explicitly flagged list of invalid or non-parseable SMILES strings

## How to apply

Retrieve the MIBiG 3.0 JSON-formatted entries from the mibig-secmet/mibig-json GitHub repository, where current annotations live in the `data` directory and entry status is tracked via the `cluster.status` field. Parse each JSON entry and extract every SMILES string from both the chemical structure fields and the protein-domain substrate fields. Feed each extracted SMILES string through the PIKAChU toolkit, recording whether it parses successfully as a per-entry validation status. Aggregate the per-entry parse outcomes into a single report (smiles_validation_report.csv), explicitly marking any flagged (invalid/non-parseable) SMILES; strings the paper corrected during curation must not remain as unresolved failures. The rationale is that MIBiG 3.0 specifically targeted annotation quality of compound structures, so syntax errors would undermine the database's intended use for training ML models of sequence-structure-function relationships.

## Related tools

- **PIKAChU** (Parses each extracted SMILES string to check syntax correctness; strings that fail to parse are flagged in the validation report)
- **mibig-secmet/mibig-json** (Source repository of MIBiG curation data in JSON format from which SMILES strings are extracted) — https://github.com/mibig-secmet

## Evaluation signals

- smiles_validation_report.csv exists and contains one validation status per MIBiG JSON entry
- Every invalid or non-parseable SMILES string is explicitly marked (no silent drops)
- SMILES strings corrected during MIBiG curation do not appear as unresolved failures in the report
- Both chemical-structure SMILES and protein-domain substrate SMILES are covered by the validation
- Successfully parsed SMILES correspond to entries whose cluster.status in the JSON indicates accepted curation

## Limitations

- Syntax validation only confirms that PIKAChU can parse the SMILES string; it does not guarantee the structure is the correct compound for the BGC
- SMILES for protein-domain substrates must also be validated, as these can fail independently of compound-structure SMILES

## Evidence

- [intro] PIKAChU validates MIBiG SMILES syntax: "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [readme] MIBiG JSON data location and status tracking: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [supplementary] Source of JSON data standard: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
