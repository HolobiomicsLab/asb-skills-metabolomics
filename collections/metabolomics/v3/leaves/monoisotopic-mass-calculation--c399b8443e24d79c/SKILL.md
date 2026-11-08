---
name: monoisotopic-mass-calculation
description: Use when compiling a DNA adduct database or mass spectral library and
  you need charged monoisotopic masses for adduct ions, including deoxyribose-loss
  fragments, from molecular formulas.
when_to_use_negative:
- When the goal is detecting adducts in experimental LC-HRMS data rather than building
  the reference mass list — the database supports annotation but does not itself process
  raw spectra
- 'When adducts require MS/MS fragments: monoisotopic mass calculation alone does
  not generate fragments (use CFM-ID or experimental MS/MS of reference standards
  for that)'
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3520
tools:
- name: ChemDraw
  role: Drawing adduct structures and molecular formulas and calculating monoisotopic
    masses (v.19.1.0.8)
- name: ChemScript
  role: Converting .cdxml structure files to .mol files and IUPAC nomenclature
- name: Open Babel
  role: Converting structures to SMILES, InChI, and InChIKey (v.3.1.1)
- name: rstudio/DT
  role: R DataTables interface used to present the database as interactive web datatables
  repo: https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:f1a28fa769de08643eae0429b951bd68779a8349145684111ff40d2b5953f8d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - ChemScript
  - Open Babel
  - rstudio/DT
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# monoisotopic-mass-calculation

## Summary

Calculates charged monoisotopic masses for DNA adducts — [M+H]+/[M]+ and neutral-loss forms [M+H-dR]+/[M-dR]+ — as performed in the 582-entry DNA adductomics database built from 279 literature-reported and 303 suspected genotoxicant–nucleobase adducts.

## When to use

Use when compiling a DNA adduct database or mass spectral library and you need charged monoisotopic masses for adduct ions, including deoxyribose-loss fragments, from molecular formulas.

## When NOT to use

- When the goal is detecting adducts in experimental LC-HRMS data rather than building the reference mass list — the database supports annotation but does not itself process raw spectra
- When adducts require MS/MS fragments: monoisotopic mass calculation alone does not generate fragments (use CFM-ID or experimental MS/MS of reference standards for that)

## Inputs

- Structures and molecular formulas of DNA adducts (279 literature-reported plus 303 suspected genotoxicant–nucleobase combinations), drawn as .cdxml files
- Curated table of literature-reported DNA adducts with harmonized names and formulas

## Outputs

- Charged monoisotopic masses [M+H]+ or [M]+ and [M+H-dR]+ or [M-dR]+ for every DNA adduct
- Structure identifiers per adduct: .mol files, SMILES, InChI, InChIKey, IUPAC nomenclature
- Merged .xlsx database of 582 entries with hyperlinks to .cdxml, .mol files and references

## How to apply

Enumerate each adduct (draw structure and formula in ChemDraw v.19.1.0.8), then compute the charged monoisotopic mass as [M+H]+ or [M]+ depending on whether the modification sits on a ring nitrogen that is protonated in the charged form (N3/N7 of dG, N1/N3/N7 of dA, N3 of dC). For each adduct also compute the deoxyribose-loss (dR) ion, i.e. [M+H-dR]+ or [M-dR]+, so both parent and nucleobase-level ions can be matched in LC-HRMS adductomics. Convert .cdxml structure files to .mol and IUPAC names with ChemScript, then to SMILES, InChI and InChIKey with Open Babel 3.1.1, and merge suspected entries (no literature reference) with curated literature entries into an .xlsx table with hyperlinks. Validate the counts: 279 literature plus 303 suspected adducts must total exactly 582 entries.

## Related tools

- **ChemDraw** (Drawing adduct structures and molecular formulas and calculating monoisotopic masses (v.19.1.0.8))
- **ChemScript** (Converting .cdxml structure files to .mol files and IUPAC nomenclature)
- **Open Babel** (Converting structures to SMILES, InChI, and InChIKey (v.3.1.1))
- **rstudio/DT** (R DataTables interface used to present the database as interactive web datatables) — https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database

## Examples

```
obabel -i mol adduct.mol -o can -O adduct.smi  # generate SMILES (Open Babel 3.1.1) after ChemDraw/ChemScript mass calculation
```

## Evaluation signals

- Final merged database contains exactly 582 entries: 279 literature-reported plus 303 suspected
- Suspected adducts are distinguishable by having no linked reference
- Every adduct has both [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+ monoisotopic masses, with the positively charged form correctly chosen for N3/N7 of dG, N1/N3/N7 of dA, and N3 of dC modifications
- Each entry carries complete structure identifiers: molecular formula, .cdxml, .mol, SMILES, InChI, InChIKey, and IUPAC name
- dT adducts are fewer among the suspected set, consistent with e.g. etheno-dT being chemically implausible without an amino group

## Limitations

- Suspected adducts are theoretical combinations of genotoxicants with nucleobases; modification positions are left unspecified in their names and they have no literature references
- Only chemically plausible combinations were enumerated (e.g. etheno-dT cannot form without an amino group), so the suspected set is not exhaustive
- The database provides monoisotopic masses and predicted fragments, not retention times — no chromatographic separation details are given for the entries

## Evidence

- [abstract] 582-entry database composition: "The database, including 582 entries, is publicly available"
- [abstract] 303 suspected adducts from genotoxicant-nucleobase combinations: "implementing the database with 303 new potential DNA adducts coming from different combinations of genotoxicants with nucleobases"
- [abstract] structure identifiers exported per adduct: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] in silico fragments for all adducts: "generating in silico MS/MS fragments for all the adducts"
- [other] public repository location: "https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database"
