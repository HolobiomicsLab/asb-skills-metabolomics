---
name: nmr-chemical-shift-database-construction
description: Use when a ROI-based NMR metabolite identification tool needs reference
  chemical shift and J-coupling (SRRM) libraries, and you must derive them from HMDB
  records and organize them by biological sample type.
when_to_use_negative:
- You need absolute quantification of compounds — the program this database supports
  was designed for identification/categorization, and 'absolute quantification of
  the compounds is not the primary aim of the program'.
- You need automated ROI identification — automated identification of ROIs is beyond
  the scope of the current version of the program.
- You need continuously auto-updated chemical shift libraries — periodic and automatic
  synchronization of NMR chemical shift databases is planned but not yet implemented.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ROIAL-NMR
  role: 'Downstream consumer of the chemical shift and SRRM databases: it identifies
    potential metabolites from user-defined ROIs by matching against HMDB-derived
    shift data'
  repo: github.com/Leo-Cheng-Lab/ROIAL-NMR
- name: Python
  role: Language used (>=3.9) to parse HMDB records and assemble the databases
- name: pandas
  role: Parsing and tabulating metabolite chemical shift and SRRM records
- name: openpyxl
  role: Exporting the databases and per-biosample tables to an Excel workbook
- name: XlsxWriter
  role: Alternative engine for writing the exported database workbook
- name: Human Metabolome Database (HMDB)
  role: Source of the 891 metabolites' NMR assignments; used as the demonstration/reference
    platform
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:9fe6dc8c259057d8295ad263fb73fc1c2d678197f00f440d3928f229c430f9d5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Human Metabolome Database (HMDB)
  - Python
  - ROIAL-NMR
  - XlsxWriter
  - openpyxl
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nmr-chemical-shift-database-construction

## Summary

Construct program-specific metabolic reference databases — one holding 1H chemical shift values and one holding spectral region resonance multiplets (SRRMs) — by parsing Human Metabolome Database (HMDB) NMR records with Python, and split them into biosample-specific tables (serum, saliva, sweat, urine, CSF, tissues) so that tools like ROIAL-NMR can match user-defined ROIs to candidate metabolites.

## When to use

Use when a ROI-based NMR metabolite identification tool needs reference chemical shift and J-coupling (SRRM) libraries, and you must derive them from HMDB records and organize them by biological sample type.

## When NOT to use

- You need absolute quantification of compounds — the program this database supports was designed for identification/categorization, and 'absolute quantification of the compounds is not the primary aim of the program'.
- You need automated ROI identification — automated identification of ROIs is beyond the scope of the current version of the program.
- You need continuously auto-updated chemical shift libraries — periodic and automatic synchronization of NMR chemical shift databases is planned but not yet implemented.

## Inputs

- HMDB metabolite records containing 1H NMR chemical shift assignments for metabolites detectable in biological samples/biofluids (serum, plasma, urine, etc.)
- Python >=3.9 environment with pandas 2.2.3, openpyxl 3.1.5, XlsxWriter 3.2.2

## Outputs

- Chemical shift database (metabolite → peak ppm values) covering 891 metabolites
- SRRM database (metabolite → spectral region resonance multiplet intervals from J-couplings)
- Biosample-specific database tables (plasma, serum, CSF, feces, saliva, sweat, urine) exported to an Excel workbook

## How to apply

Download HMDB metabolite records carrying 1H NMR chemical shift assignments for metabolites detectable in biological samples/biofluids, and parse each record with Python and pandas (2.2.3) to extract individual chemical shift peak values (ppm) and the SRRMs arising from J-couplings. Build two databases per the ROIAL-NMR design — one of chemical shift values and one of SRRMs — because the article found both are required for accurate metabolite identification. Organize records into separate biosample-specific tables (e.g., plasma, serum, CSF, feces, saliva, sweat, urine) so that identification can take 'the intersection of all inputs (ROI specifications, metabolic NMR information, and sample-specific information)'. Export both databases and the per-biosample tables to an Excel workbook with openpyxl (3.1.5) or XlsxWriter (3.2.2). Spot-check a known entry: lactate must contain two SRRMs (1.30–1.33 and 4.08–4.13 ppm) with six peaks (1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm). Validation target: the compiled database covers 891 metabolites with NMR assignments from the HMDB NMR metabolite database, the reference platform the program uses to enable identification within 5 seconds.

## Related tools

- **ROIAL-NMR** (Downstream consumer of the chemical shift and SRRM databases: it identifies potential metabolites from user-defined ROIs by matching against HMDB-derived shift data) — github.com/Leo-Cheng-Lab/ROIAL-NMR
- **Python** (Language used (>=3.9) to parse HMDB records and assemble the databases)
- **pandas** (Parsing and tabulating metabolite chemical shift and SRRM records)
- **openpyxl** (Exporting the databases and per-biosample tables to an Excel workbook)
- **XlsxWriter** (Alternative engine for writing the exported database workbook)
- **Human Metabolome Database (HMDB)** (Source of the 891 metabolites' NMR assignments; used as the demonstration/reference platform)

## Examples

```
python main.py
```

## Evaluation signals

- Compiled database contains 891 metabolites with NMR assignments sourced from the HMDB NMR metabolite database
- Spot-check lactate entry reproduces two SRRMs (1.30–1.33 and 4.08–4.13 ppm) with six peaks (1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm)
- Separate biosample-specific tables exist for plasma, serum, CSF, feces, saliva, sweat, and urine
- Both database types (chemical shift values and SRRMs) are present, since both are required for accurate metabolite identification
- Excel export opens correctly via openpyxl/XlsxWriter and preserves ppm values and multiplet intervals

## Limitations

- HMDB was used as a demonstration platform; ROIAL-NMR can utilize any metabolite database containing NMR assignments (e.g., BMRB), so coverage is bounded by the source database's 891 NMR-assigned metabolites
- Periodic and automatic synchronization of NMR chemical shift databases is planned but not yet implemented — updates require manual re-derivation
- Database content is filtered to metabolites with detectable NMR assignments; metabolites lacking assigned peaks cannot be identified downstream (e.g., the program excludes metabolites with concentration limits below 5 μM, the lower limit of detection by most NMR instruments)

## Evidence
