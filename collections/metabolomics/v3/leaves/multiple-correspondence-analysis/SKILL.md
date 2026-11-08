---
name: multiple-correspondence-analysis
description: Use when behavioral questionnaire items are categorical (e.g., CBCL item
  scores) and you need to reduce them to a few underlying dimensions and quantify
  per-dimension explained variance.
when_to_use_negative:
- The behavioral data are continuous rather than categorical item scores — MCA is
  designed for categorical variables; use a method suited to continuous data (e.g.,
  PCA/MOFA as used for the omics blocks)
- The goal is integrating multi-omics (methylation/metabolomics) data blocks — the
  workflow used MOFA2 for those, MCA only for the behavioral CBCL items
- You need significance testing between groups — the workflow ran downstream Mann–Whitney
  U / Kruskal–Wallis / GEE tests on the MCA dimension scores as a separate step
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: FactoMineR
  role: R library used to run the MCA on the categorical behavioral items and extract
    eigenvalues
- name: Nextflow
  role: Workflow manager that orchestrates the MCA step within the ACTIONdemonstrator
    pipeline
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:fc3b605e3b4d49aa12adf2e411ec2e7747052addcd24338157eea00a0a55d802
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - FactoMineR
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# multiple-correspondence-analysis

## Summary

Apply multiple correspondence analysis (MCA) with the R library FactoMineR (v2.4) to categorical parent-rated ASEBA CBCL externalizing behavior items to obtain behavioral dimensions and their explained variance fractions in the ACTION cohorts (NTR and LUMC-Curium).

## When to use

Use when behavioral questionnaire items are categorical (e.g., CBCL item scores) and you need to reduce them to a few underlying dimensions and quantify per-dimension explained variance.

## When NOT to use

- The behavioral data are continuous rather than categorical item scores — MCA is designed for categorical variables; use a method suited to continuous data (e.g., PCA/MOFA as used for the omics blocks)
- The goal is integrating multi-omics (methylation/metabolomics) data blocks — the workflow used MOFA2 for those, MCA only for the behavioral CBCL items
- You need significance testing between groups — the workflow ran downstream Mann–Whitney U / Kruskal–Wallis / GEE tests on the MCA dimension scores as a separate step

## Inputs

- Imputed parent-rated CBCL externalizing behavior item scores as categorical variables (CSV, e.g. Synthetic_data/synthetic_cbcl_data.csv)
- CBCL item label mapping (e.g. Synthetic_data/cbcl_labels.csv)

## Outputs

- MCA factor/dimension scores per subject for the behavioral data
- mca_dimension_variance.csv — table of dimension index and fraction of total variation explained per dimension

## How to apply

Load the imputed parent-rated externalizing behavior item scores (e.g., Synthetic_data/synthetic_cbcl_data.csv) into R and run MCA on the categorical CBCL items with the R library FactoMineR (v2.4), as done in the ACTIONdemonstrator workflow to obtain factors for the behavioral data. Extract the eigenvalues from the MCA result and compute the fraction of total variation explained by each dimension. Report the dimensions mirroring the paper's figures as a table with dimension index and fraction of variation explained, then write it to mca_dimension_variance.csv. In the reference study, the first 3 MCA dimensions jointly explained 30% of the variation in 26 externalizing behavior items in NTR (dimensions 1 and 2 explaining 16% and 9%); validate your computed fractions against these reported per-dimension ranges.

## Related tools

- **FactoMineR** (R library used to run the MCA on the categorical behavioral items and extract eigenvalues)
- **Nextflow** (Workflow manager that orchestrates the MCA step within the ACTIONdemonstrator pipeline) — https://github.com/Xomics/ACTIONdemonstrator_workflow

## Examples

```
library(FactoMineR); mca <- MCA(cbcl_items, graph=FALSE)
```

## Evaluation signals

- Computed per-dimension variance fractions reproduce the reported NTR figures: dimension 1 ~16%, dimension 2 ~9%, dimension 3 <5%
- First 3 dimensions jointly explain ~30% of variation in the 26 externalizing behavior items (NTR)
- For LUMC-Curium, the first 2 dimensions together explain ~30% of variation
- Output file mca_dimension_variance.csv exists with the expected columns: dimension index and fraction of variation explained
- MCA runs successfully on the categorical CBCL item matrix within the Nextflow pipeline (action.nf) using FactoMineR v2.4

## Limitations

- Only a synthetic dataset is shared because privacy regulations prohibit sharing potentially personally identifiable clinical data, so exact cohort-level variance fractions may not be reproducible on the synthetic data
- Total explained variance is modest (first 3 dimensions ~30% in NTR), so MCA dimensions capture only part of the behavioral variation
- Describing unsupervised learning workflow steps such as MCA in RO-Crate required the eNanoMapper Ontology because no matching EDAM term was available

## Evidence

- [discussion] FactoMineR used for MCA on behavioral data: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [readme] Behavioral input file for the workflow: "--behavioral_data Synthetic_data/synthetic_cbcl_data.csv"
- [discussion] Downstream use of MCA dimension scores: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters)"
