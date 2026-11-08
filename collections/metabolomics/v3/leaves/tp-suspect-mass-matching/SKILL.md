---
name: tp-suspect-mass-matching
description: Use when in-silico TPs (e.g. BioTransformer predictions for carbamazepine
  or benzotriazole) must be matched to detected features by mass, and parent/TP features
  grouped into components.
when_to_use_negative:
- No processed feature list exists yet — TP suspect screening requires features from
  a completed classical patRoon workflow, not raw vendor files
- Parents are not confirmed/predicted — mass matching presupposes TP structures from
  prediction (generateTPs) or a TP library
- The goal is general feature annotation without parent/TP relationships — plain suspect
  screening or formula/compound annotation is the appropriate tool
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: patRoon 2.0
  role: R platform providing TP suspect screening (convertToSuspects/screenSuspects),
    TP componentization and HTML reporting
  repo: https://github.com/rickhelmus/patRoon
- name: BioTransformer
  role: In-silico prediction of TP structures for confirmed parents, feeding the suspect
    list to be mass-matched
- name: patRoonData
  role: Example LC-HRMS data (standard-X standards, solvent-X blanks) used to run
    and validate the TP screening workflow
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:379a0e86de928cb993192e377c520cffed6f8050110824525e467d949dacb1ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - BioTransformer
  - R
  - patRoon 2.0
  - patRoonData
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tp-suspect-mass-matching

## Summary

Matches predicted transformation product (TP) structures with detected LC-HRMS features by mass within patRoon 2.0's TP screening workflow, then groups each parent feature with its corresponding TP features into TP components for reporting.

## When to use

Use when in-silico TPs (e.g. BioTransformer predictions for carbamazepine or benzotriazole) must be matched to detected features by mass, and parent/TP features grouped into components.

## When NOT to use

- No processed feature list exists yet — TP suspect screening requires features from a completed classical patRoon workflow, not raw vendor files
- Parents are not confirmed/predicted — mass matching presupposes TP structures from prediction (generateTPs) or a TP library
- The goal is general feature annotation without parent/TP relationships — plain suspect screening or formula/compound annotation is the appropriate tool

## Inputs

- Aligned feature table from a patRoon classical NTA workflow (positive and negative ionization sets)
- Parent suspect screening results (e.g. carbamazepine, benzotriazole) or a target list of parents
- In-silico TP predictions from generateTPs (BioTransformer, enviPath, CTS or library data)

## Outputs

- TP suspect screening results linking predicted TP structures to detected features by mass
- TP components (parent feature grouped with corresponding TP features) from componentsTP
- Interactive HTML report (report.html) of the complete TP screening workflow

## How to apply

Start from a classical patRoon workflow producing aligned features from mixed positive/negative ionization LC-HRMS data (patRoonData standard-X standards vs solvent-X blanks). Convert predicted TP structures into a suspect list (e.g. output of generateTPs('biotransformer') passed through convertToSuspects) and run TP suspect screening (screenSuspects), which 'matches candidate TPs with detected features by mass'. Retained parent/TP feature pairs can then be pruned with patRoon's TP filter steps (RT direction, structure similarity, explained transformation, duplicates, remove isomers). Finally run TP componentization (generateComponentsTP) so each parent feature is linked with corresponding TP features and grouped in a TP component, suitable for reportHTML output. Rationale: mass matching alone is not definitive, hence patRoon couples it with MS similarity and parent/TP feature classification filters before componentization.

## Related tools

- **patRoon 2.0** (R platform providing TP suspect screening (convertToSuspects/screenSuspects), TP componentization and HTML reporting) — https://github.com/rickhelmus/patRoon
- **BioTransformer** (In-silico prediction of TP structures for confirmed parents, feeding the suspect list to be mass-matched)
- **patRoonData** (Example LC-HRMS data (standard-X standards, solvent-X blanks) used to run and validate the TP screening workflow) — https://github.com/rickhelmus/patRoonData

## Examples

```
tps <- generateTPs(parentSuspects, algorithm="biotransformer"); tpSuspects <- convertToSuspects(tps); fPairs <- screenSuspects(fObjects, tpSuspects); compsTP <- generateComponentsTP(fObjects, fPairs, parents)
```

## Evaluation signals

- Suspect screening produces feature-TP matches whose measured m/z correspond to the predicted TP neutral masses within the configured mass tolerance
- generateComponentsTP returns TP components in which each confirmed parent feature is grouped with its corresponding TP features
- The script runs end-to-end on patRoonData example data and report.html exists and is a valid, browsable interactive HTML document
- TP filter steps (RT direction, duplicates, remove isomers, explained transformation) do not remove chemically expected parent/TP pairs from the example workflow

## Limitations

- Mass matching alone cannot resolve TPs that are isomers of the parent or of each other — patRoon provides a 'remove isomers' filter because isomers 'can be difficult or impossible to distinguish with MS'
- Duplicate TPs formed from the same parent through different pathways must be removed via the duplicates filter
- The patRoonData demo data are filtered to the first ten minutes of the chromatographic run and m/z 75-450, so TP matches are limited to that window

## Evidence

- [abstract] TP suspect screening then matches candidate TPs with detected features by mass.: "TP suspect screening then matches candidate TPs with detected features by mass."
- [abstract] During TP componentization, each parent feature is linked with corresponding TP features and grouped in a TP component.: "During TP componentization (Figure 1F), each parent feature is linked with corresponding TP features and grouped in a TP component."
- [abstract] Parents for (B) are specified from (1) a target list, (2) results of a suspect screening to find parents by mass, or (3) candidates of feature compound annotation: "Parents for (B) are specified from (1) a target list, (2) results of a suspect screening to find parents by mass, or (3) candidates of feature compound annotation"
- [readme] Suspect screening: Finding features with suspected presence by MS and chromatographic data.: [evidence span withheld — anchor status: external_doc]
