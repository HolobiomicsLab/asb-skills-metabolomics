---
name: parent-suspect-screening
description: Use when you have patRoon feature groups from classical NTA feature finding
  and a target list of parent compounds whose transformation products you want to
  predict and screen in a patRoon 2.0 TP workflow.
when_to_use_negative:
- You already have confirmed parents and want TP results — use TP prediction (generateTPs)
  and TP suspect screening instead; this skill only establishes the parents.
- You are screening predicted transformation products rather than known parents —
  use patRoon's TP suspect screening (convertToSuspects of predicted TPs), not the
  parent suspect screening step.
- Your data has no blank replicates to subtract, since this workflow relies on blank
  subtraction via the blank column in analysis information.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: patRoon 2.0
  role: Provides the suspect screening functionality (convertToSuspects/screenSuspects)
    that matches parent suspects to detected features by MS and chromatographic data
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Supplies the example raw LC-HRMS data (standard-X standards, solvent-X blanks)
    on which the parent suspect screening is demonstrated
  repo: https://github.com/rickhelmus/patRoonData
- name: BioTransformer
  role: 'Downstream consumer of the confirmed parents: predicts transformation products
    via patRoon''s predictTPs interface after parent screening'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:c14530f2522e0174800d4acbe876be708c7a80704e2b74a0880b7e9c7831e278
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - BioTransformer
  - R
  - patRoon 2.0
  - patRoonData
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# parent-suspect-screening

## Summary

Screens processed LC-HRMS features for known parent compounds (e.g., carbamazepine, benzotriazole) using patRoon's suspect screening, matching suspects to detected features by mass and chromatographic data to obtain confirmed parents for downstream TP prediction.

## When to use

Use when you have patRoon feature groups from classical NTA feature finding and a target list of parent compounds whose transformation products you want to predict and screen in a patRoon 2.0 TP workflow.

## When NOT to use

- You already have confirmed parents and want TP results — use TP prediction (generateTPs) and TP suspect screening instead; this skill only establishes the parents.
- You are screening predicted transformation products rather than known parents — use patRoon's TP suspect screening (convertToSuspects of predicted TPs), not the parent suspect screening step.
- Your data has no blank replicates to subtract, since this workflow relies on blank subtraction via the blank column in analysis information.

## Inputs

- Processed patRoon feature groups (from feature finding, grouping/alignment on LC-HRMS data)
- patRoonData example raw LC-HRMS data (standard-X standards and solvent-X blanks, positive and negative ionization, mzML/mzXML)
- Parent suspect list with structures (e.g., Carbamazepine, Benzotriazole)

## Outputs

- Screened parent suspect results (features matched to carbamazepine and benzotriazole by mass and chromatographic data)
- Confirmed parent structures/SMILES for in silico TP prediction (BioTransformer via predictTPs)
- Background-subtracted suspect hits (blank subtraction against solvent-X triplicates)

## How to apply

Start from feature groups produced by a classical patRoon NTA workflow (feature finding, grouping/alignment, and annotation) on the patRoonData example data (standard-X standards and solvent-X blanks in positive and negative ionization). Assemble a parent suspect list containing the target structures (e.g., Carbamazepine and Benzotriazole) and convert it with patRoon's convertToSuspects interface. Run patRoon's suspect screening (screenSuspects), which finds features with suspected presence by MS and chromatographic data, using blank subtraction via the blank column in analysis information (solvent-X triplicates). The resulting screened parents are the confirmed parental structures that are subsequently fed to in silico TP prediction (e.g., generateTPs with BioTransformer), so retention time and mass plausibility of the matches matter before continuing.

## Related tools

- **patRoon 2.0** (Provides the suspect screening functionality (convertToSuspects/screenSuspects) that matches parent suspects to detected features by MS and chromatographic data) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Supplies the example raw LC-HRMS data (standard-X standards, solvent-X blanks) on which the parent suspect screening is demonstrated) — https://github.com/rickhelmus/patRoonData
- **BioTransformer** (Downstream consumer of the confirmed parents: predicts transformation products via patRoon's predictTPs interface after parent screening)

## Examples

```
suspects <- convertToSuspects(parentList); fGroupsScr <- screenSuspects(fGroups, suspects, blankThreshold=5); tps <- generateTPs(fGroupsScr, algorithm="biotransformer")
```

## Evaluation signals

- The script runs end-to-end on the patRoonData example data without errors, producing screened parents for both Carbamazepine and Benzotriazole.
- Suspect screening matches features only where the suspect mass corresponds to a detected feature (finding features with suspected presence by MS and chromatographic data).
- Blank subtraction using the solvent-X blank column removes background features from the suspect hits.
- Downstream generateTPs("biotransformer") succeeds only with valid confirmed parents, confirming the parent screening output is usable.
- The final report.html exists and is a valid, browsable interactive HTML document covering the complete workflow.

## Limitations

- patRoonData example data was filtered to only contain the first ten minutes of the chromatographic run with m/z 75-450, so parents/fragments outside that range cannot be demonstrated.
- Suspect screening finds features by MS and chromatographic data only; identification confidence relies on patRoon's rule-based estimation of identification levels rather than full structural confirmation.
- The tutorial is a simple NTA demonstration and 'is not meant to give a detailed overview of patRoon', so parameter tuning beyond the example is not covered.

## Evidence

- [readme] Suspect screening finds features by mass and chromatographic data: [evidence span withheld — anchor status: external_doc]
- [abstract] TP suspect screening matches candidates with detected features by mass: "TP suspect screening then matches candidate TPs with detected features by mass."
- [intro] Blank subtraction uses solvent blanks in analysis information: "the solvents should be used for blank subtraction"
- [readme] patRoonData provides standard and blank LC-HRMS data for the tutorial: [evidence span withheld — anchor status: external_doc]
- [other] Parent suspect screening step precedes BioTransformer TP prediction in the runnable example: [evidence span withheld — anchor status: not_found]
