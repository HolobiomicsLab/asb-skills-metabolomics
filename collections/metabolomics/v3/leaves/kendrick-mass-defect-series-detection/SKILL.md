---
name: kendrick-mass-defect-series-detection
description: Use when a blank-filtered LC-HRMS feature table from PFAS samples (e.g.
  AFFF) must be grouped into homologous series based on CF2-normalized Kendrick mass
  defect and RT-order logic, beyond suspect-list exact mass matching.
when_to_use_negative:
- The input is already an annotated suspect list — use exact mass matching to catalogued
  PFAS instead; homologous-series detection is for non-targeted grouping of unmatched
  features
- The goal is MS/MS-based annotation (class-based MS/MS screening, fragment screening,
  or predicted MS/MS from SMILES structures) rather than mass-defect-based series
  detection
- The analyte classes have no CF2-like repeating unit (e.g. non-polyfluorinated lipids)
  — the 50 Da CF2 normalization is specific to PFAS
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: FluoroMatch Modular
  role: 'Runs the homologous-series detection module: computes CF2-normalized Kendrick
    mass defects on the blank-filtered feature table, groups series within 0.005 Da,
    validates RT order, and writes homologous_series_table.csv'
  repo: https://github.com/InnovativeOmics/Core-Match
- name: FluoroMatch 2.0
  role: Parent non-targeted PFAS annotation software whose workflow includes homologous
    series detection as a key step
  repo: https://github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:9827ab68aa2a5ac2a0c7891f35e6978a1749dd74bd67a1ff69b615438dd886c6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/InnovativeOmics/Core-Match
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# kendrick-mass-defect-series-detection

## Summary

Automates non-targeted detection of PFAS homologous series in LC-HRMS feature tables by grouping features with CF2-normalized Kendrick mass defects (within 0.005 Da) separated by CF2-unit mass differences (divisible by 50 Da), then validating retention-time order. It is the FluoroMatch 2.0 homologous-series detection module that found 50 series with ≥3 members in legacy AFFF.

## When to use

Use when a blank-filtered LC-HRMS feature table from PFAS samples (e.g. AFFF) must be grouped into homologous series based on CF2-normalized Kendrick mass defect and RT-order logic, beyond suspect-list exact mass matching.

## When NOT to use

- The input is already an annotated suspect list — use exact mass matching to catalogued PFAS instead; homologous-series detection is for non-targeted grouping of unmatched features
- The goal is MS/MS-based annotation (class-based MS/MS screening, fragment screening, or predicted MS/MS from SMILES structures) rather than mass-defect-based series detection
- The analyte classes have no CF2-like repeating unit (e.g. non-polyfluorinated lipids) — the 50 Da CF2 normalization is specific to PFAS

## Inputs

- Blank-filtered LC-HRMS feature table (m/z, retention time, intensity per feature, e.g. csv feature table from feature detection and blank filtering)
- csvInput feature table (FluoroMatch Modular csv input mode)
- MS/MS spectra (used downstream to corroborate flagged series)

## Outputs

- homologous_series_table.csv (series-membership table with ≥3-member CF2-normalized series)
- Flags for series violating RT-order logic
- Flags for series lacking MS/MS corroboration for expert review

## How to apply

Load the blank-filtered feature table (output of feature blank filtering) into FluoroMatch Modular's homologous-series detection module (run in Modular mode with FLOW <- FALSE). Compute the Kendrick mass defect for every feature normalized to the CF2 (50 Da) repeating unit. Group features into series where member Kendrick mass defects are within 0.005 Da of each other and pairwise mass differences are divisible by 50 Da (the CF2 unit). Validate each candidate series by checking retention time increases in the correct order along the series, flagging series that violate RT-order logic. Retain only series with at least 3 members and write the full series-membership table to homologous_series_table.csv, flagging series lacking MS/MS corroboration (13 of 50 in the legacy AFFF run) for expert review.

## Related tools

- **FluoroMatch Modular** (Runs the homologous-series detection module: computes CF2-normalized Kendrick mass defects on the blank-filtered feature table, groups series within 0.005 Da, validates RT order, and writes homologous_series_table.csv) — https://github.com/InnovativeOmics/Core-Match
- **FluoroMatch 2.0** (Parent non-targeted PFAS annotation software whose workflow includes homologous series detection as a key step) — https://github.com/InnovativeOmics/Core-Match

## Examples

```
R -e "FLOW <- FALSE; csvInput <- TRUE; ManuallyInputVariables <- FALSE; source('LipidMatch-4.2/FluoroMatch_Modular/Modular.r')"
```

## Evaluation signals

- All series members have Kendrick mass defects within 0.005 Da of each other when normalized to the 50 Da CF2 unit
- Pairwise mass differences between series members are divisible by 50 Da (CF2 spacing)
- Retention time increases monotonically in the correct order along each retained series
- Only series with ≥3 members are retained (legacy AFFF benchmark: 50 such series detected)
- Output file homologous_series_table.csv exists and flags series lacking MS/MS corroboration (13 of 50 in the AFFF run) for expert review

## Limitations

- Series without MS/MS corroboration are flagged for expert review rather than confirmed (13 of 50 series in the legacy AFFF sample lacked MS/MS evidence)
- Series that violate retention-time order logic are flagged, not auto-corrected, requiring manual inspection
- Detection depends on prior feature blank filtering quality; the 50-series count was obtained on a blank-filtered feature table

## Evidence

- [results] 50 series with ≥3 members detected on legacy AFFF via CF2-normalized Kendrick mass defect grouping with RT-order validation: [evidence span withheld — anchor status: not_found]
