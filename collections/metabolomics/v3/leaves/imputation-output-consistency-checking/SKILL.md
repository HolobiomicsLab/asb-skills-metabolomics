---
name: imputation-output-consistency-checking
description: Use when a random-forest (missForest) imputation has been run on CBCL
  behavioral item data in the Nextflow multi-omics workflow and the output matrix
  must be checked before feeding it to the FactoMineR MCA step.
when_to_use_negative:
- The input matrix is not the imputed behavioral output but an omics table (e.g. methylation
  probes or urine metabolites), which uses different QC rules such as discarding samples/metabolites
  with 15% or more missing values
- The behavioral data file contains no missing CBCL item values, so no imputation-and-verification
  round trip is needed
- The output is destined for MOFA factor extraction (MOFA2) on methylation/metabolomics
  rather than the FactoMineR MCA behavioral step
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: missForest
  role: R library (v1.4) performing the nonparametric random-forest imputation of
    missing CBCL items whose output is validated by this skill
- name: Nextflow
  role: Workflow manager in which the imputation and validation step runs as a modular,
    containerized pipeline process
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime that captures the software dependencies of the imputation
    process pulled automatically by Nextflow
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:67f49fc12cc895cec77aded49766e2331941d35c1ee62d8d6e4fa6403ce73a78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - Nextflow
  - missForest
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# imputation-output-consistency-checking

## Summary

Validation skill for verifying that a completed (imputed) behavioral data matrix — here ASEBA CBCL externalizing behavior items imputed with missForest — retains the exact subject-row and item-column structure of the input and contains no remaining missing values before downstream MCA.

## When to use

Use when a random-forest (missForest) imputation has been run on CBCL behavioral item data in the Nextflow multi-omics workflow and the output matrix must be checked before feeding it to the FactoMineR MCA step.

## When NOT to use

- The input matrix is not the imputed behavioral output but an omics table (e.g. methylation probes or urine metabolites), which uses different QC rules such as discarding samples/metabolites with 15% or more missing values
- The behavioral data file contains no missing CBCL item values, so no imputation-and-verification round trip is needed
- The output is destined for MOFA factor extraction (MOFA2) on methylation/metabolomics rather than the FactoMineR MCA behavioral step

## Inputs

- Original ASEBA CBCL item response matrix (CSV with subject IDs as row labels, e.g. Synthetic_data/synthetic_cbcl_data.csv)
- Imputed output matrix (imputed_cbcl_data.csv) produced by missForest v1.4

## Outputs

- Validation result confirming the imputed matrix has zero remaining missing values
- Confirmation that subject and CBCL item sets and their ordering match the input matrix

## How to apply

Load the raw CBCL item matrix from the --behavioral_data input (packaged example: Synthetic_data/synthetic_cbcl_data.csv) and record its row (subject) and column (item) labels before imputation. After running the R library missForest (v1.4), which iteratively regresses each missing item on the other items with random forests until convergence, assemble the completed matrix preserving the original column (item) order and row (subject) order. Then verify the output imputed_cbcl_data.csv exists, has no remaining missing (NA) entries, and contains exactly the same set of subjects and CBCL items as the input matrix. This structural invariance check is essential because the completed matrix is passed downstream to multiple correspondence analysis (MCA, R library FactoMineR) within the Nextflow workflow, where row/column misalignment would silently corrupt factor estimation.

## Related tools

- **missForest** (R library (v1.4) performing the nonparametric random-forest imputation of missing CBCL items whose output is validated by this skill)
- **Nextflow** (Workflow manager in which the imputation and validation step runs as a modular, containerized pipeline process) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime that captures the software dependencies of the imputation process pulled automatically by Nextflow) — https://github.com/Xomics/Docker_containers.git

## Examples

```
Rscript -e 'library(missForest); imp <- missForest(read.csv("synthetic_cbcl_data.csv")); write.csv(imp$ximp, "imputed_cbcl_data.csv", row.names=FALSE); stopifnot(!any(is.na(imp$ximp)))'
```

## Evaluation signals

- Output file imputed_cbcl_data.csv exists after the workflow step completes
- imputed_cbcl_data.csv contains zero NA/missing entries in any cell
- Set of subject row labels in the output equals the set in the input --behavioral_data file
- Set and ordering of CBCL item columns in the output matches the original matrix
- Downstream MCA (FactoMineR) step runs without subject/item misalignment errors

## Limitations

- Only the packaged synthetic dataset is shared publicly because privacy regulations prohibit sharing personally identifiable omics or clinical data, so validation against real ACTION Biomarker Study CBCL data must occur in secure environments
- missForest is an iterative method; non-convergence or excessive iterations would leave residual structure issues that a purely structural consistency check will not detect
- Structural identity of rows/columns does not guarantee plausibility of imputed item values; no distribution-level checks are specified in the workflow

## Evidence

- [discussion] MCA applied to behavioral data: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [readme] Synthetic example data: "Synthetic data is provided as example data in `Synthetic_data/`"
- [readme] Behavioral input parameter: "--behavioral_data Synthetic_data/synthetic_cbcl_data.csv"
- [readme] Containerized execution: "Nextflow automatically pulls all necessary Docker containers from Dockerhub, when using the default `nextflow.config` file"
