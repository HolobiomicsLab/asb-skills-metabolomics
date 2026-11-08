---
name: lipid-annotation-counting
description: Use when reproducing published lipidomics annotation counts from a LipidMatch/MS-DIAL/Greazy
  annotation workbook (e.g. Additional file 3) or validating counts of unique lipid
  ions, features, and lipid types per polarity.
when_to_use_negative:
- Input is already a validated count table rather than raw annotation rows
- Annotations come from a tool other than LipidMatch (e.g. only MS-DIAL or Greazy)
  and the counting convention differs
- The workbook is a fragmentation library (e.g. LipidMatch in silico libraries) rather
  than feature-level annotation output
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: pandas
  role: Load and parse the annotation worksheet and compute counts of unique lipid
    ions, features, and lipid types
- name: openpyxl
  role: Excel engine for reading the .xlsx annotation workbook worksheets
- name: LipidMatch
  role: Source of the per-feature ranked lipid ion annotations being counted
  repo: http://secim.ufl.edu/secim-tools/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:3dca704c8f19f305988343f6cff3d26e2e47b7cf4df71f59e385c526ecacc8fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidMatch
  - R
  - openpyxl
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# lipid-annotation-counting

## Summary

Recount and cross-check lipid annotation counts (unique lipid ions, distinct features, and lipid types) from LipidMatch output spreadsheets in a given ionization mode, using pandas/openpyxl over the paper's Additional file 3 worksheets.

## When to use

Use when reproducing published lipidomics annotation counts from a LipidMatch/MS-DIAL/Greazy annotation workbook (e.g. Additional file 3) or validating counts of unique lipid ions, features, and lipid types per polarity.

## When NOT to use

- Input is already a validated count table rather than raw annotation rows
- Annotations come from a tool other than LipidMatch (e.g. only MS-DIAL or Greazy) and the counting convention differs
- The workbook is a fragmentation library (e.g. LipidMatch in silico libraries) rather than feature-level annotation output

## Inputs

- Additional file 3: Table S4 annotation workbook (positive- and negative-polarity worksheets containing LipidMatch, MS-DIAL, and Greazy annotations)
- Feature table with m/z and retention time columns and ranked LipidMatch lipid ion entries

## Outputs

- annotation_counts_positive.csv with one row per metric: unique lipid ion count, feature count, lipid type count

## How to apply

Load the additional-file annotation workbook (e.g. Additional file 3: Table S4) with pandas and openpyxl, selecting the polarity-specific worksheet such as ec-positive-mode. Restrict to the LipidMatch annotation columns, which contain per-feature ranked lipid ion entries, lipid class/type labels, and adduct designations. Count unique lipid ions by tallying every annotated lipid ion row, treating multiple adducts of a given lipid species as distinct lipid ions per the paper's counting convention, and separately count distinct features (unique m/z and retention time entries) with at least one LipidMatch annotation. Also count distinct lipid types covered by the positive- (or negative-) mode annotations, e.g. 5159 unique lipid ions across 1401 features and 26 lipid types in the positive-polarity IE-ddMS2-top5 Red Cross plasma case. Accept the recount only when all three metrics match the reported figures; write one row per metric to annotation_counts_positive.csv.

## Related tools

- **pandas** (Load and parse the annotation worksheet and compute counts of unique lipid ions, features, and lipid types)
- **openpyxl** (Excel engine for reading the .xlsx annotation workbook worksheets)
- **LipidMatch** (Source of the per-feature ranked lipid ion annotations being counted) — http://secim.ufl.edu/secim-tools/

## Examples

```
python -c "import pandas as pd; df = pd.read_excel('Additional_file_3.xlsx', sheet_name='ec-positive-mode'); print(df[['lipid_ion','feature','lipid_type']].drop_duplicates().agg(['nunique']))"
```

## Evaluation signals

- Recount of unique lipid ions equals 5159 for the positive-polarity Red Cross plasma case
- Recount of annotated features equals 1401
- Recount of distinct lipid types equals 26
- Output CSV contains one row per reported metric with matching column labels
- Counts are consistent across LipidMatch, MS-DIAL, and Greazy annotation columns where all three are present

## Limitations

- Counting convention treats multiple adducts of the same lipid species as distinct lipid ions; a recount must adopt this convention or counts will not match
- The workbook is only provided as an additional file, not archived in a public repository, so the recount depends on access to the paper's supplementary data
- Counts are polarity-specific (positive vs negative worksheets) and must not be pooled across modes

## Evidence

- [methods] positive-mode count metric: "Once the user runs LipidMatch and enters user parameters, LipidMatch will automatically append identifications to each feature table using MS/MS files contained in that feature table’s subfolder."
- [abstract] feature ranking / counting convention: "Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities."
- [readme] annotation columns appended to feature tables: "combine results from other lipidomics software"
- [discussion] comparison scope (LipidMatch vs MS-DIAL vs Greazy): "Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma"
