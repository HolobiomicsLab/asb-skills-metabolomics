---
name: cross-software-annotation-table-reconciliation
description: Use when you have lipid annotations from two or more open-source lipidomics
  software aligned to the same features and need to quantify cross-software agreement
  or benchmark one annotator against the others.
when_to_use_negative:
- You have annotations from only a single software and no second annotation source
  to reconcile against
- You need rule-based fragment-level lipid identification itself rather than post-hoc
  comparison of existing annotations (run LipidMatch's MS/MS filter pipeline instead)
- Features lack retention time or m/z alignment windows required for cross-software
  feature matching (e.g., direct infusion data without RT)
edam_operation: http://edamontology.org/operation_0226
tools:
- name: LipidMatch
  role: Primary annotator whose identifications are assessed for corroboration by
    other software
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MS-DIAL
  role: Comparison annotator; its identifications are merged onto the feature table
    and used to corroborate LipidMatch assignments
- name: Greazy
  role: Comparison annotator; its identifications are merged onto the feature table
    and used to corroborate LipidMatch assignments
- name: R / CombineSoftwareIDs.R
  role: Appends identifications from LipidMatch, MS-DIAL, and Greazy to the MZmine
    feature table using 10 ppm m/z and 0.2 min RT alignment windows
  repo: http://secim.ufl.edu/secim-tools/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:a5493d0a13d357ba6a26c292f44d2f0e62d91ad523fb387c6dceafbe4887b0b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - CombineSoftwareIDs.R
  - Greazy
  - LipidMatch
  - MS-DIAL
  - R
  - R / CombineSoftwareIDs.R
  edam_operation: http://edamontology.org/operation_0226
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-software-annotation-table-reconciliation

## Summary

Reconciles lipid identifications produced by multiple annotation engines (LipidMatch, MS-DIAL, Greazy) by appending all identifications onto a shared MZmine feature table, then measures how many of one tool's fatty acyl assignments are corroborated by at least one other tool per ion polarity.

## When to use

Use when you have lipid annotations from two or more open-source lipidomics software aligned to the same features and need to quantify cross-software agreement or benchmark one annotator against the others.

## When NOT to use

- You have annotations from only a single software and no second annotation source to reconcile against
- You need rule-based fragment-level lipid identification itself rather than post-hoc comparison of existing annotations (run LipidMatch's MS/MS filter pipeline instead)
- Features lack retention time or m/z alignment windows required for cross-software feature matching (e.g., direct infusion data without RT)

## Inputs

- Aligned MZmine feature table with appended identifications from LipidMatch, MS-DIAL, and Greazy (e.g., Additional file 3: Table S4 workbook produced by CombineSoftwareIDs.R)
- Per-software annotation tables keyed by m/z and retention time
- Alignment parameters: 10 ppm m/z tolerance, 0.2 min retention time tolerance

## Outputs

- Per-polarity corroboration percentages for fatty acyl constituent identifications (expected: 98% negative mode, 92% positive mode)
- Total annotated feature counts per software per polarity (LipidMatch: 159 negative / 1401 positive; MS-DIAL: 143 negative / 411 positive; Greazy: 94 negative / 180 positive)
- Summary CSV of cross-software corroboration results

## How to apply

First append identifications from each software onto the aligned feature table (e.g., with CombineSoftwareIDs.R, using alignment windows of 10 ppm m/z and 0.2 min retention time). Filter separately by polarity and retain only features carrying identifications from all compared software (in the Red Cross plasma study: 65 negative-mode and 68 positive-mode features). For each LipidMatch annotation, check whether the fatty acyl constituent composition matches an identification from MS-DIAL or Greazy, treating plasmenyl/plasmanyl pairs differing by one degree of unsaturation as the same identification. Compute the corroboration percentage per polarity (reported: 98% negative, 92% positive) and count total annotated features per software per polarity to confirm coverage differences.

## Related tools

- **LipidMatch** (Primary annotator whose identifications are assessed for corroboration by other software) — https://github.com/GarrettLab-UF/LipidMatch
- **MS-DIAL** (Comparison annotator; its identifications are merged onto the feature table and used to corroborate LipidMatch assignments)
- **Greazy** (Comparison annotator; its identifications are merged onto the feature table and used to corroborate LipidMatch assignments)
- **R / CombineSoftwareIDs.R** (Appends identifications from LipidMatch, MS-DIAL, and Greazy to the MZmine feature table using 10 ppm m/z and 0.2 min RT alignment windows) — http://secim.ufl.edu/secim-tools/

## Examples

```
Rscript CombineSoftwareIDs.R --mz-tol 10ppm --rt-tol 0.2min
```

## Evaluation signals

- Per-polarity corroboration percentages fall in the reported ranges: 98% (negative ion mode) and 92% (positive ion mode)
- Per-software annotated feature counts show LipidMatch exceeding both MS-DIAL and Greazy in each polarity
- The comparison subsets contain the expected numbers of features identified by all three software (65 negative-mode, 68 positive-mode in the Red Cross plasma study)
- Plasmenyl/plasmanyl pairs differing by one degree of unsaturation are consistently collapsed as the same identification
- Output summary CSV contains one corroboration percentage per polarity and one count per software per polarity

## Limitations

- Corroboration is assessed only on features identified by all compared software, so unique annotations (a large share of LipidMatch's identifications) are excluded from the agreement metric
- Agreement is at the fatty acyl constituent level; structural resolution differences between software can inflate or deflate apparent concordance
- The original study used Red Cross plasma data from UHPLC-HRMS/MS (Q-Exactive orbitrap); agreement benchmarks may not transfer to other sample types or instruments

## Evidence

- [abstract] cross-software corroboration benchmark: "92% of LipidMatch identifications by fatty acyl constituents were corroborated by at least one other software in positive mode and 98% in negative ion mode."
- [methods] appending identifications from other software: "Once lipid identifications are obtained using LipidMatch, identifications from any other software such as Greazy [17], LipidSearch (Thermo Scientific, San Jose, CA), and MS-DIAL can be appended"
- [readme] modular integration in existing workflows: "LipidMatch is modular, allowing it to fit in various workflows you may have in your lab. For example LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and"
- [discussion] LipidMatch yields most identifications: "Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma"
