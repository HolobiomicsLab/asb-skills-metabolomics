---
name: semiparametric-regression-implementation
description: Use when testing phenotype–pathway association in metabolomics data where
  pathway graphs are available and you need a kernel machine score test whose kernel
  is built on regularized (smoothed) pathway features rather than raw metabolite intensities.
when_to_use_negative:
- When no pathway graph connectivity information exists for the metabolites — the
  regularized normalized Laplacian requires graph structure
- When the pathway graph is disconnected and no inspection/regularization decision
  has been made — the article warns that the unregularized normalized Laplacian 'hindered
  testing performance' on disconnected graphs
- When the goal is simply associating a phenotype with individual metabolites rather
  than pathway-level tests
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: R
  role: Implementation language for the kernel machine model fit and score test
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Compute the normalized Laplacian of the pathway graph prior to regularization
- name: impute (R package)
  role: k-nearest neighbors imputation (k = 10) of missing metabolite values before
    model fitting
- name: KEGGREST
  role: Retrieve pathway-metabolite links (keggLink function) defining which metabolites
    belong to each pathway graph
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:2882d1a63231ae24530f518fb10d4f3f7f62793429c20668f2c78e0486abad74
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  repo_url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - KEGGREST
  - R
  - igraph
  - impute (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# semiparametric-regression-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits the semi-parametric kernel machine regression model g(Y)=Xβ+h(Z)+ϵ underlying PaIRKAT, estimating covariate effects β under the null and treating h(Z) as a random effect with covariance proportional to a pathway-informed Gaussian kernel, then computing the kernel score test statistic and p-value per pathway.

## When to use

Use when testing phenotype–pathway association in metabolomics data where pathway graphs are available and you need a kernel machine score test whose kernel is built on regularized (smoothed) pathway features rather than raw metabolite intensities.

## When NOT to use

- When no pathway graph connectivity information exists for the metabolites — the regularized normalized Laplacian requires graph structure
- When the pathway graph is disconnected and no inspection/regularization decision has been made — the article warns that the unregularized normalized Laplacian 'hindered testing performance' on disconnected graphs
- When the goal is simply associating a phenotype with individual metabolites rather than pathway-level tests

## Inputs

- Pathway graph adjacency matrix (edges from the KEGG reaction database)
- Metabolite intensity matrix (KEGG-annotated metabolites; e.g., 436 metabolites mapped to KEGG pathways)
- Covariate matrix (e.g., white blood cell count, percent eosinophil, lymphocytes, monocytes, neutrophils, hemoglobin)
- Phenotype vector (e.g., log FEV1/FVC ratio or percent emphysema from COPDGene)

## Outputs

- Kernel score test statistic per pathway
- P-value per pathway (e.g., for the 28 KEGG pathways with measurements on at least 10 metabolites)
- Estimated covariate effects β under the null model

## How to apply

In R, load the pathway graph adjacency matrix plus the metabolite, covariate, and phenotype matrices from the PaIRKAT example workflow (github.com/CharlieCarpenter/PaIRKAT). Compute the normalized Laplacian of the pathway graph with igraph and apply the regularization kernel with τ = 1 to obtain the regularized normalized Laplacian that smooths metabolite features over pathway structure. Project the pathway metabolite features through this regularized Laplacian and build a Gaussian (radial basis) kernel on the smoothed features with bandwidth ρ set to the median pairwise distance. Fit g(Y)=Xβ+h(Z)+ϵ by estimating β under the null while treating h(Z) as a random effect with covariance proportional to the kernel matrix, then compute the variance-component kernel score test statistic and its p-value as described in Section 2 of the paper. Precede the fit with the paper's preprocessing: divide metabolite intensities by run-day median and rescale to the overall metabolite median, kNN-impute missing values (k = 10, impute package), natural log transform and standardize, and use partial residuals from per-metabolite covariate regressions (white blood cell count, percent eosinophil/lymphocytes/monocytes/neutrophils, hemoglobin).

## Related tools

- **R** (Implementation language for the kernel machine model fit and score test) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Compute the normalized Laplacian of the pathway graph prior to regularization)
- **impute (R package)** (k-nearest neighbors imputation (k = 10) of missing metabolite values before model fitting)
- **KEGGREST** (Retrieve pathway-metabolite links (keggLink function) defining which metabolites belong to each pathway graph)

## Examples

```
Rscript PaIRKAT_example.R  # loads pathway adjacency + metabolite/covariate/phenotype matrices, regularizes Laplacian with tau=1, fits g(Y)=Xbeta+h(Z)+epsilon, returns score test p-value per pathway
```

## Evaluation signals

- Running the example workflow with τ = 1 produces a score test statistic and p-value for each pathway
- Including pathway information via the regularized normalized Laplacian yields more powerful tests (lower p-values / higher power) than ignoring pathway information
- The unregularized normalized Laplacian variant should underperform PaIRAT on disconnected graphs — a sanity contrast for the regularization step
- P-values from competing methods should be indistinguishable for large datasets (over 1,000 observations); differences appear only in downsampled subsets of 100–500 subjects
- Filtered analysis set should match paper expectations, e.g., 1,113 complete cases for FEV1/FVC and 1,065 for percent emphysema after removal of incomplete cases

## Limitations

- Regularization parameter τ was fixed to 1 in all simulations and data analyses with no sensitivity analysis reported
- It is important to examine graph structure before applying regularization; sparsity in metabolomic pathways can greatly hinder performance
- The filtered dataset did not contain every metabolite in the selected KEGG pathways, so some analyzed pathways had fewer than 10 metabolites
- Real-data performance differences were demonstrated only on random subsets of 100–500 subjects, not the full cohort (over 1,000 observations showed indistinguishable p-values)

## Evidence
