---
name: spreadsheet-worksheet-subsetting
description: Use when a paper's supplementary data ships as a multi-worksheet Excel
  file (e.g. Additional file 3, Table S4) and a published figure (e.g. 210 lipid ions,
  159 features, 15 lipid types) must be recounted from a single software's annotation
  columns.
when_to_use_negative:
- The input is already a software-specific feature table containing only LipidMatch
  results — no worksheet selection or column filtering is needed
- The goal is cross-software corroboration rates (92%/98% agreement) rather than per-tool
  recounting, since that requires retaining MS-DIAL and Greazy columns
- The data are raw vendor files or .ms2 spectra rather than a finished annotation
  spreadsheet
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: pandas
  role: Loads the negative-polarity worksheet of Additional file 3 Table S4 and filters
    rows/columns to LipidMatch annotations for recounting
- name: LipidMatch
  role: Source of the annotation columns being subset and recounted from the shared
    feature table
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MS-DIAL
  role: Other software's annotation columns in Table S4 that must be excluded during
    subsetting
- name: Greazy
  role: Other software's annotation columns in Table S4 that must be excluded during
    subsetting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:94255f22674670458977b2c03a6e371bbcb9fbcfb96c16dd741d41c24a045c48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - Greazy
  - LipidMatch
  - MS-DIAL
  - R
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spreadsheet-worksheet-subsetting

## Summary

Selects the correct worksheet from a multi-sheet Excel supplementary workbook and subsets rows/columns to one software's annotations (here, LipidMatch columns in Additional file 3 Table S4) so counts can be independently recomputed with pandas.

## When to use

Use when a paper's supplementary data ships as a multi-worksheet Excel file (e.g. Additional file 3, Table S4) and a published figure (e.g. 210 lipid ions, 159 features, 15 lipid types) must be recounted from a single software's annotation columns.

## When NOT to use

- The input is already a software-specific feature table containing only LipidMatch results — no worksheet selection or column filtering is needed
- The goal is cross-software corroboration rates (92%/98% agreement) rather than per-tool recounting, since that requires retaining MS-DIAL and Greazy columns
- The data are raw vendor files or .ms2 spectra rather than a finished annotation spreadsheet

## Inputs

- Multi-worksheet Excel supplementary table (Additional file 3, Table S4) with per-feature annotations from LipidMatch, MS-DIAL, and Greazy
- Published reference values to verify (210 lipid ions, 159 features, 15 lipid types in negative mode)

## Outputs

- pandas DataFrame subset containing only LipidMatch negative-mode annotations
- Counts of distinct lipid ions, distinct features (m/z + RT pairs), and distinct lipid types
- verification_summary.csv with recounted vs. published values and pass/fail flags

## How to apply

Open the workbook and select the polarity-specific worksheet — for this verification, the negative-polarity sheet of Additional file 3 Table S4, which 'includes all features identified in Red Cross plasma, with LipidMatch, MS-DIAL, and Greazy annotations.' Load the sheet with pandas, then restrict the frame to the LipidMatch annotation columns and drop rows lacking a LipidMatch lipid identification so MS-DIAL and Greazy entries do not inflate counts. From the subset, compute three distinct tallies: unique lipid ion annotations (lipid identification entries), unique features defined by m/z + retention time pairs with at least one LipidMatch annotation, and unique lipid types parsed from the class prefix of each annotation (e.g. CE, Cer, LPC, LPE, PA, PE, PG, PI, PS, SM, TAG). Finally, write recounted-versus-published comparisons with pass/fail flags to verification_summary.csv. The rationale is that mixed-software supplementary tables must be filtered to one tool's columns before any per-tool count is meaningful.

## Related tools

- **pandas** (Loads the negative-polarity worksheet of Additional file 3 Table S4 and filters rows/columns to LipidMatch annotations for recounting)
- **LipidMatch** (Source of the annotation columns being subset and recounted from the shared feature table) — https://github.com/GarrettLab-UF/LipidMatch
- **MS-DIAL** (Other software's annotation columns in Table S4 that must be excluded during subsetting)
- **Greazy** (Other software's annotation columns in Table S4 that must be excluded during subsetting)

## Examples

```
import pandas as pd; df = pd.read_excel('Additional_file3_TableS4.xlsx', sheet_name='negative_polarity'); lm = df.dropna(subset=['LipidMatch_annotation']); print(lm['m/z_RT'].nunique(), lm['lipid_ion'].nunique(), lm['lipid_type'].nunique())
```

## Evaluation signals

- Recounted lipid ions match the published 210 in negative mode (pass flag in verification_summary.csv)
- Recounted features with at least one LipidMatch annotation equal 159
- Parsed lipid-type count equals 15, with class prefixes drawn from the expected set (e.g. CE, Cer, LPC, LPE, PA, PE, PG, PI, PS, SM, TAG)
- MS-DIAL and Greazy rows/columns are verifiably absent from the subset (no non-LipidMatch annotations leak into counts)
- Each unique feature is keyed by a unique m/z + retention time pair, avoiding duplicate counting of co-annotated features

## Limitations

- Only the LipidMatch annotation columns can be trusted for LipidMatch counts; retaining other software's columns double-counts features identified by multiple tools
- The workflow depends on the worksheet being present as an additional file rather than an archived repository artifact
- Lipid-type parsing relies on consistent class-prefix formatting in the annotation strings

## Evidence

- [supplementary] Table S4 scope: "Additional file 3: Table S4 includes all features identified in Red Cross plasma, with LipidMatch, MS-DIAL, and Greazy annotations."
- [discussion] Most identifications from LipidMatch: "Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma"
- [abstract] Agreement metric grounding: "92% of LipidMatch identifications by fatty acyl constituents were corroborated by at least one other software in positive mode and 98% in negative ion mode."
- [intro] Feature definition: "features (peaks defined by a mass to charge ratio (m/z) and retention time) often contain multiple co-eluting molecules with similar m/z values"
- [readme] README library scope: "LipidMatch identifications are obtained by matching experimental fragment m/z values with simulated library m/z values using in-silico fragmentation libraries"
