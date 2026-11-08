---
name: biotransformer-tp-structure-prediction
description: Use when confirmed parent suspects (e.g., carbamazepine, benzotriazole)
  from LC-HRMS suspect screening need candidate transformation product structures
  before TP suspect screening and componentization.
when_to_use_negative:
- You only need parent suspect screening or classical NTA feature finding/annotation
  without transformation products
- A curated experimental TP library exists and in-silico prediction is unnecessary
  (patRoon also supports library-based TP screening)
- The data is not mass-spectrometry based or lacks chromatographic feature information
  to link predicted TPs to
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0121
tools:
- name: patRoon
  role: Wraps BioTransformer via its predictTPs/generateTPs interface and feeds predicted
    TPs into TP suspect screening and componentization
  repo: https://github.com/rickhelmus/patRoon
- name: BioTransformer
  role: In-silico engine that predicts TP structures for the confirmed parent compounds
- name: patRoonData
  role: Provides the example LC-HRMS data (standard-X, solvent-X) on which the parent
    screening feeding TP prediction is demonstrated
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:9b45d0888a0454493c47e9974c55540423835e420118be622edd85df51bf6d8e
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# biotransformer-tp-structure-prediction

## Summary

In-silico prediction of transformation product (TP) structures for confirmed parent compounds using BioTransformer through patRoon's generateTPs interface, providing the candidate TP suspects that feed patRoon 2.0's automated TP screening workflow.

## When to use

Use when confirmed parent suspects (e.g., carbamazepine, benzotriazole) from LC-HRMS suspect screening need candidate transformation product structures before TP suspect screening and componentization.

## When NOT to use

- You only need parent suspect screening or classical NTA feature finding/annotation without transformation products
- A curated experimental TP library exists and in-silico prediction is unnecessary (patRoon also supports library-based TP screening)
- The data is not mass-spectrometry based or lacks chromatographic feature information to link predicted TPs to

## Inputs

- Confirmed parent compound structures (e.g., carbamazepine, benzotriazole) from patRoon suspect screening results, a target list, or feature compound annotation candidates
- Processed patRoon features from LC-HRMS data (patRoonData standard-X / solvent-X, positive and negative ionization)

## Outputs

- Predicted TP structures with associated formulas (candidates for TP suspect screening)
- TP suspect list convertible via convertToSuspects for screenSuspects matching against detected features by mass

## How to apply

First complete the classical patRoon NTA steps on the patRoonData example data (standard-X standards and solvent-X blanks, positive and negative ionization) and confirm parents via suspect screening on the processed features. Then call patRoon's predictTPs interface with algorithm set to BioTransformer (generateTPs(..., algorithm = "biotransformer")) on the confirmed parent structures to obtain predicted TP structures and formulas. BioTransformer is one of several in-silico TP predictors patRoon wraps — enviPath and CTS are alternatives — but all are accessed through the same consistent interface, removing the need for manual data conversions. The resulting TP structures are converted to suspects with convertToSuspects and screened against detected features by mass via screenSuspects, then parents and TP features are grouped into TP components. Rationale: automated in-silico prediction avoids relying on curated TP libraries, which may not cover novel or emerging contaminants.

## Related tools

- **patRoon** (Wraps BioTransformer via its predictTPs/generateTPs interface and feeds predicted TPs into TP suspect screening and componentization) — https://github.com/rickhelmus/patRoon
- **BioTransformer** (In-silico engine that predicts TP structures for the confirmed parent compounds)
- **patRoonData** (Provides the example LC-HRMS data (standard-X, solvent-X) on which the parent screening feeding TP prediction is demonstrated) — https://github.com/rickhelmus/patRoonData

## Examples

```
tps <- generateTPs(parentSuspects, algorithm = "biotransformer")  # then suspects <- convertToSuspects(tps); scr <- screenSuspects(features, suspects)
```

## Evaluation signals

- generateTPs returns predicted TP structures for every confirmed parent (carbamazepine and benzotriazole) without errors
- Downstream convertToSuspects/screenSuspects matches predicted TPs to detected features by mass, producing non-empty TP screening results
- componentsTP links each parent feature with corresponding TP features into TP components
- reportHTML emits a browsable interactive report.html covering the complete TP screening workflow
- The script runs end-to-end on patRoonData example data in both positive and negative ionization modes

## Limitations

- In-silico TP prediction yields candidate structures only; candidates still require mass-based screening, filtering (RT direction, structure similarity, explained transformation, duplicates, isomers) and feature linking before they can be considered detected
- BioTransformer is one of several wrapped predictors (enviPath, CTS); predicted TP sets differ between algorithms, and patRoon 2.0 supports complementary TP screening approaches to compensate
- Example patRoonData files are truncated (first ten minutes, m/z 75-450), so TPs outside that range cannot be recovered in the demo

## Evidence

- [abstract] BioTransformer as in-silico TP structure source: [evidence span withheld — anchor status: not_found]
- [abstract] TP suspect screening consumes predicted TPs: "TP suspect screening then matches candidate TPs with detected features by mass."
- [abstract] Parent candidates can come from suspect screening or annotation: "Parents for (B) are specified from (1) a target list, (2) results of a suspect screening to find parents by mass, or (3) candidates of feature compound annotation"
- [intro] Example data used for the demonstrated workflow: "we will use example data provided within the [patRoonData](https://github.com/rickhelmus/patRoonData) package"
- [readme] patRoonData purpose: [evidence span withheld — anchor status: external_doc]
