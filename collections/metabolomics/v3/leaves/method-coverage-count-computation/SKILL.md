---
name: method-coverage-count-computation
description: Use when a multi-platform metabolomics study reports a standards table
  with per-method detection flags (checkmark/cross plus RT/MT ± sd) and you must quantify
  how many metabolites each method or method combination covers.
when_to_use_negative:
- Input is already a processed feature table or abundance matrix rather than a per-method
  detection/standards table — coverage counting from flags does not apply
- Single-platform datasets where no combination coverage (HILIC vs CE vs all five)
  can be computed
- Yeast sample feature augmentation or chemometric evaluation tasks — use the AriumMS
  ROI/augmentation pipeline instead of coverage counting
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Source of the per-analyte detection checkmarks in the standards table that
    are normalized to boolean detection flags for coverage counting
  repo: https://github.com/AdrianHaun/AriumMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:713149413326a989a566549ab907f74afb8e7e0c53842ead76fa464f3aa9a78f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AdrianHaun/AriumMS
  tools:
  - AriumMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# method-coverage-count-computation

## Summary

Computes per-method and per-method-combination detection coverage of a metabolite standard mixture across multi-platform LC-MS/CE-MS methods (e.g., HILIC-MS ESI+/ESI− plus three CE-MS methods), yielding counts such as 30/36 (HILIC combined), 32/36 (CE combined), and 36/36 (all five methods).

## When to use

Use when a multi-platform metabolomics study reports a standards table with per-method detection flags (checkmark/cross plus RT/MT ± sd) and you must quantify how many metabolites each method or method combination covers.

## When NOT to use

- Input is already a processed feature table or abundance matrix rather than a per-method detection/standards table — coverage counting from flags does not apply
- Single-platform datasets where no combination coverage (HILIC vs CE vs all five) can be computed
- Yeast sample feature augmentation or chemometric evaluation tasks — use the AriumMS ROI/augmentation pipeline instead of coverage counting

## Inputs

- Electronic supplementary file (Supplementary file1, DOCX format, ~1.51 MB) containing Table 1 of metabolite standard analytes
- Table 1 rows with detection flags per platform (HILIC ESI+, HILIC ESI−, CE alkaline anion, CE acidic anion, CE cation) and RT/MT ± sd values
- AriumMS detection checkmark column for each analyte

## Outputs

- table1_standard_analytes.csv with 40 transcribed analyte rows and 3 appended summary coverage rows
- Boolean detection matrix (checkmark/cross normalized) per analyte per platform
- Coverage summary counts: 30/36 (HILIC combined), 32/36 (CE combined), 36/36 (all five methods combined)

## How to apply

Open the electronic supplementary file (Supplementary file1, DOCX, ~1.51 MB) and locate Table 1 listing the metabolite standards analyzed across the five separation methods. Parse the DOCX table with python-docx, transcribing each of the 40 standard analyte rows with the detection flag for each of the five platforms (HILIC ESI+, HILIC ESI−, CE alkaline anion, CE acidic anion, CE cation) plus the RT (LC-MS) or MT (CE-MS) value with its standard deviation and the AriumMS detection checkmark. Cast the RT/MT ± sd columns to numeric and normalize checkmark/cross symbols to boolean detection flags. Compute coverage summary counts per method combination — HILIC combined 30/36, CE combined 32/36, all five methods combined 36/36 — and append them as summary rows to table1_standard_analytes.csv. Validate that the file contains 40 analyte rows plus 3 summary coverage rows and that recomputed counts match the reported values.

## Related tools

- **AriumMS** (Source of the per-analyte detection checkmarks in the standards table that are normalized to boolean detection flags for coverage counting) — https://github.com/AdrianHaun/AriumMS

## Examples

```
import docx; rows = docx.Document('Supplementary file1.docx').tables[0].rows
```

## Evaluation signals

- Output file table1_standard_analytes.csv exists and contains exactly 40 analyte rows plus 3 summary coverage rows
- Recomputed coverage counts match the reported values: 30/36 (HILIC combined), 32/36 (CE combined), 36/36 (all five methods)
- Checkmark/cross symbols are consistently normalized to boolean detection flags with no residual symbols in the boolean columns
- RT (LC-MS) and MT (CE-MS) ± sd values parse to numeric columns without coercion errors (40 rows, 5 platform columns)
- All-five-methods coverage equals 36/36, confirming that the union of HILIC-MS and CE-MS flags spans the full 36-metabolite standard

## Limitations

- The table contains 40 standard analyte rows, but coverage counts are reported over 36 metabolites — the analyte-row-to-metabolite mapping must be resolved before recomputing counts
- RT columns (LC-MS) and MT columns (CE-MS) have different semantics and units; they must not be merged without keeping platform labels distinct
- Detection flags depend on the AriumMS checkmark annotation in the supplementary table; no MS/MS spectral networking is implemented in AriumMS, so cross-platform feature linkage is not automated
- No direct access link/URL is given for the supplementary DOCX in the article text; it must be retrieved from the publisher page

## Evidence

- [intro] multi-platform standard measurement scope: "Therefore, the metabolic standard mixtures and the yeast extracts were measured within a multi-platform approach combining HILIC-MS (ESI positive/ESI negative) with three CE-MS methods"
- [discussion] supplementary file format: "Below is the link to the electronic supplementary material.Supplementary file1 (DOCX 1.51 MB)"
- [abstract] AriumMS as multi-platform toolbox: "we introduce AriumMS (augmented region of interest for untargeted metabolomics mass spectrometry) as a reliable toolbox for multi-platform metabolomics"
- [readme] AriumMS tool scope: "All in one tool for untargeted Metabolomics by ROI and augmentation of multiple Data sets."
