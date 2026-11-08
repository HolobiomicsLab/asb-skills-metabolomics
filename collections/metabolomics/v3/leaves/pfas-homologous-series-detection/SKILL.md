---
name: pfas-homologous-series-detection
description: Use when you have a blank-filtered HRMS feature table (e.g., from LC-HRMS
  AFFF or environmental samples) and need to flag features belonging to CF2 homologous
  series as non-targeted evidence of PFAS identity.
when_to_use_negative:
- Features have not yet been blank-filtered against solvent blanks — running series
  detection on all ~31132 raw features inflates false series and downstream scoring
- The analyte class is not PFAS and CF2 mass-difference logic does not apply (e.g.,
  lipidomics), unless the Lipid/TWeen parameter toggles are deliberately changed from
  the PFAS defaults
- You need MS/MS-level annotation of individual PFAS rather than series-level non-targeted
  evidence — use the suspect-screening or class-based MS/MS screening steps instead
tools:
- name: FluoroMatch Modular
  role: Runs the homologous-series detection module on the blank-filtered feature
    table (Kendrick mass defect grouping, RT-order validation, >=3-member series retention)
  repo: https://github.com/InnovativeOmics/Core-Match
- name: FluoroMatch 2.0
  role: Software whose non-targeted workflow incorporates homologous-series detection
    as one evidence layer feeding the confidence-level scoring framework
  repo: https://github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Flow 2.0
  role: End-to-end pipeline covering feature detection through homologous series detection
    in the LC-HRMS PFAS discovery workflow
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:d722b22008042f702fa15b74e628c7e3c932d3e06e797b2c67a88edcae375d63
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/InnovativeOmics/Core-Match
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  - LipidMatch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# pfas-homologous-series-detection

## Summary

Automated detection of CF2-based homologous series among non-targeted PFAS features in FluoroMatch 2.0, using Kendrick mass defect normalized to the CF2 (50 Da) repeating unit with retention-time-order validation. This is a key non-targeted evidence layer for flagging features likely to be PFAS without library spectra.

## When to use

Use when you have a blank-filtered HRMS feature table (e.g., from LC-HRMS AFFF or environmental samples) and need to flag features belonging to CF2 homologous series as non-targeted evidence of PFAS identity.

## When NOT to use

- Features have not yet been blank-filtered against solvent blanks — running series detection on all ~31132 raw features inflates false series and downstream scoring
- The analyte class is not PFAS and CF2 mass-difference logic does not apply (e.g., lipidomics), unless the Lipid/TWeen parameter toggles are deliberately changed from the PFAS defaults
- You need MS/MS-level annotation of individual PFAS rather than series-level non-targeted evidence — use the suspect-screening or class-based MS/MS screening steps instead

## Inputs

- Blank-filtered feature table from FluoroMatch feature detection/blank filtering (e.g., 3740 rows with m/z, retention time, and intensity columns)
- Vendor LC-HRMS raw files plus solvent blanks (e.g., legacy AFFF 1999 holding-tank sample and three solvent blanks)
- FluoroMatch Modular scripts placed in RapidTest/LipidMatch_Libraries/Scripts/

## Outputs

- Homologous-series membership table (homologous_series_table.csv) listing features grouped into CF2-normalized series
- Series flags (>=3 members) and flags for series lacking MS/MS corroboration for expert review
- Homologous-series evidence contributing to per-feature confidence-level scores (1238 probable PFAS in the AFFF validation)

## How to apply

Load the blank-filtered feature table (e.g., the 3740-feature table retained from 31132 detected AFFF features) into FluoroMatch Modular's homologous-series detection module. Compute Kendrick mass defects for every feature normalized to the CF2 repeating unit (50 Da), then group features into series where member Kendrick mass defects agree within 0.005 Da and pairwise mass differences are divisible by the 50 Da CF2 unit. Validate each candidate series by checking that retention time increases in the correct order along the series, flagging any series violating RT-order logic. Retain only series with at least 3 members; on the legacy AFFF 1999 holding-tank sample this procedure automatically detected 50 such series. Homologous-series membership combines with exact mass matching (>8000 EPA CompTox PFAS structures), mass defect filtering, fragment evidence, and confidence scoring to convert potential PFAS (2290) into probable PFAS (1238).

## Related tools

- **FluoroMatch Modular** (Runs the homologous-series detection module on the blank-filtered feature table (Kendrick mass defect grouping, RT-order validation, >=3-member series retention)) — https://github.com/InnovativeOmics/Core-Match
- **FluoroMatch 2.0** (Software whose non-targeted workflow incorporates homologous-series detection as one evidence layer feeding the confidence-level scoring framework) — https://github.com/InnovativeOmics/Core-Match
- **FluoroMatch Flow 2.0** (End-to-end pipeline covering feature detection through homologous series detection in the LC-HRMS PFAS discovery workflow)

## Examples

```
R: FLOW <- FALSE; csvInput <- TRUE; Lipid <- FALSE; TWeen_pos <- FALSE; source('Modular.r')  # defaults to PFAS analysis per Core-Match README
```

## Evaluation signals

- On the legacy AFFF validation sample, the module detects exactly 50 homologous series with 3 or more members from the 3740 blank-filtered features
- Every retained series has pairwise member mass differences divisible by the 50 Da CF2 unit and Kendrick mass defects within 0.005 Da
- Retention time increases monotonically in the correct order along each validated series; series violating RT-order logic are flagged
- Downstream confidence scoring on the AFFF data yields 2290 potential PFAS and 1238 probable PFAS, consistent with the published 96% feature-list reduction
- homologous_series_table.csv is produced and flags the 13 series lacking MS/MS corroboration for expert review

## Limitations

- Series detection alone does not confirm structures — 13 of the detected AFFF series lacked MS/MS corroboration and require expert review
- The method depends on correct RT ordering; series violating RT-order logic are flagged rather than automatically trusted
- Integration requires manually placing edited scripts from the GitHub repository into the InnovativeOmics.com distribution directories and toggling FLOW/csvInput/Lipid parameters; misconfiguration changes the analysis mode

## Evidence
