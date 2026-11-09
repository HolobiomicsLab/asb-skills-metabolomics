---
name: retention-migration-time-transcription
description: Use when a supplementary DOCX table reports per-method RT/MT ± sd and
  checkmark/cross detection flags for standard analytes and these must be converted
  into a machine-readable coverage matrix for verifying multi-platform metabolite
  coverage claims.
when_to_use_negative:
- The goal is feature detection or augmentation from raw mzML/mzXML MS data — use
  the AriumMS ROI/augmentation pipeline instead, which this table merely summarizes
- The analyte data are already available as a structured feature table or CSV — re-transcribing
  the DOCX adds no information
- Only single-platform (e.g. HILIC-MS alone) coverage is of interest — the value of
  this skill lies in cross-method (CE-MS vs HILIC-MS) coverage comparison
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: python-docx
  role: parses the supplementary DOCX table to transcribe analyte rows, RT/MT ± sd
    values, and checkmark flags
- name: AriumMS
  role: produced the per-platform detection checkmarks (AriumMS detection) that the
    transcription normalizes into boolean flags
  repo: github.com/AdrianHaun/AriumMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:74190171f1a249804a674a8c49d36cebf580510c76b1b4dc8fd8d399321bfd83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - python-docx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# retention-migration-time-transcription

## Summary

Transcribes the retention time (RT, LC-MS) and migration time (MT, CE-MS) values with standard deviations plus per-platform detection flags from the standard-analyte table (Table 1) of the AriumMS electronic supplementary file into a structured CSV, and recomputes multi-platform coverage counts (HILIC 30/36, CE 32/36, all five methods 36/36).

## When to use

Use when a supplementary DOCX table reports per-method RT/MT ± sd and checkmark/cross detection flags for standard analytes and these must be converted into a machine-readable coverage matrix for verifying multi-platform metabolite coverage claims.

## When NOT to use

- The goal is feature detection or augmentation from raw mzML/mzXML MS data — use the AriumMS ROI/augmentation pipeline instead, which this table merely summarizes
- The analyte data are already available as a structured feature table or CSV — re-transcribing the DOCX adds no information
- Only single-platform (e.g. HILIC-MS alone) coverage is of interest — the value of this skill lies in cross-method (CE-MS vs HILIC-MS) coverage comparison

## Inputs

- Electronic supplementary file1 (DOCX, ~1.51 MB) containing Table 1 of standard analytes
- Table 1 columns: analyte names, RT (LC-MS) or MT (CE-MS) values with standard deviations, AriumMS detection checkmarks for HILIC ESI+, HILIC ESI−, CE alkaline anion, CE acidic anion, CE cation

## Outputs

- table1_standard_analytes.csv with 40 analyte rows plus 3 summary coverage rows (HILIC combined 30/36, CE combined 32/36, all five methods 36/36)
- Boolean detection-flag matrix (5 platforms × 40 analytes) and numeric RT/MT ± sd columns

## How to apply

Open the electronic supplementary file (Supplementary file1, DOCX, ~1.51 MB) and locate Table 1, which lists the metabolite standards measured across the five separation methods (HILIC ESI+, HILIC ESI−, CE alkaline anion, CE acidic anion, CE cation). Parse the DOCX table with python-docx, transcribing each of the 40 standard analyte rows while capturing the RT (LC-MS) or MT (CE-MS) value with its standard deviation and the AriumMS detection checkmark for each platform. Cast RT/MT ± sd strings to numeric columns and normalize checkmark/cross symbols to boolean detection flags — RT and MT come from different separation dimensions, so keep them in clearly separated columns. Compute coverage summary counts per method combination (HILIC combined 30/36, CE combined 32/36, all five methods combined 36/36) and append them as summary rows to the 40 analyte rows. Write the result to table1_standard_analytes.csv and validate against the reported counts.

## Related tools

- **python-docx** (parses the supplementary DOCX table to transcribe analyte rows, RT/MT ± sd values, and checkmark flags)
- **AriumMS** (produced the per-platform detection checkmarks (AriumMS detection) that the transcription normalizes into boolean flags) — github.com/AdrianHaun/AriumMS

## Examples

```
python -c "import docx; t = docx.Document('Supplementary file1.docx').tables[0]; [print([c.text for c in r.cells]) for r in t.rows]"
```

## Evaluation signals

- Output CSV contains exactly 40 analyte rows plus 3 summary coverage rows
- Recomputed coverage counts match the reported values: 30/36 (HILIC combined), 32/36 (CE combined), 36/36 (all five methods)
- All RT/MT ± sd cells parse to numeric columns without residual non-numeric characters
- Checkmark/cross symbols map 1:1 to boolean flags with no missing values across the 5 platform columns
- Coverage is computed over 36 unique metabolites although 40 analyte rows exist, so row-to-metabolite mapping is checked before counting

## Limitations

- The supplementary material is referenced without a direct access URL in the article section, so the DOCX must be obtained from the publisher page (DOI 10.1007/s00216-023-04715-6)
- The 40 analyte rows exceed the 36 metabolites used for coverage counting; some metabolites appear in multiple rows/forms and must be consolidated before computing coverage
- RT and MT values are platform-specific (LC-MS vs CE-MS) and not directly comparable; they must be kept in separate columns rather than merged into one time axis

## Evidence

- [supplementary] Combining the three CE-MS methods detects 32 of 36 standard metabolites, and applying all five analytical methods (HILIC-MS plus CE-MS) makes all metabolites of the standard detectable.: [evidence span withheld — anchor status: not_found]
- [intro] The metabolic standard mixtures and yeast extracts were measured across five separation methods: "the metabolic standard mixtures and the yeast extracts were measured within a multi-platform approach combining HILIC-MS (ESI positive/ESI negative) with three CE-MS methods"
- [discussion] Supplementary file1 (DOCX 1.51 MB) is the source document for Table 1: "Below is the link to the electronic supplementary material.Supplementary file1 (DOCX 1.51 MB)"
- [readme] AriumMS ROI search and augmentation parameters underlie the detection flags transcribed from Table 1: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation."
