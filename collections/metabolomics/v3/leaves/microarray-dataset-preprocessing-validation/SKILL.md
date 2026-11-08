---
name: microarray-dataset-preprocessing-validation
description: Use when you have a gene-expression matrix (e.g. 17 DM2, 17 NGT, 9 IGT
  muscle profiles, or smoker/non-smoker airway epithelia data) and a set of predefined
  pathway gene lists (KEGG/Biocarta, ~400 pathways), and you need standardized per-gene
  expression plus a validated singular-value decomposition.
when_to_use_negative:
- When the input is already a pathway-activity table or a downstream result matrix
  — this skill's purpose is upstream standardization and SVD verification, not result
  post-processing.
- When the dataset lacks predefined pathway gene lists (e.g. no KEGG or Biocarta annotations)
  — SVD of arbitrary gene subsets without a curated pathway definition is outside
  the scope of PLAGE.
- When a non-SVD pathway-activity method (e.g. GSEA enrichment scores) is already
  applied to the same standardized matrix and the goal is simply to compare scores,
  not to generate the activity levels themselves.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_0621
tools:
- name: PLAGE
  role: Web program implementing the pathway-level SVD analysis described here; used
    to perform the standardization, decomposition, and validation workflow.
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:5e9ea09220ad1f99765a26f0f175695f7c2e84def5c15f0a74bb0807481223f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# microarray-dataset-preprocessing-validation

## Summary

Standardizes each gene's microarray expression levels to zero mean and unit variance across samples, then verifies the integrity of the resulting pathway-level SVD decomposition (orthonormality of metagenes and coefficient vectors, eigenvalue ordering, reconstruction accuracy) before downstream pathway-activity analysis.

## When to use

Use when you have a gene-expression matrix (e.g. 17 DM2, 17 NGT, 9 IGT muscle profiles, or smoker/non-smoker airway epithelia data) and a set of predefined pathway gene lists (KEGG/Biocarta, ~400 pathways), and you need standardized per-gene expression plus a validated singular-value decomposition before computing pathway activity levels.

## When NOT to use

- When the input is already a pathway-activity table or a downstream result matrix — this skill's purpose is upstream standardization and SVD verification, not result post-processing.
- When the dataset lacks predefined pathway gene lists (e.g. no KEGG or Biocarta annotations) — SVD of arbitrary gene subsets without a curated pathway definition is outside the scope of PLAGE.
- When a non-SVD pathway-activity method (e.g. GSEA enrichment scores) is already applied to the same standardized matrix and the goal is simply to compare scores, not to generate the activity levels themselves.

## Inputs

- Raw gene-expression matrix (samples × genes), e.g. type 2 diabetes muscle expression data (17 DM2, 17 NGT, 9 IGT profiles)
- Pathway gene lists (~400 pathways) from KEGG and Biocarta
- Sample-group labels (e.g. diabetic vs non-diabetic, smoker vs non-smoker) for downstream permutation testing

## Outputs

- Standardized (zero-mean, unit-variance per gene) expression matrix ready for pathway-level SVD
- Per-pathway singular value decomposition components: first metagene weights (first column of W) and activity-level coefficients c_j (first row of C)
- Numerical validation report confirming WᵀW = I, CCᵀ = I, eigenvalue ordering, and reconstruction of Y from WDC
- Output table of all pathways' first metagene weights and activity-level coefficients (e.g. pathways_plage_activities.csv)

## How to apply

Load the raw expression matrix (e.g. 43 diabetes samples or the cigarette-smoke airway epithelia set) and the ~400 pathway gene sets from KEGG and Biocarta. For each gene, standardize its expression levels to zero mean and unit variance across all samples in the dataset. Then, for each pathway, build the pathway matrix Y (rows = that pathway's genes, columns = samples) and compute its singular value decomposition Y = WDC using a linear-algebra library such as NumPy. Validation consists of checking that Y is numerically reproduced by WDC, that columns of W and rows of C are orthonormal (WᵀW = I and CCᵀ = I), and that the eigenvalues in D are ordered from highest to lowest, as required by the PLAGE method's first-metagene definition.

## Related tools

- **PLAGE** (Web program implementing the pathway-level SVD analysis described here; used to perform the standardization, decomposition, and validation workflow.) — http://dulci.biostat.edu/pathways

## Examples

```
python -c "import numpy as np; Y = np.loadtxt('pathway_matrix.csv'); W, D, C = np.linalg.svd(Y, full_matrices=False); assert np.allclose(W @ np.diag(D) @ C, Y, atol=1e-10); assert np.allclose(W.T @ W, np.eye(W.shape[1]), atol=1e-10); assert np.all(np.diff(D) <= 0)"
```

## Evaluation signals

- Check that the reconstructed matrix W·D·Cᵀ matches the original standardized pathway matrix Y within a small numerical tolerance (e.g. element-wise absolute error < 1e-10).
- Verify that WᵀW ≈ I (columns of W orthonormal) and that CCᵀ ≈ I (rows of C orthonormal), as required by the SVD definition used in the PLAGE method.
- Confirm that the diagonal entries of D (eigenvalues) are sorted in descending order, so that the first metagene and its associated activity level correspond to the largest component of variation.
- Sanity-check a known pathway: in the analogous smoking dataset, the ribosome metagene accounts for ~53% of total variation (λ²/(n·(n−1)) ≈ 0.53); the corresponding value for the diabetes dataset should be a similarly large fraction, indicating the first component dominates.
- Verify that the output table (pathways_plage_activities.csv) contains one row per pathway with the first metagene weights and one activity-level coefficient c_j per sample, and that no pathway is missing its SVD components.

## Limitations

- The KEGG and Biocarta pathways continue to expand and evolve; the versions used for this article are only available at the PLAGE website and no version numbers or accessions are stated in the text.
- The t-statistic formula referenced for downstream comparison is referenced but not rendered in the provided text, so permutation-based significance testing cannot be fully reproduced from this excerpt alone.
- The SVD is computed per pathway separately; this skill does not validate cross-pathway consistency or the biological relevance of the extracted metagenes beyond the orthonormality and ordering invariants.

## Evidence

- [methods] Standardization to zero mean and unit variance before pathway SVD: [evidence span withheld — anchor status: not_found]
- [methods] Definition of pathway activity level as the first SVD metagene: [evidence span withheld — anchor status: not_found]
- [methods] Validation of orthonormality and eigenvalue ordering: [evidence span withheld — anchor status: not_found]
- [discussion] Permutation testing parameters for downstream comparison: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels"
- [abstract] PLAGE web tool availability: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
