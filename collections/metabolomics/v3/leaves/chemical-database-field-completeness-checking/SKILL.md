---
name: chemical-database-field-completeness-checking
description: Use when a curated chemical/adduct database is distributed as structure
  files (.mol/.cdxml) plus a spreadsheet, and you must verify identifier columns are
  populated and reproducible entry-by-entry.
when_to_use_negative:
- The input structures already carry canonical identifiers and the goal is spectral
  matching or fragment prediction (CFM-ID) rather than identifier completeness auditing
- No machine-readable structure files (.mol) are available, so identifiers cannot
  be independently regenerated and only presence/absence can be checked
- The task is generating new identifiers for novel adducts rather than auditing an
  existing deposited database
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Open Babel
  role: Batch-converts the 582 .mol files (optionally via .SDF) into SMILES, InChI
    and InChIKey for comparison against deposited fields; version 3.1.1 as used in
    the paper
- name: ChemDraw
  role: Source of the .cdxml structure files that were converted into .mol files and
    IUPAC nomenclature with ChemScript, forming the input structures being audited
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:a0c06f20b62b4b1d9c52baedd12b4b9a2f0c7e8db7244aa7b10ee6418f90fb1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - Open Babel
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Chemical database field completeness checking

## Summary

Batch-validates that every chemical database entry carries populated, machine-readable structural identifiers (SMILES, InChI, InChIKey) by regenerating them from deposited .mol files with Open Babel 3.1.1 and diffing against the deposited .xlsx fields.

## When to use

Use when a curated chemical/adduct database is distributed as structure files (.mol/.cdxml) plus a spreadsheet, and you must verify identifier columns are populated and reproducible entry-by-entry.

## When NOT to use

- The input structures already carry canonical identifiers and the goal is spectral matching or fragment prediction (CFM-ID) rather than identifier completeness auditing
- No machine-readable structure files (.mol) are available, so identifiers cannot be independently regenerated and only presence/absence can be checked
- The task is generating new identifiers for novel adducts rather than auditing an existing deposited database

## Inputs

- Zipped deposit from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database containing one .xlsx file and 582 .mol structure files
- .mol (MDL Molfile) files, optionally converted through .SDF intermediates
- Deposited .xlsx database columns: name, SMILES, InChI, InChIKey, IUPAC nomenclature

## Outputs

- identifier_validation.csv with 582 rows (one per database entry) and columns for entry name, deposited and regenerated SMILES/InChI/InChIKey, and a populated/matched status flag
- Counts of entries with populated SMILES, InChI and InChIKey fields

## How to apply

Retrieve the zipped deposit (.xlsx, .cdxml, .mol files) from the GitLab repository and extract the 582 .mol structure files. Batch-convert each .mol file to SMILES, InChI and InChIKey with Open Babel version 3.1.1 (the version used in the paper), optionally via .SDF intermediates. Load the .xlsx with pandas and pull the name, SMILES, InChI, InChIKey and IUPAC nomenclature columns for all 582 entries (279 literature-reported plus 303 suspected adducts). Compare each regenerated identifier against the deposited field entry-by-entry, handling positively charged ring-nitrogen adduct representations consistently. Write the per-entry result to identifier_validation.csv with deposited vs regenerated values and a populated/matched status flag.

## Related tools

- **Open Babel** (Batch-converts the 582 .mol files (optionally via .SDF) into SMILES, InChI and InChIKey for comparison against deposited fields; version 3.1.1 as used in the paper)
- **ChemDraw** (Source of the .cdxml structure files that were converted into .mol files and IUPAC nomenclature with ChemScript, forming the input structures being audited)

## Examples

```
obabel -imol adducts/*.mol -osmi -O identifier_validation_input.smi
```

## Evaluation signals

- identifier_validation.csv contains exactly 582 rows, one per database entry (279 literature-reported plus 303 suspected adducts)
- The report states the count of entries with populated SMILES, InChI and InChIKey fields, and no entry has an unflagged empty identifier field
- Regenerated SMILES, InChI and InChIKey match the corresponding deposited .xlsx values entry-by-entry, with mismatches explicitly flagged
- Positively charged ring-nitrogen adduct representations are treated consistently across the deposited and regenerated identifier sets

## Limitations

- Identifier regeneration depends on the exact converter version — Open Babel 3.1.1 as used in the paper — so other versions may produce non-matching canonical forms
- Positively charged ring-nitrogen adduct representations require a consistent convention; inconsistent handling causes spurious deposited-vs-regenerated mismatches
- ChemScript-derived .mol files and IUPAC nomenclature are upstream inputs; errors made during that ChemDraw conversion propagate into the completeness check

## Evidence

- [methods] Identifier regeneration pipeline: [evidence span withheld — anchor status: not_found]
- [methods] Validation output schema: [evidence span withheld — anchor status: not_found]
- [abstract] Database scope and identifiers: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] Database availability: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
