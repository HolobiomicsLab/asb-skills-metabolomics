---
name: schymanski-identification-level-assignment
description: Use when patRoon suspect screening or formula/compound annotation results
  (e.g. from MetFrag/SIRIUS/GenForm on LC-HRMS features) need automatic, reproducible
  Schymanski 1-5 confidence level assignment.
when_to_use_negative:
- When identification levels are already assigned manually or curated by an expert
  — re-estimation would overwrite curation
- When the analysis does not involve patRoon formula/compound annotation objects (the
  rules are applied to patRoon annotation data structures)
- When the goal is transformation product (TP) screening or prioritization rather
  than confidence-level annotation of suspects
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: patRoon 2.0
  role: Provides genIDLevelRulesFile() and estimateIDConfidence() with the native
    ID confidence module implementing the configurable rule-based identification level
    estimation
  repo: https://github.com/rickhelmus/patRoon
- name: R
  role: Execution environment in which the patRoon annotation objects are loaded,
    the YAML rule file is generated/edited, and identification levels are estimated
    and exported
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:d55ff0ca6c5ba9342a73842282858f74c6c9e8475458ce87aac40fc25ef8494c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - R
  - patRoon 2.0
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# schymanski-identification-level-assignment

## Summary

Automatically assign Schymanski et al. (2014) identification levels (1-5) to suspect screening annotations in patRoon 2.0 using a configurable, extensible YAML rule file that maps annotation scores, candidate rankings and supporting data to confidence levels.

## When to use

Use when patRoon suspect screening or formula/compound annotation results (e.g. from MetFrag/SIRIUS/GenForm on LC-HRMS features) need automatic, reproducible Schymanski 1-5 confidence level assignment.

## When NOT to use

- When identification levels are already assigned manually or curated by an expert — re-estimation would overwrite curation
- When the analysis does not involve patRoon formula/compound annotation objects (the rules are applied to patRoon annotation data structures)
- When the goal is transformation product (TP) screening or prioritization rather than confidence-level annotation of suspects

## Inputs

- patRoon formula annotation results (e.g. from GenForm or SIRIUS) for detected features
- patRoon compound structure annotation results (e.g. from MetFrag or SIRIUS) with candidate rankings and scores
- YAML identification-level rules file (e.g. idlevelrules.yml), generated via genIDLevelRulesFile() and optionally edited
- Suspect screening results from patRoon features (mzML-derived, e.g. patRoonData standard-X/solvent-X data)

## Outputs

- Annotated features with an identification level column containing Schymanski et al. (2014) levels 1-5
- Generated/curated YAML rules file mapping scores and rankings to identification levels
- Table of features with candidates, scores and assigned identification levels (e.g. identification_levels.csv)

## How to apply

Load the formula and compound structure annotation results generated for the features of a patRoon workflow (e.g. the patRoonData standard-X / solvent-X LC-HRMS triplicate data). Call genIDLevelRulesFile() to generate a template YAML file containing the default rule set that maps annotation scores, candidate rankings and supporting data (e.g. presence of a reference standard, MS/MS match) to identification levels. Inspect and, if needed, adjust rule thresholds so levels follow the Schymanski et al. (2014) convention, then run estimateIDConfidence() with the annotation results and the YAML file (e.g. idlevelrules.yml) to assign a level to every annotated feature. The rule-based approach is configurable and extensible, so it can be tuned to project-specific confidence criteria rather than relying on hard-coded cutoffs.

## Related tools

- **patRoon 2.0** (Provides genIDLevelRulesFile() and estimateIDConfidence() with the native ID confidence module implementing the configurable rule-based identification level estimation) — https://github.com/rickhelmus/patRoon
- **R** (Execution environment in which the patRoon annotation objects are loaded, the YAML rule file is generated/edited, and identification levels are estimated and exported)

## Examples

```
rulesFile <- genIDLevelRulesFile(); annotations <- estimateIDConfidence(annotations, IDFile = "idlevelrules.yml")
```

## Evaluation signals

- identification_levels.csv exists and contains an identification-level column with values restricted to the 1-5 range for all annotated candidates
- The generated YAML rules file (e.g. idlevelrules.yml) is present and parseable as valid YAML
- Every annotated feature in the returned annotations object carries a valid identification level (no NA/missing levels for formula or compound candidates)
- Assigned levels are consistent with the rule thresholds in the YAML file (e.g. rank-1 candidates meeting minimum score thresholds receive higher confidence levels)
- Suspect features found only in the blank solvent-X data (after blank subtraction) should not carry confident identification levels

## Limitations

- Levels are estimated from annotation scores, candidate rankings and supporting data — they are only as reliable as the underlying formula/compound annotations (GenForm, SIRIUS, MetFrag)
- Default rules are templates; without inspecting and adjusting thresholds (e.g. minimum score for a rank-1 candidate, reference standard or MS/MS match presence) the Schymanski convention may not be faithfully reproduced
- Suspect screening relies on MS and chromatographic data; isomers that are difficult or impossible to distinguish with MS can undermine confident level assignment

## Evidence

- [abstract] Improved suspect screening includes automatic estimation of identification levels using a configurable rule-based approach: "Automatic estimation of identification levels (Schymanski et al., 2014) using a configurable and extensible rule based approach"
- [readme] ID confidence is native patRoon functionality: [evidence span withheld — anchor status: external_doc]
- [readme] patRoon includes estimating identification levels as novel functionality: "`patRoon` is vendor agnostic and includes novel functionality, such as combining positive and negative ionization data, estimating identification
levels"
- [readme] patRoonData provides the standard-X and solvent-X LC-HRMS example data: [evidence span withheld — anchor status: external_doc]
- [readme] Suspect screening finds features with suspected presence by MS and chromatographic data: [evidence span withheld — anchor status: external_doc]
