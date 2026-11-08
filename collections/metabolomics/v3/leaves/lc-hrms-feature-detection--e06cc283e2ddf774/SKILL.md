---
name: lc-hrms-feature-detection
description: Use when raw LC-HRMS/MS vendor files (e.g. AFFF foam samples plus solvent
  blanks) must be converted into a detected feature list as the entry point for PFAS
  non-targeted screening.
when_to_use_negative:
- Input is already a blank-filtered or annotated feature table — skip to feature blank
  filtering or the confidence-scoring stage instead
- Data are direct-infusion or non-chromatographic MS lacking the retention-time dimension
  FluoroMatch Flow 2.0 expects for LC-HRMS/MS PFAS processing
- The goal is targeted quantification of a few known PFAS (e.g. PFOA/PFOS only) rather
  than non-targeted feature discovery
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: FluoroMatch Flow 2.0
  role: Executes the full data-processing pipeline whose first stage is feature detection
    of LC-HRMS/MS samples for PFAS discovery
  repo: https://github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Standalone modular mode of the same codebase (FLOW <- FALSE) for running feature
    detection outside the full Flow pipeline
  repo: https://github.com/InnovativeOmics/Core-Match
- name: LipidMatch 4.2
  role: Distribution package whose directories (e.g. LipidMatch-4.2\Flow\LipidMatch_Distribution,
    FluoroMatch_Modular) host the detection scripts and manuals
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:bc88ec5421d3be81043c0b032e54fdbae3a46d09d29e6021bcb0dd5927123cb5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/InnovativeOmics/Core-Match
  tools:
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  - LipidMatch
  - LipidMatch 4.2
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# LC-HRMS feature detection

## Summary

Performs the first stage of the FluoroMatch Flow 2.0 non-targeted PFAS pipeline: detecting chromatographic/mass features from LC-HRMS/MS vendor files, producing the full feature list (e.g. 31,132 features for the legacy AFFF sample) that all downstream filtering and annotation stages operate on.

## When to use

Use when raw LC-HRMS/MS vendor files (e.g. AFFF foam samples plus solvent blanks) must be converted into a detected feature list as the entry point for PFAS non-targeted screening.

## When NOT to use

- Input is already a blank-filtered or annotated feature table — skip to feature blank filtering or the confidence-scoring stage instead
- Data are direct-infusion or non-chromatographic MS lacking the retention-time dimension FluoroMatch Flow 2.0 expects for LC-HRMS/MS PFAS processing
- The goal is targeted quantification of a few known PFAS (e.g. PFOA/PFOS only) rather than non-targeted feature discovery

## Inputs

- LC-HRMS/MS vendor data files for samples (e.g. legacy AFFF 1999 holding-tank sample)
- LC-HRMS/MS vendor data files for blanks (e.g. three solvent blanks)
- FluoroMatch Flow 2.0 / Modular project directory with Scripts folder and libraries

## Outputs

- Full detected feature list (~31,132 features for the AFFF sample) consumed by downstream blank filtering and PFAS scoring stages
- Feature table handed to exact mass matching, mass defect filtering, and homologous series detection

## How to apply

Install the FluoroMatch/LipidMatch distribution from innovativeomics.com and, if editing algorithms, clone the InnovativeOmics/Core-Match repo and copy the root Scripts contents into RapidTest/LipidMatch_Libraries/Scripts/. In Modular.r, choose the deployment mode: set FLOW <- TRUE for the full Flow pipeline (or FALSE for modular use), keep csvInput and ManuallyInputVariables as desired, and ensure Lipid and TWeen_pos are FALSE (or both FALSE) so the run defaults to PFAS analysis. Load the sample vendor files together with the accompanying solvent blanks (e.g. the legacy 1999 AFFF holding-tank sample and its three solvent blanks) and run feature detection to yield the complete feature list. This feature list is the input to subsequent stages — feature blank filtering, exact mass matching, mass defect filtering, and homologous series detection — so detection completeness directly controls what later PFAS evidence layers can recover.

## Related tools

- **FluoroMatch Flow 2.0** (Executes the full data-processing pipeline whose first stage is feature detection of LC-HRMS/MS samples for PFAS discovery) — https://github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Standalone modular mode of the same codebase (FLOW <- FALSE) for running feature detection outside the full Flow pipeline) — https://github.com/InnovativeOmics/Core-Match
- **LipidMatch 4.2** (Distribution package whose directories (e.g. LipidMatch-4.2\Flow\LipidMatch_Distribution, FluoroMatch_Modular) host the detection scripts and manuals)

## Examples

```
Rscript Modular.r  # with Modular.r set: FLOW <- TRUE; csvInput <- FALSE; ManuallyInputVariables <- FALSE; Lipid <- FALSE; TWeen_pos <- FALSE
```

## Evaluation signals

- The detected feature list reproduces the published scale (~31,132 features for the legacy AFFF sample before any filtering)
- Feature detection is run on both samples and the three solvent blanks so that downstream feature blank filtering can retain 3,740 features
- Modular.r flags are set correctly for PFAS analysis (Lipid and TWeen_pos FALSE, or both FALSE defaulting to PFAS)
- Downstream stage counts remain traceable: 31,132 total → 3,740 blank-filtered → 2,290 potential PFAS → 1,238 probable PFAS (96% reduction)
- Edited scripts (genEIC.r, MS1Spectragen.r, Stats.R) are placed in LipidMatch_Libraries\Scripts and Modular.r in both FluoroMatch_Modular and LipidMatch_Distribution

## Limitations

- The user must install the stable release from innovativeomics.com for production use; the GitHub Core-Match repo is for developer-edited algorithms that must be manually integrated into the distribution directories
- Detection must include the solvent blanks in the same run, otherwise the blank-filtering stage cannot reduce the feature list as designed
- Common setup issues are deferred to FluoroMatch_Modular\Trouble_Shooting_Common_Issues.docx and the Flow/Modular manuals rather than being resolved in code

## Evidence
