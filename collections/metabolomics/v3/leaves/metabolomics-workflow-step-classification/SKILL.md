---
name: metabolomics-workflow-step-classification
description: Use when curating or reproducing a catalogue of metabolomics R/Bioconductor
  packages that must be classified by workflow step (data processing, biostatistics,
  annotation/identification, network/pathway analysis) and repository (CRAN, Bioconductor,
  GitHub).
when_to_use_negative:
- The goal is to run or execute a metabolomics pipeline rather than catalogue and
  classify packages — this skill produces a reference table, not processed data.
- The source list of packages comes from a different review or registry whose section
  structure differs from the metaRbolomics workflow steps; classifications would not
  match this review's placement.
- The input is already a structured package table with workflow-step labels; reclassification
  would only duplicate existing metadata.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3391
tools:
- name: R
  role: The programming and statistics environment whose metabolomics-specific packages
    are being catalogued and classified
- name: pandas
  role: Assembles the extracted package records into the required CSV catalogue with
    validated columns
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:07049d5b3929a4a681849175f044e7ea8689f9d0b20a35b0b00b68c9c8b22538
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# metabolomics-workflow-step-classification

## Summary

A curation skill for building a structured catalogue of metabolomics-specific R packages (from the metaRbolomics review, DOI 10.3390/metabo9100200) in which every package is assigned to the computational metabolomics workflow step covered by the review's section structure and tagged with its hosting repository.

## When to use

Use when curating or reproducing a catalogue of metabolomics R/Bioconductor packages that must be classified by workflow step (data processing, biostatistics, annotation/identification, network/pathway analysis) and repository (CRAN, Bioconductor, GitHub).

## When NOT to use

- The goal is to run or execute a metabolomics pipeline rather than catalogue and classify packages — this skill produces a reference table, not processed data.
- The source list of packages comes from a different review or registry whose section structure differs from the metaRbolomics workflow steps; classifications would not match this review's placement.
- The input is already a structured package table with workflow-step labels; reclassification would only duplicate existing metadata.

## Inputs

- Full text of the metaRbolomics review article (DOI 10.3390/metabo9100200), including its section structure (Tables 1-12)
- Package names and repository attributions as stated verbatim in the review

## Outputs

- metaRbolomics_package_catalogue.csv with columns package, repository, workflow_step, description and >=200 data rows

## How to apply

Read the metaRbolomics review (10.3390/metabo9100200) and extract every metabolomics-specific R package name verbatim; the review summarises more than two hundred such packages. For each package, record its hosting repository exactly as stated in the review (CRAN, Bioconductor, GitHub, or other). Assign each package to the workflow step corresponding to the review's section placement — MS pre-processing/data processing, metabolite annotation and identification, biostatistics, biochemical network and pathway analysis, or multifunctional workflow — and write a one-line description per package. Assemble all records (columns: package, repository, workflow_step, description) into a CSV (e.g. metaRbolomics_package_catalogue.csv) using pandas. Validate by checking the file exists, contains at least 200 data rows, all four required columns are present, and an LLM judge confirms workflow-step classifications agree with the review's section placement.

## Related tools

- **R** (The programming and statistics environment whose metabolomics-specific packages are being catalogued and classified)
- **pandas** (Assembles the extracted package records into the required CSV catalogue with validated columns)

## Examples

```
import pandas as pd; df = pd.DataFrame(records, columns=['package','repository','workflow_step','description']); df.to_csv('metaRbolomics_package_catalogue.csv', index=False)
```

## Evaluation signals

- Output CSV file exists and contains at least 200 data rows, matching the review's 'more than two hundred' package count.
- All four required columns (package, repository, workflow_step, description) are present with no missing values.
- Every package name matches the review's verbatim spelling (e.g. cross-check against Tables 1-12).
- Every repository value is one of CRAN, Bioconductor, GitHub, or other, as stated in the review.
- An LLM judge confirms each workflow_step classification agrees with the review's section placement of the package.

## Limitations

- Repository and workflow-step attributions reflect the review's 2019 snapshot; packages may have moved repositories or gained new functionality since publication.
- Packages described as multifunctional span multiple workflow steps, so single-step classification can be ambiguous.
- Classification depends on the review's section structure; packages not mentioned in the review cannot be classified by this method.

## Evidence

- [abstract] More than two hundred packages exist: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [abstract] Workflow steps covered: "This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow, including data processing, biostatistics, metabolite"
- [abstract] R as target environment: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
