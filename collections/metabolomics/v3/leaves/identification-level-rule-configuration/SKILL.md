---
name: identification-level-rule-configuration
description: Use when patRoon suspect screening or formula/compound annotation results
  need automatic Schymanski et al. (2014) ID-level assignment, and rule thresholds
  (score, rank, reference standard, MS/MS match) must be inspectable or adjustable
  in YAML.
when_to_use_negative:
- Input is raw vendor LC-MS data or an unprocessed feature table — feature extraction,
  grouping and formula/compound annotation must already be done.
- The analysis is not mass spectrometry based NTA/suspect screening (e.g. non-MS omics
  data), where Schymanski identification levels do not apply.
- Levels must follow a different confidence scheme than Schymanski et al. (2014) levels
  1–5 without restating every rule in the YAML file.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: patRoon 2.0
  role: Provides genIDLevelRulesFile() to generate the YAML rule template and estimateIDConfidence()
    to apply the rules to formula and compound annotations
  repo: https://rickhelmus.github.io/patRoon
- name: R
  role: Runtime environment in which the patRoon identification-level estimation workflow
    is executed
- name: patRoonData
  role: Example LC-HRMS data (standard-X standard mixture, solvent-X blank) used to
    exercise the annotation and level-estimation workflow
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:ffc7db203a8c31b9cf4ffd240677f1b1fc34af64fcc9343f18cb56e5016ff160
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - patRoon 2.0
  - patRoonData
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# identification-level-rule-configuration

## Summary

Configure and apply patRoon 2.0's configurable, extensible rule-based approach (YAML rules file) to automatically estimate Schymanski et al. (2014) identification confidence levels (1–5) for suspect screening results from formula and compound structure annotations.

## When to use

Use when patRoon suspect screening or formula/compound annotation results need automatic Schymanski et al. (2014) ID-level assignment, and rule thresholds (score, rank, reference standard, MS/MS match) must be inspectable or adjustable in YAML.

## When NOT to use

- Input is raw vendor LC-MS data or an unprocessed feature table — feature extraction, grouping and formula/compound annotation must already be done.
- The analysis is not mass spectrometry based NTA/suspect screening (e.g. non-MS omics data), where Schymanski identification levels do not apply.
- Levels must follow a different confidence scheme than Schymanski et al. (2014) levels 1–5 without restating every rule in the YAML file.

## Inputs

- patRoon formula annotation results (formula candidates per feature)
- patRoon compound structure annotation results (candidate rankings and scores per feature)
- YAML identification-level rules file (e.g. idlevelrules.yml), generated via genIDLevelRulesFile()
- Features from LC-HRMS suspect screening (e.g. patRoonData standard-X / solvent-X data in mzML)

## Outputs

- Identification level (1–5 per Schymanski et al., 2014) per annotated feature/candidate
- identification_levels.csv with features, candidates, scores and assigned identification levels
- Generated/edited YAML rules file (idlevelrules.yml)

## How to apply

After running feature extraction, grouping/alignment and formula plus compound structure annotation in patRoon (e.g. on the patRoonData standard-X/solvent-X LC-HRMS data), call genIDLevelRulesFile() to write a template YAML file (e.g. idlevelrules.yml) containing the default rules that map annotation scores, candidate rankings and supporting data (reference standard, MS/MS match) to levels 1–5. Inspect and adjust the rule thresholds so levels follow the Schymanski et al. (2014) convention, then call estimateIDConfidence() on the annotation results with the IDFile argument pointing to the YAML file. Verify every annotated feature receives a valid level (1–5) and export features with candidates, scores and assigned levels to identification_levels.csv. The rationale is that patRoon 2.0 deliberately makes level estimation configurable and extensible so that laboratory-specific evidence requirements can be encoded rather than hard-coded.

## Related tools

- **patRoon 2.0** (Provides genIDLevelRulesFile() to generate the YAML rule template and estimateIDConfidence() to apply the rules to formula and compound annotations) — https://rickhelmus.github.io/patRoon
- **R** (Runtime environment in which the patRoon identification-level estimation workflow is executed)
- **patRoonData** (Example LC-HRMS data (standard-X standard mixture, solvent-X blank) used to exercise the annotation and level-estimation workflow) — https://github.com/rickhelmus/patRoonData

## Examples

```
idlevelrules.yml <- genIDLevelRulesFile(); levels <- estimateIDConfidence(compoundAnnotations, IDFile = "idlevelrules.yml"); write.csv(as.data.frame(levels), "identification_levels.csv")
```

## Evaluation signals

- identification_levels.csv exists and contains an identification-level column with values in the 1–5 range for all annotated candidates
- The YAML rules file (e.g. idlevelrules.yml) is present and parseable as valid YAML
- Every annotated feature in the returned annotations object carries a valid identification level
- Rank-1 candidates meeting the configured score threshold (and/or with reference standard or MS/MS match evidence) receive the expected higher levels (e.g. level 1 or 2)
- Altering a rule threshold in the YAML file changes the assigned levels in the expected direction, confirming the rules are actually applied

## Limitations

- Levels are only as good as the underlying formula and compound annotations; the rules map existing scores/rankings and do not add new evidence.
- Distinguishing isomers is difficult or impossible with MS alone, which constrains how high a level isomers can reach.
- The default rule set encodes Schymanski et al. (2014) conventions; laboratory-specific evidence types not represented in the YAML rules cannot influence the estimate without extending the rules.

## Evidence

- [abstract] Automatic estimation of identification levels (Schymanski et al., 2014) using a configurable and extensible rule based approach: "Automatic estimation of identification levels (Schymanski et al., 2014) using a configurable and extensible rule based approach"
- [readme] ID confidence table row: [evidence span withheld — anchor status: external_doc]
- [readme] vendor agnostic novel functionality: "`patRoon` is vendor agnostic and includes novel functionality, such as combining positive and negative ionization data, estimating identification
levels"
