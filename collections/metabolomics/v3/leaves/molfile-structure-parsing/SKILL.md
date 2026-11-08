---
name: molfile-structure-parsing
description: Use when a curated compound database (e.g. a .xlsx of DNA adducts) must
  be populated or validated with SMILES/InChI/InChIKey from structure files, or when
  regenerating identifiers from deposited .mol files for entry-by-entry comparison.
when_to_use_negative:
- The goal is generating in silico MS/MS fragments — that is handled separately with
  CFM-ID in the spectral library step, not with Open Babel identifier generation.
- The input is raw LC-HRMS/MS spectral data rather than curated structure files —
  this skill operates on .mol/.cdxml/.SDF chemical representations only.
- The deliverable is a browsable interactive web datatable of the database (rstudio/DT
  task), not structure-identifier generation or validation.
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: Open Babel
  role: Batch-converts each .mol file (optionally via .SDF intermediates) into SMILES,
    InChI and InChIKey at version 3.1.1
- name: ChemDraw (ChemScript)
  role: Upstream conversion of .cdxml files into .mol files and IUPAC nomenclature
    before identifier generation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:4f59cdf5972db51aadfa443dc6e40f58f47f948f254ba588b522c5582bb3c1b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - ChemDraw (ChemScript)
  - Open Babel
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# molfile-structure-parsing

## Summary

Converts curated chemical structure files (.mol, optionally via .SDF) into the standard computer-readable identifiers SMILES, InChI and InChIKey using Open Babel 3.1.1, as done to populate the 582-entry DNA adduct database.

## When to use

Use when a curated compound database (e.g. a .xlsx of DNA adducts) must be populated or validated with SMILES/InChI/InChIKey from structure files, or when regenerating identifiers from deposited .mol files for entry-by-entry comparison.

## When NOT to use

- The goal is generating in silico MS/MS fragments — that is handled separately with CFM-ID in the spectral library step, not with Open Babel identifier generation.
- The input is raw LC-HRMS/MS spectral data rather than curated structure files — this skill operates on .mol/.cdxml/.SDF chemical representations only.
- The deliverable is a browsable interactive web datatable of the database (rstudio/DT task), not structure-identifier generation or validation.

## Inputs

- .mol structure files (582 entries: 279 literature-reported + 303 suspected adducts)
- .cdxml ChemDraw files (if starting from the original drawings; require ChemScript conversion)
- .xlsx database file with name, SMILES, InChI, InChIKey and IUPAC nomenclature columns

## Outputs

- SMILES, InChI and InChIKey strings per structure (batch output, optionally via .SDF intermediates)
- identifier_validation.csv with per-entry deposited vs. regenerated identifiers and populated/matched status flags

## How to apply

Retrieve the deposit from the GitLab repository (https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database) containing the .xlsx database and 582 .mol structure files (279 literature-reported plus 303 suspected adducts); any source .cdxml files must first be converted to .mol and IUPAC nomenclature with ChemScript, as Open Babel does not read ChemDraw format. Batch-convert each .mol file to SMILES, InChI and InChIKey with Open Babel at the version used in the paper (3.1.1), optionally via .SDF intermediates as described in section 2.3. Load the .xlsx with pandas and extract the name, SMILES, InChI, InChIKey and IUPAC nomenclature columns, then compare each regenerated identifier against the deposited field entry-by-entry, flagging missing or non-matching values and handling positively charged ring-nitrogen adduct representations consistently. Write a per-entry comparison table (identifier_validation.csv) with deposited and regenerated SMILES/InChI/InChIKey and a populated/matched status flag.

## Related tools

- **Open Babel** (Batch-converts each .mol file (optionally via .SDF intermediates) into SMILES, InChI and InChIKey at version 3.1.1)
- **ChemDraw (ChemScript)** (Upstream conversion of .cdxml files into .mol files and IUPAC nomenclature before identifier generation)

## Examples

```
obabel structures/*.mol -osmi -O identifiers.smi --append InChI InChIKey
```

## Evaluation signals

- identifier_validation.csv contains exactly 582 rows, one per database entry
- The report states the count of entries with populated SMILES, InChI and InChIKey fields
- Each regenerated SMILES/InChI/InChIKey matches the corresponding deposited field in the .xlsx entry-by-entry
- Positively charged ring-nitrogen adduct representations are handled consistently (no spurious mismatches from tautomer/charge notation)
- Entries flagged as missing or non-matching are explicitly enumerated rather than silently dropped

## Limitations

- Open Babel version must be pinned to 3.1.1 as used in the paper; different versions can yield differing canonical strings.
- Positively charged ring-nitrogen adducts require consistent representation conventions or identifiers will mismatch.
- .cdxml source files cannot be parsed directly by Open Babel — a ChemScript/ChemDraw step is required first.

## Evidence

- [methods] identifier pipeline: [evidence span withheld — anchor status: not_found]
- [abstract] database contents: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] database availability: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
