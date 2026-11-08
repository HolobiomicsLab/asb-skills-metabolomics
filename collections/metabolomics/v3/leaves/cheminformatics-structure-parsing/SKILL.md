---
name: cheminformatics-structure-parsing
description: Use when a dataset contains SMILES strings for small molecules or enzyme
  substrate structures (e.g. JSON curation entries) and you must confirm each string
  parses as syntactically correct before downstream use or database release.
when_to_use_negative:
- The strings to check are InChI or other structure identifiers rather than SMILES
  — PIKAChU syntax validation as applied here targets SMILES
- The goal is predicting or comparing chemical structures (e.g. BGC-to-structure prediction),
  not verifying the syntax of already-curated structure annotations
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3315
tools:
- name: PIKAChU
  role: Chemistry toolkit used to validate the correctness of SMILES syntax for compound
    structures and protein-domain substrates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:efc03664395c246b46f0dd1cd6bdd22fed39dcdb31a677805eb28cd4383ae5a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cheminformatics-structure-parsing

## Summary

A cheminformatics skill for validating the correctness of SMILES strings encoding chemical structures and protein-domain substrates, exemplified by the PIKAChU-based syntax validation used to curate compound structures in MIBiG 3.0.

## When to use

Use when a dataset contains SMILES strings for small molecules or enzyme substrate structures (e.g. JSON curation entries) and you must confirm each string parses as syntactically correct before downstream use or database release.

## When NOT to use

- The strings to check are InChI or other structure identifiers rather than SMILES — PIKAChU syntax validation as applied here targets SMILES
- The goal is predicting or comparing chemical structures (e.g. BGC-to-structure prediction), not verifying the syntax of already-curated structure annotations

## Inputs

- JSON-formatted curation entries (e.g. MIBiG annotations from the mibig-secmet/mibig-json repository `data` directory)
- SMILES strings extracted from chemical structure fields
- SMILES strings extracted from protein-domain substrate fields

## Outputs

- smiles_validation_report.csv with one validation status per entry
- Flagged list of SMILES strings that fail syntax validation or cannot be parsed

## How to apply

Retrieve the JSON-formatted curation entries (for MIBiG 3.0 these come from the mibig-secmet/mibig-json GitHub repository, where the `data` directory tracks current annotations). Parse each JSON entry and extract every SMILES string from both the chemical structure fields and the protein-domain substrate fields, since MIBiG 3.0 curators annotated compound structures (1188 structures cross-linked) and NRPS A-domain substrates (2020 domains). Feed each extracted SMILES through the PIKAChU chemistry toolkit to check whether it parses successfully, recording a per-string validation status. Aggregate the outcomes into a single report (smiles_validation_report.csv) that one status per entry and explicitly flags every SMILES that fails syntax validation or cannot be parsed; any strings the paper corrected during curation must not remain as unresolved failures.

## Related tools

- **PIKAChU** (Chemistry toolkit used to validate the correctness of SMILES syntax for compound structures and protein-domain substrates)

## Evaluation signals

- smiles_validation_report.csv exists and contains exactly one validation status per MIBiG JSON entry
- Every SMILES string that fails parsing or syntax validation is explicitly marked in the report — no silent drops
- No SMILES string that the MIBiG 3.0 curators corrected during annotation appears as an unresolved failure in the final report
- Both chemical-structure fields and protein-domain substrate fields are covered by the validation, matching the paper's scope of 1188 chemical structures and 2020 protein domain substrates

## Limitations

- PIKAChU-based checking validates SMILES syntax only — parseability does not guarantee the structure annotation itself is chemically correct
- The validation depends on the repository JSON layout; the README notes entry status is tracked via the `cluster.status` field, so outdated or superseded entries must be handled when iterating over the `data` directory

## Evidence

- [intro] PIKAChU validates SMILES syntax: "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [readme] MIBiG JSON repository contents: "This repository tracks the current [MIBiG] annotations in JSON format."
