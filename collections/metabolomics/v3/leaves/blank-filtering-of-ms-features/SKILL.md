---
name: blank-filtering-of-ms-features
description: Use when processing raw LC-HRMS vendor files with matched solvent blanks
  (e.g., 3 solvent blanks for a legacy AFFF sample) and the full feature list (~31,132
  features) must be stripped of non-sample background features before PFAS annotation.
when_to_use_negative:
- Input is already a blank-filtered feature table (e.g., the 3,740-row annotated output)
  — re-filtering would discard legitimate sample features.
- No matched blank samples exist for the acquisition — blank subtraction cannot be
  justified and downstream exact mass/mass-defect/homologous-series stages should
  carry the reduction instead.
- The analysis target is lipidomics rather than PFAS — set Lipid <- TRUE in the LipidMatch/FluoroMatch
  scripts instead of relying on the PFAS-default pipeline.
tools:
- name: FluoroMatch Flow 2.0
  role: End-to-end pipeline (feature detection → blank filtering → PFAS screening)
    in which blank filtering is the second step
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular scripts variant where the blank-filtering step is executed; installed
    by copying Scripts into RapidTest/LipidMatch_Libraries/Scripts/
  repo: github.com/InnovativeOmics/Core-Match
- name: LipidMatch 4.2
  role: Software distribution hosting the FluoroMatch_Modular directory and scripts
    used to run the blank-filtering workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:c9ab2e3f924762cf2ea0474295183e3068196f0f979c56038c72c151269e89b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  - LipidMatch
  - LipidMatch 4.2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# Blank filtering of MS features

## Summary

A first-pass feature-reduction step in the FluoroMatch Flow 2.0 PFAS pipeline that removes features detected in LC-HRMS vendor files when they are also present in solvent blanks, cutting background noise before PFAS-specific screening.

## When to use

Use when processing raw LC-HRMS vendor files with matched solvent blanks (e.g., 3 solvent blanks for a legacy AFFF sample) and the full feature list (~31,132 features) must be stripped of non-sample background features before PFAS annotation.

## When NOT to use

- Input is already a blank-filtered feature table (e.g., the 3,740-row annotated output) — re-filtering would discard legitimate sample features.
- No matched blank samples exist for the acquisition — blank subtraction cannot be justified and downstream exact mass/mass-defect/homologous-series stages should carry the reduction instead.
- The analysis target is lipidomics rather than PFAS — set Lipid <- TRUE in the LipidMatch/FluoroMatch scripts instead of relying on the PFAS-default pipeline.

## Inputs

- Raw LC-HRMS sample vendor files (e.g., legacy AFFF 1999 holding-tank sample)
- Matched solvent blank vendor files (three solvent blanks)
- FluoroMatch Flow 2.0 / FluoroMatch Modular software distribution (LipidMatch-4.2)

## Outputs

- Blank-filtered feature table (3,740 rows retained from 31,132 detected features)
- Stage-by-stage feature count summary mirroring Fig. 5i–ii (31,132 → 3,740 → 2,290 → 1,238)

## How to apply

Set up the FluoroMatch Flow 2.0 modular scripts by copying the root Scripts folder into RapidTest/LipidMatch_Libraries/Scripts/ per the Core-Match README. Load the sample vendor files together with the solvent blanks (here, the 1999 holding-tank AFFF sample and three solvent blanks) and run feature detection to yield the full feature list (~31,132 features). Then apply feature blank filtering against the three solvent blanks to remove features attributable to background, retaining 3,740 features. The rationale is that the raw list is dominated by non-PFAS background; blank filtering is the first stage in the cascade that continues with exact mass matching to the EPA CompTox PFAS structure list (>8,000 structures), mass defect filtering, and CF2-based homologous series detection (2,290 potential PFAS, 1,238 probable PFAS). The blank-filtered table is the input to the subsequent confidence-level scoring framework, where 39% of retained features scored E. Verify the step by confirming the filtered table has exactly 3,740 rows and the stage counts match the published reduction (31,132 → 3,740 → 2,290 → 1,238; 96% reduction).

## Related tools

- **FluoroMatch Flow 2.0** (End-to-end pipeline (feature detection → blank filtering → PFAS screening) in which blank filtering is the second step) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular scripts variant where the blank-filtering step is executed; installed by copying Scripts into RapidTest/LipidMatch_Libraries/Scripts/) — github.com/InnovativeOmics/Core-Match
- **LipidMatch 4.2** (Software distribution hosting the FluoroMatch_Modular directory and scripts used to run the blank-filtering workflow)

## Evaluation signals

- Blank-filtered feature table contains exactly 3,740 rows from 31,132 detected features.
- Overall cascade reduction matches the published 96% of features likely not PFAS filtered out.
- Downstream counts remain consistent: 2,290 potential PFAS and 1,238 probable PFAS after mass defect/homologous-series stages.
- Score distribution after the full pipeline shows 39% of retained features scored E, matching Fig. 5i–ii.
- Script integration is correct: Modular.r placed in LipidMatch-4.2\Flow\LipidMatch_Distribution and LipidMatch-4.2\FluoroMatch_Modular.

## Limitations

- Blank filtering only removes background features; it cannot distinguish sample features that are non-PFAS, so the 3,740 retained features still require exact mass matching, mass defect filtering, and homologous-series detection to reach 1,238 probable PFAS.
- The reduction magnitude depends on the number and quality of blanks (three solvent blanks in this application); different blank sets would change the retained-feature count.
- The user must correctly configure FLOW/csvInput/ManuallyInputVariables/Lipid flags in the modular scripts; misconfiguration changes pipeline behavior.

## Evidence

- [abstract] workflow order abstract: "These steps include feature detection, feature blank filtering, exact mass matching to catalogued PFAS, mass defect filtering, homologous series detection"
- [abstract] 96% reduction abstract: "Furthermore, we were able to filter out 96% of features which were likely not PFAS."
- [readme] README setup: "* Copy the contents of the root Scripts folder into: RapidTest/LipidMatch_Libraries/Scripts/"
