---
name: lipid-type-classification-tallying
description: Use when you have a feature-level annotation workbook (e.g. Additional
  file 3, Table S4) from LipidMatch, MS-DIAL, and Greazy and need to count unique
  lipid ions, features, and lipid types for a given polarity such as positive-mode
  IE-ddMS2-top5 data.
when_to_use_negative:
- Input is raw vendor MS files or unannotated feature tables — run the LipidMatch
  identification workflow (peak picking, MSConvert .ms2 conversion, LipidMatch rule-based
  matching) before tallying
- Goal is annotation counts in negative ion mode or a different sample type — the
  validated figures (5159 ions / 1401 features / 26 lipid types) are specific to positive-mode
  Red Cross plasma
- Goal is comparing annotation overlap between LipidMatch, MS-DIAL, and Greazy rather
  than reproducing LipidMatch-only counts
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidMatch
  role: R tool whose appended identifications (ranked lipid ions, lipid type labels,
    adducts) are the rows being tallied
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: pandas/openpyxl
  role: Loading and filtering the positive-polarity worksheet of the Additional file
    3 annotation workbook
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:4c6543cb8d351dc68d3f05a7c7e26c1bf0bf78bbb019c7e99c25cbd131fe7da8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - R
  - pandas/openpyxl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lipid-type-classification-tallying

## Summary

Tallies lipid annotation coverage from LipidMatch output workbooks — counting unique lipid ions, annotated features, and distinct lipid types per polarity — to reproduce or audit the annotation-count metrics reported in the LipidMatch paper for Red Cross plasma.

## When to use

Use when you have a feature-level annotation workbook (e.g. Additional file 3, Table S4) from LipidMatch, MS-DIAL, and Greazy and need to count unique lipid ions, features, and lipid types for a given polarity such as positive-mode IE-ddMS2-top5 data.

## When NOT to use

- Input is raw vendor MS files or unannotated feature tables — run the LipidMatch identification workflow (peak picking, MSConvert .ms2 conversion, LipidMatch rule-based matching) before tallying
- Goal is annotation counts in negative ion mode or a different sample type — the validated figures (5159 ions / 1401 features / 26 lipid types) are specific to positive-mode Red Cross plasma
- Goal is comparing annotation overlap between LipidMatch, MS-DIAL, and Greazy rather than reproducing LipidMatch-only counts

## Inputs

- Additional file 3: Table S4 annotation workbook (positive-polarity worksheet) with LipidMatch, MS-DIAL, and Greazy annotations
- LipidMatch-annotated feature tables keyed by m/z and retention time
- LipidMatch adduct and lipid class/type label columns

## Outputs

- annotation_counts_positive.csv with one row per reported metric (unique lipid ions, features, lipid types)
- Recounted lipid-ion, feature, and lipid-type counts per polarity worksheet

## How to apply

Load the annotation workbook (e.g. the ec-positive-mode worksheet of Additional file 3) with pandas using the openpyxl engine, then restrict to the LipidMatch annotation columns: per-feature ranked lipid ion entries, lipid class/type labels, and adduct designations. Count unique lipid ions by tallying every annotated lipid ion row, deliberately treating multiple adducts of the same lipid species as distinct ions, per the paper's counting convention. Separately count the distinct features (unique m/z + retention-time entries) carrying at least one LipidMatch annotation, and the number of distinct lipid types those annotations span. For positive-mode Red Cross plasma the recount must equal the published figures: 5159 unique lipid ions, 1401 features, and 26 lipid types; write one row per metric to annotation_counts_positive.csv.

## Related tools

- **LipidMatch** (R tool whose appended identifications (ranked lipid ions, lipid type labels, adducts) are the rows being tallied) — https://github.com/GarrettLab-UF/LipidMatch
- **pandas/openpyxl** (Loading and filtering the positive-polarity worksheet of the Additional file 3 annotation workbook)

## Examples

```
df = pd.read_excel('Additional_file3_TableS4.xlsx', sheet_name='ec-positive', engine='openpyxl'); counts = {'unique_lipid_ions': df['lipid_ion'].nunique(), 'features': df[['mz','rt']].drop_duplicates().shape[0], 'lipid_types': df['lipid_type'].nunique()}
```

## Evaluation signals

- Recounted unique lipid ion count equals 5159 for positive-mode Red Cross plasma
- Feature count (unique m/z + retention time entries with ≥1 LipidMatch annotation) equals 1401
- Lipid type count equals 26 distinct lipid types in positive mode
- Multiple adducts of one lipid species are counted as distinct lipid ions, matching the paper's convention
- annotation_counts_positive.csv contains exactly one row per reported metric

## Limitations

- Key artifacts (software zip, supplemental figures/tables, annotation workbook) are only provided as additional files, not archived in a public repository
- Counts depend on the polarity and acquisition method (IE-ddMS2-top5); do not transfer positive-mode figures to other worksheets
- LipidMatch library coverage (e.g. 250,000+ species across 56 lipid types in the paper; README cites over 500,000 species and 60+ types in later versions) means achievable lipid-type counts vary with the library version used

## Evidence

- [abstract] We introduce LipidMatch, an R-based tool for lipid identification for liquid chromatography tandem mass spectrometry workflows.: "We introduce LipidMatch, an R-based tool for lipid identification for liquid chromatography tandem mass spectrometry workflows."
- [discussion] Red Cross plasma provides the most identifications: "Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma"
- [methods] LipidMatch written in R: "LipidMatch was written in R [9]. The user interface for LipidMatch consists of a series of dialogue boxes developed using gWidgets API and the tcltk R package."
- [readme] Lipid libraries as csv files: "The link at the bottom of this page contains a manual, lipid libraries in .csv format, a batch file for lipidomics with MZmine processing, and LipidMatch software/scripts."
