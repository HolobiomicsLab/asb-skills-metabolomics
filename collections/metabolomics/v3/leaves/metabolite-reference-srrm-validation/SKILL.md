---
name: metabolite-reference-srrm-validation
description: Use when preparing or verifying the reference libraries (chemical shifts
  + SRRMs per metabolite, split by serum/plasma/urine/CSF/saliva/sweat/feces/tissue)
  that a ROI-based NMR identification program consumes.
when_to_use_negative:
- The goal is absolute quantification of metabolites — the ROIAL-NMR workflow explicitly
  does not aim at absolute quantification
- The user expects automated ROI detection — automated identification of ROIs is beyond
  the scope of the current version and ROIs must be user-defined
- The reference metabolites come from a database without NMR assignments — ROIAL-NMR
  requires metabolite databases containing NMR chemical shift data (HMDB, BMRB, etc.)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3883
tools:
- name: ROIAL-NMR
  role: Consumes the compiled chemical shift + SRRM reference databases (HMDB-derived,
    organized by sample type) to identify potential metabolites from user-defined
    proton NMR ROIs
  repo: github.com/Leo-Cheng-Lab/ROIAL-NMR
- name: Python >=3.9
  role: Runtime environment (installed via Anaconda) for parsing HMDB records and
    running ROIAL-NMR
  repo: https://www.anaconda.com/download/
- name: pandas 2.2.3
  role: Parsing HMDB metabolite records and structuring the chemical shift and SRRM
    database tables
- name: openpyxl 3.1.5
  role: Exporting the compiled databases and biosample-specific tables to Excel workbooks
- name: XlsxWriter 3.2.2
  role: Writing the database tables to .xlsx output files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:4a96330f1a3239e960b3a2e1e06df92b99f359f1d2e6e1d21483379389f7e9b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Python >=3.9
  - ROIAL-NMR
  - XlsxWriter
  - XlsxWriter 3.2.2
  - openpyxl
  - openpyxl 3.1.5
  - pandas
  - pandas 2.2.3
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3883
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metabolite-reference-srrm-validation

## Summary

Build and validate program-specific metabolic reference databases — chemical shift values and spectral region resonance multiplets (SRRMs) — derived from the HMDB NMR metabolite database and organized by biological sample type, so that ROIAL-NMR can identify metabolites from user-defined proton NMR regions-of-interest.

## When to use

Use when preparing or verifying the reference libraries (chemical shifts + SRRMs per metabolite, split by serum/plasma/urine/CSF/saliva/sweat/feces/tissue) that a ROI-based NMR identification program consumes.

## When NOT to use

- The goal is absolute quantification of metabolites — the ROIAL-NMR workflow explicitly does not aim at absolute quantification
- The user expects automated ROI detection — automated identification of ROIs is beyond the scope of the current version and ROIs must be user-defined
- The reference metabolites come from a database without NMR assignments — ROIAL-NMR requires metabolite databases containing NMR chemical shift data (HMDB, BMRB, etc.)

## Inputs

- HMDB metabolite records containing 1H NMR chemical shift assignments (for metabolites detectable in serum, plasma, urine, CSF, saliva, sweat, feces, tissues)
- Per-metabolite chemical shift peak lists (ppm) and spectral region resonance multiplet (SRRM) regions
- Metabolite concentration limit metadata (lower limit of detection ~5 μM)

## Outputs

- Chemical shift database table (891 HMDB metabolites)
- SRRM database table (891 HMDB metabolites)
- Biosample-specific database tables (plasma, serum, CSF, feces, saliva, sweat, urine)
- Excel workbook export (openpyxl/XlsxWriter .xlsx)

## How to apply

Download HMDB metabolite records carrying 1H NMR chemical shift assignments for metabolites detectable in biological samples, and parse each record (e.g. with Python/pandas) to extract individual chemical shift peaks in ppm and the J-coupling-derived SRRM regions. Compile two tables — one of chemical shift values, one of SRRMs — for every metabolite with NMR assignments (891 metabolites from HMDB), and generate separate tables per sample type (plasma, serum, CSF, feces, saliva, sweat, urine). Export both tables and the per-biosample tables to an Excel workbook via openpyxl/XlsxWriter, then spot-check known entries: lactate must contain two SRRMs (1.30-1.33 and 4.08-4.13 ppm) with six peaks (1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm). The rationale is that ROIAL-NMR's identification takes the intersection of user-defined ROI specifications, metabolic NMR information, and sample-specific information, so both chemical shifts and SRRMs are required for accurate metabolite identification; metabolites are then scored by a match ratio (>=50% of SRRMs within analyzed ROIs) and excluded if concentration limits fall below 5 μM.

## Related tools

- **ROIAL-NMR** (Consumes the compiled chemical shift + SRRM reference databases (HMDB-derived, organized by sample type) to identify potential metabolites from user-defined proton NMR ROIs) — github.com/Leo-Cheng-Lab/ROIAL-NMR
- **Python >=3.9** (Runtime environment (installed via Anaconda) for parsing HMDB records and running ROIAL-NMR) — https://www.anaconda.com/download/
- **pandas 2.2.3** (Parsing HMDB metabolite records and structuring the chemical shift and SRRM database tables)
- **openpyxl 3.1.5** (Exporting the compiled databases and biosample-specific tables to Excel workbooks)
- **XlsxWriter 3.2.2** (Writing the database tables to .xlsx output files)

## Examples

```
python main.py
```

## Evaluation signals

- The compiled database contains exactly 891 metabolites with NMR assignments sourced from the HMDB NMR metabolite database
- Lactate spot-check passes: two SRRMs at 1.30-1.33 and 4.08-4.13 ppm containing six peaks at 1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm
- Separate biosample-specific tables exist for plasma, serum, CSF, feces, saliva, sweat, and urine
- Both databases and all per-biosample tables export successfully to an Excel workbook (openpyxl/XlsxWriter)
- Identification run against a real spectrum returns candidate metabolites whose SRRMs fall within the user-defined ROIs with a match ratio >= 50%, in roughly 5 seconds

## Limitations

- Automatic synchronization of NMR chemical shift databases is planned but not yet implemented, so reference libraries must be manually updated
- Results depend on user consistency in creating ROIs; different users may obtain variable results from the same data
- ROIAL-NMR was not built to validate the accuracy of user-defined ROIs, so erroneous ROI definitions propagate into identifications regardless of database quality
- If more than 50% of a potential metabolite's peaks lie beyond the analyzed region of interest, those peaks are excluded from the match-ratio calculation, which can affect identifications near spectral edges

## Evidence
