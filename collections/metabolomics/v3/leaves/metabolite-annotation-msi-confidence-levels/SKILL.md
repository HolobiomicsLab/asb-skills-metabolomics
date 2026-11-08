---
name: metabolite-annotation-msi-confidence-levels
description: Use when LC-MS untargeted metabolomics features (e.g. a cleaned mass_dataset
  object from tidyMass) need metabolite identification against curated in-house databases
  with standardized MSI confidence-level reporting.
when_to_use_negative:
- The input is still raw LC-MS data rather than a processed/cleaned feature table
  — peak picking and data cleaning must be done first (e.g. with tidyMass or tools
  like MZmine, MS-DIAL, XCMS)
- Annotations at MSI levels other than level 2 are required, since only level 2 annotations
  were used for subsequent analysis in the tidyMass case study
- A single in-house database (one chromatographic mode) fully covers the analytes
  and merging across HILIC/RPLC/public databases with annotation-score-based redundancy
  removal is unnecessary
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metID
  role: Performs the annotation of cleaned features against the in-house HILIC and
    RPLC databases and the public database, matching on accurate m/z and retention
    time
- name: massdataset
  role: Loads and manages the cleaned feature table as a mass_dataset object that
    metID annotates
- name: tidyMass
  role: Overall R-based framework whose uniform mass_dataset data structure and metID
    module carry out the metabolite annotation step of the workflow
  repo: https://github.com/tidymass/tidymass
- name: R
  role: Language in which the annotation workflow is implemented and executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:f8105a3be1e77ceb1b02d002834f070654291e94d8c703eabf3ee921bf304949
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - massdataset
  - metID
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metabolite-annotation-msi-confidence-levels

## Summary

Annotates LC-MS features using the tidyMass metID package against in-house HILIC (71 metabolites) and RPLC (55 metabolites) databases plus a public database, removes redundant annotations by annotation score, and retains only MSI confidence level 2 identifications for downstream analysis.

## When to use

Use when LC-MS untargeted metabolomics features (e.g. a cleaned mass_dataset object from tidyMass) need metabolite identification against curated in-house databases with standardized MSI confidence-level reporting.

## When NOT to use

- The input is still raw LC-MS data rather than a processed/cleaned feature table — peak picking and data cleaning must be done first (e.g. with tidyMass or tools like MZmine, MS-DIAL, XCMS)
- Annotations at MSI levels other than level 2 are required, since only level 2 annotations were used for subsequent analysis in the tidyMass case study
- A single in-house database (one chromatographic mode) fully covers the analytes and merging across HILIC/RPLC/public databases with annotation-score-based redundancy removal is unnecessary

## Inputs

- Cleaned feature table as a tidyMass mass_dataset object (R, massdataset package)
- In-house metabolite databases built with metID: HILIC database (71 metabolites) and RPLC database (55 metabolites)
- A public metabolite database for extended annotation coverage
- Accurate m/z values and retention times of the features

## Outputs

- Merged, deduplicated annotation results (redundant annotations removed by annotation score)
- MSI level 2 annotation table (e.g. annotations_level2.csv), with 74 annotated metabolites in the CRC case study

## How to apply

Load the cleaned feature table as a mass_dataset object in R using massdataset. Run metID annotation against the in-house HILIC database (71 metabolites) and the in-house RPLC database (55 metabolites), matching on accurate m/z and retention time, then run a further metID annotation against a public metabolite database to extend coverage. Merge the annotation results and remove redundant annotations based on the annotation score. Filter to keep only annotations at MSI confidence level 2 — the tidyMass CRC case study retained 74 metabolites after this filtering — and write the level 2 annotation table (e.g. annotations_level2.csv) for subsequent statistical analysis.

## Related tools

- **metID** (Performs the annotation of cleaned features against the in-house HILIC and RPLC databases and the public database, matching on accurate m/z and retention time)
- **massdataset** (Loads and manages the cleaned feature table as a mass_dataset object that metID annotates)
- **tidyMass** (Overall R-based framework whose uniform mass_dataset data structure and metID module carry out the metabolite annotation step of the workflow) — https://github.com/tidymass/tidymass
- **R** (Language in which the annotation workflow is implemented and executed)

## Examples

```
remotes::install_github("tidymass/tidymass")
```

## Evaluation signals

- The merged annotation table contains no redundant annotations after filtering by annotation score
- All retained annotations are at MSI confidence level 2 (only level 2 annotations are carried into subsequent analysis)
- In the tidyMass CRC case study, exactly 74 metabolites remain annotated after redundancy removal — 71 from HILIC and 55 from RPLC database coverage
- The final level 2 table is written to annotations_level2.csv and is traceable back to the mass_dataset object and processing parameters
- Each annotation is supported by matching on both accurate m/z and retention time against the in-house database

## Limitations

- Only MSI level 2 annotations are retained for subsequent analysis, so level 1 (standard) or level 3+ identifications are not part of the downstream feature set
- Coverage depends on the in-house databases (only 71 HILIC and 55 RPLC metabolites) plus whatever the public database adds
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, although mass_dataset can be converted to other tools' data structures
- Redundant annotations are resolved solely by annotation score, which may not resolve all ambiguous identifications

## Evidence

- [intro] two in-house databases were constructed using metID, that contain 71 and 55 metabolites in HILIC and RPLC modes, respectively: "two in-house databases were constructed using metID, that contain 71 and 55 metabolites in HILIC and RPLC modes, respectively"
- [intro] Only the annotations with level 2 were used for subsequent analysis: "Only the annotations with level 2 were used for subsequent analysis"
- [intro] the redundant annotations were removed based on the annotation score: "the redundant annotations were removed based on the annotation score"
- [readme] tidyMass is a comprehensive R-based computational framework: [evidence span withheld — anchor status: external_doc]
