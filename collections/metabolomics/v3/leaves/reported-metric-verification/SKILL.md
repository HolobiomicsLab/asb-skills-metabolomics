---
name: reported-metric-verification
description: Use when a paper reports aggregate counts (e.g., 5159 unique lipid ions,
  1401 features, 26 lipid types) that must be verified by recounting them from the
  underlying per-feature annotation workbook in its supplementary files.
when_to_use_negative:
- The input is raw vendor data or .ms2 files rather than an already-annotated feature
  table workbook — run the LipidMatch identification workflow (peak picking, MSConvert,
  identification) first
- The metric to verify is a software-performance comparison across tools rather than
  a recountable count from a supplementary workbook
- The workbook mixes polarities or tools in a single sheet without polarity/annotation-source
  labels that allow isolating LipidMatch positive-mode columns
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: LipidMatch
  role: Source of the per-feature ranked lipid ion annotations being recounted and
    verified in the workbook
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MS-DIAL
  role: Comparison annotations present in the same worksheet; its columns are excluded
    when isolating LipidMatch counts
- name: Greazy
  role: Comparison annotations present in the same worksheet; its columns are excluded
    when isolating LipidMatch counts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:cf902efa8a9b0d7fff7595cca94bf4446da606f71b179b16bf7ecc9e4365fb3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - Greazy
  - LipidMatch
  - MS-DIAL
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# reported-metric-verification

## Summary

Independently recounts summary statistics reported in a publication (here, LipidMatch annotation counts for positive-mode IE-ddMS2-top5 Red Cross plasma) directly from the authors' supplementary annotation workbook, accepting the recount only when it exactly matches the reported values.

## When to use

Use when a paper reports aggregate counts (e.g., 5159 unique lipid ions, 1401 features, 26 lipid types) that must be verified by recounting them from the underlying per-feature annotation workbook in its supplementary files.

## When NOT to use

- The input is raw vendor data or .ms2 files rather than an already-annotated feature table workbook — run the LipidMatch identification workflow (peak picking, MSConvert, identification) first
- The metric to verify is a software-performance comparison across tools rather than a recountable count from a supplementary workbook
- The workbook mixes polarities or tools in a single sheet without polarity/annotation-source labels that allow isolating LipidMatch positive-mode columns

## Inputs

- Additional file 3: Table S4 annotation workbook (Excel, positive-polarity worksheet) with feature-level annotations from LipidMatch, MS-DIAL, and Greazy
- Reported target metrics from the paper: 5159 unique lipid ions, 1401 features, 26 lipid types (positive mode)

## Outputs

- annotation_counts_positive.csv with one row per reported metric (unique lipid ions, features, lipid types) and the recounted value
- Pass/fail validation status comparing each recount to the reported figure

## How to apply

Load the reported workbook (Additional file 3: Table S4, the ec-positive-mode worksheet) with pandas using the openpyxl engine so per-feature LipidMatch annotations are readable. Filter to the LipidMatch annotation columns, retaining the per-feature ranked lipid ion entries, lipid class/type labels, and adduct designations. Count unique lipid ions by tallying every annotated lipid ion row, following the paper's convention that multiple adducts of one lipid species count as distinct lipid ions; count features as unique m/z and retention-time entries carrying at least one LipidMatch annotation; and count distinct lipid types covered. Accept the verification only on exact equality: 5159 unique lipid ions, 1401 features, and 26 lipid types for positive ion mode, writing one row per reported metric to annotation_counts_positive.csv.

## Related tools

- **LipidMatch** (Source of the per-feature ranked lipid ion annotations being recounted and verified in the workbook) — https://github.com/GarrettLab-UF/LipidMatch
- **MS-DIAL** (Comparison annotations present in the same worksheet; its columns are excluded when isolating LipidMatch counts)
- **Greazy** (Comparison annotations present in the same worksheet; its columns are excluded when isolating LipidMatch counts)

## Examples

```
import pandas as pd; df = pd.read_excel('Additional_file3_TableS4.xlsx', sheet_name='ec_positive', engine='openpyxl'); df[df['software']=='LipidMatch'].groupby(['feature_id','lipid_ion']).size()
```

## Evaluation signals

- Recounted unique lipid ion count equals exactly 5159 in positive mode
- Recounted annotated feature count (unique m/z + retention time entries with ≥1 LipidMatch annotation) equals exactly 1401
- Recounted lipid type count equals exactly 26
- annotation_counts_positive.csv contains one row per reported metric with the recount and the reported value
- Counting treats multiple adducts of a lipid species as distinct lipid ions, consistent with the paper's convention

## Limitations

- The annotation workbook is only distributed as a journal additional file, not archived in a public repository, so reproducibility depends on obtaining it from the publication
- Counting conventions (e.g., adducts counted as distinct ions) must match the paper's; a different convention yields different counts
- The reported metrics are specific to positive-polarity IE-ddMS2-top5 Red Cross plasma data; other datasets or polarities need their own targets

## Evidence

- [supplementary] LipidMatch annotations in the ec-positive-mode worksheet of Additional file 3 report 5159 unique lipid ions across 1401 features and 26 lipid types.: [evidence span withheld — anchor status: not_found]
- [discussion] Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma: "Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma"
- [methods] Once the user runs LipidMatch and enters user parameters, LipidMatch will automatically append identifications to each feature table: "Once the user runs LipidMatch and enters user parameters, LipidMatch will automatically append identifications to each feature table using MS/MS files contained in that feature table’s subfolder."
- [methods] the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches: "the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches, including those not used for confirmation (Step 7, Fig. 3)."
- [methods] identifications from any other software such as Greazy, LipidSearch (Thermo Scientific, San Jose, CA), and MS-DIAL can be appended: "Once lipid identifications are obtained using LipidMatch, identifications from any other software such as Greazy [17], LipidSearch (Thermo Scientific, San Jose, CA), and MS-DIAL can be appended in"
