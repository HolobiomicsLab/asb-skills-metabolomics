---
name: dna-adduct-database-tabulation
description: Use when compiling or verifying a curated DNA adduct database (.xlsx
  plus metadata .docx) from literature-reported and suspected adducts, or when recomputing
  its descriptive statistics such as mass range and study-design splits.
when_to_use_negative:
- The database already exists and you only need to annotate experimental MS/MS spectra
  against it rather than rebuild the tabulation.
- Input spectra are raw LC-HRMS feature lists awaiting annotation — this skill concerns
  the reference database itself, not spectral matching against it.
- Structures are unavailable or purely spectral fragments lack molecular formulas,
  so monoisotopic [M+H]+ and [M-dR]+ masses cannot be calculated.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0607
tools:
- name: ChemDraw
  role: Draws each suspected adduct structure and molecular formula (v.19.1.0.8) and
    exports .cdxml files
- name: CFM-ID
  role: Generates in silico MS/MS fragments for all adducts included in the datatables
- name: rstudio/DT
  role: R/JavaScript DataTables wrapper used to render the database and its MS/MS
    fragment data as interactive web datatables
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:b3c3afd54c0de75194419669d985c86b8c126855ae2105ca95695271f5bcbee9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - ChemDraw
  - rstudio/DT
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0607
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# dna-adduct-database-tabulation

## Summary

Tabulates the 582-entry DNA adductomics database (279 literature-reported plus 303 suspected genotoxicant–nucleobase adducts) into an .xlsx file with structural identifiers and interactive web datatables of experimental and predicted MS/MS fragments.

## When to use

Use when compiling or verifying a curated DNA adduct database (.xlsx plus metadata .docx) from literature-reported and suspected adducts, or when recomputing its descriptive statistics such as mass range and study-design splits.

## When NOT to use

- The database already exists and you only need to annotate experimental MS/MS spectra against it rather than rebuild the tabulation.
- Input spectra are raw LC-HRMS feature lists awaiting annotation — this skill concerns the reference database itself, not spectral matching against it.
- Structures are unavailable or purely spectral fragments lack molecular formulas, so monoisotopic [M+H]+ and [M-dR]+ masses cannot be calculated.

## Inputs

- Curated table of 279 literature-reported DNA adducts
- Theoretical genotoxicant–nucleobase combinations drawn in ChemDraw (.cdxml files)
- Metadata .docx accompanying the database

## Outputs

- An .xlsx database of 582 entries (279 literature + 303 suspected) with hyperlinks to .cdxml and .mol files and references
- SMILES, InChI, InChIKey and IUPAC nomenclature for each adduct
- Interactive web datatables (rstudio/DT) reporting experimental and in silico MS/MS fragments

## How to apply

Assemble the 279 literature-reported DNA adducts with harmonized structures, molecular formulas and names, then enumerate the 303 suspected adducts from theoretical combinations of genotoxicants with the four nucleobases, drawing each structure in ChemDraw v.19.1.0.8. For every entry calculate the charged monoisotopic [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+ masses, noting that positively charged forms apply for modifications on N3/N7 of dG, N1/N3/N7 of dA and N3 of dC. Convert .cdxml structure files to .mol files and IUPAC nomenclature with ChemScript, then to SMILES, InChI and InChIKey with Open Babel 3.1.1. Merge the literature and suspected entries into a single .xlsx file with hyperlinks to the .cdxml/.mol files and references, and publish it from the GitLab repository for annotation in untargeted LC-HRMS DNA adductomics.

## Related tools

- **ChemDraw** (Draws each suspected adduct structure and molecular formula (v.19.1.0.8) and exports .cdxml files)
- **CFM-ID** (Generates in silico MS/MS fragments for all adducts included in the datatables)
- **rstudio/DT** (R/JavaScript DataTables wrapper used to render the database and its MS/MS fragment data as interactive web datatables) — https://github.com/rstudio/DT

## Examples

```
library(DT); adducts <- readxl::read_xlsx('dna_adductomics_database.xlsx'); datatable(adducts)
```

## Evaluation signals

- Final .xlsx must contain exactly 582 entries: 279 literature-reported plus 303 suspected adducts distinguishable by having no linked reference.
- Recomputed precursor mass range must match the reported 220–640 Da for [M+H]+/[M]+ ions and stay unchanged after adding the 303 suspected adducts.
- Study-design split recomputed from the .docx metadata must match the reported in vitro 38%, in vivo 35%, human 27%.
- Technique shares must match the reported 78% MS and 16% HRMS/MS, with 16 genotoxicant classes and nine source classes.
- Every entry should carry SMILES, InChI and InChIKey conversions from Open Babel 3.1.1 and hyperlinks to its .cdxml/.mol files and references.

## Limitations

- Suspected adducts (303) carry no literature references and leave modification positions unspecified in the name, so entries need curation before confident annotation.
- dT adducts are fewer among suspected combinations because e.g. etheno-dT cannot form without an amino group.
- Only 15 reference standards contributed experimental MS/MS spectra; most adducts rely on in silico predicted fragments (CFM-ID).

## Evidence

- [abstract] Database tabulation into 582-entry .xlsx: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [abstract] Four-step build and spectral library: "In this paper, we present a four-step approach used to compile and curate a database for the annotation of DNA adducts in biological samples."
- [abstract] Interactive web datatables of fragments: "reporting both experimental and predicted fragments into interactive web datatables"
- [readme] DT readme install/usage: [evidence span withheld — anchor status: external_doc]
- [results] Fig. 2 descriptive statistics: [evidence span withheld — anchor status: not_found]
