---
name: interactive-html-report-generation
description: Use when a patRoon NTA/TP-screening workflow in R has finished and the
  resulting suspects, TP components and annotations must be delivered as a single
  interactive HTML document (report.html) for review.
when_to_use_negative:
- Input is raw vendor MS data that still needs conversion to mzML/mzXML and pre-treatment
  such as mass re-calibration — reporting presumes a completed workflow
- The analysis is not a patRoon workflow (reportHTML is patRoon-specific); other pipelines
  need their own reporting mechanisms
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: patRoon
  role: R package providing reportHTML to render the interactive HTML report of the
    TP screening workflow
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Example LC-HRMS data package whose processed workflow results are reported
    in the HTML document
  repo: https://github.com/rickhelmus/patRoonData
- name: BioTransformer
  role: In-silico TP prediction backend whose predicted TPs (via generateTPs("biotransformer"))
    appear in the reported workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:ab92e8819765d00eeb3a0cea3f40de83da8b813f8bb440b0f8c276e9cb60c65d
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# interactive-html-report-generation

## Summary

Generate a browsable interactive HTML report documenting the complete patRoon TP screening workflow (parent suspect screening, BioTransformer TP prediction, TP suspect screening, TP componentization) so results can be inspected and shared without re-running R code.

## When to use

Use when a patRoon NTA/TP-screening workflow in R has finished and the resulting suspects, TP components and annotations must be delivered as a single interactive HTML document (report.html) for review.

## When NOT to use

- Input is raw vendor MS data that still needs conversion to mzML/mzXML and pre-treatment such as mass re-calibration — reporting presumes a completed workflow
- The analysis is not a patRoon workflow (reportHTML is patRoon-specific); other pipelines need their own reporting mechanisms

## Inputs

- Processed patRoon feature objects (features, annotations) from patRoonData example data (standard-X standards and solvent-X blanks, positive and negative ionization)
- Parent suspect screening results for target parents (carbamazepine, benzotriazole)
- Predicted TP suspect results from generateTPs("biotransformer") / convertToSuspects / screenSuspects
- TP components object from componentsTP

## Outputs

- report.html — a valid, browsable interactive HTML document covering the complete TP screening workflow

## How to apply

After completing the patRoon workflow steps — feature finding, grouping/alignment and annotation on patRoonData example data, parent suspect screening (e.g., carbamazepine, benzotriazole), BioTransformer TP prediction via generateTPs("biotransformer"), TP suspect screening via convertToSuspects/screenSuspects, and TP componentization via componentsTP — call patRoon's reportHTML function to emit the interactive HTML report as report.html. The report bundles the full TP screening workflow output so parent/TP feature links and component structure remain browsable. patRoon 2.2's new reporting interface brings 'a much improved HTML interface' with many optimizations, so newer versions may expose improved reporting options. Validate the skill by confirming report.html exists and opens as an interactive document.

## Related tools

- **patRoon** (R package providing reportHTML to render the interactive HTML report of the TP screening workflow) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Example LC-HRMS data package whose processed workflow results are reported in the HTML document) — https://github.com/rickhelmus/patRoonData
- **BioTransformer** (In-silico TP prediction backend whose predicted TPs (via generateTPs("biotransformer")) appear in the reported workflow)

## Examples

```
reportHTML(fGroups, susports, compsTP = compsTP, path = "report.html")
```

## Evaluation signals

- report.html exists on disk after the script runs end-to-end
- report.html opens in a browser and is an interactive (not static/empty) HTML document
- The report includes content from all workflow stages: parent suspects, predicted/screened TPs, and TP components
- Regenerating the report after re-running the patRoon tutorial script on patRoonData yields consistent content
- No R errors are raised during the reportHTML call

## Limitations

- patRoon 2.0's reporting was substantially revised in 2.2 with 'a new reporting interface, which brings a much improved HTML interface, many optimizations and other important new functionality' — results and options differ across versions
- The documented validation is limited to the patRoonData example dataset; larger or non-example datasets may need additional reporting configuration
- No changelog found in the source article, so version-specific reporting behavior may be underdocumented

## Evidence

- [full_text] reportHTML emits the interactive HTML report: [evidence span withheld — anchor status: not_found]
- [abstract] TP componentization groups parent and TP features: "During TP componentization (Figure 1F), each parent feature is linked with corresponding TP features and grouped in a TP component."
- [readme] patRoonData example data description: [evidence span withheld — anchor status: external_doc]
- [readme] Improved HTML reporting interface in patRoon 2.2: [evidence span withheld — anchor status: external_doc]
- [abstract] TP suspect screening matches candidates by mass: "TP suspect screening then matches candidate TPs with detected features by mass."
