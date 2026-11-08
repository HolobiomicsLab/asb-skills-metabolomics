---
name: spectral-region-resonance-multiplet-annotation
description: Use when ROIAL-NMR users need a metabolic reference library built from
  a source other than the HMDB (e.g., BMRB), with per-metabolite chemical-shift peaks
  and J-coupling multiplet (SRRM) regions in the expected Excel workbook schema.
when_to_use_negative:
- The HMDB-derived library already covers the metabolites of interest — ROIAL-NMR
  ships with chemical-shift data for 891 HMDB metabolites and the HMDB remains the
  paper's demonstration platform
- Automated ROI identification or absolute metabolite quantification is the goal —
  the article explicitly states both are beyond ROIAL-NMR's scope and this skill only
  supplies the reference database
- Validating the accuracy of user-defined ROIs — ROIAL-NMR was not built to test ROI
  accuracy
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: ROIAL-NMR
  role: 'Consumer of the BMRB-derived library: loads the generated .xlsx workbook
    as a user-selected metabolic database and computes match ratios of peaks within
    user-defined ROIs per SRRM'
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Parsing BMRB assignment records and structuring metabolite/chemical-shift/multiplicity
    data into DataFrames before export
- name: XlsxWriter
  role: Writing the chemical-shift and SRRM tables into the single Excel workbook
    in ROIAL-NMR's expected sheet layout
- name: openpyxl
  role: Re-opening the generated workbook to validate that it loads cleanly and rows
    expose both chemical-shift peaks and SRRM regions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:fdbb8b08ff2743ee12dbde6c4c85280ebd41ab45fb23bea7e080798dee76eed7
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spectral-region-resonance-multiplet-annotation

## Summary

Builds a metabolite NMR assignment library from an alternative biological NMR reference database (BMRB instead of HMDB), producing the two Excel tables — per-metabolite 1H chemical-shift values and spectral region resonance multiplets (SRRMs) — that ROIAL-NMR requires as a user-selected metabolic database.

## When to use

Use when ROIAL-NMR users need a metabolic reference library built from a source other than the HMDB (e.g., BMRB), with per-metabolite chemical-shift peaks and J-coupling multiplet (SRRM) regions in the expected Excel workbook schema.

## When NOT to use

- The HMDB-derived library already covers the metabolites of interest — ROIAL-NMR ships with chemical-shift data for 891 HMDB metabolites and the HMDB remains the paper's demonstration platform
- Automated ROI identification or absolute metabolite quantification is the goal — the article explicitly states both are beyond ROIAL-NMR's scope and this skill only supplies the reference database
- Validating the accuracy of user-defined ROIs — ROIAL-NMR was not built to test ROI accuracy

## Inputs

- BMRB metabolite 1H chemical-shift assignment records (metabolite names, biofluid/sample-type context, peak ppm values, J-coupling multiplicity)
- ROIAL-NMR HMDB-derived library workbook as schema template (chemical-shift table + SRRM table in a single .xlsx file)

## Outputs

- Single Excel workbook (written with XlsxWriter) containing a per-metabolite 1H chemical-shift table
- Per-metabolite SRRM (spectral region resonance multiplet) table in the same workbook, drop-in compatible as a ROIAL-NMR user-selected metabolic database

## How to apply

Download metabolite 1H chemical-shift assignment records from BMRB, which the paper names alongside the HMDB as a valid NMR assignment source. Parse the records with pandas (>=2.2.3), retaining metabolite names, biofluid/sample-type context, 1H chemical-shift peak values, and J-coupling multiplicity information. Generate a first table of chemical-shift values per metabolite and a second table of SRRMs (multiplets arising from J-couplings) per metabolite, mirroring the HMDB-derived libraries already used by ROIAL-NMR (891 metabolites at the time of the paper). Write both tables into a single Excel workbook with XlsxWriter 3.2.2 using the same sheet layout and column schema so ROIAL-NMR's match-ratio logic — peak-in-ROI matching per SRRM (e.g., lactate's 2/2 example, metabolites kept at match ratio >= 50%) — can be computed without format errors.

## Related tools

- **ROIAL-NMR** (Consumer of the BMRB-derived library: loads the generated .xlsx workbook as a user-selected metabolic database and computes match ratios of peaks within user-defined ROIs per SRRM) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Parsing BMRB assignment records and structuring metabolite/chemical-shift/multiplicity data into DataFrames before export)
- **XlsxWriter** (Writing the chemical-shift and SRRM tables into the single Excel workbook in ROIAL-NMR's expected sheet layout)
- **openpyxl** (Re-opening the generated workbook to validate that it loads cleanly and rows expose both chemical-shift peaks and SRRM regions)

## Examples

```
import pandas as pd; shifts = pd.read_csv('bmrb_assignments.csv'); srrm = pd.read_csv('bmrb_srrms.csv');
with pd.ExcelWriter('bmrb_library.xlsx', engine='xlsxwriter') as w:
    shifts.to_excel(w, sheet_name='ChemicalShifts', index=False)
    srrm.to_excel(w, sheet_name='SRRMs', index=False)
```

## Evaluation signals

- Workbook opens cleanly with openpyxl/pandas and contains both the chemical-shift and SRRM sheets in the HMDB-derived schema
- Every metabolite row exposes both chemical-shift peak values and at least one SRRM region, since 'both are required for accurate metabolite identification'
- Spot-check a known metabolite (e.g., lactate) shows its full SRRM set so a 2/2 (100%) match ratio is computable when all its peaks fall inside user ROIs
- ROIAL-NMR loads the workbook as a user-selected database and completes analysis without format errors, in its expected ~5 seconds for single-population analysis

## Limitations

- Databases such as BMRB and HMDB lack a mechanism to collectively identify metabolites present in multiple ROIs simultaneously — ROIAL-NMR supplies that aggregation, but the library itself must still be curated
- Auto-generated library entries inherit the source database's chemical-shift coverage; metabolites missing from BMRB assignments will not be identified
- Results remain dependent on user consistency in defining ROIs; variable results between users are acknowledged by the authors

## Evidence
