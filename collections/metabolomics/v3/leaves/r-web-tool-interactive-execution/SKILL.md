---
name: r-web-tool-interactive-execution
description: Use when a published R/Shiny web tool must be run interactively (via
  its web URL or locally from GitHub) to generate simulated pooled-QC sample data
  from a feature/sample intensity matrix for correction and normalization.
when_to_use_negative:
- The dataset already contains real, measured pooled-QC samples and only needs drift
  correction — pseudoQC is for simulating QC data when actual QC samples are absent
- The goal is regression modeling or normalization on the raw data directly rather
  than generating simulated QC sample data as intermediate inputs
- The input is not a metabolomics or proteomics feature/sample intensity matrix (pseudoQC
  targets these two data types)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0511
tools:
- name: pseudoQC
  role: Interactive R-based web software that simulates QC sample data for actual
    metabolomics and proteomics datasets using four machine learning-based regression
    methods
  repo: https://www.omicsolution.org/wukong/pseudoQC/
- name: R
  role: Language in which pseudoQC is developed; used to run the tool locally from
    the GitHub repository
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201900264
    title: 'pseudoQC: A Regression‐Based Simulation Software for Correction and Normalization
      of Complex Metabolomics and Proteomics Datasets'
  source_content_hash: sha256:8bfe7dbb156cfde070f3d13f2152f1684ccde9d0d2c4c6166cb29ded3106c5ef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - pseudoQC
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0511
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# r-web-tool-interactive-execution

## Summary

Execute an R-developed, interactive web-based tool (pseudoQC) through its web interface or locally from its GitHub repository to simulate QC sample data for metabolomics and proteomics datasets using four machine learning-based regression methods.

## When to use

Use when a published R/Shiny web tool must be run interactively (via its web URL or locally from GitHub) to generate simulated pooled-QC sample data from a feature/sample intensity matrix for correction and normalization.

## When NOT to use

- The dataset already contains real, measured pooled-QC samples and only needs drift correction — pseudoQC is for simulating QC data when actual QC samples are absent
- The goal is regression modeling or normalization on the raw data directly rather than generating simulated QC sample data as intermediate inputs
- The input is not a metabolomics or proteomics feature/sample intensity matrix (pseudoQC targets these two data types)

## Inputs

- Feature/sample intensity matrix from a metabolomics or proteomics dataset (input table loaded into pseudoQC)

## Outputs

- Simulated pooled-QC sample data table per regression method (four tables total), with the same features as the input dataset, for use in correction and normalization

## How to apply

Obtain the open-source pseudoQC software, which is developed in R, either by opening the web interface at https://www.omicsolution.org/wukong/pseudoQC/ or by running the app locally from the GitHub repository (YanglabWCH/pseudoQC). Load the input metabolomics or proteomics dataset — a feature/sample intensity matrix — into the tool. Select one of the four machine learning-based regression methods implemented in pseudoQC and run the regression-based simulation to generate simulated QC sample data from the actual dataset. Export one simulated QC data table per regression method (four tables total), preserving the original feature/sample identifiers, so the tables can be used downstream for correction and normalization of the two published datasets. For detailed operation steps, consult the pseudoQC.Supplementary.pdf file linked in the repository README.

## Related tools

- **pseudoQC** (Interactive R-based web software that simulates QC sample data for actual metabolomics and proteomics datasets using four machine learning-based regression methods) — https://www.omicsolution.org/wukong/pseudoQC/
- **R** (Language in which pseudoQC is developed; used to run the tool locally from the GitHub repository) — https://www.r-project.org/

## Examples

```
Rscript -e "shiny::runGitHub('YanglabWCH/pseudoQC')"
```

## Evaluation signals

- Four simulated QC data tables are produced, one per regression method
- Each simulated QC table has the same features as the input dataset, with original feature/sample identifiers preserved
- The simulated QC tables can be applied directly to correction and normalization of the published datasets
- Applying the simulated QC data reproduces the paper's finding that nonlinear regression methods perform better than linear ones for correction and normalization
- Detailed operation steps were followed from pseudoQC.Supplementary.pdf in the GitHub repository

## Limitations

- The four individual machine learning-based regression methods are not named in the article's methods section (flagged as a missing signal)
- The tool is accessed through an interactive web interface or local R execution; the README provides no command-line or scripting API, referring users to the supplementary PDF for detailed operation
- Contact for bugs or questions is via a GitHub issue or e-mail (<maintainer e-mail, see the repository README>); no other support channel is documented

## Evidence

- [abstract] the simulated data are used for correction and normalization of the two published datasets, and the obtained results suggest that nonlinear regression methods perform better than linear ones: [evidence span withheld — anchor status: not_found]
