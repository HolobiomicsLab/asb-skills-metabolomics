---
name: feature-table-parsing
description: Use when input is a peak-picked feature table (e.g., MZmine/MS-DIAL/XCMS
  export) with m/z and retention time per feature, and the goal is to prepare it for
  LipidMatch MS1 precursor matching or to tally annotated features per polarity.
when_to_use_negative:
- Input is still raw vendor mass spec data — run feature detection with MZmine, XCMS,
  or MS-DIAL first, since LipidMatch consumes feature tables, not raw files
- MS/MS raw spectra still need peak picking — convert and peak-pick to .ms2 with MSConvert
  before parsing
- Data already carries appended lipid identifications from LipidMatch/MS-DIAL/Greazy
  and the task is comparison rather than table preparation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidMatch
  role: Consumes the parsed feature tables, matches each feature's m/z against in
    silico precursor ions, and appends identifications automatically
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MZmine
  role: Peak-picking software used to create the feature tables that this skill parses
- name: MSConvert
  role: Peak-picks MS/MS data and converts it to .ms2 file format used alongside feature
    tables
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:4d7111d295ab0563d3c1faebf65eff96c7e8de5dd88489418a8c66b1f56b0f4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - MSConvert
  - MZmine
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# feature-table-parsing

## Summary

Load and standardize peak-picked feature tables (m/z, retention time, intensities per feature) produced by tools like MZmine, XCMS, or MS-DIAL so they can be organized alongside .ms2 MS/MS files for LipidMatch's MS1 exact-mass matching and downstream annotation.

## When to use

Use when input is a peak-picked feature table (e.g., MZmine/MS-DIAL/XCMS export) with m/z and retention time per feature, and the goal is to prepare it for LipidMatch MS1 precursor matching or to tally annotated features per polarity.

## When NOT to use

- Input is still raw vendor mass spec data — run feature detection with MZmine, XCMS, or MS-DIAL first, since LipidMatch consumes feature tables, not raw files
- MS/MS raw spectra still need peak picking — convert and peak-pick to .ms2 with MSConvert before parsing
- Data already carries appended lipid identifications from LipidMatch/MS-DIAL/Greazy and the task is comparison rather than table preparation

## Inputs

- Peak-picked feature table CSV (e.g., MZmine export) with m/z, retention time, and sample intensities per feature
- Feature tables per polarity type (positive and negative)
- MS/MS peak-picked .ms2 files from MSConvert, organized in subfolders

## Outputs

- Parsed feature table keyed by unique m/z and retention time entries per feature
- Directory-organized feature tables with appended LipidMatch identifications
- Annotation count tables (e.g., annotation_counts_positive.csv with unique lipid ion, feature, and lipid type counts)

## How to apply

Read the feature table (CSV from MZmine or equivalent) with one row per feature defined by m/z and retention time, plus sample intensity columns, using pandas with openpyxl for xlsx workbooks or R for LipidMatch's pipeline. Separate tables per polarity type, since LipidMatch requires feature tables for each polarity created from raw vendor files by peak-picking software such as MZmine, XCMS, or MS-DIAL. Retain m/z, RT, and intensity columns and exclude dimers, multimers, and in-source fragments when matching against in silico precursor ions, which is done at the MS1 level with a user-defined m/z tolerance in Da (Step 1; Fig. 3). Place the feature table(s) and .ms2 MS/MS data (generated via MSConvert) into the directory structure shown in Fig. 2, with MS/MS files in each feature table's subfolder. When the task is annotation counting, filter to the annotated rows and count distinct features by unique m/z and retention time entries.

## Related tools

- **LipidMatch** (Consumes the parsed feature tables, matches each feature's m/z against in silico precursor ions, and appends identifications automatically) — https://github.com/GarrettLab-UF/LipidMatch
- **MZmine** (Peak-picking software used to create the feature tables that this skill parses)
- **MSConvert** (Peak-picks MS/MS data and converts it to .ms2 file format used alongside feature tables)

## Examples

```
features <- read.csv("mzmine_feature_table.csv")  # one row per feature: m/z, RT, sample intensities
```

## Evaluation signals

- Parsed table contains one row per feature defined by m/z and retention time, with intensity columns preserved for downstream fragment-intensity ranking
- Feature tables are split by polarity type (positive/negative) as LipidMatch requires feature tables for each polarity
- Directory layout matches Fig. 2: feature table(s) with MS/MS .ms2 files in the matching subfolder
- When recounting annotations, unique feature counts by m/z+RT reproduce reported values (e.g., 1401 annotated features in positive-mode Red Cross plasma)
- No dimers, multimers, or in-source fragments are included among MS1 precursor matches

## Limitations

- LipidMatch does not currently support Waters files, so feature tables from Waters acquisitions cannot enter this workflow directly
- Feature table quality depends entirely on the upstream peak-picking software (MZmine, XCMS, MS-DIAL, or other); parsing cannot recover missed features
- Feature table(s) and MS/MS data must be manually placed into the required directory structure before running LipidMatch

## Evidence

- [methods] Feature tables per polarity and directory organization: [evidence span withheld — anchor status: not_found]
- [methods] Directory structure requirement: "Feature table(s) and MS/MS data are placed into a directory as shown in Fig. 2."
- [methods] MS1 exact mass matching on parsed features: "LipidMatch performs exact mass matching at the MS1 level between in silico precursor ions and each features m/z using a user defined m/z tolerance (Da)"
- [readme] Compatible peak-picking tools per README: "LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and Compound Discoverer)"
