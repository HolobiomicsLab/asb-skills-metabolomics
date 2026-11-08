---
name: nmr-chemical-shift-curation
description: Use when building or extending a metabolite NMR chemical-shift reference
  library for ROI-based metabolite identification, e.g., swapping the HMDB source
  for BMRB assignments to drop into ROIAL-NMR.
when_to_use_negative:
- The goal is identifying ROIs automatically from raw spectra — the paper states automated
  ROI identification is beyond the current scope; ROIs must be user-defined first
- The goal is absolute quantification of metabolite concentrations — the program's
  primary aim is identification/categorization, not quantification
- The input is already a validated ROIAL-NMR-format database workbook (re-curation
  is unnecessary)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0605
tools:
- name: ROIAL-NMR
  role: Consumer of the curated two-table database workbook; its match-ratio logic
    (peak-in-ROI matching per SRRM) drives metabolite identification
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Parsing BMRB assignment records and structuring metabolite names, chemical
    shifts, and multiplicity into dataframes (pandas 2.2.3)
- name: XlsxWriter
  role: Writing the chemical-shift and SRRM tables into the single .xlsx workbook
    consumed by ROIAL-NMR (XlsxWriter 3.2.2)
- name: openpyxl
  role: Validation read-back of the generated workbook to confirm it opens cleanly
    (openpyxl 3.1.5)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:209b11d0c129ded2ac9849e23a7a9015147ee87217463213d2e922d92a7b69ad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Python
  - ROIAL-NMR
  - XlsxWriter
  - openpyxl
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# nmr-chemical-shift-curation

## Summary

Curate a metabolite proton NMR assignment library from a biological NMR reference database (e.g., BMRB as an alternative to the HMDB) and format it as the two-table Excel workbook (chemical-shift values plus SRRM multiplets) that ROIAL-NMR consumes as a user-selected metabolic database.

## When to use

Use when building or extending a metabolite NMR chemical-shift reference library for ROI-based metabolite identification, e.g., swapping the HMDB source for BMRB assignments to drop into ROIAL-NMR.

## When NOT to use

- The goal is identifying ROIs automatically from raw spectra — the paper states automated ROI identification is beyond the current scope; ROIs must be user-defined first
- The goal is absolute quantification of metabolite concentrations — the program's primary aim is identification/categorization, not quantification
- The input is already a validated ROIAL-NMR-format database workbook (re-curation is unnecessary)

## Inputs

- BMRB (or HMDB) metabolite 1H chemical shift assignment records (downloaded database entries)
- Reference schema of the existing HMDB-derived chemical-shift and SRRM library workbook used by ROIAL-NMR

## Outputs

- Excel workbook (.xlsx) with one sheet/table of chemical shift values per metabolite and one of SRRM (J-coupling multiplet) regions per metabolite, matching the ROIAL-NMR database column schema

## How to apply

Download metabolite 1H chemical shift assignment records from the chosen source database (BMRB is named in the paper as an alternative alongside the HMDB, which supplies chemical shift data for 891 metabolites). Parse the records with pandas, retaining metabolite names, sample-type/biofluid context, 1H chemical shift peak values, and J-coupling multiplicity information. Build two tables mirroring the HMDB-derived libraries: (1) chemical shift values per metabolite, and (2) spectral region resonance multiplets (SRRMs) per metabolite — the paper found both are required for accurate metabolite identification. Write both tables into a single Excel workbook with XlsxWriter using the same sheet layout and column schema as the HMDB-derived libraries, so the file can be selected as the metabolic database inside ROIAL-NMR (Python >=3.9, pandas 2.2.3, XlsxWriter 3.2.2, openpyxl 3.1.5).

## Related tools

- **ROIAL-NMR** (Consumer of the curated two-table database workbook; its match-ratio logic (peak-in-ROI matching per SRRM) drives metabolite identification) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Parsing BMRB assignment records and structuring metabolite names, chemical shifts, and multiplicity into dataframes (pandas 2.2.3))
- **XlsxWriter** (Writing the chemical-shift and SRRM tables into the single .xlsx workbook consumed by ROIAL-NMR (XlsxWriter 3.2.2))
- **openpyxl** (Validation read-back of the generated workbook to confirm it opens cleanly (openpyxl 3.1.5))

## Examples

```
import pandas as pd; bmrb = pd.read_csv('bmrb_shifts.csv'); xlsx = pd.ExcelWriter('BMRB_library.xlsx', engine='xlsxwriter'); bmrb[['metabolite','shift_ppm']].to_excel(xlsx, sheet_name='chemical_shifts', index=False); bmrb[['metabolite','srrm']].to_excel(xlsx, sheet_name='SRRMs', index=False); xlsx.close()
```

## Evaluation signals

- Workbook opens cleanly with openpyxl/pandas and contains exactly the two expected tables (chemical shifts and SRRMs) in the same sheet layout as the HMDB-derived libraries
- Every metabolite row exposes both chemical-shift peaks and SRRM regions so ROIAL-NMR's match-ratio logic (e.g., lactate's 2/2 peak-in-ROI example) computes without format errors
- Chemical shift peak values and J-coupling multiplets retained from source records match the BMRB entries after parsing
- Dropping the workbook into ROIAL-NMR as a user-selected metabolic database yields identified metabolites with match ratios in the 'All Metabolites' window

## Limitations

- Source databases themselves lack a mechanism to collectively identify metabolites present in multiple ROIs simultaneously — curation only supplies the per-metabolite assignment data
- Planned automatic synchronization of NMR chemical shift databases is not yet implemented, so the library must be re-curated manually
- The paper demonstrated with HMDB data; BMRB-derived tables must be structurally mirrored to the HMDB-derived schema and validated for compatibility

## Evidence

- [intro] BMRB as alternative source: [evidence span withheld — anchor status: not_found]
