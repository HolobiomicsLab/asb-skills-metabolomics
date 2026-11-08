---
name: pfas-annotation-confidence-scoring
description: Use when you have blank-filtered LC-HRMS/MS feature lists from PFAS suspect
  samples (e.g. AFFF) and need to assign each feature an annotation-confidence level
  before reporting compounds.
when_to_use_negative:
- You already have confirmed compound identities with library MS/MS matches — the
  scoring framework is for candidate features needing confidence grading, not for
  re-verifying annotations
- Your data are not LC-HRMS/MS feature detections amenable to blank filtering, mass
  defect filtering, and CF2 homologous-series detection (e.g. targeted single-analyte
  assays)
- Only an empty/small feature list without blank controls exists — confidence scoring
  integrates multiple evidence layers that cannot be built without sample/blank contrast
  and MS/MS or series evidence
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FluoroMatch 2.0
  role: Automates the full non-targeted PFAS identification workflow whose output
    feeds the confidence-level scoring framework
- name: FluoroMatch Flow 2.0
  role: End-to-end pipeline (feature detection, blank filtering, exact mass matching,
    mass defect filtering, homologous series detection) generating the evidence layers
    scored per feature
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular scripts (copy root Scripts folder into RapidTest/LipidMatch_Libraries/Scripts/)
    for running individual scoring/evidence steps; configured with FLOW <- FALSE
  repo: github.com/InnovativeOmics/Core-Match
- name: LipidMatch
  role: Software distribution containing the FluoroMatch_Modular and Flow directories
    whose scripts must be installed before scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:09048465894f8385672afe8901a13030ec9489a78bed477590b3b55bf4ebccfd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  - LipidMatch
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pfas-annotation-confidence-scoring

## Summary

Applies FluoroMatch 2.0's comprehensive confidence-level scoring framework to candidate PFAS features, integrating layers of evidence (exact mass match, mass defect, homologous series, MS/MS) to grade annotation certainty and reduce overreporting.

## When to use

Use when you have blank-filtered LC-HRMS/MS feature lists from PFAS suspect samples (e.g. AFFF) and need to assign each feature an annotation-confidence level before reporting compounds.

## When NOT to use

- You already have confirmed compound identities with library MS/MS matches — the scoring framework is for candidate features needing confidence grading, not for re-verifying annotations
- Your data are not LC-HRMS/MS feature detections amenable to blank filtering, mass defect filtering, and CF2 homologous-series detection (e.g. targeted single-analyte assays)
- Only an empty/small feature list without blank controls exists — confidence scoring integrates multiple evidence layers that cannot be built without sample/blank contrast and MS/MS or series evidence

## Inputs

- Vendor LC-HRMS/MS sample files (e.g. legacy AFFF holding-tank sample) plus associated solvent blank runs
- EPA CompTox Chemicals Dashboard PFAS structure list (over 8000 PFAS structures)
- Candidate feature table from feature detection and blank filtering (e.g. 3740 rows)
- SMILES structures for predicted MS/MS

## Outputs

- Per-feature annotated output table with multi-layer evidence and confidence-level score (e.g. score E fraction)
- Stage-by-stage feature count-reduction table (total features → blank-filtered → potential PFAS → probable PFAS)
- Score distribution across candidate features (e.g. 39% scored E)

## How to apply

First reduce the raw feature list with FluoroMatch Flow 2.0 stages: feature detection, feature blank filtering against solvent blanks, exact mass matching to the EPA CompTox Chemicals Dashboard PFAS structure list (over 8000 structures), mass defect filtering, and CF2-based homologous series detection. Then build per-feature evidence layers via retention time pattern analysis, class-based MS/MS screening, fragment screening, and predicted MS/MS from SMILES structures. Finally apply the comprehensive confidence-level criterion, which integrates all evidence layers into a single score (e.g. in the legacy AFFF application, 39% of the 3740 blank-filtered features scored E). Compile the per-feature annotated output table so every reported PFAS carries its confidence grade, and count features at each reduction stage (e.g. 31132 → 3740 → 2290 potential PFAS → 1238 probable PFAS).

## Related tools

- **FluoroMatch 2.0** (Automates the full non-targeted PFAS identification workflow whose output feeds the confidence-level scoring framework)
- **FluoroMatch Flow 2.0** (End-to-end pipeline (feature detection, blank filtering, exact mass matching, mass defect filtering, homologous series detection) generating the evidence layers scored per feature) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular scripts (copy root Scripts folder into RapidTest/LipidMatch_Libraries/Scripts/) for running individual scoring/evidence steps; configured with FLOW <- FALSE) — github.com/InnovativeOmics/Core-Match
- **LipidMatch** (Software distribution containing the FluoroMatch_Modular and Flow directories whose scripts must be installed before scoring)

## Examples

```
cp Scripts/* RapidTest/LipidMatch_Libraries/Scripts/ && Rscript Modular.r  # with FLOW <- FALSE; ManuallyInputVariables <- FALSE
```

## Evaluation signals

- Filtered feature table contains the expected row count (e.g. exactly 3740 blank-filtered features from 31132 total)
- Stage-wise counts match published reduction (31132 → 3740 → 2290 potential PFAS → 1238 probable PFAS; 96% filtered out)
- Score distribution reproduces the reported fraction (39% of blank-filtered features scored E)
- Every reported PFAS row carries an explicit confidence-level grade and its underlying evidence flags (homologous-series and/or fluorine-fragment evidence)
- Total number of likely PFAS reported is consistent with the published count (over one thousand for the AFFF application)

## Limitations

- High confidence-score fraction can still include uncertain annotations — the criterion exists precisely to reduce overreporting, so low grades (e.g. E) must be reported as such
- Scoring quality depends on the completeness of the reference structure list (EPA CompTox PFAS list, >8000 structures); PFAS absent from it may be misgraded
- Requires solvent blanks and MS/MS (or SMILES-predicted MS/MS) inputs; without these, evidence layers and confidence grades cannot be fully constructed

## Evidence
