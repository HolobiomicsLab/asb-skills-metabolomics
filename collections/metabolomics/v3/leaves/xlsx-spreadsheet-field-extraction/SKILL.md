---
name: xlsx-spreadsheet-field-extraction
description: Use when you have a deposited .xlsx chemical database file and need to
  pull per-entry name, SMILES, InChI, InChIKey and IUPAC fields for validation, comparison
  or downstream identifier generation.
when_to_use_negative:
- Input is a raw .mol or .cdxml structure file rather than the consolidated .xlsx
  database — use the structure-to-identifier conversion skill instead
- Only a subset of entries is needed and a quick manual lookup suffices; the scripted
  extraction is designed for the full 582-entry batch
- The target is predicted MS/MS fragment data in the interactive web datatables, which
  lives outside the .xlsx file
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3473
tools:
- name: pandas
  role: loads the .xlsx file and extracts the name, SMILES, InChI, InChIKey and IUPAC
    nomenclature columns for all 582 entries
- name: Open Babel
  role: used to generate the SMILES, InChI and InChIKey identifiers that the extracted
    .xlsx fields are compared against
- name: ChemDraw
  role: source of the .cdxml/.mol structures and IUPAC nomenclature originally written
    into the .xlsx database
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:f3ab85069ea582e2a113a33487f16aa9ecde08667d40a6273d9380f065f7ca89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - Open Babel
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# xlsx-spreadsheet-field-extraction

## Summary

Extracts structured chemical-identifier fields (name, SMILES, InChI, InChIKey, IUPAC nomenclature) from the .xlsx database file accompanying the 582-entry DNA adductomics database, so that each entry can be validated or compared against regenerated identifiers.

## When to use

Use when you have a deposited .xlsx chemical database file and need to pull per-entry name, SMILES, InChI, InChIKey and IUPAC fields for validation, comparison or downstream identifier generation.

## When NOT to use

- Input is a raw .mol or .cdxml structure file rather than the consolidated .xlsx database — use the structure-to-identifier conversion skill instead
- Only a subset of entries is needed and a quick manual lookup suffices; the scripted extraction is designed for the full 582-entry batch
- The target is predicted MS/MS fragment data in the interactive web datatables, which lives outside the .xlsx file

## Inputs

- .xlsx database file with 582 DNA adduct entries (name, SMILES, InChI, InChIKey, IUPAC nomenclature columns)
- optionally, regenerated SMILES/InChI/InChIKey from .mol files via Open Babel 3.1.1 for cross-checking

## Outputs

- per-entry field table (582 rows) with deposited name, SMILES, InChI, InChIKey and IUPAC nomenclature
- identifier_validation.csv with deposited vs regenerated identifiers and populated/matched status flags

## How to apply

Load the .xlsx database file with pandas, treating each row as one of the 582 DNA adduct entries (279 literature-reported plus 303 suspected adducts). Extract the name, SMILES, InChI, InChIKey and IUPAC nomenclature columns, handling the positively charged ring-nitrogen adduct representations consistently so protonation states match the deposited values. Cross-check each extracted field against either the corresponding column in the same file or a regenerated identifier (e.g. SMILES/InChI/InChIKey produced from the .mol files via Open Babel) to flag missing or non-matching values. Write a per-entry comparison table with columns for deposited and regenerated identifiers plus a populated/matched status flag; in this paper's validation the report contained exactly 582 rows.

## Related tools

- **pandas** (loads the .xlsx file and extracts the name, SMILES, InChI, InChIKey and IUPAC nomenclature columns for all 582 entries)
- **Open Babel** (used to generate the SMILES, InChI and InChIKey identifiers that the extracted .xlsx fields are compared against)
- **ChemDraw** (source of the .cdxml/.mol structures and IUPAC nomenclature originally written into the .xlsx database)

## Examples

```
df = pd.read_excel('dna_adduct_database.xlsx'); ids = df[['name','SMILES','InChI','InChIKey','IUPAC nomenclature']].to_csv('identifier_validation.csv', index=False)
```

## Evaluation signals

- The extracted table contains exactly 582 rows, one per database entry
- The count of entries with populated SMILES, InChI and InChIKey fields is reported and no field is silently dropped
- Regenerated SMILES/InChI/InChIKey match the deposited values entry-by-entry; mismatched rows are flagged, not silently overwritten
- Positively charged ring-nitrogen adduct representations are handled consistently, so identical chemical entries yield identical strings

## Limitations

- Field extraction only covers the .xlsx spreadsheet; experimental MS/MS spectra of 15 reference standards and in silico fragments from CFM-ID are in separate interactive web datatables, not in the spreadsheet
- Charged species (ring-nitrogen adducts) need explicit handling to avoid false mismatches between deposited and regenerated identifiers
- Version sensitivity: identifiers were generated with Open Babel 3.1.1; other versions may produce different canonical strings

## Evidence

- [abstract] Database file field extraction: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] Database scale: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [results] Identifier comparison: [evidence span withheld — anchor status: not_found]
