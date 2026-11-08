---
name: metabolite-database-format-replication
description: Use when a study needs ROIAL-NMR to match metabolites from a reference
  source other than the default HMDB — e.g., BMRB — and a drop-in replacement metabolic
  database workbook must be generated in the program's expected format.
when_to_use_negative:
- When absolute quantification is the goal — the article states 'absolute quantification
  of the compounds is not the primary aim of the program' and the database supplies
  only assignments
- When automated ROI identification is needed — this is beyond the current program
  scope and unrelated to database construction
- When the alternative database lacks metabolite assignments keyed to proton NMR chemical
  shift values — the source data must contain chemical shift-keyed assignments to
  be usable
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0091
tools:
- name: ROIAL-NMR
  role: Target program whose metabolic database workbook format is being replicated;
    validates the library via match-ratio logic over ROIs
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Parse and structure the BMRB chemical shift assignment records into the two
    required tables
- name: XlsxWriter
  role: Write the two-table chemical-shift and SRRM library into a single Excel workbook
    in the HMDB-derived schema
- name: openpyxl
  role: Validate that the generated workbook opens cleanly and rows expose both peaks
    and SRRM regions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:8eee9cc756d03c2376adb5a9f2fed92d21bd78bdd3168c59509f5c1dee8e41f0
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolite-database-format-replication

## Summary

Build a ROIAL-NMR-compatible metabolite NMR assignment library from an alternative source (e.g., BMRB) by replicating the exact two-table Excel workbook schema of the HMDB-derived libraries: one table of 1H chemical shift values per metabolite and one table of spectral region resonance multiplets (SRRMs) reflecting J-coupling multiplicity.

## When to use

Use when a study needs ROIAL-NMR to match metabolites from a reference source other than the default HMDB — e.g., BMRB — and a drop-in replacement metabolic database workbook must be generated in the program's expected format.

## When NOT to use

- When absolute quantification is the goal — the article states 'absolute quantification of the compounds is not the primary aim of the program' and the database supplies only assignments
- When automated ROI identification is needed — this is beyond the current program scope and unrelated to database construction
- When the alternative database lacks metabolite assignments keyed to proton NMR chemical shift values — the source data must contain chemical shift-keyed assignments to be usable

## Inputs

- Metabolite 1H chemical shift assignment records from an alternative NMR reference database (e.g., BMRB)
- Metabolite names, biofluid/sample-type context, 1H chemical shift peak values, and J-coupling multiplicity fields
- Reference HMDB-derived ROIAL-NMR metabolic database workbook as the schema template

## Outputs

- Single Excel workbook with two sheets: a chemical-shift table and an SRRM (spectral region resonance multiplet) table per metabolite, in the HMDB-derived library column schema
- Drop-in user-selected metabolic database file loadable by ROIAL-NMR

## How to apply

Download metabolite 1H chemical shift assignment records from the alternative database (the paper names BMRB alongside HMDB as a valid NMR assignment source) and parse them with pandas, retaining metabolite names, biofluid/sample-type context, chemical shift peak values, and J-coupling multiplicity. Generate two tables mirroring the HMDB-derived libraries: (1) chemical shift values per metabolite with NMR assignments, and (2) per-metabolite SRRMs (multiplets due to J-couplings). Write both tables into a single Excel workbook with XlsxWriter using the same sheet layout and column schema as the HMDB-derived libraries so ROIAL-NMR can accept it as a user-selected metabolic database. Validate the workbook opens cleanly (openpyxl/pandas) and that each metabolite row exposes both chemical-shift peaks and SRRM regions, because ROIAL-NMR requires both for accurate identification and computes match ratios as peaks-in-ROI per SRRM (e.g., lactate's 2/2 example).

## Related tools

- **ROIAL-NMR** (Target program whose metabolic database workbook format is being replicated; validates the library via match-ratio logic over ROIs) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Parse and structure the BMRB chemical shift assignment records into the two required tables)
- **XlsxWriter** (Write the two-table chemical-shift and SRRM library into a single Excel workbook in the HMDB-derived schema)
- **openpyxl** (Validate that the generated workbook opens cleanly and rows expose both peaks and SRRM regions)

## Examples

```
python main.py  # then select the generated BMRB-derived .xlsx metabolic database workbook in the ROIAL-NMR GUI
```

## Evaluation signals

- The workbook opens cleanly with openpyxl/pandas without format errors
- Each metabolite row exposes both chemical-shift peaks and SRRM regions so ROIAL-NMR's match-ratio logic (peak-in-ROI matching per SRRM) can be computed
- Sheet layout and column schema match the HMDB-derived libraries exactly, allowing the file to be selected as a user metabolic database in ROIAL-NMR
- SRRM multiplicity reflects J-coupling information retained from the source records (e.g., lactate yields 2/2 SRRM matches)
- Both tables required by ROIAL-NMR are present — the paper found that chemical shift values and metabolite-specific SRRMs are both required for accurate metabolite identification

## Limitations

- Databases such as BMRB lack a mechanism to collectively identify metabolites across multiple ROIs — the library only supplies assignments; cross-ROI identification must happen in ROIAL-NMR
- Automatic synchronization of NMR chemical shift databases is planned but not yet implemented, so the library must be rebuilt manually when the source database updates
- Results depend on user consistency; variable ROI definitions between users yield variable identifications regardless of library quality

## Evidence
