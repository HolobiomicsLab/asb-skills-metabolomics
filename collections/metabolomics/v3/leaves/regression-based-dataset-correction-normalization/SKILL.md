---
name: regression-based-dataset-correction-normalization
description: Use when a metabolomics or proteomics feature/sample intensity matrix
  lacks actual pooled-QC samples and you need simulated QC data to perform regression-based
  correction and normalization of the dataset.
when_to_use_negative:
- The dataset already contains actual pooled-QC samples, so QC data need not be simulated
- The input is raw instrument output rather than a feature/sample intensity matrix
  (pseudoQC takes actual metabolomics and proteomics datasets as feature/sample matrices)
- The analysis goal does not involve correction or normalization driven by QC sample
  data
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pseudoQC
  role: Interactive web-based software that simulates pooled-QC sample data from an
    actual metabolomics or proteomics dataset using four machine learning-based regression
    methods, for use in correction and normalization
  repo: https://github.com/qade544/pseudoQC
- name: R
  role: Language in which pseudoQC is developed; the software can be run via its R-based
    web interface
  repo: https://www.omicsolution.org/wukong/pseudoQC/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201900264
    title: 'pseudoQC: A Regression‐Based Simulation Software for Correction and Normalization
      of Complex Metabolomics and Proteomics Datasets'
  source_content_hash: sha256:803d6bd79f93c204c5ba50a2b371ff013003e0706343606487a712805b611454
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/qade544/pseudoQC
  tools:
  - R
  - pseudoQC
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# regression-based-dataset-correction-normalization

## Summary

Simulates pooled-QC sample data for a feature/sample intensity matrix (metabolomics or proteomics) using pseudoQC's four machine learning-based regression methods, then uses the simulated QC data to drive correction and normalization of the original dataset. Motivated by pseudoQC's finding that nonlinear regression methods outperform linear ones for this correction/normalization task.

## When to use

Use when a metabolomics or proteomics feature/sample intensity matrix lacks actual pooled-QC samples and you need simulated QC data to perform regression-based correction and normalization of the dataset.

## When NOT to use

- The dataset already contains actual pooled-QC samples, so QC data need not be simulated
- The input is raw instrument output rather than a feature/sample intensity matrix (pseudoQC takes actual metabolomics and proteomics datasets as feature/sample matrices)
- The analysis goal does not involve correction or normalization driven by QC sample data

## Inputs

- metabolomics or proteomics feature/sample intensity matrix (actual dataset)
- choice of regression method among the four machine learning-based regression methods implemented in pseudoQC

## Outputs

- one simulated pooled-QC sample data table per regression method (four tables total), with features matching the input dataset
- corrected and normalized versions of the original dataset derived using the simulated QC data

## How to apply

Obtain the open-source pseudoQC software (R-based; usable via the web interface at https://www.omicsolution.org/wukong/pseudoQC/ or the GitHub repository) and load the input feature/sample intensity matrix. For each of the four machine learning-based regression methods implemented in pseudoQC, run the regression-based simulation to generate simulated pooled-QC sample data from the actual dataset, exporting one simulated QC data table per method (four tables total). Preserve the original feature/sample identifiers so the simulated QC tables can be directly used downstream for correction and normalization. Apply the simulated QC data to correct and normalize the published dataset(s); the paper's comparative result — that nonlinear regression methods perform better than linear ones — is the benchmark expectation. Detailed operation instructions are provided in the pseudoQC.Supplementary.pdf file in the GitHub repository.

## Related tools

- **pseudoQC** (Interactive web-based software that simulates pooled-QC sample data from an actual metabolomics or proteomics dataset using four machine learning-based regression methods, for use in correction and normalization) — https://github.com/qade544/pseudoQC
- **R** (Language in which pseudoQC is developed; the software can be run via its R-based web interface) — https://www.omicsolution.org/wukong/pseudoQC/

## Examples

```
browseURL("https://www.omicsolution.org/wukong/pseudoQC/")
```

## Evaluation signals

- Exactly four simulated QC data tables are produced, one per regression method
- Each simulated QC table contains the same features as the input dataset, with original feature/sample identifiers preserved
- The simulated QC data successfully drive correction and normalization of the target dataset (e.g. one of the two published metabolomics or proteomics datasets referenced in the paper)
- Comparative results across the four regression methods reproduce the paper's finding that nonlinear regression methods perform better than linear ones
- Run details match the operating instructions in pseudoQC.Supplementary.pdf

## Limitations

- The four machine learning-based regression methods are not individually named in the article's extracted text, so per-method parameter details must be obtained from the supplementary PDF
- pseudoQC is a web-based (Shiny-style) application rather than a command-line tool; operation details live in pseudoQC.Supplementary.pdf, not in the README
- Performance conclusions are grounded in two published datasets (metabolomics and proteomics); behavior on other dataset types is not established by this paper

## Evidence

- [abstract] Nonlinear methods outperform linear ones: [evidence span withheld — anchor status: not_found]
- [readme] R implementation and web availability: [evidence span withheld — anchor status: not_found]
