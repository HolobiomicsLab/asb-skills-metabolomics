---
name: monoisotopic-mass-range-computation
description: Use when building or auditing a DNA adduct reference database and you
  need to verify the monoisotopic mass span of its entries, or when you need to confirm
  a newly added set of potential adducts does not change the reported mass range.
when_to_use_negative:
- You need to annotate adducts in experimental LC-HRMS data by spectral matching —
  this skill only characterizes database mass coverage, not MS/MS fragment annotation
- The input table lacks monoisotopic mass or precursor-ion columns and only holds
  nominal masses or retention times
- Your database entries are nucleobases or genotoxicants alone rather than combined
  DNA adduct structures
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: ChemDraw
  role: Source of the harmonized chemical structure identifiers (SMILES, InChI, .mol
    files) from which monoisotopic masses and formulas are derived
- name: rstudio/DT
  role: R DataTables package used to render the database, including mass and formula
    fields, as interactive web datatables
  repo: https://github.com/rstudio/DT
- name: pandas/openpyxl
  role: Python stack for loading the .xlsx database and computing min/max monoisotopic
    masses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:4ddefe518583274aaa96ad34270dc662fc098dcf74c1db7631e6fc016b91b759
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - ChemDraw
  - pandas/openpyxl
  - rstudio/DT
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# monoisotopic-mass-range-computation

## Summary

Computation of the monoisotopic [M+H]+/[M]+ precursor-ion mass range from a curated DNA adduct database (e.g. the 582-entry database released with the DNA adductomics paper) to characterize and validate its mass coverage, reported as 220–640 Da.

## When to use

Use when building or auditing a DNA adduct reference database and you need to verify the monoisotopic mass span of its entries, or when you need to confirm a newly added set of potential adducts does not change the reported mass range.

## When NOT to use

- You need to annotate adducts in experimental LC-HRMS data by spectral matching — this skill only characterizes database mass coverage, not MS/MS fragment annotation
- The input table lacks monoisotopic mass or precursor-ion columns and only holds nominal masses or retention times
- Your database entries are nucleobases or genotoxicants alone rather than combined DNA adduct structures

## Inputs

- DNA adduct database spreadsheet (.xlsx) with monoisotopic masses, chemical formulas, and [M+H]+/[M]+ precursor ions per entry
- Metadata document (.docx) describing study design, genotoxicant classes, and source classes
- Chemical structure identifiers per entry (SMILES, InChI, InChIKey, .mol/.cdxml files)

## Outputs

- Minimum and maximum monoisotopic [M+H]+/[M]+ precursor mass of the database (e.g. 220–640 Da)
- Mass distribution summary usable in a descriptive-statistics figure (e.g. Figure 2)
- Validation flag if recomputed mass range differs from the published value

## How to apply

Load the database (a .xlsx spreadsheet plus metadata .docx) from the GitLab repository with pandas via openpyxl, and identify the [M+H]+/[M]+ precursor-ion masses for each entry. Compute the minimum and maximum monoisotopic mass — for the 582-entry database this should reproduce the reported 220–640 Da range. The structures, molecular formulas, and monoisotopic masses in the database were harmonized from 279 literature-reported DNA adducts and extended with 303 new potential adducts from genotoxicant–nucleobase combinations, so masses derive from the curated chemical formulas (e.g. generated with ChemDraw identifiers such as SMILES, InChI, and .mol files). Re-run the computation after appending new entries: the paper notes the mass distribution is unchanged after adding the 303 suspected adducts, so an altered range signals a data-harmonization error. Report the min–max span alongside other descriptive statistics (study design shares, identification-technique shares) to characterize database coverage.

## Related tools

- **ChemDraw** (Source of the harmonized chemical structure identifiers (SMILES, InChI, .mol files) from which monoisotopic masses and formulas are derived)
- **rstudio/DT** (R DataTables package used to render the database, including mass and formula fields, as interactive web datatables) — https://github.com/rstudio/DT
- **pandas/openpyxl** (Python stack for loading the .xlsx database and computing min/max monoisotopic masses)

## Examples

```
import pandas as pd; df = pd.read_excel('dna_adduct_database.xlsx'); m = df['mono_mass']; print(m.min(), m.max())
```

## Evaluation signals

- Recomputed min and max monoisotopic mass reproduce the published 220–640 Da range for the 582-entry database
- Mass range is unchanged after merging the 303 suspected/potential adducts into the 279 literature-reported set
- Number of entries remains 582 (279 + 303) when the range is computed
- Every entry's mass matches the mass implied by its harmonized chemical formula and structure identifiers
- Recomputed statistics agree with Figure 2 of the paper; any mismatch is flagged

## Limitations

- The range covers only monoisotopic [M+H]+/[M]+ precursor ions; isotopic or alternative charge states are not represented
- Mass harmonization depends on correct structure/formula curation; errors in SMILES or .mol identifiers propagate to wrong masses
- Newly added potential adducts (303 of the 582) are in silico combinations of genotoxicants with nucleobases and have not been experimentally confirmed

## Evidence

- [abstract] The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database ).: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [abstract] implementing the database with 303 new potential DNA adducts: "implementing the database with 303 new potential DNA adducts coming from different combinations of genotoxicants with nucleobases"
- [abstract] monoisotopic masses, chemical formulas, .cdxml files: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [results] 220–640 Da mass range: [evidence span withheld — anchor status: not_found]
