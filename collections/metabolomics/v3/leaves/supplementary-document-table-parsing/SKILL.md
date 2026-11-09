---
name: supplementary-document-table-parsing
description: Use when a paper's quantitative claim (e.g. how many of 36 standard metabolites
  are covered by HILIC ESI+, HILIC ESI−, CE alkaline/acidic anion and CE cation methods)
  exists only as a table inside an electronic supplementary file that must be transcribed
  to structured data.
when_to_use_negative:
- The detection/coverage data are already available as a machine-readable table (CSV/Excel)
  — transcribing from DOCX would only add transcription errors
- The claim depends on raw MS feature data (mzXML/mzML peak detection) rather than
  a static published table — process with AriumMS ROI/augmentation instead
- The supplementary table is embedded as an image rather than a DOCX table object
  — python-docx cell extraction cannot recover it
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: python-docx
  role: Parses the supplementary DOCX Table 1 cell-by-cell to transcribe the 40 analyte
    rows and their five-platform detection flags
- name: AriumMS
  role: Source of the per-metabolite detection checkmarks and RT/MT values recorded
    in the supplementary table
  repo: github.com/AdrianHaun/AriumMS
- name: manual expert review
  role: Validates that normalized checkmark/cross symbols and the 36-vs-40 row reconciliation
    match the reported coverage counts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:cac507ca501f670afe5544480a8030ce02e5e1ca7d96c4c3eedc1a2f6286487d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - manual expert review
  - python-docx
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# supplementary-document-table-parsing

## Summary

Extracts the multi-platform detection matrix for the 36-metabolite standard (Table 1) from the paper's supplementary DOCX file and converts it into a structured CSV with per-method coverage summaries, making the HILIC-MS/CE-MS coverage claims (30/36, 32/36, 36/36) machine-checkable.

## When to use

Use when a paper's quantitative claim (e.g. how many of 36 standard metabolites are covered by HILIC ESI+, HILIC ESI−, CE alkaline/acidic anion and CE cation methods) exists only as a table inside an electronic supplementary file that must be transcribed to structured data.

## When NOT to use

- The detection/coverage data are already available as a machine-readable table (CSV/Excel) — transcribing from DOCX would only add transcription errors
- The claim depends on raw MS feature data (mzXML/mzML peak detection) rather than a static published table — process with AriumMS ROI/augmentation instead
- The supplementary table is embedded as an image rather than a DOCX table object — python-docx cell extraction cannot recover it

## Inputs

- Supplementary file1 (DOCX, ~1.51 MB) containing Table 1 with 40 standard analyte rows
- Per-row detection flags for five platforms: HILIC ESI+, HILIC ESI−, CE alkaline anion, CE acidic anion, CE cation
- RT (LC-MS) or MT (CE-MS) values with standard deviations and AriumMS detection checkmarks

## Outputs

- table1_standard_analytes.csv with 40 transcribed analyte rows (numeric RT/MT ± sd, boolean detection flags)
- 3 appended summary coverage rows: 30/36 (HILIC combined), 32/36 (CE combined), 36/36 (all five methods combined)

## How to apply

Open the electronic supplementary file (Supplementary file1, DOCX, ~1.51 MB) and locate Table 1, which lists the metabolite standards analyzed across the five separation methods. Parse the DOCX table with python-docx and transcribe each of the 40 standard analyte rows, capturing the detection flag per platform (HILIC ESI+, HILIC ESI−, CE alkaline anion, CE acidic anion, CE cation) plus the RT (LC-MS) or MT (CE-MS) value with its standard deviation and the AriumMS detection checkmark. Cast the RT/MT ± sd values to numeric columns and normalize checkmark/cross symbols to boolean detection flags. Compute coverage counts per method combination (HILIC combined 30/36, CE combined 32/36, all five methods 36/36) and append them as summary rows before writing table1_standard_analytes.csv. Note the reconciliation decision point: the table holds 40 analyte rows but the coverage denominators count 36 metabolites, so rows must be grouped to the 36-metabolite standard before summing flags.

## Related tools

- **python-docx** (Parses the supplementary DOCX Table 1 cell-by-cell to transcribe the 40 analyte rows and their five-platform detection flags)
- **AriumMS** (Source of the per-metabolite detection checkmarks and RT/MT values recorded in the supplementary table) — github.com/AdrianHaun/AriumMS
- **manual expert review** (Validates that normalized checkmark/cross symbols and the 36-vs-40 row reconciliation match the reported coverage counts)

## Examples

```
import docx; rows = [[c.text for c in r.cells] for r in docx.Document('Supplementary file1.docx').tables[0].rows]
```

## Evaluation signals

- Output file table1_standard_analytes.csv exists and contains exactly 40 analyte rows plus 3 summary coverage rows
- Recomputed coverage counts match the reported values: 30/36 (HILIC combined), 32/36 (CE combined), 36/36 (all five methods)
- All RT/MT ± sd strings parse to numeric columns with no coercion errors or NaNs in detection-flag columns
- Every checkmark/cross symbol maps to a boolean detection flag with zero unmapped symbols
- The 40 analyte rows reconcile to the 36 standard metabolites used as coverage denominators

## Limitations

- The article section provides no direct access link/URL for the supplementary material beyond the reference 'Supplementary file1 (DOCX 1.51 MB)', so the file must be obtained from the publisher page
- Checkmark/cross glyphs in DOCX are encoding-sensitive; manual symbol normalization to booleans is required and error-prone
- The 40-row table vs 36-metabolite coverage denominator requires manual reconciliation that could silently miscount if rows are duplicated or grouped incorrectly

## Evidence

- [other] Supplementary file referenced in article: "Below is the link to the electronic supplementary material.Supplementary file1 (DOCX 1.51 MB)"
- [intro] Multi-platform standard measurements underlie the table: "the metabolic standard mixtures and the yeast extracts were measured within a multi-platform approach combining HILIC-MS (ESI positive/ESI negative) with three CE-MS methods"
- [readme] File conversion context from AriumMS README: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
