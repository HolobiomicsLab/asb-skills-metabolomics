---
name: categorical-item-factor-extraction
description: Use when a set of categorical behavioral questionnaire items (e.g., ASEBA
  CBCL item scores) must be reduced to a small number of continuous dimensions whose
  explained variance can be reported per dimension.
when_to_use_negative:
- The data are continuous molecular measurements (DNA methylation, metabolomics) —
  the ACTIONdemonstrator uses MOFA2 for those omics layers, not MCA
- The input is already a reduced factor or dimension table rather than raw categorical
  item scores
- The questionnaire items are not categorical or have not been imputed/prepared by
  the ACTIONdemonstrator_preprocessing workflow
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: FactoMineR
  role: R library (v2.4) used to run multiple correspondence analysis (MCA) on the
    categorical behavioral items to obtain factors
- name: Nextflow
  role: Workflow manager that packages and executes the MCA step as part of the modular
    ACTIONdemonstrator analysis pipeline
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:5047c0a913484051c27419201ecc1f98a2b895f46bce72f759429fd46e14376e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow.git
  tools:
  - FactoMineR
  - Nextflow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Categorical item factor extraction

## Summary

Runs multiple correspondence analysis (MCA) with the R library FactoMineR (v2.4) on categorical questionnaire items—here parent-rated ASEBA CBCL externalizing behavior items from the ACTION cohorts—to extract dimensions (factors) that capture underlying behavioral patterns.

## When to use

Use when a set of categorical behavioral questionnaire items (e.g., ASEBA CBCL item scores) must be reduced to a small number of continuous dimensions whose explained variance can be reported per dimension.

## When NOT to use

- The data are continuous molecular measurements (DNA methylation, metabolomics) — the ACTIONdemonstrator uses MOFA2 for those omics layers, not MCA
- The input is already a reduced factor or dimension table rather than raw categorical item scores
- The questionnaire items are not categorical or have not been imputed/prepared by the ACTIONdemonstrator_preprocessing workflow

## Inputs

- Imputed parent-rated externalizing behavior item scores from the ASEBA CBCL as a CSV file (e.g., Synthetic_data/synthetic_cbcl_data.csv)
- CBCL item label mapping (e.g., Synthetic_data/cbcl_labels.csv)
- 26 categorical behavioral items per subject from the NTR or LUMC-Curium ACTION cohorts

## Outputs

- MCA dimension/factor scores per subject for the behavioral data
- Table of per-dimension fraction of variation explained, written to mca_dimension_variance.csv

## How to apply

Load the imputed parent-rated externalizing behavior item scores (e.g., Synthetic_data/synthetic_cbcl_data.csv) into R and run multiple correspondence analysis (MCA) on the categorical items using the R library FactoMineR (v2.4), as in the ACTIONdemonstrator workflow. Extract the eigenvalues from the MCA result and compute the fraction of total variation explained by each dimension. Report the dimensions mirroring the paper's figures as a table with columns for dimension index and fraction of variation explained, and write it to mca_dimension_variance.csv. In the NTR cohort the first 3 MCA dimensions jointly explain 30% of the variation in 26 externalizing behavior items (dimension 1 ~16%, dimension 2 ~9%, dimension 3 <5%); for LUMC-Curium the first 2 dimensions together explain ~30%. The resulting behavioral dimension scores can subsequently be compared across SNF clusters or correlated with omics factors via Spearman rank correlation.

## Related tools

- **FactoMineR** (R library (v2.4) used to run multiple correspondence analysis (MCA) on the categorical behavioral items to obtain factors)
- **Nextflow** (Workflow manager that packages and executes the MCA step as part of the modular ACTIONdemonstrator analysis pipeline) — https://github.com/Xomics/ACTIONdemonstrator_workflow.git

## Examples

```
Rscript -e "library(FactoMineR); res <- MCA(cbcl_items, graph=FALSE); write.csv(data.frame(dimension=1:3, variance_explained=(res$eig[1:3,2]/sum(res$eig[,2]))), 'mca_dimension_variance.csv', row.names=FALSE)"
```

## Evaluation signals

- Per-dimension variance fractions from mca_dimension_variance.csv should reproduce the reported figures for NTR: dimension 1 ~16%, dimension 2 ~9%, dimension 3 <5%
- The first 3 MCA dimensions of the 26 NTR externalizing behavior items should jointly explain ~30% of total variation
- For the LUMC-Curium cohort, the first 2 dimensions together should explain ~30% of variation
- Output table contains the required columns: dimension index and fraction of variation explained
- The number of items entering the MCA matches the 26 parent-rated externalizing behavior items of the ASEBA CBCL

## Limitations

- MCA dimensions explain modest fractions of variation (e.g., only ~16% and ~9% for NTR dimensions 1 and 2), so individual dimensions should not be over-interpreted as complete representations of externalizing behavior
- Only synthetic data (Synthetic_data/synthetic_cbcl_data.csv) is publicly shared because privacy regulations prohibit sharing potentially personally identifiable clinical data; results on synthetic data may differ from the paper's reported values
- FactoMineR version matters for reproducibility (paper uses v2.4) and is executed inside a container dependency managed by Nextflow/Singularity

## Evidence

- [discussion] MCA applied to behavioral data: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [other] Per-dimension variance fractions for NTR: [evidence span withheld — anchor status: not_found]
- [readme] Synthetic data example inputs in the workflow README: "--behavioral_data Synthetic_data/synthetic_cbcl_data.csv"
- [discussion] Synthetic data availability limitation: "privacy regulations prohibit sharing of potentially personally identifiable data such as omics measurements or clinical information. To demonstrate the functionality of the workflow, we shared a"
