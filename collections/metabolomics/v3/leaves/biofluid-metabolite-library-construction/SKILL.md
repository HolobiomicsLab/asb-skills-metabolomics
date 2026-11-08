---
name: biofluid-metabolite-library-construction
description: Use when building or extending the reference metabolite database behind
  ROIAL-NMR — e.g., deriving 1H chemical shift and J-coupling multiplet (SRRM) tables
  from BMRB or HMDB records for biofluids like serum, urine, saliva, or CSF.
when_to_use_negative:
- Automated ROI identification or peak picking from raw spectra — the article states
  'Automated identification of ROIs is beyond the scope of the current version of
  the program'
- Absolute quantification of metabolite concentrations — 'absolute quantification
  of the compounds is not the primary aim of the program'
- Collectively identifying metabolites across multiple ROIs — the databases themselves
  (HMDB, BMRB) lack this mechanism; that is what ROIAL-NMR's match-ratio logic does
  downstream
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: pandas
  role: Parse database assignment records and build the chemical-shift and SRRM tables
    (pandas 2.2.3)
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: XlsxWriter
  role: Write the two-library tables into a single Excel workbook in ROIAL-NMR's sheet
    layout (XlsxWriter 3.2.2)
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: openpyxl
  role: Reopen/validate the generated workbook so it loads cleanly (openpyxl 3.1.5)
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: ROIAL-NMR
  role: 'Downstream consumer: loads the constructed library as a user-selected metabolic
    database for ROI-based metabolite identification'
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Biological Magnetic Resonance Bank (BMRB)
  role: Alternative source of 1H chemical shift assignments from which the library
    tables are derived
- name: Human Metabolome Database (HMDB)
  role: Default reference platform; its NMR assignments for 891 metabolites define
    the library schema being replicated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:dacd942af8b3e4132b6fb0fcd17c8e29cb2428d0d5f0167e000e79783da0c0dc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Biological Magnetic Resonance Bank (BMRB)
  - Human Metabolome Database (HMDB)
  - Python
  - ROIAL-NMR
  - XlsxWriter
  - openpyxl
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# biofluid-metabolite-library-construction

## Summary

Construct a program-specific metabolite NMR assignment library (chemical-shift table plus spectral region resonance multiplet/SRRM table) from a biological NMR reference database such as HMDB or BMRB, formatted as an Excel workbook that ROIAL-NMR can ingest as a user-selected metabolic database.

## When to use

Use when building or extending the reference metabolite database behind ROIAL-NMR — e.g., deriving 1H chemical shift and J-coupling multiplet (SRRM) tables from BMRB or HMDB records for biofluids like serum, urine, saliva, or CSF.

## When NOT to use

- Automated ROI identification or peak picking from raw spectra — the article states 'Automated identification of ROIs is beyond the scope of the current version of the program'
- Absolute quantification of metabolite concentrations — 'absolute quantification of the compounds is not the primary aim of the program'
- Collectively identifying metabolites across multiple ROIs — the databases themselves (HMDB, BMRB) lack this mechanism; that is what ROIAL-NMR's match-ratio logic does downstream

## Inputs

- Metabolite 1H chemical shift assignment records from a biological NMR database (HMDB or BMRB), including metabolite names, biofluid/sample-type context, peak chemical shift values, and J-coupling multiplicity
- Existing HMDB-derived ROIAL-NMR library workbook as a schema/structure template

## Outputs

- Single Excel workbook (written with XlsxWriter) containing two sheets: a per-metabolite chemical shift value table and a per-metabolite SRRM (multiplet) table, matching the ROIAL-NMR column schema
- Validated library loadable by ROIAL-NMR as a user-selected metabolic database

## How to apply

Download metabolite 1H chemical shift assignment records from the chosen reference (e.g., BMRB as an alternative to the HMDB, which anchors ROIAL-NMR's default library of 891 metabolites). Parse the records with pandas (>=2.2.3), retaining metabolite names, biofluid/sample-type context, 1H chemical shift peak values, and J-coupling multiplicity information. Generate two tables mirroring the HMDB-derived library schema: one of chemical shift values per metabolite and one of spectral region resonance multiplets (SRRMs) per metabolite, since ROIAL-NMR was found to require both chemical shift values and metabolite-specific SRRMs for accurate identification. Write both tables into a single Excel workbook with XlsxWriter 3.2.2 using the same sheet layout and column schema as the HMDB-derived libraries, so the file can be dropped into ROIAL-NMR as a user-selected metabolic database. Validate by reopening the workbook with openpyxl/pandas and confirming each metabolite row exposes both chemical-shift peaks and SRRM regions so match-ratio logic (e.g., lactate's 2/2 peak-in-ROI example) computes without format errors.

## Related tools

- **pandas** (Parse database assignment records and build the chemical-shift and SRRM tables (pandas 2.2.3)) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **XlsxWriter** (Write the two-library tables into a single Excel workbook in ROIAL-NMR's sheet layout (XlsxWriter 3.2.2)) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **openpyxl** (Reopen/validate the generated workbook so it loads cleanly (openpyxl 3.1.5)) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **ROIAL-NMR** (Downstream consumer: loads the constructed library as a user-selected metabolic database for ROI-based metabolite identification) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Biological Magnetic Resonance Bank (BMRB)** (Alternative source of 1H chemical shift assignments from which the library tables are derived)
- **Human Metabolome Database (HMDB)** (Default reference platform; its NMR assignments for 891 metabolites define the library schema being replicated)

## Examples

```
python main.py
```

## Evaluation signals

- Workbook opens cleanly with openpyxl/pandas and follows the same sheet layout and column schema as the HMDB-derived libraries
- Each metabolite row exposes both chemical-shift peaks and SRRM regions — both are required for accurate metabolite identification per the article
- Match-ratio logic can be computed per SRRM without format errors (e.g., lactate achieving a 2/2 peak-in-ROI match)
- Coverage check: HMDB-derived library anchors 891 metabolites; a BMRB-derived library should report its metabolite count for comparability
- ROIAL-NMR accepts the workbook as a user-selected metabolic database and completes single-population analysis in about 5 seconds

## Limitations

- The source databases (HMDB, BMRB) have no mechanism to collectively identify metabolites across multiple ROIs — that logic lives in ROIAL-NMR, not the library
- Automatic synchronization of NMR chemical shift databases is planned but not yet implemented; library updates are manual
- Library-derived identifications depend on user-defined ROIs, which can lead to variable results between users
- ROIAL-NMR can utilize any metabolite database containing NMR assignments, but the constructed workbook must match the expected schema or loading will fail

## Evidence

- [intro] BMRB as library source: [evidence span withheld — anchor status: not_found]
- [methods] both tables required: "we found that both are required for accurate metabolite identification"
- [discussion] any metabolite database with NMR assignments: "Here, we used HMDB as a demonstrating platform; however, ROIAL-NMR can utilize any metabolite database containing NMR assignments"
- [intro] default library size: "it enables the rapid (within 5 seconds) identification of a broad range of metabolites from ROIs by utilizing chemical shift data for 891 metabolites from the HMDB NMR metabolite database"
- [intro] databases lack cross-ROI mechanism: "there is no mechanism built into these databases that can collectively identify potential metabolites that may exist in multiple spectral regions-of-interest (ROIs) simultaneously"
