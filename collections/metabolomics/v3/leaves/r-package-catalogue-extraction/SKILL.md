---
name: r-package-catalogue-extraction
description: Use when building or updating a structured inventory of R packages for
  metabolomics, where each package must be attributed to a repository (CRAN, Bioconductor,
  GitHub) and a workflow step (data processing, biostatistics, annotation, pathway
  analysis).
when_to_use_negative:
- The goal is tool recommendation or execution of a metabolomics analysis rather than
  building a package inventory — this skill only catalogues and classifies packages,
  it does not run them.
- The source text is not the metaRbolomics review or an equivalent curated review
  article; package names, repositories, and workflow-step placements must come from
  the review's own section structure, not from ad-hoc web searches.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: R
  role: The programming and statistics environment whose metabolomics-specific packages
    are being catalogued and classified
- name: pandas
  role: Used to assemble the extracted package records into the catalogue CSV with
    the required columns
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:c96e8d21ef56816654b9d4f6ce15d4f77faf3983a78a827252750bab4d6399fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# r-package-catalogue-extraction

## Summary

A curation skill for reproducing the curated catalogue of metabolomics-specific R packages from the metaRbolomics review (DOI 10.3390/metabo9100200), classifying each of the 200+ packages by hosting repository (CRAN, Bioconductor, GitHub) and by computational metabolomics workflow step.

## When to use

Use when building or updating a structured inventory of R packages for metabolomics, where each package must be attributed to a repository (CRAN, Bioconductor, GitHub) and a workflow step (data processing, biostatistics, annotation, pathway analysis).

## When NOT to use

- The goal is tool recommendation or execution of a metabolomics analysis rather than building a package inventory — this skill only catalogues and classifies packages, it does not run them.
- The source text is not the metaRbolomics review or an equivalent curated review article; package names, repositories, and workflow-step placements must come from the review's own section structure, not from ad-hoc web searches.

## Inputs

- Review article full text (DOI 10.3390/metabo9100200, 'The metaRbolomics Toolbox in Bioconductor and beyond')
- Review section structure defining workflow-step categories (MS pre-processing/data processing, annotation and identification, biostatistics, biochemical network and pathway analysis, multifunctional workflow)

## Outputs

- metaRbolomics_package_catalogue.csv with columns (package, repository, workflow_step, description) and >= 200 data rows

## How to apply

Read the review article and extract every metabolomics-specific R package name verbatim, since the review summarises more than two hundred such packages. For each package, record its hosting repository exactly as stated in the review (CRAN, Bioconductor, or GitHub). Assign each package to the workflow step corresponding to the review's section structure — MS pre-processing/data processing, metabolite annotation and identification, biostatistics, biochemical network and pathway analysis, or multifunctional workflow — and write a one-line short description. Assemble all records with pandas into a CSV with the required columns (package, repository, workflow_step, description) and save it as metaRbolomics_package_catalogue.csv; validation requires at least 200 data rows, all four columns present, and LLM-judge agreement of workflow-step classification with the review's section placement.

## Related tools

- **R** (The programming and statistics environment whose metabolomics-specific packages are being catalogued and classified)
- **pandas** (Used to assemble the extracted package records into the catalogue CSV with the required columns)

## Examples

```
df = pd.DataFrame(records, columns=['package','repository','workflow_step','description']); df.to_csv('metaRbolomics_package_catalogue.csv', index=False)
```

## Evaluation signals

- metaRbolomics_package_catalogue.csv exists on disk after the run
- The CSV contains at least 200 data rows, consistent with the review's claim of more than two hundred packages
- All four required columns are present: package, repository, workflow_step, description
- An LLM judge confirms that workflow-step classifications agree with the review's section placement of each package
- Every repository value is one of CRAN, Bioconductor, GitHub, or another hosting source as stated in the review

## Limitations

- Workflow-step assignments are limited to the categories defined by the review's section structure; packages spanning multiple steps must be reconciled with the review's placement rather than re-classified independently.
- Repository attribution reflects what the review states; packages may have moved repositories since publication (2019).
- No changelog is available for the curated list (the source notes '_No changelog found._'), so drift from the published catalogue cannot be tracked.

## Evidence

- [abstract] abstract: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [other] other: [evidence span withheld — anchor status: not_found]
