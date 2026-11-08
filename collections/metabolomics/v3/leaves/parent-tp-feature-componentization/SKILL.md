---
name: parent-tp-feature-componentization
description: Use when confirmed parent suspect-screening hits (e.g. carbamazepine,
  benzotriazole) and mass-matched TP features must be grouped into parent/TP components
  before reporting in an NTA workflow.
when_to_use_negative:
- You only have a feature table without parent suspect screening results or TP predictions
  — componentization requires confirmed parent/TP feature pairs to link
- The goal is plain compound annotation or adduct/isotope componentization rather
  than transformation-product parent/TP grouping
- This tutorial-level NTA is not meant as a detailed overview; do not use it as a
  production TP screening protocol without tuning filters for your matrix
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: patRoon
  role: Provides the TP screening workflow, TP suspect screening, TP componentization
    (componentsTP), filter steps and HTML reporting
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Supplies the example LC-HRMS data (standard-X standard mixture and solvent-X
    blank, triplicate) used to demonstrate and validate the TP componentization workflow
  repo: https://github.com/rickhelmus/patRoonData
- name: BioTransformer
  role: In-silico prediction of TP structures for confirmed parents, which are converted
    to suspects and screened before componentization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:92fdaf023eb43f5f21646c4c4fa1bed6f6af6bfe407cbabd825257c614eb94bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - BioTransformer
  - R
  - patRoon
  - patRoon 2.0
  - patRoonData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# parent-tp-feature-componentization

## Summary

Links each confirmed parent feature with its corresponding transformation-product features and groups them into TP components using patRoon 2.0's TP componentization, the final linking step of the automated TP screening workflow.

## When to use

Use when confirmed parent suspect-screening hits (e.g. carbamazepine, benzotriazole) and mass-matched TP features must be grouped into parent/TP components before reporting in an NTA workflow.

## When NOT to use

- You only have a feature table without parent suspect screening results or TP predictions — componentization requires confirmed parent/TP feature pairs to link
- The goal is plain compound annotation or adduct/isotope componentization rather than transformation-product parent/TP grouping
- This tutorial-level NTA is not meant as a detailed overview; do not use it as a production TP screening protocol without tuning filters for your matrix

## Inputs

- Processed patRoon feature objects from the classical NTA workflow (feature finding, grouping/alignment, annotation) on LC-HRMS data in mixed positive/negative ionization mode
- Parent suspect screening results for target parents (e.g. carbamazepine, benzotriazole)
- Predicted TP structures from BioTransformer converted to suspect lists (convertToSuspects) and screened against features by mass

## Outputs

- TP components (componentsTP) grouping each parent feature with its corresponding TP features
- Interactive HTML report (report.html) of the complete TP screening workflow

## How to apply

First run the classical patRoon workflow on the patRoonData example data (standard-X standards and solvent-X blanks, positive and negative ionization): feature finding, grouping/alignment, and formula/compound annotation. Perform parent suspect screening for the target parents, then predict TP structures in silico with generateTPs("biotransformer") and screen the predicted TP suspects against detected features by mass via convertToSuspects/screenSuspects. Then generate TP components so that each parent feature is linked with its corresponding TP features and grouped in a TP component. Apply patRoon's configurable filter steps during componentization — RT direction (expected relative retention direction from chemical polarities), structure similarity, explained transformation, remove isomers, and duplicates — to prune spurious parent/TP links. Finally emit reportHTML so components are browsable in an interactive HTML report.

## Related tools

- **patRoon** (Provides the TP screening workflow, TP suspect screening, TP componentization (componentsTP), filter steps and HTML reporting) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Supplies the example LC-HRMS data (standard-X standard mixture and solvent-X blank, triplicate) used to demonstrate and validate the TP componentization workflow) — https://github.com/rickhelmus/patRoonData
- **BioTransformer** (In-silico prediction of TP structures for confirmed parents, which are converted to suspects and screened before componentization)

## Examples

```
tps <- generateTPs(parentSuspects = sus, algorithm = "biotransformer"); compsTP <- generateComponentsTP(parentSuspects = sus, tps = tps)
```

## Evaluation signals

- TP components contain one parent feature grouped with one or more mass-matched TP features; each TP feature traces back to a screened suspect from the BioTransformer prediction
- Spurious links are reduced: duplicates from different pathways removed, isomers flagged for removal, RT direction consistent with chemical polarities, explained transformation consistent with formula annotations
- Componentization runs end-to-end on patRoonData example data without errors in the scripted workflow
- report.html is generated and is a valid, browsable interactive HTML document showing the parent/TP components

## Limitations

- Isomers can be difficult or impossible to distinguish with MS, so the remove-isomers filter may discard or merge chemically distinct TPs
- TP componentization depends on upstream TP suspect screening by mass; TPs absent from the in-silico prediction or below detection cannot be linked
- patRoonData is filtered to the first ten minutes of the chromatographic run with m/z 75-450, limiting the demonstrable TP space

## Evidence

- [abstract] TP componentization groups parent with TP features: "During TP componentization (Figure 1F), each parent feature is linked with corresponding TP features and grouped in a TP component."
- [abstract] TP suspect screening matches by mass: "TP suspect screening then matches candidate TPs with detected features by mass."
- [abstract] Isomer distinguishability limit: "Remove isomers Removes isomers, which can be difficult or impossible to distinguish with MS."
- [readme] patRoonData purpose: [evidence span withheld — anchor status: external_doc]
- [intro] Workflow scope: "`patRoon` aims to provide comprehensive mass spectrometry based non-target analysis (NTA) workflows for environmental analysis."
