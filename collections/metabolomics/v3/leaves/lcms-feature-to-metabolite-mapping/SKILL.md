---
name: lcms-feature-to-metabolite-mapping
description: Use when a cleaned LC-MS feature table (mass_dataset object) needs metabolite
  annotations, and database-level identifications (MSI level 2) must be produced by
  matching m/z and retention time.
when_to_use_negative:
- Input is raw LC-MS data that has not been peak-picked and cleaned — run the tidyMass
  raw data processing and data cleaning steps first
- You require MSI level 1 identifications or MS/MS spectral verification — this skill
  only guarantees annotations up to MSI level 2
- You need to annotate GC-MS or MS/MS data — the skill is grounded in LC-MS feature
  tables only
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: metID
  role: Performs metabolite annotation of features against in-house HILIC/RPLC and
    public databases by m/z and retention time matching
  repo: https://github.com/tidymass/tidymass
- name: massdataset
  role: Loads and manages the cleaned feature table as a mass_dataset object that
    feeds the annotation step
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidyMass
  role: Object-oriented framework providing the uniform mass_dataset data structure
    and workflow context for annotation
  repo: https://github.com/tidymass/tidymass
- name: R
  role: Language in which the annotation workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:69c1e465fda516f4a79caa0cbd556a80dc69cd87e4e9b022dc3c2fbb6810ca0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - massdataset
  - metID
  - tidyMass
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# lcms-feature-to-metabolite-mapping

## Summary

Annotates LC-MS peak features to putative metabolites using metID within tidyMass, matching accurate m/z and retention time against in-house HILIC (71 metabolites) and RPLC (55 metabolites) databases plus public databases, then deduplicates annotations by annotation score and retains MSI level 2 identifications.

## When to use

Use when a cleaned LC-MS feature table (mass_dataset object) needs metabolite annotations, and database-level identifications (MSI level 2) must be produced by matching m/z and retention time.

## When NOT to use

- Input is raw LC-MS data that has not been peak-picked and cleaned — run the tidyMass raw data processing and data cleaning steps first
- You require MSI level 1 identifications or MS/MS spectral verification — this skill only guarantees annotations up to MSI level 2
- You need to annotate GC-MS or MS/MS data — the skill is grounded in LC-MS feature tables only

## Inputs

- Cleaned LC-MS feature table as a mass_dataset object (R, massdataset package)
- In-house HILIC metabolite database (71 metabolites, metID format)
- In-house RPLC metabolite database (55 metabolites, metID format)
- Optional public metabolite database for extended coverage

## Outputs

- Metabolite annotation table restricted to MSI level 2 (e.g. annotations_level2.csv)
- Per-feature annotations with database matches, annotation scores, and confidence levels

## How to apply

Load the cleaned feature table as a mass_dataset object with massdataset, then run metID annotation against the in-house HILIC database (71 metabolites) and the in-house RPLC database (55 metabolites), matching on accurate m/z and retention time. Optionally annotate against a public database to extend coverage. Merge all annotation results and remove redundant annotations based on the annotation score, since the same feature can match multiple database entries across modes. Finally, filter to keep only MSI confidence level 2 annotations — the paper restricts downstream analysis to level 2 — and export the resulting annotation table (74 metabolites remained in the CRC case study).

## Related tools

- **metID** (Performs metabolite annotation of features against in-house HILIC/RPLC and public databases by m/z and retention time matching) — https://github.com/tidymass/tidymass
- **massdataset** (Loads and manages the cleaned feature table as a mass_dataset object that feeds the annotation step) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidyMass** (Object-oriented framework providing the uniform mass_dataset data structure and workflow context for annotation) — https://github.com/tidymass/tidymass
- **R** (Language in which the annotation workflow is executed)

## Examples

```
remotes::install_gitlab("tidymass/tidymass"); library(massdataset); annotated <- metID::annotate_features(mass_dataset, database = c(hilic_db, rplc_db)); annotations_level2 <- filter_msi_level2(remove_redundant(annotated))
```

## Evaluation signals

- All retained annotations are at MSI confidence level 2 (annotations at other levels are filtered out)
- No redundant annotations remain after deduplication by annotation score
- In the CRC case study, exactly 74 metabolites remain annotated after filtering (from HILIC: 71 metabolite database; RPLC: 55 metabolite database)
- The output is a valid level 2 annotation table (e.g. annotations_level2.csv) keyed to features in the input mass_dataset object

## Limitations

- Annotations are limited to MSI level 2 putative identifications, not confirmed level 1 identifications
- Coverage depends on the database contents — the in-house databases contain only 71 (HILIC) and 55 (RPLC) metabolites
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, requiring conversion of the mass_dataset class to other data structures

## Evidence

- [intro] two in-house databases were constructed using metID, that contain 71 and 55 metabolites in HILIC and RPLC modes, respectively: "two in-house databases were constructed using metID, that contain 71 and 55 metabolites in HILIC and RPLC modes, respectively"
- [intro] Only the annotations with level 2 were used for subsequent analysis: "Only the annotations with level 2 were used for subsequent analysis"
- [intro] the redundant annotations were removed based on the annotation score24: "the redundant annotations were removed based on the annotation score24"
- [intro] requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation"
- [intro] tidyMass has high interoperability with other existing tools: "tidyMass has high interoperability with other existing tools"
