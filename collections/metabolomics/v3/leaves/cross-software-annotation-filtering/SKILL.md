---
name: cross-software-annotation-filtering
description: Use when LipidMatch rule-based annotations on ddMS2-topN feature tables
  need independent validation against other open-source lipid identification software
  such as MS-DIAL or Greazy.
when_to_use_negative:
- The feature table already contains cross-software annotations and you only need
  to re-run LipidMatch scoring
- The task is annotation of a lipid class (e.g., cardiolipins) that MS-DIAL and Greazy
  libraries do not cover — cross-validation is unavailable for those types
- Only a single identification level (e.g., precursor-m/z-only, level 4) is at stake,
  where corroboration by other tools is not informative
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: LipidMatch
  role: Primary rule-based identification engine; its output feature tables receive
    appended annotations from other software for cross-comparison
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MS-DIAL
  role: One of two comparison tools whose annotations are appended to the same feature
    table to compute corroboration rates
- name: Greazy
  role: Second comparison tool used in the three-way cross-software evaluation
- name: R
  role: Language in which LipidMatch is written and through which the annotation-appending
    and corroboration workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:f2ec294958cf6f92071d4a3b1e383cfe47a1e3933450aa09881a593700ea42e6
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cross-software-annotation-filtering

## Summary

Append and compare lipid identifications from LipidMatch, MS-DIAL, and Greazy on the same UHPLC-HRMS/MS feature table, then compute cross-software corroboration rates (e.g., 92% positive-mode, 98% negative-mode agreement) to vet rule-based annotations.

## When to use

Use when LipidMatch rule-based annotations on ddMS2-topN feature tables need independent validation against other open-source lipid identification software such as MS-DIAL or Greazy.

## When NOT to use

- The feature table already contains cross-software annotations and you only need to re-run LipidMatch scoring
- The task is annotation of a lipid class (e.g., cardiolipins) that MS-DIAL and Greazy libraries do not cover — cross-validation is unavailable for those types
- Only a single identification level (e.g., precursor-m/z-only, level 4) is at stake, where corroboration by other tools is not informative

## Inputs

- Feature table(s) from peak picking (MZmine, XCMS, MS-DIAL, Compound Discoverer)
- .ms2 files from MSConvert
- LipidMatch in silico libraries (.csv)
- LipidMatch output feature tables with appended identifications
- MS-DIAL and Greazy annotation outputs

## Outputs

- Feature table with LipidMatch, MS-DIAL, and Greazy annotations side-by-side per feature
- Cross-software corroboration rates (e.g., % of LipidMatch identifications corroborated)
- Agreement statistics split by ionization polarity (positive vs. negative mode)

## How to apply

After running LipidMatch on .ms2 files with exact mass matching at MS1 and fragment matching in ppm, append identifications from MS-DIAL and Greazy to the same feature table in an additional column, as LipidMatch's workflow explicitly supports. Align annotations by feature (m/z plus retention time) and evaluate agreement at two levels: fatty acyl constituents and lipid class. Compute the percentage of LipidMatch identifications corroborated by at least one other tool — the article reports 92% in positive mode and 98% in negative ion mode for Red Cross plasma features identified by all three tools. Use these rates to prioritize which LipidMatch rule-based identifications (levels 1-4) are most defensible.

## Related tools

- **LipidMatch** (Primary rule-based identification engine; its output feature tables receive appended annotations from other software for cross-comparison) — https://github.com/GarrettLab-UF/LipidMatch
- **MS-DIAL** (One of two comparison tools whose annotations are appended to the same feature table to compute corroboration rates)
- **Greazy** (Second comparison tool used in the three-way cross-software evaluation)
- **R** (Language in which LipidMatch is written and through which the annotation-appending and corroboration workflow is executed)

## Examples

```
lipidmatch --feature-table features.csv --ms2 pooled.ms2 --polarity negative --append-other MS-DIAL Greazy
```

## Evaluation signals

- Percentage of LipidMatch fatty acyl identifications corroborated by at least one other software should be ~92% in positive mode and ~98% in negative mode on Red Cross plasma features identified by all three tools
- Every feature row should have at most one annotation per tool, aligned on the same m/z and retention time key
- Agreement should be computed only on features with identifications from all three software, not on the full feature table
- Disagreements should resolve to known library-coverage gaps (e.g., 56 vs. 34 vs. 24 lipid types for LipidMatch/MS-DIAL/Greazy)
- Annotations should be appended as an additional column without altering LipidMatch's own identification-level assignments (1-4)

## Limitations

- LipidMatch covers 56 lipid types whereas MS-DIAL covers 34 and Greazy 24, so lower corroboration on LipidMatch-only classes reflects library differences, not false positives
- The workflow is validated primarily on Windows (7-10) with R 3.3.3 and depends on external MSConvert for .ms2 generation
- Cross-software comparison assumes consistent m/z and retention-time alignment across tools; drift will inflate apparent disagreement
- Only Q-Exactive orbitrap and Q-TOF (Agilent, Bruker, SCIEX) data are validated; Waters files are not supported

## Evidence

- [methods] Append identifications from other software: "Once lipid identifications are obtained using LipidMatch, identifications from any other software such as Greazy [17], LipidSearch (Thermo Scientific, San Jose, CA), and MS-DIAL can be appended in"
- [abstract] 92% positive / 98% negative corroboration: "92% of LipidMatch identifications by fatty acyl constituents were corroborated by at least one other software in positive mode and 98% in negative ion mode."
- [methods] Cross-tool library coverage differences: "With 56 lipid types, LipidMatch in silico libraries cover the greatest number of lipid types of any open source software to date, with MS-DIAL containing 34 lipid types, and Greazy containing 24"
- [readme] Integration with peak-picking software: "For example LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and Compound Discoverer), and combine results from other lipidomics software."
- [readme] Waters file limitation: "The software does not currently support Waters files."
