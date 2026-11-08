---
name: gaussian-kernel-bandwidth-calibration
description: Use when fitting a semi-parametric kernel machine regression g(Y)=Xβ+h(Z)+ϵ
  on pathway metabolite features and need a Gaussian kernel whose bandwidth is calibrated
  from the data.
when_to_use_negative:
- The pathway graph is disconnected and has not been regularized — the article notes
  the unregularized normalized Laplacian hindered testing performance on disconnected
  graphs
- Very large cohorts (over 1,000 observations) where p-values from the compared methods
  were indistinguishable from one another, so the pathway-smoothed kernel adds little
  practical gain
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
tools:
- name: R
  role: Implementation language for the kernel machine regression, Laplacian smoothing,
    and Gaussian kernel construction
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Computes the normalized Laplacian of the pathway graph prior to regularization
    and kernel construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:2679c26ad1de14235f3fc1e4aec625534f3d8a33778e38fa2407fe19113806a9
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
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gaussian-kernel-bandwidth-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the Gaussian (radial basis) kernel for PaIRKAT's kernel machine regression using metabolite features smoothed through the regularized normalized Laplacian of a pathway graph, with the bandwidth parameter set to the median pairwise distance.

## When to use

Use when fitting a semi-parametric kernel machine regression g(Y)=Xβ+h(Z)+ϵ on pathway metabolite features and need a Gaussian kernel whose bandwidth is calibrated from the data.

## When NOT to use

- The pathway graph is disconnected and has not been regularized — the article notes the unregularized normalized Laplacian hindered testing performance on disconnected graphs
- Very large cohorts (over 1,000 observations) where p-values from the compared methods were indistinguishable from one another, so the pathway-smoothed kernel adds little practical gain

## Inputs

- Metabolite feature matrix restricted to a single KEGG pathway (e.g. one of the 28 pathways with measurements on ≥10 metabolites)
- Regularized normalized Laplacian of the pathway graph (built from KEGG reaction-database edges)
- Covariate matrix X and phenotype vector Y

## Outputs

- Gaussian (radial basis) kernel matrix K on the smoothed pathway features with bandwidth ρ = median pairwise distance
- Kernel score test statistic for the pathway variance component
- P-value for the phenotype–pathway association test

## How to apply

First smooth the pathway metabolite features through the regularized normalized Laplacian of the pathway graph, computed with igraph and the regularization kernel with τ = 1. Then compute pairwise distances between the smoothed feature vectors and construct the Gaussian (radial basis) kernel, setting the bandwidth ρ to the median pairwise distance. This kernel enters the kernel machine model as the covariance of the random effect h(Z), so that pathway graph topography directly shapes the similarity structure of the test statistic. The score test statistic for the variance component and its p-value are then computed as described in Section 2 of the paper, with τ = 1 fixed across simulations and the COPDGene real-data analyses.

## Related tools

- **R** (Implementation language for the kernel machine regression, Laplacian smoothing, and Gaussian kernel construction) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Computes the normalized Laplacian of the pathway graph prior to regularization and kernel construction)

## Examples

```
R: source('PaIRKAT.R'); K <- gaussKernel(L_reg %*% Z, rho = median(dist(L_reg %*% Z)))
```

## Evaluation signals

- The kernel matrix is symmetric positive semi-definite with unit diagonal after Gaussian construction on the smoothed features
- Bandwidth ρ equals the median of the pairwise distance matrix computed on the Laplacian-smoothed features (not the raw metabolite intensities)
- Running the example workflow with τ = 1 yields a score test statistic and p-value for each of the 28 KEGG pathways analyzed
- Comparing against the unregularized Laplacian on a disconnected graph shows performance not degraded, per the paper's reported robustness

## Limitations

- The regularization parameter τ is fixed to 1 for all simulations and data analyses, with no sensitivity analysis of τ reported
- Mathematical formulas for the regularization kernel appear as blank placeholders in the document text, so the exact regularization function must be recovered from the cited derivation reference
- Method performance differences in real data were only demonstrated on downsampled subsets (100–500 subjects), not the full COPDGene cohort
- The filtered dataset did not contain every metabolite within the 28 KEGG pathways selected, so some analyzed pathways had fewer than 10 metabolites

## Evidence

- [discussion] Using the normalized Laplacian, hindered testing performance compared to using PaIRKAT or ignoring the pathway information when a graph was disconnected.: [evidence span withheld — anchor status: not_found]
