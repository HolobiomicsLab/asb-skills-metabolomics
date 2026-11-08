---
name: feature-intensity-matrix-preparation
description: Use when an actual metabolomics or proteomics dataset must be formatted
  as a feature/sample intensity matrix as input to pseudoQC's regression-based QC
  simulation before correction and normalization.
when_to_use_negative:
- The input is already simulated pooled-QC data rather than an actual metabolomics
  or proteomics dataset needing QC simulation
- The analysis goal is batch correction or normalization itself rather than generating
  simulated QC sample data via regression
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pseudoQC
  role: Interactive web-based software that simulates QC sample data for actual metabolomics
    and proteomics datasets using four machine learning-based regression methods
  repo: https://www.omicsolution.org/wukong/pseudoQC/
- name: R
  role: Language in which pseudoQC is developed; used to run the simulation locally
    from the GitHub repository
  repo: https://github.com/qade544/pseudoQC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201900264
    title: 'pseudoQC: A Regression‐Based Simulation Software for Correction and Normalization
      of Complex Metabolomics and Proteomics Datasets'
  source_content_hash: sha256:d0f2ab440261b9fdac6d3a474472bff60465b987bf1cf8168559f8101d32b70a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - pseudoQC
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# feature-intensity-matrix-preparation

## Summary

Prepares a feature/sample intensity matrix from a metabolomics or proteomics dataset so it can be loaded into pseudoQC, the R-based web software that simulates QC sample data using four machine learning-based regression methods for downstream correction and normalization.

## When to use

Use when an actual metabolomics or proteomics dataset must be formatted as a feature/sample intensity matrix as input to pseudoQC's regression-based QC simulation before correction and normalization.

## When NOT to use

- The input is already simulated pooled-QC data rather than an actual metabolomics or proteomics dataset needing QC simulation
- The analysis goal is batch correction or normalization itself rather than generating simulated QC sample data via regression

## Inputs

- Feature/sample intensity matrix from a metabolomics or proteomics dataset (rows = features, columns = samples, with original feature and sample identifiers preserved)

## Outputs

- Simulated QC sample data table per regression method (four tables total, one per machine learning-based regression method), preserving original feature/sample identifiers for downstream correction and normalization

## How to apply

Assemble the dataset (e.g., one of the two published metabolomics or proteomics datasets used in the paper) as a feature/sample intensity matrix in which rows correspond to features and columns to samples, preserving the original feature and sample identifiers. Load this matrix into pseudoQC, the interactive web-based software developed with R and available at https://www.omicsolution.org/wukong/pseudoQC/ (source on GitHub at YanglabWCH/pseudoQC). Follow the detailed operation instructions in pseudoQC.Supplementary.pdf to run the simulation, which applies four different machine learning-based regression methods to generate simulated pooled-QC sample data. The rationale is that regression-based simulation of QC sample data from the real dataset enables subsequent correction and normalization, where the article's results indicate nonlinear regression methods perform better than linear ones.

## Related tools

- **pseudoQC** (Interactive web-based software that simulates QC sample data for actual metabolomics and proteomics datasets using four machine learning-based regression methods) — https://www.omicsolution.org/wukong/pseudoQC/
- **R** (Language in which pseudoQC is developed; used to run the simulation locally from the GitHub repository) — https://github.com/qade544/pseudoQC

## Examples

```
Open https://www.omicsolution.org/wukong/pseudoQC/ and upload the feature/sample intensity matrix of your metabolomics or proteomics dataset to run the four regression-based QC simulations (see pseudoQC.Supplementary.pdf for detailed operation).
```

## Evaluation signals

- Four simulated QC data tables are produced, one per machine learning-based regression method
- Each simulated QC table contains the same features as the input dataset
- Original feature and sample identifiers are preserved so the simulated data can be used for correction and normalization
- The simulated QC data, when applied to correction and normalization of the published datasets, reproduce the paper's result that nonlinear regression methods perform better than linear ones
- Detailed operation steps match those documented in pseudoQC.Supplementary.pdf

## Limitations

- The four machine learning-based regression methods are not individually named in the article's methods section
- Detailed operating instructions reside only in the pseudoQC.Supplementary.pdf file, so exact parameter settings are not given in the article or README
- The software is web-based; a working web interface (or local R installation of the GitHub repository) is required

## Evidence

- [abstract] pseudoQC simulation of QC data via four regression methods: "simulate QC sample data for actual metabolomics and proteomics datasets using four different machine learning-based regression methods"
- [abstract] Use of simulated data downstream: "The simulated data are used for correction and normalization of the two published datasets"
- [readme] pseudoQC is R-based web software with a hosted URL: "It is developed with [R](https://www.r-project.org/) and the software can also be found here: [https://www.omicsolution.org/wukong/pseudoQC/](https://www.omicsolution.org/wukong/pseudoQC/)."
- [readme] Detailed usage documentation location: "The detailed operation about **<font size='5'> pseudoQC </font>** can be found in [pseudoQC.Supplementary.pdf](https://github.com/qade544/pseudoQC/blob/master/pseudoQC.Supplementary.pdf) file."
- [abstract] Nonlinear methods outperform linear ones: "the obtained results suggest that nonlinear regression methods perform better than linear ones"
