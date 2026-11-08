---
name: gene-set-submatrix-decomposition
description: Use when raw gene-expression data must be translated into per-sample,
  per-pathway activity levels using predefined KEGG/Biocarta gene sets before group
  comparisons such as diabetic vs. non-diabetic or smoker vs. non-smoker.
when_to_use_negative:
- When the goal is to evaluate individual genes rather than group-level pathway activity;
  the article states the pathway-based results can be complementary to one more focused
  on individual genes but this skill itself operates on gene-set submatrices.
- When no predefined pathway/gene-set annotations exist for the organism or platform
  — the SVD must be run per defined pathway submatrix.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Web program implementing pathway level analysis of gene expression via SVD;
    performs the pathway-activity decomposition described here.
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:430a88d1ab99c00725c278d6bf304b06695f8b515ede0d859338344674dc0cc4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  - PLAGE (Pathway Level Analysis of Gene Expression)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gene-set-submatrix-decomposition

## Summary

Decomposes the standardized expression submatrix of each predefined gene set (pathway) via singular value decomposition, taking the first SVD metagene's gene weights and its associated activity-level coefficients per sample as the pathway's activity profile (PLAGE).

## When to use

Use when raw gene-expression data must be translated into per-sample, per-pathway activity levels using predefined KEGG/Biocarta gene sets before group comparisons such as diabetic vs. non-diabetic or smoker vs. non-smoker.

## When NOT to use

- When the goal is to evaluate individual genes rather than group-level pathway activity; the article states the pathway-based results can be complementary to one more focused on individual genes but this skill itself operates on gene-set submatrices.
- When no predefined pathway/gene-set annotations exist for the organism or platform — the SVD must be run per defined pathway submatrix.

## Inputs

- Gene expression matrix (genes x samples) from a microarray study, e.g. the type 2 diabetes muscle expression data with 43 samples (17 DM2, 17 NGT, 9 IGT)
- Predefined pathway gene sets, e.g. ~400 pathways and complexes from KEGG and Biocarta websites

## Outputs

- Per-pathway first metagene weights (first column of W) for each gene in the pathway
- Per-pathway, per-sample activity-level coefficients c_j (first row of C)
- Optional per-pathway lambda squared values quantifying the variation accounted for by the first metagene

## How to apply

Standardize each gene's expression levels to zero mean and unit variance across all samples (e.g. the 43-sample type 2 diabetes muscle dataset with 17 DM2, 17 NGT and 9 IGT profiles). For each pathway from the ~400 KEGG/Biocarta gene sets, assemble the matrix Y whose rows are that pathway's genes and whose columns are samples. Compute the SVD Y = WDC with a linear-algebra library such as NumPy: columns of W are orthonormal metagenes, D holds ordered eigenvalues, rows of C are orthonormal coefficient vectors. Record the first metagene's gene weights (first column of W) and the per-sample activity levels (first row of C); optionally note lambda squared as the fraction of variation captured by the first metagene. The rationale is that features defined by groups of genes are more robust to individual-gene variation than single-gene features, and the first metagene together with its activity levels captures the dominant component of variation in the pathway's expression.

## Related tools

- **PLAGE** (Web program implementing pathway level analysis of gene expression via SVD; performs the pathway-activity decomposition described here.) — http://dulci.biostat.edu/pathways

## Examples

```
import numpy as np
U, s, Vt = np.linalg.svd(Y, full_matrices=False)  # Y: pathway genes x samples
weights, activities = U[:, 0], Vt[0, :]
```

## Evaluation signals

- Verify numerically that the computed SVD reproduces Y = W D C for each pathway submatrix.
- Check that columns of W are orthonormal (W^T W = I) and rows of C are orthonormal (C C^T = I).
- Confirm eigenvalues in D are ordered from highest to lowest as the method defines.
- Spot-check that the first metagene accounts for a large fraction of total variation, e.g. lambda^2/(n_g(n_s-1)) near 0.53 in the analogous smoking dataset.
- Ensure each pathway's submatrix contains only the standardized expression values of that pathway's genes.

## Limitations

- The KEGG and Biocarta pathway versions evolve over time; specific version numbers are not stated and the versions used are only available via the PLAGE website.
- Only the first SVD metagene is used; pathways whose dominant variation is not biologically relevant may be poorly represented.
- Standardization to zero mean and unit variance is required per gene before decomposition; applying the method to unstandardized data is unsupported.

## Evidence

- [abstract] PLAGE defines pathway activity via first SVD metagene: "In this article we present a new pathway based approach to the analysis of gene expression that, while similar in spirit to GSEA, has a number of potential advantages."
- [intro] Objective: derive pathway activity levels via SVD: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [intro] Demonstration datasets: "We demonstrate the approach using the same expression data analyzed by Mootha et al. [9] in their study of type 2 diabetes, and also with expression data from airway epithelia of smokers and"
- [discussion] Robustness rationale: "For one, features defined by groups of genes will tend to be more robust in the face of variation at the level of individual genes."
- [discussion] Workflow: quantify pathway activity per sample: "the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons"
- [discussion] Web program PLAGE: "The SVD-based pathways analysis method has been implemented in a web program [11] called PLAGE (pathway level analysis of gene expression)."
