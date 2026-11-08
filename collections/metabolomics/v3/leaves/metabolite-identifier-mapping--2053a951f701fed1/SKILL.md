---
name: metabolite-identifier-mapping
description: Use when a study needs ROIAL-NMR to run against a metabolite NMR reference
  other than the HMDB — e.g., building a BMRB-derived library from metabolite 1H chemical
  shift assignments for user-selected database loading.
when_to_use_negative:
- The goal is automated identification of spectral ROIs — the article states 'Automated
  identification of ROIs is beyond the scope of the current version of the program'.
- The goal is absolute quantification of metabolites — ROIAL-NMR explicitly does not
  target absolute quantification.
- The goal is validating the accuracy of user-defined ROIs — 'ROIAL-NMR was not built
  to test the accuracy of the user-defined ROIs'.
tools:
- name: ROIAL-NMR
  role: Target program whose user-selected metabolic database the generated workbook
    plugs into; its match-ratio logic defines the required schema
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Parses BMRB chemical shift assignment records into structured tables (version
    2.2.3 per the article's requirements)
- name: XlsxWriter
  role: Writes the chemical-shift and SRRM tables into the single Excel workbook consumed
    by ROIAL-NMR (version 3.2.2)
- name: openpyxl
  role: Used to validate that the generated workbook opens cleanly (version 3.1.5
    listed in requirements)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:fa635b06e16b922478d0cf7dbded0559881f98cbcab26761a9c254c4db4026c3
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# metabolite-identifier-mapping

## Summary

Builds a ROIAL-NMR-compatible metabolite NMR assignment library from an alternative source database (e.g., BMRB) by parsing its 1H chemical shift records into the two-table Excel schema (chemical-shift values and SRRMs) that ROIAL-NMR's match-ratio logic consumes, mirroring the HMDB-derived demonstration libraries.

## When to use

Use when a study needs ROIAL-NMR to run against a metabolite NMR reference other than the HMDB — e.g., building a BMRB-derived library from metabolite 1H chemical shift assignments for user-selected database loading.

## When NOT to use

- The goal is automated identification of spectral ROIs — the article states 'Automated identification of ROIs is beyond the scope of the current version of the program'.
- The goal is absolute quantification of metabolites — ROIAL-NMR explicitly does not target absolute quantification.
- The goal is validating the accuracy of user-defined ROIs — 'ROIAL-NMR was not built to test the accuracy of the user-defined ROIs'.

## Inputs

- BMRB metabolite 1H chemical shift assignment records (metabolite names, biofluid/sample-type context, 1H peak values, J-coupling multiplicity)
- HMDB-derived ROIAL-NMR library workbook used as the schema template

## Outputs

- Single Excel workbook containing two tables: per-metabolite 1H chemical shift values and per-metabolite SRRM regions, in the ROIAL-NMR database schema

## How to apply

Download metabolite 1H chemical shift assignment records from the alternative source database (BMRB is named in the article as an alternative to the HMDB demonstration platform) and parse them with pandas (2.2.3), retaining metabolite names, biofluid/sample-type context, 1H chemical shift peak values, and J-coupling multiplicity information. Generate the first program-specific table of chemical shift values per metabolite and the second table of spectral region resonance multiplets (SRRMs, i.e., multiplets due to J-couplings), mirroring the structure of the HMDB-derived libraries from the original task. Write both tables into a single Excel workbook with XlsxWriter 3.2.2 using the same sheet layout and column schema as the HMDB-derived libraries so the file can be dropped into ROIAL-NMR as a user-selected metabolic database. Both chemical shift values and metabolite-specific SRRMs are required for accurate metabolite identification, because ROIAL-NMR scores each metabolite by the fraction of its SRRMs matched inside user-defined ROIs (only metabolites with a match ratio >= 50% are kept), so the SRRM sheet cannot be omitted.

## Related tools

- **ROIAL-NMR** (Target program whose user-selected metabolic database the generated workbook plugs into; its match-ratio logic defines the required schema) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Parses BMRB chemical shift assignment records into structured tables (version 2.2.3 per the article's requirements))
- **XlsxWriter** (Writes the chemical-shift and SRRM tables into the single Excel workbook consumed by ROIAL-NMR (version 3.2.2))
- **openpyxl** (Used to validate that the generated workbook opens cleanly (version 3.1.5 listed in requirements))

## Examples

```
python main.py
```

## Evaluation signals

- The generated workbook opens cleanly with openpyxl/pandas without format errors.
- Each metabolite row exposes both chemical-shift peaks and SRRM regions so ROIAL-NMR's match-ratio logic (peak-in-ROI matching per SRRM, e.g., lactate's 2/2 example) can be computed.
- Sheet layout and column schema match the HMDB-derived chemical-shift and SRRM libraries so ROIAL-NMR accepts the file as a user-selected metabolic database.
- Metabolite records retain biofluid/sample-type context and J-coupling multiplicity needed for SRRM construction.
- ROIAL-NMR completes its analysis in its expected runtime (~5 seconds single-population, <10 seconds multi-group) when loaded with the new library.

## Limitations

- Source NMR databases (HMDB, BMRB) lack a mechanism to collectively identify metabolites present in multiple ROIs simultaneously — the cross-ROI collective identification is what ROIAL-NMR adds, not the databases.
- Automatic synchronization of NMR chemical shift databases is planned but not yet implemented, so library updates from BMRB/HMDB are manual.
- User-defined ROI creation is a source of variability: 'using user-defined NMR spectral ROIs can lead to variable results obtained by different users'.

## Evidence
