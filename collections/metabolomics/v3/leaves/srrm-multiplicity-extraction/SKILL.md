---
name: srrm-multiplicity-extraction
description: Use when you need to compile biosample-specific NMR reference databases
  (chemical shift peaks and SRRM multiplet regions) from HMDB records to power ROI-based
  metabolite identification in ROIAL-NMR.
when_to_use_negative:
- When the target metabolite database is not HMDB-based — the paper notes ROIAL-NMR
  can also use BMRB or any database containing NMR assignments, so re-deriving SRRMs
  from HMDB is unnecessary if another curated database is already loaded
- When the task is ROI identification or filtering (match ratio, significance ratio)
  rather than reference-database construction — those are downstream ROIAL-NMR steps,
  not database building
- When automated ROI identification is the goal — that is beyond the current scope
  of the program
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: ROIAL-NMR
  role: Downstream consumer of the extracted chemical shift and SRRM databases; its
    identification step matches user-defined ROIs against these per-metabolite multiplet
    regions
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Parsing HMDB metabolite records and structuring extracted peak/SRRM values
    into tabular form
- name: openpyxl
  role: Exporting the compiled chemical shift and SRRM databases and per-biosample
    tables to Excel
- name: XlsxWriter
  role: Alternative Excel export engine for writing the database workbook
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:33bbc11e8787e1300c232dda692b051ee0e4219828eadc5968bce161290f5ab1
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
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# srrm-multiplicity-extraction

## Summary

Builds the program-specific metabolic lookup databases used by ROIAL-NMR by parsing HMDB metabolite records to extract per-metabolite 1H NMR chemical shift values and spectral region resonance multiplets (SRRMs), organized by biosample type. Both chemical shifts and SRRMs are required together for accurate metabolite identification.

## When to use

Use when you need to compile biosample-specific NMR reference databases (chemical shift peaks and SRRM multiplet regions) from HMDB records to power ROI-based metabolite identification in ROIAL-NMR.

## When NOT to use

- When the target metabolite database is not HMDB-based — the paper notes ROIAL-NMR can also use BMRB or any database containing NMR assignments, so re-deriving SRRMs from HMDB is unnecessary if another curated database is already loaded
- When the task is ROI identification or filtering (match ratio, significance ratio) rather than reference-database construction — those are downstream ROIAL-NMR steps, not database building
- When automated ROI identification is the goal — that is beyond the current scope of the program

## Inputs

- HMDB metabolite records with 1H NMR chemical shift assignments
- Biosample type labels (serum, plasma, CSF, feces, saliva, sweat, urine, tissues) for organizing per-sample-type tables

## Outputs

- Chemical shift database (per-metabolite peak ppm values) for 891 HMDB metabolites
- SRRM database (per-metabolite multiplet regions)
- Biosample-specific database tables (e.g. serum, urine, CSF)
- Excel workbook (.xlsx) exported via openpyxl/XlsxWriter

## How to apply

Download HMDB metabolite records containing 1H NMR chemical shift assignments and parse each record with Python/pandas to extract individual peak values (ppm) and the spectral region resonance multiplets (SRRMs) arising from J-couplings. Build two databases — one holding chemical shift values and one holding SRRMs — for every metabolite with NMR assignments (891 metabolites in the paper), organizing records into biosample-specific tables for sample types such as serum, saliva, sweat, urine, CSF, feces, and tissues. Spot-check representative entries (e.g. lactate must contain two SRRMs at 1.30–1.33 and 4.08–4.13 ppm with six peaks: 1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm) before export. Export both databases and per-biosample tables to an Excel workbook via openpyxl/XlsxWriter for use by ROIAL-NMR's identification engine, which takes the intersection of ROI specifications, metabolic NMR information, and sample-specific information. Keep both databases in sync because the paper found that chemical shift values and metabolite-specific SRRMs are both required for accurate metabolite identification.

## Related tools

- **ROIAL-NMR** (Downstream consumer of the extracted chemical shift and SRRM databases; its identification step matches user-defined ROIs against these per-metabolite multiplet regions) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Parsing HMDB metabolite records and structuring extracted peak/SRRM values into tabular form)
- **openpyxl** (Exporting the compiled chemical shift and SRRM databases and per-biosample tables to Excel)
- **XlsxWriter** (Alternative Excel export engine for writing the database workbook)

## Examples

```
python main.py  # launch ROIAL-NMR GUI; load the compiled HMDB chemical-shift/SRRM Excel database and select sample type before ROI analysis
```

## Evaluation signals

- Compiled database contains 891 metabolites with NMR assignments sourced from the HMDB NMR metabolite database
- Lactate spot-check reproduces two SRRMs (1.30–1.33 and 4.08–4.13 ppm) with six peaks (1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm)
- Both chemical shift values and metabolite-specific SRRMs are present for each metabolite; identification quality degrades if either is missing
- Exported workbook opens with pandas/openpyxl and separates tables by biosample type (serum, plasma, CSF, feces, saliva, sweat, urine)
- Downstream ROIAL-NMR runs complete single-population analysis in about 5 seconds using the compiled database

## Limitations

- ROIAL-NMR relies on manual database input; automatic synchronization of NMR chemical shift databases is planned but not yet implemented
- The paper demonstrates the pipeline only on HMDB as a reference platform; adaptation to other databases (BMRB, etc.) requires re-extraction
- ROIAL-NMR was not built to test the accuracy of user-defined ROIs, so errors propagate from badly defined ROIs regardless of database quality

## Evidence

- [methods] we found that both are required for accurate metabolite identification: [evidence span withheld — anchor status: not_found]
- [readme] using the Human Metabolome Database (HMDB) as a reference platform: [evidence span withheld — anchor status: not_found]
