---
name: fluoromatch-scoring-framework-application
description: Use when LC-HRMS/MS feature tables and MS/MS spectra (e.g. AFFF injections,
  mzML/raw acquisitions) have been processed through FluoroMatch Modular steps and
  per-feature PFAS annotation confidence levels (score A) must be assigned.
when_to_use_negative:
- 'Input is not PFAS analysis: FluoroMatch defaults to PFAS analysis only when both
  Lipid and TWeen_pos flags are FALSE; set these appropriately or the scoring will
  target the wrong analyte class.'
- You only need raw feature detection/blank filtering without annotation confidence
  — the scoring framework is a downstream layer and requires the full screening evidence
  first.
- Data are from a targeted suspect-screening release of FluoroMatch (original release)
  rather than the truly non-targeted FluoroMatch 2.0 workflows.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: FluoroMatch Modular
  role: Runs the modular pipeline (feature detection, blank filtering, MS/MS and fragment
    screening) that feeds the scoring framework; configured with FLOW <- FALSE
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Flow 2.0
  role: Flow-mode alternative covering all steps of LC-HRMS/MS PFAS discovery data
    processing including the confidence scoring framework
- name: LipidMatch 4.2
  role: Distribution containing the FluoroMatch_Modular directory and documentation
    (FluoroMatch_Manual.docx, Trouble_Shooting_Common_Issues.docx) used to install
    and configure the pipeline
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:88506e034a99ed589e98be9abc04bee3b54e9f5479e5a37005c6b475090bb60f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  - LipidMatch
  - LipidMatch 4.2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# fluoromatch-scoring-framework-application

## Summary

Apply the FluoroMatch 2.0 scoring framework to LC-HRMS/MS feature evidence (exact mass, mass defect, homologous series, class-based MS/MS and fragment screening) to assign per-feature confidence levels for PFAS annotations. It reduces overreporting by integrating all evidence layers into a single confidence score, e.g. score A annotations in AFFF iterative-exclusion injections.

## When to use

Use when LC-HRMS/MS feature tables and MS/MS spectra (e.g. AFFF injections, mzML/raw acquisitions) have been processed through FluoroMatch Modular steps and per-feature PFAS annotation confidence levels (score A) must be assigned.

## When NOT to use

- Input is not PFAS analysis: FluoroMatch defaults to PFAS analysis only when both Lipid and TWeen_pos flags are FALSE; set these appropriately or the scoring will target the wrong analyte class.
- You only need raw feature detection/blank filtering without annotation confidence — the scoring framework is a downstream layer and requires the full screening evidence first.
- Data are from a targeted suspect-screening release of FluoroMatch (original release) rather than the truly non-targeted FluoroMatch 2.0 workflows.

## Inputs

- Four AFFF LC-HRMS/MS acquisitions (initial injection plus three iterative-exclusion rounds)
- Catalogued PFAS exact-mass/structure list (e.g. EPA CompTox Chemicals Dashboard PFAS structure list of over 8000 structures)
- FluoroMatch Modular scripts copied from the project's Scripts folder into RapidTest/LipidMatch_Libraries/Scripts/

## Outputs

- Per-feature PFAS annotations with assigned confidence levels (score A = confident)
- Round-wise counts table (e.g. iex_rounds_table.csv) of features with PFAS MS/MS evidence and score A annotations per injection round

## How to apply

Run FluoroMatch Modular (FLOW <- FALSE for manual/modular execution) on the acquired LC-HRMS/MS files, first performing feature detection and feature blank filtering, then exact mass matching to catalogued PFAS, mass defect filtering, homologous series detection, retention time pattern analysis, class-based MS/MS screening, fragment screening, and predicted MS/MS from SMILES structures. All layers of evidence are then incorporated into the scoring framework, which assigns a comprehensive confidence level to each annotation to reduce overreporting. Count per injection round how many features carry PFAS MS/MS evidence and how many reach confident (score A) level, and write these round-wise counts to a table such as iex_rounds_table.csv. In the AFFF iterative-exclusion workflow this yielded 105 PFAS MS/MS-evidence features in the initial injection rising to 399 after three IE rounds (+280%), with confident (score A) annotations improving by 139%.

## Related tools

- **FluoroMatch Modular** (Runs the modular pipeline (feature detection, blank filtering, MS/MS and fragment screening) that feeds the scoring framework; configured with FLOW <- FALSE) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Flow 2.0** (Flow-mode alternative covering all steps of LC-HRMS/MS PFAS discovery data processing including the confidence scoring framework)
- **LipidMatch 4.2** (Distribution containing the FluoroMatch_Modular directory and documentation (FluoroMatch_Manual.docx, Trouble_Shooting_Common_Issues.docx) used to install and configure the pipeline) — github.com/InnovativeOmics/Core-Match

## Examples

```
FLOW <- FALSE; ManuallyInputVariables <- FALSE; source("LipidMatch-4.2/FluoroMatch_Modular/Modular.r")
```

## Evaluation signals

- The round-wise counts table contains four round rows (initial injection plus three IE rounds) with per-round counts as exact integers (row_count_equals).
- Counts reproduce the reported trend: 105 features with PFAS MS/MS evidence in the initial injection rising to 399 after three iterative-exclusion rounds (output_matches_reference).
- Confident (score A) annotation counts per round improve by 139% relative to the initial injection, matching the reported coverage gain.
- The number of additional confident annotations decreases across subsequent IE injections, reflecting that high-abundance compounds were fragmented by the fourth injection.
- Every score A annotation is backed by the integrated evidence layers (exact mass, mass defect, homologous series, class-based and fragment MS/MS screening); annotations lacking these layers should not reach score A.

## Limitations

- Confidence scoring depends on upstream screening quality: the initial injection fragments only the highest-abundance compounds, so additional confident annotations shrink in later IE rounds.
- Overreporting risk is inherent; the comprehensive confidence level criterion exists specifically to reduce it, so skipping evidence layers will inflate counts.
- Modular configuration requires toggling Lipid/TWeen_pos parameters — misconfiguration (both not FALSE) changes the default PFAS behavior; edited Modular.r code must be manually integrated into LipidMatch-4.2\Flow\LipidMatch_Distribution and LipidMatch-4.2\FluoroMatch_Modular directories.

## Evidence
