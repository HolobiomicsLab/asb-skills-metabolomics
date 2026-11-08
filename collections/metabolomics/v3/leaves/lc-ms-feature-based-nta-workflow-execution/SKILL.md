---
name: lc-ms-feature-based-nta-workflow-execution
description: Use when raw LC-HRMS data (e.g. standard/blank triplicates in positive
  and negative ionization) must be turned into annotated features and linked parent–transformation-product
  components using an R-based NTA workflow.
when_to_use_negative:
- Input is already a processed feature table or annotation list rather than raw/pre-treated
  mzML-mzXML LC-HRMS data
- The tutorial-style quick workflow is not meant as a detailed overview; for production
  analyses needing fine control over every algorithm parameter, consult the full patRoon
  handbook instead
- Only simple library suspect screening of known compounds is needed, with no interest
  in in-silico TP prediction or parent–TP componentization
tools:
- name: patRoon
  role: 'R package providing the complete NTA workflow interface: feature finding,
    grouping/alignment, annotation, suspect screening, generateTPs, componentsTP,
    and reportHTML'
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Supplies the example LC-HRMS standard and blank triplicate data on which the
    end-to-end workflow is demonstrated
  repo: https://github.com/rickhelmus/patRoonData
- name: BioTransformer
  role: In-silico predictor invoked through generateTPs('biotransformer') to obtain
    candidate transformation product structures for confirmed parents
- name: ProteoWizard
  role: msconvert used to pre-treat vendor MS files to open formats such as mzML/mzXML
    before feature extraction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:f7ffd3d2840927fca308911b86036abe36db72a4c265b329e448ac66356d7051
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - BioTransformer
  - ProteoWizard
  - R
  - patRoon
  - patRoon 2.0
  - patRoonData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lc-ms-feature-based-nta-workflow-execution

## Summary

Executes a complete LC-HRMS non-target analysis workflow in patRoon 2.0: feature finding on raw mzML/mzXML data, grouping/alignment across analyses, formula/compound annotation, suspect screening, and transformation product (TP) screening with BioTransformer, ending in an interactive HTML report.

## When to use

Use when raw LC-HRMS data (e.g. standard/blank triplicates in positive and negative ionization) must be turned into annotated features and linked parent–transformation-product components using an R-based NTA workflow.

## When NOT to use

- Input is already a processed feature table or annotation list rather than raw/pre-treated mzML-mzXML LC-HRMS data
- The tutorial-style quick workflow is not meant as a detailed overview; for production analyses needing fine control over every algorithm parameter, consult the full patRoon handbook instead
- Only simple library suspect screening of known compounds is needed, with no interest in in-silico TP prediction or parent–TP componentization

## Inputs

- Raw LC-HRMS data converted to mzML or mzXML (vendor formats may require ProteoWizard msconvert and optional mass re-calibration)
- patRoonData example data: triplicate standard mixture (standard-X) and blank solvent (solvent-X) in positive and negative ionization
- Parent suspect list (e.g. carbamazepine, benzotriazole)
- Blank solvent analyses for blank subtraction via the blank column in analysis information

## Outputs

- Feature table with grouped/aligned features across analyses
- Formula and compound structure annotation candidates
- BioTransformer-predicted TP structures converted to suspect lists
- TP screening results matching candidate TPs with detected features by mass
- TP components grouping each parent feature with its corresponding TP features
- Interactive HTML report (report.html) of the complete workflow

## How to apply

Load the patRoonData example data (standard-X standards and solvent-X blanks, positive and negative ionization) after converting vendor files to mzML/mzXML, then run feature finding followed by grouping and alignment across analyses in R with patRoon 2.0. Screen parent suspects such as carbamazepine and benzotriazole against the processed features, then predict TPs in silico with generateTPs('biotransformer') and convert them to a suspect list via convertToSuspects. Screen the predicted TP suspects against detected features by mass, then run TP componentization so each parent feature is grouped with its corresponding TP features into a TP component. Finish by emitting the interactive HTML report with reportHTML, and optionally apply TP filters such as the RT direction filter (verifying expected relative retention time direction from chemical polarities) and the duplicates and remove-isomers filters to prune implausible TPs. In sets workflows, feature m/z values are replaced with neutral masses calculated from adduct information and aligned/grouped across polarities, so positive and negative mode data can be processed simultaneously.

## Related tools

- **patRoon** (R package providing the complete NTA workflow interface: feature finding, grouping/alignment, annotation, suspect screening, generateTPs, componentsTP, and reportHTML) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Supplies the example LC-HRMS standard and blank triplicate data on which the end-to-end workflow is demonstrated) — https://github.com/rickhelmus/patRoonData
- **BioTransformer** (In-silico predictor invoked through generateTPs('biotransformer') to obtain candidate transformation product structures for confirmed parents)
- **ProteoWizard** (msconvert used to pre-treat vendor MS files to open formats such as mzML/mzXML before feature extraction)

## Examples

```
tps <- generateTPs(parents, algorithm = "biotransformer"); suspects <- convertToSuspects(tps); tpsScr <- screenSuspects(featSets, suspects); compsTP <- componentsTP(featSets, parentsScr, tpsScr); reportHTML(featSets, parents = parentsScr, TPs = tpsScr, components = compsTP)
```

## Evaluation signals

- The R script runs end-to-end on the patRoonData example data without errors
- report.html exists and is a valid, browsable interactive HTML document
- Parent suspect screening confirms the target parents carbamazepine and benzotriazole in the standard mixture
- Each generated TP component links a parent feature with at least one corresponding TP feature grouped by mass and retention relationship
- Blank subtraction against the solvent-X triplicate removes features present in blanks

## Limitations

- patRoonData demo data is truncated: only the first ten minutes of the chromatographic run with m/z 75–450 are present, so full-range results differ
- TP candidates may include isomers that are difficult or impossible to distinguish with MS, and duplicates formed from the same parent via different pathways, requiring the remove-isomers and duplicates filters
- Feature m/z values in sets workflows are replaced with neutral masses from adduct information, so mis-annotated adducts propagate into TP screening

## Evidence

- [abstract] TP suspect screening matches candidate TPs with detected features by mass: "TP suspect screening then matches candidate TPs with detected features by mass."
- [abstract] TP componentization groups parent and TP features: "During TP componentization (Figure 1F), each parent feature is linked with corresponding TP features and grouped in a TP component."
- [intro] patRoon's scope as an NTA platform: "`patRoon` aims to provide comprehensive mass spectrometry based non-target analysis (NTA) workflows for environmental analysis."
- [intro] patRoonData example dataset composition: "The example dataset contains LC-MS data for a standard mixture with known composition ('standard-X') and a blank solvent ('solvent-X'), both in triplicate"
- [readme] README on patRoonData content and filtering: [evidence span withheld — anchor status: external_doc]
- [readme] README on blank utility of solvent analyses: [evidence span withheld — anchor status: external_doc]
