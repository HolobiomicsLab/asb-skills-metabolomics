---
name: study-design-stratification-of-adduct-entries
description: Use when a DNA adductomics database (.xlsx with metadata) must be profiled
  or validated by stratum — study design, identification technique, genotoxicant class,
  source, or precursor mass range.
when_to_use_negative:
- The goal is spectral annotation or MS/MS fragment matching rather than database
  profiling — use the spectral-library workflow (CFM-ID in silico fragments, 15 reference-standard
  experimental MS/MS spectra) instead
- Input is raw LC-HRMS sample data rather than the curated adduct database — stratification
  of entries does not apply to unannotated files
- Entry-level structure harmonization (SMILES, InChIKey, .cdxml/.mol files) is the
  target, not distribution statistics
tools:
- name: pandas / openpyxl
  role: loads the .xlsx adduct database and computes per-stratum counts and mass minima/maxima
  repo: https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- name: rstudio/DT
  role: R DataTables interface used to display database entries (and their stratified
    views) as interactive web datatables
  repo: https://github.com/rstudio/DT
- name: ChemDraw
  role: source of structural identifiers (monoisotopic masses, chemical formulas,
    .cdxml/.mol files) whose precursor masses underpin the mass-range check
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:064f284a63102ae0b4a1b825d1387954b3c2c7746e1581df138d77a5a82c4e40
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - pandas / openpyxl
  - rstudio/DT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# study-design-stratification-of-adduct-entries

## Summary

Tabulates and cross-checks the descriptive statistics of the 582-entry DNA adduct database — monoisotopic precursor mass range (220–640 Da), study-design split (in vitro 38%, in vivo 35%, human 27%), identification-technique shares (78% MS, 16% HRMS/MS), and grouping into 16 genotoxicant classes and nine source classes — against Figure 2 of La Barbera et al. 2022.

## When to use

Use when a DNA adductomics database (.xlsx with metadata) must be profiled or validated by stratum — study design, identification technique, genotoxicant class, source, or precursor mass range.

## When NOT to use

- The goal is spectral annotation or MS/MS fragment matching rather than database profiling — use the spectral-library workflow (CFM-ID in silico fragments, 15 reference-standard experimental MS/MS spectra) instead
- Input is raw LC-HRMS sample data rather than the curated adduct database — stratification of entries does not apply to unannotated files
- Entry-level structure harmonization (SMILES, InChIKey, .cdxml/.mol files) is the target, not distribution statistics

## Inputs

- 582-entry DNA adduct database as .xlsx from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- accompanying metadata file (.docx) describing study design, technique, genotoxicant and source per entry

## Outputs

- tabulated study-design distribution (in vitro / in vivo / human percentages)
- identification-technique share table (MS, HRMS/MS, NMR, UV/fluorescence/immunoanalysis, 32P-postlabeling)
- genotoxicant-class and source-class group counts (16 and 9 classes respectively)
- verified precursor monoisotopic mass range (220–640 Da) with min/max values

## How to apply

Download the 582-entry database (.xlsx plus metadata .docx) from the GitLab repository and load it with pandas via openpyxl. Compute the minimum and maximum monoisotopic masses of the [M+H]+/[M]+ precursor ions to verify the reported 220–640 Da range, and confirm the distribution is unchanged after adding the 303 suspected (potential) adducts generated from combinations of genotoxicants with nucleobases. Tabulate the study-design distribution (in vitro / in vivo / human) from the metadata and, for human entries, further count target-tissue versus surrogate-sample detections. Categorize each entry by identification technique (MS, HRMS/MS, NMR, UV/fluorescence/immunoanalysis, 32P-postlabeling) and compute technique shares. Finally, group entries by causative genotoxicant and by source, and cross-check every recomputed statistic against the values reported in Figure 2, flagging any mismatch.

## Related tools

- **pandas / openpyxl** (loads the .xlsx adduct database and computes per-stratum counts and mass minima/maxima) — https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- **rstudio/DT** (R DataTables interface used to display database entries (and their stratified views) as interactive web datatables) — https://github.com/rstudio/DT
- **ChemDraw** (source of structural identifiers (monoisotopic masses, chemical formulas, .cdxml/.mol files) whose precursor masses underpin the mass-range check)

## Examples

```
import pandas as pd; db = pd.read_excel('dna_adductomics_database.xlsx', engine='openpyxl'); print(db['monoisotopic_mass'].min(), db['monoisotopic_mass'].max())
```

## Evaluation signals

- Recomputed precursor mass min/max equals 220 Da and 640 Da, and the distribution is unchanged after adding the 303 suspected adducts
- Study-design tabulation reproduces the 38% in vitro / 35% in vivo / 27% human split
- Technique shares reproduce 78% MS and 16% HRMS/MS
- Grouping by causative genotoxicant and by source yields exactly 16 genotoxicant classes and nine source classes
- Every recomputed statistic matches Figure 2 of the paper; mismatches are explicitly flagged

## Limitations

- The statistics are only defined on the curated 582-entry literature database plus the 303 potential adducts; they do not describe a user's own experimental adduct list
- Human entries require an additional target-tissue versus surrogate-sample distinction that must come from the metadata .docx, not the mass/structure fields
- The DT R package is in maintenance-only mode, so interactive datatable rendering bugs may not be fixed upstream

## Evidence

- [results] Figure 2 summarizes descriptive statistics of the 582-entry database: 220–640 Da mass range, in vitro 38%, in vivo 35%, human 27%, 78% MS, 16% HRMS/MS, 16 genotoxicant classes, and nine source classes.: [evidence span withheld — anchor status: not_found]
- [abstract] The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database ).: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [readme] This package provides a function `datatable()` to display R data via the [DataTables](https://datatables.net/) library (N.B. not to be confused with the **data.table** package).: [evidence span withheld — anchor status: external_doc]
