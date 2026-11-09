---
name: smiles-syntax-validation
description: Use when a curation dataset stores chemical structures as SMILES strings
  (e.g. in JSON entries) and you must confirm every string is syntactically valid
  before release or downstream use.
when_to_use_negative:
- Structures are stored in a format other than SMILES (e.g. only InChI or Molfiles),
  so PIKAChU syntax parsing is not applicable
- You need to verify chemical correctness or identity of a structure rather than SMILES
  syntax — parsing success does not confirm the structure is the right molecule
- Entries are already published MIBiG data that have passed the community curation
  and validation cycle described for MIBiG 3.0
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0154
tools:
- name: PIKAChU
  role: Parses each extracted SMILES string to check correctness of SMILES syntax
    and record whether it parses successfully
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:fff2fd43361a2ddfa183fa920d6fa9c09067a575b87b1ed2a45bc6864d119cf5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# smiles-syntax-validation

## Summary

Validates the syntactic correctness of SMILES strings collected for compound structures in MIBiG 3.0 by parsing each string with the PIKAChU toolkit and flagging any that fail to parse.

## When to use

Use when a curation dataset stores chemical structures as SMILES strings (e.g. in JSON entries) and you must confirm every string is syntactically valid before release or downstream use.

## When NOT to use

- Structures are stored in a format other than SMILES (e.g. only InChI or Molfiles), so PIKAChU syntax parsing is not applicable
- You need to verify chemical correctness or identity of a structure rather than SMILES syntax — parsing success does not confirm the structure is the right molecule
- Entries are already published MIBiG data that have passed the community curation and validation cycle described for MIBiG 3.0

## Inputs

- JSON-formatted MIBiG 3.0 entries from the mibig-secmet/mibig-json GitHub repository
- SMILES strings extracted from chemical structure fields of each entry
- SMILES strings extracted from protein-domain substrate fields (e.g. A-domain substrates) of each entry

## Outputs

- smiles_validation_report.csv containing one validation status per entry, with all flagged (invalid/non-parseable) SMILES explicitly marked

## How to apply

Retrieve the JSON-formatted MIBiG 3.0 entries from the mibig-secmet/mibig-json GitHub repository, then parse each entry and extract all SMILES strings from both the chemical structure fields and the protein-domain substrate fields. Feed every extracted SMILES string through the PIKAChU toolkit (used in the paper because SMILES correctness was a curation focus alongside compound structure cross-linking and protein domain selectivity) and record whether each string parses successfully. Aggregate the per-string parse outcomes into a single validation report, explicitly marking any string that fails syntax validation or cannot be parsed. Because MIBiG 3.0 curation included correction of SMILES during annotation, strings the curators already fixed should not surface as unresolved failures.

## Related tools

- **PIKAChU** (Parses each extracted SMILES string to check correctness of SMILES syntax and record whether it parses successfully)

## Evaluation signals

- smiles_validation_report.csv exists and contains one validation status per entry
- Every invalid or non-parseable SMILES string is explicitly marked as flagged in the report
- SMILES strings corrected during MIBiG 3.0 curation do not appear as unresolved failures in the report
- Every SMILES in both chemical structure fields and protein-domain substrate fields was extracted and tested, none skipped

## Limitations

- Parsing success only confirms syntax validity, not that the SMILES encodes the chemically correct structure for the compound
- Strings may legitimately fail if they were annotated incorrectly upstream, so failures must be traced back to curation rather than silently discarded
- Substrate SMILES in protein-domain fields (e.g. the >500 known A-domain substrates) may be more heterogeneous than compound-structure SMILES, complicating uniform validation

## Evidence

- [intro] PIKAChU used for SMILES syntax validation: "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [abstract] curation focus on compound structures and domain selectivities: "Particular attention was paid to the annotation of compound structures and biological activities, as well as protein domain selectivities."
- [intro] scope of chemical structure annotation in MIBiG 3.0: "We focused on three features: the characterisation and cross-linking of 1188 chemical structures, the annotation of 1002 bioactivities of BGC products, and the validation and annotation of 2020"
- [readme] data location for MIBiG JSON entries: "This repository tracks the current [MIBiG] annotations in JSON format."
