---
name: inchikey-canonicalization-verification
description: Use when a compound database ships structure files (.mol/.cdxml) plus
  identifier columns that must be regenerated and validated, or when checking whether
  deposited SMILES/InChI/InChIKey fields are populated and canonical.
when_to_use_negative:
- When the goal is predicting MS/MS fragments for adduct annotation — that is the
  CFM-ID in silico fragment generation step, not identifier regeneration
- When no structure files (.mol/.cdxml) are available and only spectral peak lists
  must be processed
- When the deposit's identifier columns are already verified and the task is spectral
  library building from experimental MS/MS spectra of reference standards
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Open Babel
  role: Batch-converts each .mol file to SMILES, InChI and InChIKey (version 3.1.1
    as used in the paper), optionally via .SDF intermediates per section 2.3
- name: ChemDraw
  role: 'Source format for the structures: .cdxml files are the starting point that
    ChemScript converts into .mol files and IUPAC nomenclature'
- name: ChemScript
  role: Converts ChemDraw .cdxml files into .mol files and IUPAC nomenclature before
    Open Babel identifier generation
- name: pandas
  role: Loads the .xlsx database file and extracts the name, SMILES, InChI, InChIKey
    and IUPAC nomenclature columns for entry-by-entry comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:5004a96c89fa2260072c0c293ea7a53f2161f70fa9b60574434318e1d38fe1c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - ChemScript
  - Open Babel
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# inchikey-canonicalization-verification

## Summary

Regenerates and verifies the machine-readable structural identifiers (SMILES, InChI, InChIKey) deposited for every entry of a compound database, following the DNA adductomics database pipeline that converted 582 .mol structures with Open Babel 3.1.1 and cross-checked the resulting identifiers against the deposited .xlsx fields.

## When to use

Use when a compound database ships structure files (.mol/.cdxml) plus identifier columns that must be regenerated and validated, or when checking whether deposited SMILES/InChI/InChIKey fields are populated and canonical.

## When NOT to use

- When the goal is predicting MS/MS fragments for adduct annotation — that is the CFM-ID in silico fragment generation step, not identifier regeneration
- When no structure files (.mol/.cdxml) are available and only spectral peak lists must be processed
- When the deposit's identifier columns are already verified and the task is spectral library building from experimental MS/MS spectra of reference standards

## Inputs

- Zipped deposit containing .xlsx database file, .cdxml structure files, and 582 .mol structure files (from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database)
- Database table with per-entry name, SMILES, InChI, InChIKey and IUPAC nomenclature columns (readable with pandas)

## Outputs

- identifier_validation.csv with one row per database entry (582 rows): entry name, deposited SMILES/InChI/InChIKey, regenerated SMILES/InChI/InChIKey, and populated/matched status flag
- Counts of entries with populated SMILES, InChI and InChIKey fields

## How to apply

Retrieve the deposit (here .xlsx plus 582 .cdxml/.mol files from the GitLab dna_adductomics_database repo) and extract the structure files. Regenerate identifiers exactly as the paper did: ChemScript converts .cdxml files to .mol files and IUPAC nomenclature, then Open Babel 3.1.1 converts each .mol file (optionally via .SDF intermediates, section 2.3) to SMILES, InChI and InChIKey. Load the .xlsx with pandas and pull the name, SMILES, InChI, InChIKey and IUPAC nomenclature columns for all 582 entries (279 literature-reported adducts plus 303 suspected adducts). Compare each regenerated identifier against the deposited value entry-by-entry, flagging missing or non-matching fields, and treat positively charged ring-nitrogen adduct representations consistently since these are a known canonicalization edge case. Write the comparison to identifier_validation.csv with columns for entry name, deposited vs regenerated SMILES/InChI/InChIKey, and a populated/matched status flag.

## Related tools

- **Open Babel** (Batch-converts each .mol file to SMILES, InChI and InChIKey (version 3.1.1 as used in the paper), optionally via .SDF intermediates per section 2.3)
- **ChemDraw** (Source format for the structures: .cdxml files are the starting point that ChemScript converts into .mol files and IUPAC nomenclature)
- **ChemScript** (Converts ChemDraw .cdxml files into .mol files and IUPAC nomenclature before Open Babel identifier generation)
- **pandas** (Loads the .xlsx database file and extracts the name, SMILES, InChI, InChIKey and IUPAC nomenclature columns for entry-by-entry comparison)

## Examples

```
obabel dna_adducts/*.mol -osmi -O all_smiles.smi && obabel dna_adducts/*.mol -oinchi -O all_inchi.inchi
```

## Evaluation signals

- identifier_validation.csv contains exactly 582 rows, one per database entry
- The report gives counts of entries with populated SMILES, InChI and InChIKey fields, covering both the 279 literature-reported and 303 suspected adducts
- Regenerated identifiers match the corresponding deposited field entry-by-entry, with any mismatches explicitly flagged
- Positively charged ring-nitrogen adduct representations are handled consistently between deposited and regenerated identifiers
- Conversion was run with Open Babel version 3.1.1, matching the version used in the paper

## Limitations

- Identifier strings depend on the converter version — the paper pinned Open Babel 3.1.1, so regenerating with a different version may yield non-matching canonical forms
- Positively charged ring-nitrogen adducts are an edge case whose representation must be harmonized consistently between deposited and regenerated fields
- .SDF intermediates may be required for some conversions (section 2.3), so direct .mol-to-identifier conversion does not always suffice

## Evidence

- [abstract] monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database ).: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [methods] ChemDraw files (.cdxml) were converted into .mol files and IUPAC nomenclature with ChemScript, then .mol files were converted to SMILES, InChI and InChIKey using Open Babel 3.1.1: [evidence span withheld — anchor status: not_found]
