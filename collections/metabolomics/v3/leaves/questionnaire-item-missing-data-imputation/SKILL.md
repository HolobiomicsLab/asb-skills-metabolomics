---
name: questionnaire-item-missing-data-imputation
description: Use when a questionnaire item matrix (e.g., parent-rated ASEBA CBCL externalizing
  behavior items per child) contains missing entries that must be filled before downstream
  factor extraction such as multiple correspondence analysis.
when_to_use_negative:
- The CBCL item matrix already has no missing values — imputation is unnecessary and
  would alter observed data
- The target data is continuous omics data (e.g., urine metabolomics or DNA methylation
  probes) with its own filtering rules (e.g., discarding samples/metabolites with
  ≥15% missing values) rather than questionnaire items to be imputed
- You need a factor model other than MCA on the behavioral items that has different
  requirements than a complete item matrix
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: missForest (R library, v1.4)
  role: Performs the nonparametric random-forest imputation of missing CBCL items,
    iteratively regressing each missing item on the others until convergence
- name: Nextflow
  role: Workflow manager orchestrating the imputation step and passing the completed
    matrix to downstream MCA
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Provides containerized software dependencies for the workflow module (containers
    pulled automatically from Dockerhub via nextflow.config)
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:fff30d05d3b396c7c8233409cebbfb313ee4c421c78344f4d01c5e33c415df48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - Nextflow
  - missForest (R library, v1.4)
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# questionnaire-item-missing-data-imputation

## Summary

Imputes missing item responses in behavioral questionnaire data (ASEBA CBCL externalizing behavior items) using missForest, a non-parametric random-forest-based missing value imputation method for mixed-type data, so a complete item matrix can be passed to downstream factor analysis (MCA).

## When to use

Use when a questionnaire item matrix (e.g., parent-rated ASEBA CBCL externalizing behavior items per child) contains missing entries that must be filled before downstream factor extraction such as multiple correspondence analysis.

## When NOT to use

- The CBCL item matrix already has no missing values — imputation is unnecessary and would alter observed data
- The target data is continuous omics data (e.g., urine metabolomics or DNA methylation probes) with its own filtering rules (e.g., discarding samples/metabolites with ≥15% missing values) rather than questionnaire items to be imputed
- You need a factor model other than MCA on the behavioral items that has different requirements than a complete item matrix

## Inputs

- Behavioral data CSV of ASEBA CBCL item responses per child (e.g., synthetic_cbcl_data.csv), with missing entries and subject identifiers as row labels

## Outputs

- Completed behavioral item matrix (imputed_cbcl_data.csv) with no remaining missing values, preserving original subject row order and CBCL item column order

## How to apply

Load the behavioral data file provided via the --behavioral_data input (in the packaged example run: Synthetic_data/synthetic_cbcl_data.csv), which contains ASEBA CBCL item responses per child, keeping subject identifiers as row labels and preserving the original item (column) order. Identify the missing-value pattern, treating unobserved items as entries to impute. Run the nonparametric random-forest imputation with the R library missForest (v1.4), which iteratively fills in each missing item by regressing it on the other items with random forests until the imputed values converge. Assemble the completed behavioral item matrix, preserving row and column order, so it can be passed downstream to the MCA step (R library FactoMineR 2.4) of the Nextflow workflow. Validate that the output imputed_cbcl_data.csv has no remaining missing values and the same subjects and items as the input.

## Related tools

- **missForest (R library, v1.4)** (Performs the nonparametric random-forest imputation of missing CBCL items, iteratively regressing each missing item on the others until convergence)
- **Nextflow** (Workflow manager orchestrating the imputation step and passing the completed matrix to downstream MCA) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Provides containerized software dependencies for the workflow module (containers pulled automatically from Dockerhub via nextflow.config)) — https://github.com/Xomics/Docker_containers.git

## Examples

```
library(missForest); cbcl_imputed <- missForest(cbcl_matrix)$ximp; write.csv(cbcl_imputed, "imputed_cbcl_data.csv", row.names = TRUE)
```

## Evaluation signals

- Output file imputed_cbcl_data.csv exists after the run
- Imputed matrix contains no remaining missing (NA) values
- Imputed matrix contains exactly the same set of subjects (rows) and CBCL items (columns) as the input matrix
- Original column (item) order and row (subject) order are preserved so downstream MCA (FactoMineR 2.4) receives a correctly ordered matrix
- Observed (non-missing) entries are unchanged by the imputation; only previously missing entries are filled

## Limitations

- Privacy regulations prohibit sharing the real behavioral/omics data; only a synthetic dataset (Synthetic_data/) is distributed, so imputation behavior on the true cohort data must be reproduced by users with authorized access
- missForest was chosen as a non-parametric method for mixed-type data; performance depends on convergence of the iterative random-forest regressions, and the article does not report accuracy metrics on held-out true values

## Evidence

- [full_text] The paper cites MissForest, a non-parametric missing value imputation method for mixed-type data, as the reference for imputing missing values in the behavioral item data.: [evidence span withheld — anchor status: not_found]
- [readme] Behavioral data input is the synthetic CBCL file: "--behavioral_data Synthetic_data/synthetic_cbcl_data.csv"
- [readme] Synthetic example data ships with the workflow repository: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
- [discussion] Imputed behavioral matrix feeds the MCA step: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
