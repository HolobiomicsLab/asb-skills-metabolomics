---
name: excel-report-generation
description: Use when a margheRita R analysis produces a result table (identifications,
  associations, statistics) that must be persisted as an .xlsx workbook with named
  sheets for sharing or archiving.
when_to_use_negative:
- The data has not yet been processed by margheRita (no mRList object) and there is
  no result table to export
- The goal is raw instrument or MS-Dial output archiving (e.g. tab-delimited feature
  data files), which margheRita consumes rather than produces
- Downstream tools require machine-readable tab-delimited text rather than an Excel
  workbook
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: margheRita
  role: Produces the result tables (e.g. metabolite_identification() associations)
    and writes them to the 'associations' sheet of metabolite_identification.xlsx
    via its Excel writer
  repo: https://github.com/emosca-cnr/margheRita
- name: R
  role: Runtime environment in which margheRita performs the analysis and Excel export
- name: MS-Dial
  role: Upstream generator of the feature data files that margheRita reads before
    results are exported to Excel
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:cbfc21acb47b4ea42f50a47edc51d20f8afb17146f66e86c1e1442d6d74f222f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emosca-cnr/margheRita
  tools:
  - MS-Dial
  - R
  - margheRita
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# excel-report-generation

## Summary

Writing margheRita analysis results (e.g. the feature-metabolite association table from metabolite_identification()) into structured Excel workbooks such as metabolite_identification.xlsx, with named sheets like 'associations', so downstream users receive a single portable report file.

## When to use

Use when a margheRita R analysis produces a result table (identifications, associations, statistics) that must be persisted as an .xlsx workbook with named sheets for sharing or archiving.

## When NOT to use

- The data has not yet been processed by margheRita (no mRList object) and there is no result table to export
- The goal is raw instrument or MS-Dial output archiving (e.g. tab-delimited feature data files), which margheRita consumes rather than produces
- Downstream tools require machine-readable tab-delimited text rather than an Excel workbook

## Inputs

- feature-metabolite association table (data.frame) from margheRita metabolite_identification(), with identification level, ppm mass error, RT error, and retention-index error columns
- other margheRita result tables (filtering, normalization, statistics) to be exported as sheets

## Outputs

- Excel workbook metabolite_identification.xlsx with an 'associations' sheet containing the feature-metabolite associations

## How to apply

After calling margheRita functions such as metabolite_identification() on a normalized/filtered mRList, extract the resulting feature-metabolite association table, which carries the assigned identification level plus mass-accuracy error (ppm), retention-time error, and retention-index error columns per matched feature. Write this table to a named sheet (e.g. 'associations') of an Excel file such as metabolite_identification.xlsx using the openxlsx/writexl-style writer used by margheRita, and save it to disk. Keep one logical result per sheet so the workbook mirrors the margheRita pipeline stage that produced it; the input association table originates from matching feature m/z, retention time, and MS/MS spectra against the margheRita reference spectral library for the column/modality used (e.g. RPShort positive mode).

## Related tools

- **margheRita** (Produces the result tables (e.g. metabolite_identification() associations) and writes them to the 'associations' sheet of metabolite_identification.xlsx via its Excel writer) — https://github.com/emosca-cnr/margheRita
- **R** (Runtime environment in which margheRita performs the analysis and Excel export)
- **MS-Dial** (Upstream generator of the feature data files that margheRita reads before results are exported to Excel)

## Examples

```
mRList <- metabolite_identification(mRList, library = "RPShort_pos"); write_xlsx(list(associations = mRList$metabolite_identification), "metabolite_identification.xlsx")
```

## Evaluation signals

- metabolite_identification.xlsx exists on disk and contains a sheet named 'associations'
- The 'associations' sheet includes identification-level, mass-accuracy error (ppm), retention-time error, and retention-index error columns for each matched feature
- The number of rows in 'associations' matches the number of features returned by metabolite_identification() on the mRList
- Assignments respect the library capability: margheRita's in-house spectral libraries yield up to level-1 identifications, whereas the MS-Dial library is limited to level 2
- Values are readable by an independent Excel/R reader with no truncation of identifiers or error values

## Limitations

- Level-1 identification confidence is only possible when the required MS/MS information is available in the selected reference spectral library; otherwise identifications may be lower confidence or absent
- The margheRita in-house spectral library covers ~800 metabolites across HILIC, LipC8, pZIC, RPLong and RPShort columns, far fewer than the ~10^5 metabolites in the MS-Dial library
- The export is only as complete as the upstream pre-processing: features removed by m/z, NA, or CV-ratio filtering will not appear in the report

## Evidence

- [intro] metabolite identification with margheRita is possible up to level-1 confidence, provided that the required MS/MS information is available in the selected reference spectral library: "these libraries provideup to level 1 identifications in positive and negative modalities for "HILIC", "LipC8", "pZIC", "RPLong" and "RPShort" chromatographic columns"
- [intro] MS-Dial library is larger but limited to level 2: "**MS-Dial**, which covers a much larger set of metabolites ($10^5$), but is limited to level 2 identifications in positive and negative modalities."
- [readme] margheRita performs metabolite annotation up to level-1 based on in-house and freely available spectral libraries: "metabolite annotation up to level-1, based on in-house spectral libraries as well as freely available libraries;"
- [intro] MS-Dial generates the input feature file consumed by margheRita: "usually, this file is generated by MS-Dial"
- [intro] margheRita covers the full workflow including identification, whose results are exported: "The R package margheRita addresses the complete workflow for metabolomic profiling in untargeted studies based on liquid chromatography (LC) coupled with tandem mass spectrometry (MS/MS)."
