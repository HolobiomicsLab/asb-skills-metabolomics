---
name: lipid-annotation-count-reconciliation
description: Use when auditing or reproducing published lipidomics annotation counts
  from a supplemental annotation workbook — e.g., recounting LipidMatch negative-mode
  identifications on Red Cross plasma (210 lipid ions, 159 features, 15 lipid types)
  from Additional file 3 Table S4.
when_to_use_negative:
- Feature tables and MS/MS spectra are raw and unannotated — run the full LipidMatch
  pipeline first (peak picking with MZmine/XCMS/MS-DIAL, MSConvert conversion to .ms2,
  then LipidMatch matching) rather than recounting annotations
- The annotation table contains no LipidMatch annotation column/entries (e.g., only
  MS-DIAL or Greazy output) — this skill specifically isolates and recounts LipidMatch
  identifications
- Fresh identifications are requested using current LipidMatch libraries (over 500,000
  species across over 60 lipid types per README) and compared against 2017 published
  counts (250,000 species, 56 types) — library drift makes such counts non-comparable
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: LipidMatch
  role: R-based rule-based lipid identifier whose negative-mode annotations (by lipid
    ion, feature, and lipid type) are the object of the recount and reconciliation
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MS-DIAL
  role: independent annotation source whose identifications corroborate LipidMatch
    counts in the cross-software comparison
- name: Greazy
  role: second independent annotator used for corroboration (98% negative-mode agreement
    reported for features identified by all three tools)
- name: pandas
  role: filters the negative-polarity Table S4 worksheet to LipidMatch annotations
    and computes the unique lipid-ion, feature, and lipid-type counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:64bef27de4fd3b1282453dcced53b70cf055f9c53d226ec4cf0b33a9a1e34648
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lipid-annotation-count-reconciliation

## Summary

Recounts distinct lipid ions, features (unique m/z + retention time pairs), and lipid types among LipidMatch annotations in a published annotation table (e.g., the negative-polarity worksheet of Additional file 3, Table S4) and reconciles the recounted figures against published counts. It verifies the reproducibility of negative-mode Red Cross plasma annotation counts (210 lipid ions, 159 features, 15 lipid types) that underpin cross-software corroboration claims.

## When to use

Use when auditing or reproducing published lipidomics annotation counts from a supplemental annotation workbook — e.g., recounting LipidMatch negative-mode identifications on Red Cross plasma (210 lipid ions, 159 features, 15 lipid types) from Additional file 3 Table S4.

## When NOT to use

- Feature tables and MS/MS spectra are raw and unannotated — run the full LipidMatch pipeline first (peak picking with MZmine/XCMS/MS-DIAL, MSConvert conversion to .ms2, then LipidMatch matching) rather than recounting annotations
- The annotation table contains no LipidMatch annotation column/entries (e.g., only MS-DIAL or Greazy output) — this skill specifically isolates and recounts LipidMatch identifications
- Fresh identifications are requested using current LipidMatch libraries (over 500,000 species across over 60 lipid types per README) and compared against 2017 published counts (250,000 species, 56 types) — library drift makes such counts non-comparable

## Inputs

- Additional file 3, Table S4 annotation workbook (Excel), negative-polarity worksheet with LipidMatch, MS-DIAL, and Greazy annotations for Red Cross plasma
- Published reference counts to reconcile against: 210 lipid ions, 159 features, 15 lipid types (negative-mode LipidMatch)

## Outputs

- Recounted negative-mode LipidMatch figures: distinct lipid ions, distinct features (unique m/z + RT pairs), distinct lipid types
- verification_summary.csv with recounted vs. published values and pass/fail flags per metric

## How to apply

Open the annotation workbook (Additional file 3, Table S4) and select the negative-polarity worksheet containing LipidMatch, MS-DIAL, and Greazy annotations for Red Cross plasma features. Using pandas, filter to LipidMatch annotation entries only, excluding MS-DIAL and Greazy columns/rows, since the target is LipidMatch's negative-mode output. Then compute three counts: (1) distinct lipid ions as unique lipid identification entries carrying a LipidMatch annotation, (2) distinct features as unique m/z + retention time pairs with at least one LipidMatch annotation, and (3) distinct lipid types by parsing the lipid class prefix (e.g., CE, Cer, LPC, LPE, PA, PE, PG, PI, PS, SM, TAG) from each annotation. Write recounted vs. published figures — 210 lipid ions, 159 features, 15 lipid types — with per-metric pass/fail flags to verification_summary.csv. Rationale: LipidMatch provided the most lipid identifications for Red Cross plasma compared to MS-DIAL and Greazy, so its per-tool counts must be isolated before computing the corroboration statistics (98% of LipidMatch identifications by fatty acyl constituents corroborated by at least one other software in negative ion mode, 92% in positive mode).

## Related tools

- **LipidMatch** (R-based rule-based lipid identifier whose negative-mode annotations (by lipid ion, feature, and lipid type) are the object of the recount and reconciliation) — https://github.com/GarrettLab-UF/LipidMatch
- **MS-DIAL** (independent annotation source whose identifications corroborate LipidMatch counts in the cross-software comparison)
- **Greazy** (second independent annotator used for corroboration (98% negative-mode agreement reported for features identified by all three tools))
- **pandas** (filters the negative-polarity Table S4 worksheet to LipidMatch annotations and computes the unique lipid-ion, feature, and lipid-type counts)

## Examples

```
python -c "import pandas as pd; df = pd.read_excel('Additional_file_3_Table_S4.xlsx', sheet_name='negative')"
```

## Evaluation signals

- Recounted negative-mode figures equal the published values: 210 lipid ions, 159 features, 15 lipid types for Red Cross plasma LipidMatch annotations
- Invariant: distinct feature count (unique m/z + RT pairs) is less than or equal to distinct lipid-ion count (159 features vs 210 ions in the published table)
- verification_summary.csv exists with one row per metric containing the recounted value, published value, and a pass/fail flag
- Parsed lipid class prefixes resolve to recognized lipid classes consistent with LipidMatch's coverage (56 lipid types in the article's in silico libraries; 15 observed in negative-mode Red Cross plasma)
- Filtered dataframe contains no MS-DIAL or Greazy annotation entries

## Limitations

- Key artifacts (annotation workbook Table S4, software zip) are provided only as journal additional files, not archived in a public repository, so the exact table version may be hard to obtain
- Article-era environment is pinned (R 3.3.3, Windows 7 through 10 only) and depends on external peak-picking software (MZmine, XCMS, MS-DIAL) and MSConvert for generating .ms2 files, making upstream re-identification nontrivial
- Current LipidMatch libraries (over 500,000 lipid species across over 60 lipid types per README) exceed the 2017 paper's libraries (250,000 species, 56 types), so freshly generated annotations will not match published counts

## Evidence

- [abstract] 92% corroboration finding: "92% of LipidMatch identifications by fatty acyl constituents were corroborated by at least one other software in positive mode and 98% in negative ion mode."
- [abstract] library scale and lipid-type coverage: "LipidMatch currently has over 250,000 lipid species spanning 56 lipid types contained in in silico fragmentation libraries."
- [methods] appending other software annotations: "Once lipid identifications are obtained using LipidMatch, identifications from any other software such as Greazy [17], LipidSearch (Thermo Scientific, San Jose, CA), and MS-DIAL can be appended"
- [discussion] Red Cross plasma annotation yield: "Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma"
- [readme] combining results from other lipidomics software: "LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and Compound Discoverer), and combine results from other lipidomics software."
