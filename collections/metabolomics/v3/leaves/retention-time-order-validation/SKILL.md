---
name: retention-time-order-validation
description: Use when a candidate PFAS homologous series (features grouped by matching
  CF2-normalized Kendrick mass defect and CF2-unit mass differences) needs confirmation
  that chromatographic retention time follows the expected order along the series.
when_to_use_negative:
- Input is raw LC-HRMS data that has not been feature-detected and blank-filtered
  yet; homologous series detection operates on a processed feature table.
- Input features are already annotated against catalogued PFAS (e.g. EPA CompTox Dashboard
  list); use exact mass matching instead of series inference.
- Non-PFAS lipid or metabolite applications without CF2 repeating-unit chemistry,
  where CF2-normalized KMD grouping is chemically meaningless.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FluoroMatch 2.0
  role: runs the automated non-targeted PFAS annotation workflow, including homologous-series
    detection with retention-time validation
  repo: https://github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: standalone R mode of FluoroMatch used to run the homologous-series detection
    module with FLOW <- FALSE
  repo: https://github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:4e0b5c2ec8eef2e3f7491ae91b8cd600639f69b18e43d729bea28a1239b51e0d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/InnovativeOmics/Core-Match
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# retention-time-order-validation

## Summary

Validates candidate homologous series in PFAS non-targeted LC-HRMS/MS data by checking that retention time increases monotonically along each series of CF2-normalized Kendrick mass defect-matched features, as implemented in FluoroMatch 2.0's homologous series detection module.

## When to use

Use when a candidate PFAS homologous series (features grouped by matching CF2-normalized Kendrick mass defect and CF2-unit mass differences) needs confirmation that chromatographic retention time follows the expected order along the series.

## When NOT to use

- Input is raw LC-HRMS data that has not been feature-detected and blank-filtered yet; homologous series detection operates on a processed feature table.
- Input features are already annotated against catalogued PFAS (e.g. EPA CompTox Dashboard list); use exact mass matching instead of series inference.
- Non-PFAS lipid or metabolite applications without CF2 repeating-unit chemistry, where CF2-normalized KMD grouping is chemically meaningless.

## Inputs

- blank-filtered LC-HRMS/MS feature table (from feature detection and feature blank filtering)
- per-feature retention times and exact m/z values
- FluoroMatch Modular configuration (FLOW <- FALSE for Modular mode)

## Outputs

- homologous_series_table.csv with series membership and flags
- flagged list of series violating retention-time order
- flagged list of series lacking MS/MS corroboration for expert review

## How to apply

After computing Kendrick mass defects normalized to the CF2 repeating unit (50 Da) for each feature in a blank-filtered feature table, group features into series whose Kendrick mass defects fall within 0.005 Da of each other and whose pairwise mass differences are divisible by 50 Da. Then validate each candidate series by checking that retention time increases in the correct chemical order along the series (longer-chain homologs elute later for typical PFAS classes); flag any series violating RT-order logic for removal or expert review. Retain only series with at least 3 members, since shorter groupings are too likely to be coincidental KMD matches. Write the validated series memberships to a table (e.g. homologous_series_table.csv) and flag series lacking MS/MS corroboration for expert review. This retention-time pattern analysis step is one layer of evidence in FluoroMatch 2.0's broader workflow, which also includes blank filtering, exact mass matching, mass defect filtering, class-based MS/MS screening, fragment screening, and predicted MS/MS from SMILES structures.

## Related tools

- **FluoroMatch 2.0** (runs the automated non-targeted PFAS annotation workflow, including homologous-series detection with retention-time validation) — https://github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (standalone R mode of FluoroMatch used to run the homologous-series detection module with FLOW <- FALSE) — https://github.com/InnovativeOmics/Core-Match

## Examples

```
FLOW <- FALSE; csvInput <- FALSE; ManuallyInputVariables <- FALSE  # FluoroMatch Modular config for PFAS homologous-series detection
```

## Evaluation signals

- Each retained series has ≥3 members with pairwise mass differences divisible by the CF2 unit (50 Da).
- All members of a retained series share CF2-normalized KMD within the 0.005 Da tolerance.
- Retention time increases monotonically in the correct order along every retained series; series violating RT order are flagged, not silently kept.
- Series lacking MS/MS corroboration are explicitly flagged for expert review rather than counted as confirmed.
- On the legacy AFFF benchmark sample, ~50 series with ≥3 members should be detected, and overall coverage of likely PFAS should be consistent with the reported >10x increase over FluoroMatch 1.x.

## Limitations

- Series without MS/MS corroboration cannot be confidently annotated and require expert review; 13 of the 50 AFFF series were flagged in the worked example.
- RT-order validation assumes the expected elution trend for the PFAS class; series with atypical chromatographic behavior may be wrongly flagged.
- Series with fewer than 3 members are not retained, so valid two-member homologs will be missed.
- The workflow depends on upstream feature detection and blank filtering quality; 96% of non-PFAS features were filtered out in the reported application, so poor blank filtering degrades series detection.

## Evidence
