---
name: hmdb-chemical-shift-parsing
description: Use when building a reference metabolite library from HMDB NMR assignments
  — i.e., extracting per-metabolite chemical shift peaks (ppm) and SRRM multiplet
  regions needed for spectral region-of-interest (ROI) matching in untargeted NMR
  metabolomics.
when_to_use_negative:
- You need automated ROI identification or absolute metabolite quantification — ROIAL-NMR
  explicitly treats both as beyond its scope
- You need a reference database beyond what HMDB NMR assignments cover — note that
  ROIAL-NMR can also ingest BMRB and other NMR-assignment databases, so forcing everything
  through HMDB parsing may be the wrong path
- You need auto-synced chemical shift databases — periodic/automatic synchronization
  of NMR chemical shift databases is planned but not yet implemented
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: ROIAL-NMR
  role: Consumer of the parsed HMDB chemical shift/SRRM libraries; its per-ROI matching
    screens 891 metabolites against user-defined ROIs
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Python >=3.9
  role: Language used to parse HMDB records and build the chemical shift and SRRM
    databases
- name: pandas 2.2.3
  role: DataFrame parsing/organization of chemical shift peaks and SRRMs per metabolite
    and biosample type
- name: openpyxl 3.1.5
  role: Export the compiled chemical shift and SRRM databases to Excel workbooks
- name: XlsxWriter 3.2.2
  role: Excel export of the compiled database tables
- name: Human Metabolome Database (HMDB)
  role: Source reference platform of 1H NMR chemical shift assignments underpinning
    the libraries
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:291c9651bae441a4b96abfb0b2e559df19a7c22af20f975ca3fff85ab8975ca8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Human Metabolome Database (HMDB)
  - Python
  - Python >=3.9
  - ROIAL-NMR
  - XlsxWriter
  - XlsxWriter 3.2.2
  - openpyxl
  - openpyxl 3.1.5
  - pandas
  - pandas 2.2.3
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# hmdb-chemical-shift-parsing

## Summary

Parse 1H NMR chemical shift assignments from the Human Metabolome Database (HMDB) into program-specific metabolic libraries of chemical shift values and spectral region resonance multiplets (SRRMs) for 891 metabolites, organized by biosample type (serum, plasma, CSF, feces, saliva, sweat, urine), so that downstream ROI-matching (ROIAL-NMR) can identify potential metabolites within ~5 seconds.

## When to use

Use when building a reference metabolite library from HMDB NMR assignments — i.e., extracting per-metabolite chemical shift peaks (ppm) and SRRM multiplet regions needed for spectral region-of-interest (ROI) matching in untargeted NMR metabolomics.

## When NOT to use

- You need automated ROI identification or absolute metabolite quantification — ROIAL-NMR explicitly treats both as beyond its scope
- You need a reference database beyond what HMDB NMR assignments cover — note that ROIAL-NMR can also ingest BMRB and other NMR-assignment databases, so forcing everything through HMDB parsing may be the wrong path
- You need auto-synced chemical shift databases — periodic/automatic synchronization of NMR chemical shift databases is planned but not yet implemented

## Inputs

- HMDB metabolite records with 1H NMR chemical shift assignments (ppm) for metabolites detectable in biological samples/biofluids
- Biosample-type labels (serum, plasma, urine, CSF, feces, saliva, sweat) used to organize biosample-specific library tables

## Outputs

- Database #1: per-metabolite chemical shift values (ppm) for 891 HMDB metabolites
- Database #2: per-metabolite spectral region resonance multiplets (SRRMs) for 891 HMDB metabolites
- Biosample-specific library tables (plasma, serum, CSF, feces, saliva, sweat, urine)
- Excel workbook of both databases and per-biosample-type tables (openpyxl/XlsxWriter)

## How to apply

Download HMDB metabolite records carrying 1H NMR chemical shift assignments for biologically detectable metabolites, then parse each record with Python/pandas to extract individual chemical shift peak values (ppm) and SRRMs — the multiplet regions arising from J-couplings. Build two databases: one holding chemical shift values and one holding SRRMs for every assigned metabolite (891 in the ROIAL-NMR build), and create separate biosample-specific tables (plasma, serum, CSF, feces, saliva, sweat, urine). Export both databases and the per-biosample tables to an Excel workbook (openpyxl/XlsxWriter). Spot-check a known entry: the lactate record must contain two SRRMs (1.30–1.33 and 4.08–4.13 ppm) with six peaks (1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm); both chemical shifts and SRRMs are required together for accurate identification downstream.

## Related tools

- **ROIAL-NMR** (Consumer of the parsed HMDB chemical shift/SRRM libraries; its per-ROI matching screens 891 metabolites against user-defined ROIs) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Python >=3.9** (Language used to parse HMDB records and build the chemical shift and SRRM databases)
- **pandas 2.2.3** (DataFrame parsing/organization of chemical shift peaks and SRRMs per metabolite and biosample type)
- **openpyxl 3.1.5** (Export the compiled chemical shift and SRRM databases to Excel workbooks)
- **XlsxWriter 3.2.2** (Excel export of the compiled database tables)
- **Human Metabolome Database (HMDB)** (Source reference platform of 1H NMR chemical shift assignments underpinning the libraries)

## Examples

```
git clone https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git; python main.py
```

## Evaluation signals

- Compiled database contains exactly 891 metabolites with NMR assignments from the HMDB
- Lactate spot-check reproduces the paper-reported entry: two SRRMs (1.30–1.33 and 4.08–4.13 ppm) and six peaks (1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm)
- Downstream validation: matching lactate's 1.324 ppm peak to ROI 1.31–1.34 and 4.111 ppm to ROI 4.10–4.12 yields match ratio 2/2
- Biosample-specific tables exist for all seven listed sample types (plasma, serum, CSF, feces, saliva, sweat, urine) and export cleanly to the Excel workbook
- Library supports the target runtime: ROIAL-NMR completes per-ROI matching across 891 metabolites in about 5 seconds

## Limitations

- Both chemical shift values and metabolite-specific SRRMs are required for accurate metabolite identification — a peaks-only library is insufficient
- ROIAL-NMR/HMDB demonstrate the approach, but database synchronization is manual; automatic synchronization of NMR chemical shift databases is planned but not implemented
- Program results depend on user consistency in creating ROIs; user-defined ROIs can lead to variable results between users
- ROIAL-NMR was not built to test the accuracy of user-defined ROIs, and absolute quantification is not the program's primary aim

## Evidence
