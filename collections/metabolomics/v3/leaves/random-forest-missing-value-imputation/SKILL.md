---
name: random-forest-missing-value-imputation
description: Use when a behavioral item matrix (e.g. ASEBA CBCL parent-rated externalizing
  items) contains missing item responses and a completed matrix is required before
  factor extraction such as multiple correspondence analysis.
when_to_use_negative:
- The omics data blocks (urine metabolomics, DNA methylation) — those use MOFA2 and
  their own QC/filters (e.g. discarding samples/metabolites with ≥15% missing values),
  not missForest
- The input matrix has no missing values — imputation is unnecessary and would only
  add noise
- A parametric imputation is required that depends on distributional assumptions;
  missForest is explicitly non-parametric for mixed-type data
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: missForest
  role: R library (v1.4) performing the iterative random-forest imputation of missing
    CBCL items
- name: Nextflow
  role: Workflow manager wrapping the imputation step as a modular, containerized
    pipeline process
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime capturing the software dependencies of the imputation step;
    Nextflow pulls containers from Dockerhub by default
  repo: https://github.com/Xomics/Docker_containers.git
- name: FactoMineR
  role: 'Downstream consumer: MCA (v2.4) is applied to the imputed behavioral data
    to obtain behavioral factors'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:5fa87004e6bee698f6ef87d265f7c25cf424a15e8b2d5d5bfbb43a7da9df1ac5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - FactoMineR
  - Nextflow
  - missForest
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# random-forest-missing-value-imputation

## Summary

Impute missing values in mixed-type behavioral data (ASEBA CBCL externalizing behavior items) using the non-parametric missForest random-forest imputation (R library missForest v1.4), so the completed item matrix can feed the downstream MCA step of the Nextflow multi-omics workflow.

## When to use

Use when a behavioral item matrix (e.g. ASEBA CBCL parent-rated externalizing items) contains missing item responses and a completed matrix is required before factor extraction such as multiple correspondence analysis.

## When NOT to use

- The omics data blocks (urine metabolomics, DNA methylation) — those use MOFA2 and their own QC/filters (e.g. discarding samples/metabolites with ≥15% missing values), not missForest
- The input matrix has no missing values — imputation is unnecessary and would only add noise
- A parametric imputation is required that depends on distributional assumptions; missForest is explicitly non-parametric for mixed-type data

## Inputs

- ASEBA CBCL behavioral item data file passed via --behavioral_data (CSV, e.g. Synthetic_data/synthetic_cbcl_data.csv), with subjects as rows and CBCL items as columns and missing item entries
- Optional: cbcl_labels.csv (--cbcl_labels) mapping item labels for the CBCL items

## Outputs

- Completed CBCL item matrix (imputed_cbcl_data.csv) with no remaining missing values, same subjects and items as the input
- Imputed behavioral matrix ordered for downstream MCA (FactoMineR) factor extraction

## How to apply

Load the behavioral data file passed via the --behavioral_data input (in the packaged example run: Synthetic_data/synthetic_cbcl_data.csv), which holds ASEBA CBCL item responses per child, keeping subject identifiers as row labels. Identify the missing-value pattern in the CBCL item matrix and treat unobserved items as entries to impute. Run the nonparametric random-forest imputation with the R library missForest (v1.4), which iteratively fills each missing item by regressing it on the other items with random forests until imputed values converge — this handles mixed-type data without distributional assumptions. Reassemble the completed matrix preserving the original item (column) and subject (row) order so it can be passed to the multiple correspondence analysis (MCA, R library FactoMineR 2.4) step of the Nextflow workflow. Validate that the output (imputed_cbcl_data.csv) exists, contains no remaining missing values, and retains the same subjects and CBCL items as the input matrix.

## Related tools

- **missForest** (R library (v1.4) performing the iterative random-forest imputation of missing CBCL items)
- **Nextflow** (Workflow manager wrapping the imputation step as a modular, containerized pipeline process) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime capturing the software dependencies of the imputation step; Nextflow pulls containers from Dockerhub by default) — https://github.com/Xomics/Docker_containers.git
- **FactoMineR** (Downstream consumer: MCA (v2.4) is applied to the imputed behavioral data to obtain behavioral factors)

## Examples

```
nextflow run action.nf --output dir/of/choice --behavioral_data Synthetic_data/synthetic_cbcl_data.csv --cbcl_labels Synthetic_data/cbcl_labels.csv
```

## Evaluation signals

- Output file imputed_cbcl_data.csv exists after the workflow step completes
- Imputed matrix contains no remaining missing (NA) values
- Set of subjects (rows) and CBCL items (columns) is identical between input and imputed output
- Original column (item) order and row (subject) order are preserved so the matrix is directly consumable by the MCA/FactoMineR step
- Imputed values fall within the plausible response range of the corresponding CBCL items

## Limitations

- Privacy regulations prohibit sharing the real, potentially personally identifiable behavioral/clinical data — only a synthetic dataset (Synthetic_data/) is provided to demonstrate the workflow, so convergence and imputation quality on real data cannot be inspected publicly
- missForest is iterative; convergence depends on the missing-value pattern, and very sparse item matrices may require attention to convergence behavior
- The workflow assumes data have already been standardized by the preprocessing workflow (Xomics/ACTIONdemonstrator_preprocessing); raw, non-standardized CBCL exports are not valid input

## Evidence

- [full_text] MissForest is cited as the imputation method for behavioral items: [evidence span withheld — anchor status: not_found]
- [full_text] missForest version and iterative procedure: [evidence span withheld — anchor status: not_found]
- [full_text] Validation of imputed output: [evidence span withheld — anchor status: not_found]
- [readme] Behavioral data input path in the packaged example run: "--behavioral_data Synthetic_data/synthetic_cbcl_data.csv"
- [discussion] MCA applied to behavioral data downstream: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [abstract] Workflow implemented as modular containerized Nextflow pipeline: "The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies."
