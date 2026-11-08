---
name: openbabel-batch-identifier-conversion
description: Use when a curated compound list (.mol/.SDF/.cdxml-derived) needs standardized
  computer-readable identifiers — SMILES, InChI, InChIKey — for database population
  or identifier cross-validation.
when_to_use_negative:
- Input is already a populated table of SMILES/InChI/InChIKey values that only needs
  cross-checking, not regeneration.
- Structures are supplied as non-.mol/.SDF authoring files (e.g. raw .cdxml) that
  must first be converted with ChemScript — Open Babel alone will not ingest them.
- The task concerns MS/MS spectral matching or adduct identification rather than structural
  identifier generation.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3390
tools:
- name: Open Babel
  role: Converts .mol/.SDF structure files to SMILES, InChI and InChIKey identifiers
    (version 3.1.1 used in the paper)
- name: ChemDraw
  role: Authors the .cdxml adduct structures that are converted to .mol files before
    Open Babel conversion
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:bb048420c03554f2cfd6ef18c74f41d36b4e092c6b4ba7b7082e2f95c8b80bd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - Open Babel
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3390
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Open Babel batch chemical identifier conversion (SMILES/InChI/InChIKey)

## Summary

Batch-convert chemical structure files (.mol/.SDF) into SMILES, InChI and InChIKey identifiers using Open Babel, as done for the 582 DNA adduct entries in the DNA adductomics database. It ensures every database record carries multiple machine-readable structural representations.

## When to use

Use when a curated compound list (.mol/.SDF/.cdxml-derived) needs standardized computer-readable identifiers — SMILES, InChI, InChIKey — for database population or identifier cross-validation.

## When NOT to use

- Input is already a populated table of SMILES/InChI/InChIKey values that only needs cross-checking, not regeneration.
- Structures are supplied as non-.mol/.SDF authoring files (e.g. raw .cdxml) that must first be converted with ChemScript — Open Babel alone will not ingest them.
- The task concerns MS/MS spectral matching or adduct identification rather than structural identifier generation.

## Inputs

- ChemDraw structure files (.cdxml)
- Structure files (.mol) extracted from the database deposit (.xlsx, .cdxml, .mol files at the GitLab repository)
- Optionally .SDF intermediate files
- .xlsx database file with name, SMILES, InChI, InChIKey and IUPAC nomenclature columns

## Outputs

- SMILES, InChI and InChIKey strings for each of the 582 database entries
- Updated .xlsx database file with populated identifier columns
- identifier_validation.csv with per-entry deposited vs. regenerated identifiers and a populated/matched status flag

## How to apply

Convert authoring-tool files first: .cdxml files are turned into .mol files and IUPAC nomenclature with ChemScript/ChemDraw, then .mol files (optionally via .SDF intermediates, per the paper's section 2.3) are batch-converted to SMILES, InChI and InChIKey with Open Babel 3.1.1. For the DNA adduct database this produced identifiers for all 582 entries — 279 literature-reported adducts plus 303 suspected adducts formed from genotoxicant–nucleobase combinations — which were written into the .xlsx database file. Positively charged ring-nitrogen adduct representations must be handled consistently so the regenerated identifiers match the deposited ones. Validate entry-by-entry: load the .xlsx with pandas, compare each regenerated identifier against the deposited name, SMILES, InChI and InChIKey fields, and flag missing or non-matching values in a per-entry CSV.

## Related tools

- **Open Babel** (Converts .mol/.SDF structure files to SMILES, InChI and InChIKey identifiers (version 3.1.1 used in the paper))
- **ChemDraw** (Authors the .cdxml adduct structures that are converted to .mol files before Open Babel conversion)

## Examples

```
obabel 582_adducts.sdf -O adducts_identifiers.csv --append SMILES InChI InChIKey  # then: pd.read_csv('adducts.xlsx')[['name','SMILES','InChI','InChIKey']]
```

## Evaluation signals

- identifier_validation.csv contains exactly 582 rows, one per database entry.
- Report counts of entries with populated SMILES, InChI and InChIKey fields; each should cover all 582 entries.
- Every regenerated identifier matches the corresponding deposited field in the .xlsx file entry-by-entry.
- Positively charged ring-nitrogen adducts are represented consistently across SMILES, InChI and InChIKey.
- The InChIKey is a fixed-length 27-character hash confirming correct InChI derivation.

## Limitations

- Identifier regeneration depends on consistent protonation/charge representation for positively charged ring-nitrogen adducts; inconsistent handling causes mismatched InChI/InChIKey values.
- The paper used Open Babel 3.1.1; other versions may produce differing InChI outputs.
- .cdxml source files require a separate ChemScript conversion step before Open Babel can process them.

## Evidence

- [other] ChemDraw files (.cdxml) were converted into .mol files and IUPAC nomenclature with ChemScript, then .mol files were converted to SMILES, InChI and InChIKey using Open Babel 3.1.1: [evidence span withheld — anchor status: not_found]
- [abstract] The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database ).: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [abstract] The database was expanded with 303 new potential DNA adducts from combinations of genotoxicants with nucleobases.: "implementing the database with 303 new potential DNA adducts coming from different combinations of genotoxicants with nucleobases"
- [abstract] Both experimental MS/MS spectra and in silico predicted fragments are reported in interactive web datatables.: "reporting both experimental and predicted fragments into interactive web datatables"
