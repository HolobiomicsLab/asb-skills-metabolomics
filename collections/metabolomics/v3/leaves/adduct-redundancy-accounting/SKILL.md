---
name: adduct-redundancy-accounting
description: Use when counting unique lipid ions, features, or lipid types from annotation
  workbooks produced by LipidMatch or comparable lipid identification software, where
  multiple adducts of the same lipid species appear across features.
when_to_use_negative:
- Input is a merged/de-duplicated lipid species table that has already collapsed adducts
  into a single molecular species entry
- You need structural-level identification counts (e.g. fatty-acyl constituent level)
  rather than ion-level counts, since the convention deliberately preserves adduct-level
  redundancy
- Feature table has not yet been annotated by LipidMatch (i.e. no adduct designations
  present)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: LipidMatch
  role: Produces the per-feature ranked lipid ion annotations with adduct designations
    that are tallied; written in R and available at secim.ufl.edu/secim-tools/
  repo: http://secim.ufl.edu/secim-tools/
- name: MS-DIAL
  role: Alternative peak-picking and identification software whose annotations can
    be compared to LipidMatch using the same adduct-redundant counting convention
- name: Greazy
  role: Alternative in-silico lipid identification software used for cross-tool comparison
    of annotation counts
- name: MSConvert
  role: Converts raw MS/MS peak-picked data to .ms2 file format required by LipidMatch
    upstream of annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:bce1382e19cb19756fd722daefed337382395e316f8c9fe52d7a3b456471cc11
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Greazy
  - LipidMatch
  - MS-DIAL
  - MSConvert
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9361702127659575
n_claims: 47
---

# Adduct redundancy accounting

## Summary

Treat every annotated adduct of a lipid species (e.g. [M+H]+, [M+NH4]+, [M-H]-) as a distinct lipid ion when tallying unique annotations from untargeted UHPLC-HRMS/MS feature tables, while separately counting distinct features (m/z + retention time) and distinct lipid types. This prevents inflation or under-counting of identifications when comparing software outputs like LipidMatch, MS-DIAL, and Greazy.

## When to use

Use when counting unique lipid ions, features, or lipid types from annotation workbooks produced by LipidMatch or comparable lipid identification software, where multiple adducts of the same lipid species appear across features.

## When NOT to use

- Input is a merged/de-duplicated lipid species table that has already collapsed adducts into a single molecular species entry
- You need structural-level identification counts (e.g. fatty-acyl constituent level) rather than ion-level counts, since the convention deliberately preserves adduct-level redundancy
- Feature table has not yet been annotated by LipidMatch (i.e. no adduct designations present)

## Inputs

- Annotation worksheet (XLSX format, e.g. Additional file 3: Table S4) with per-feature ranked lipid ion entries, lipid class/type labels, and adduct designations
- Feature table from peak-picking software (MZmine, XCMS, MS-DIAL, or Compound Discoverer) with unique m/z and retention time entries
- LipidMatch output annotations appended to feature tables

## Outputs

- annotation_counts_positive.csv with one row per reported metric (unique lipid ions, features, lipid types)
- Corresponding negative-polarity counts (unique lipid ions, features, lipid types)
- Counts suitable for software comparison (LipidMatch vs MS-DIAL vs Greazy)

## How to apply

Load the annotation worksheet (e.g. the 'ec-positive-mode' sheet of Additional file 3: Table S4) using pandas with openpyxl. Filter to the software's annotation columns — for LipidMatch, retain the per-feature ranked lipid ion entries, lipid class/type labels, and adduct designations. Count unique lipid ions by tallying every annotated lipid ion row, treating multiple adducts of a given lipid species as distinct lipid ions per the paper's counting convention. Separately count distinct features (unique m/z and retention time entries) carrying at least one annotation, and count distinct lipid types. Validate by comparing the recounted values against the reported figures (e.g. 5159 unique lipid ions, 1401 features, 26 lipid types in positive ion mode for Red Cross plasma).

## Related tools

- **LipidMatch** (Produces the per-feature ranked lipid ion annotations with adduct designations that are tallied; written in R and available at secim.ufl.edu/secim-tools/) — http://secim.ufl.edu/secim-tools/
- **MS-DIAL** (Alternative peak-picking and identification software whose annotations can be compared to LipidMatch using the same adduct-redundant counting convention)
- **Greazy** (Alternative in-silico lipid identification software used for cross-tool comparison of annotation counts)
- **MSConvert** (Converts raw MS/MS peak-picked data to .ms2 file format required by LipidMatch upstream of annotation)

## Examples

```
import pandas as pd; df = pd.read_excel('Additional_file_3_Table_S4.xlsx', sheet_name='ec-positive-mode'); print(df.shape)
```

## Evaluation signals

- Recount of unique lipid ions matches the reported figure exactly (e.g. 5159 in positive mode for Red Cross plasma)
- Feature count (distinct m/z + RT entries with ≥1 annotation) matches the reported figure (e.g. 1401 in positive mode)
- Lipid type count matches the reported figure (e.g. 26 lipid types in positive mode)
- Same convention applied symmetrically to negative-mode annotations so cross-software comparison is not skewed by differing adduct handling
- Adduct strings are parsed consistently (e.g. [M+H]+, [M+NH4]+, [M-H]-) and not collapsed or split incorrectly during counting

## Limitations

- LipidMatch was tested only on Windows 7 through 10; no cross-platform support is stated, and it is pinned to R version 3.3.3
- The annotation workbook is distributed as an Additional file rather than archived in a public repository, so reproducibility depends on access to the supplement
- Users must develop their own fragmentation libraries in Excel for full customization; the counting convention inherits whatever adduct assignments LipidMatch makes
- The workflow assumes feature tables already contain distinct m/z + retention time entries; it does not itself perform peak de-redundancy across features

## Evidence

- [results] Positive-mode counts: 5159 unique lipid ions, 1401 features, 26 lipid types: [evidence span withheld — anchor status: not_found]
- [results] Counting convention: each adduct is a distinct lipid ion: [evidence span withheld — anchor status: not_found]
- [intro] Feature defined by m/z and retention time: "features (peaks defined by a mass to charge ratio (m/z) and retention time) often contain multiple co-eluting molecules with similar m/z values"
- [methods] LipidMatch libraries cover 56 lipid types vs MS-DIAL 34, Greazy 24: "With 56 lipid types, LipidMatch in silico libraries cover the greatest number of lipid types of any open source software to date, with MS-DIAL containing 34 lipid types, and Greazy containing 24"
- [abstract] Unique fragmentation libraries include previously uncharacterized adducts such as ammoniated cardiolipins: "Unique fragmentation libraries, compared to other open source software, include oxidized lipids, bile acids, sphingosines, and previously uncharacterized adducts, including ammoniated cardiolipins."
- [methods] Feature tables created from raw vendor files by peak-picking software: [evidence span withheld — anchor status: not_found]
- [methods] LipidMatch appends identifications to each feature table using MS/MS files: "Once the user runs LipidMatch and enters user parameters, LipidMatch will automatically append identifications to each feature table using MS/MS files contained in that feature table’s subfolder."
- [methods] Identifications from other software can be appended in Excel: "Once lipid identifications are obtained using LipidMatch, identifications from any other software such as Greazy [17], LipidSearch (Thermo Scientific, San Jose, CA), and MS-DIAL can be appended in"
- [readme] Project home page and software distribution: "The link at the bottom of this page contains a manual, lipid libraries in .csv format, a batch file for lipidomics with MZmine processing, and LipidMatch software/scripts."
