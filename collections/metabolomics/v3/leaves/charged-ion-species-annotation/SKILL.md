---
name: charged-ion-species-annotation
description: 'Use when expanding a small-molecule annotation database for LC-HRMS
  adductomics: each drawn structure needs charged monoisotopic masses for detectable
  ion species and standardized identifiers (SMILES, InChI, InChIKey).'
when_to_use_negative:
- When the goal is acquiring or matching experimental MS/MS spectra of reference standards
  (step 4 of the paper's workflow) rather than enumerating theoretical ion species
  and masses
- When entries are already literature-reported adducts with harmonized structures
  and names — those belong to the curation/harmonization step, not the implementation
  step
- When adducts of nucleobases beyond dA, dC, dG and dT (e.g. RNA or protein adducts)
  are targeted — the combination enumeration here is scoped to the four DNA nucleobases
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: ChemDraw
  role: Draw each suspected adduct structure and molecular formula; calculate the
    charged monoisotopic masses and produce .cdxml files
- name: ChemScript
  role: Batch-convert .cdxml structure files to .mol files and generate IUPAC nomenclature
- name: Open Babel
  role: Convert .mol files to SMILES, InChI, and InChIKey identifiers (version 3.1.1)
- name: CFM-ID
  role: Generate in silico MS/MS fragments for all adducts in the implemented database
- name: rstudio/DT
  role: R datatable() interface used to render the database, including experimental
    and predicted fragments, as interactive web datatables
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:5479bae3a5a33e3f74b7b5b045df48c268a7a7e20da6185c15bb4bb3aa9a4364
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - ChemDraw
  - ChemScript
  - Open Babel
  - rstudio/DT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# charged-ion-species-annotation

## Summary

Enumerate theoretical DNA adducts from genotoxicant–nucleobase combinations and compute the charged monoisotopic ion masses ([M+H]+, [M]+, [M+H-dR]+, [M-dR]+) plus structural identifiers needed for HRMS annotation in DNA adductomics.

## When to use

Use when expanding a small-molecule annotation database for LC-HRMS adductomics: each drawn structure needs charged monoisotopic masses for detectable ion species and standardized identifiers (SMILES, InChI, InChIKey).

## When NOT to use

- When the goal is acquiring or matching experimental MS/MS spectra of reference standards (step 4 of the paper's workflow) rather than enumerating theoretical ion species and masses
- When entries are already literature-reported adducts with harmonized structures and names — those belong to the curation/harmonization step, not the implementation step
- When adducts of nucleobases beyond dA, dC, dG and dT (e.g. RNA or protein adducts) are targeted — the combination enumeration here is scoped to the four DNA nucleobases

## Inputs

- Theoretical genotoxicant–nucleobase combination list to enumerate
- Structure drawings in .cdxml format (ChemDraw v.19.1.0.8)
- Curated table of 279 literature-reported DNA adducts with harmonized structures, molecular formulas and names
- Database merge target in .xlsx format

## Outputs

- Charged monoisotopic masses [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+ per adduct
- .mol files and IUPAC nomenclature (via ChemScript)
- SMILES, InChI, and InChIKey identifiers (via Open Babel 3.1.1)
- Merged 582-entry .xlsx database (279 literature + 303 suspected) with hyperlinks to .cdxml, .mol files and references
- In silico MS/MS fragments generated with CFM-ID for the database entries

## How to apply

Enumerate all chemically plausible combinations of potential genotoxicants with the four nucleobases (dT adducts are fewer, since e.g. etheno-dT cannot form without an amino group), and draw each structure and molecular formula in ChemDraw v.19.1.0.8, naming suspected adducts without literature references and leaving modification positions unspecified. For every DNA adduct calculate the charged monoisotopic masses [M + H]+ or [M]+ and [M + H-dR]+ or [M-dR]+, choosing the positively charged form appropriate to the modification site: N3/N7 of dG, N1/N3/N7 of dA, and N3 of dC. Convert the .cdxml structure files to .mol files and IUPAC nomenclature with ChemScript, then to SMILES, InChI, and InChIKey with Open Babel 3.1.1. Merge the resulting suspected adducts with the curated 279 literature-reported entries in an .xlsx file that includes hyperlinks to the .cdxml files, .mol files and references, keeping suspected entries distinguishable by their missing reference link.

## Related tools

- **ChemDraw** (Draw each suspected adduct structure and molecular formula; calculate the charged monoisotopic masses and produce .cdxml files)
- **ChemScript** (Batch-convert .cdxml structure files to .mol files and generate IUPAC nomenclature)
- **Open Babel** (Convert .mol files to SMILES, InChI, and InChIKey identifiers (version 3.1.1))
- **CFM-ID** (Generate in silico MS/MS fragments for all adducts in the implemented database)
- **rstudio/DT** (R datatable() interface used to render the database, including experimental and predicted fragments, as interactive web datatables) — https://github.com/rstudio/DT

## Examples

```
obabel adduct.mol -Oadduct.inchi
```

## Evaluation signals

- The merged database contains exactly 582 entries: 279 literature-reported plus 303 suspected
- Suspected adducts are identifiable by having no linked reference in the .xlsx database
- Every entry carries both charged monoisotopic masses ([M+H]+ or [M]+, and [M+H-dR]+ or [M-dR]+), a chemical formula, and .cdxml/.mol hyperlinks
- Every entry resolves to valid SMILES, InChI and InChIKey identifiers after the Open Babel conversion
- Suspected adduct names leave modification positions unspecified, consistent with the paper's naming convention

## Limitations

- Suspected adducts are purely theoretical (no literature support) and have unspecified modification positions, so they are only tentative annotations for HRMS screening
- In silico MS/MS fragments (CFM-ID) are predicted, not experimentally validated; only 15 reference standards contributed experimental MS/MS spectra to the spectral library
- The rstudio/DT package used to display the database is in maintenance-only mode — pull requests may not be reviewed or merged

## Evidence

- [abstract] implementing the database with 303 new potential DNA adducts coming from different combinations of genotoxicants with nucleobases: "implementing the database with 303 new potential DNA adducts coming from different combinations of genotoxicants with nucleobases"
- [abstract] monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] generating in silico MS/MS fragments for all the adducts: "generating in silico MS/MS fragments for all the adducts"
- [readme] This package provides a function datatable() to display R data via the DataTables library: [evidence span withheld — anchor status: external_doc]
