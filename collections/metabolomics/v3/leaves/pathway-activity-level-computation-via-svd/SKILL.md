---
name: pathway-activity-level-computation-via-svd
description: Use when gene expression data (e.g., 43 type 2 diabetes samples) must
  be converted into per-sample activity levels for ~400 predefined KEGG/Biocarta pathways
  and correlated with a continuous phenotype such as OGTT glucose.
when_to_use_negative:
- The analysis goal is a pairwise two-group comparison (e.g., diabetic vs non-diabetic
  mean activity differences via t statistics), not correlation of pathway activity
  with a continuous phenotype variable
- The input expression data are not gene-level and cannot be standardized per gene
  to zero mean and unit variance across samples
- No predefined pathway gene sets (KEGG/Biocarta) exist for the organism or genes
  in the expression matrix
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Web program that implements the SVD-based pathway activity level computation
    and downstream pathway analyses used by this skill
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:aefa9d8b7c78e5f9bfe2478fe37ae1041349f0d2978f9e62f0e91a5d0c08b9ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pathway-activity-level-computation-via-svd

## Summary

PLAGE-style computation of per-sample pathway 'activity' levels by singular value decomposition (SVD) of standardized, pathway-restricted expression matrices, followed by correlation of activity levels with a quantitative phenotype (e.g., 1-hour OGTT blood glucose) and permutation-based significance testing.

## When to use

Use when gene expression data (e.g., 43 type 2 diabetes samples) must be converted into per-sample activity levels for ~400 predefined KEGG/Biocarta pathways and correlated with a continuous phenotype such as OGTT glucose.

## When NOT to use

- The analysis goal is a pairwise two-group comparison (e.g., diabetic vs non-diabetic mean activity differences via t statistics), not correlation of pathway activity with a continuous phenotype variable
- The input expression data are not gene-level and cannot be standardized per gene to zero mean and unit variance across samples
- No predefined pathway gene sets (KEGG/Biocarta) exist for the organism or genes in the expression matrix

## Inputs

- Gene expression matrix (samples x genes) from a microarray study, e.g., the Mootha et al. type 2 diabetes dataset with 43 samples
- Phenotype vector per sample, e.g., blood glucose concentration 1 hour after oral glucose tolerance test (OGTT)
- Predefined pathway gene sets from KEGG and Biocarta (~400 pathways and complexes), versions available at the PLAGE website

## Outputs

- Per-sample pathway activity level vector (first-row SVD coefficients cj of C) for each pathway
- Pearson correlation coefficient r between each pathway's activity levels and the phenotype (e.g., r = -0.52 for KEGG Oxidative Phosphorylation vs 1-hour OGTT glucose)
- Permutation-based p-value per pathway from 10,000 phenotype permutations with extremal-correlation statistics

## How to apply

Standardize each gene's expression to zero mean and unit variance across all samples (e.g., 43 samples in the Mootha type 2 diabetes dataset). For each of the ~400 KEGG/Biocarta pathways, form the genes-by-samples matrix Y and compute its singular value decomposition Y = WDC; take the first-row coefficients cj of C as that pathway's activity levels per sample. Correlate each pathway's activity levels with the quantitative phenotype (1-hour OGTT blood glucose) using Pearson correlation r. Estimate p-values by permuting the phenotype values 10,000 times, recording the extremal (minimum and maximum) correlations per permutation, and taking the fraction of extremal values stronger than the observed correlation in the corresponding direction. The rationale is that features defined by groups of genes are more robust than individual genes to expression variation, and that pathway activity levels can be used like gene expression levels for downstream comparisons.

## Related tools

- **PLAGE** (Web program that implements the SVD-based pathway activity level computation and downstream pathway analyses used by this skill) — http://dulci.biostat.edu/pathways

## Evaluation signals

- Computed Pearson r for KEGG Oxidative Phosphorylation vs 1-hour OGTT glucose must be approximately -0.52 (tolerance about ±0.05) with the same sign as reported
- Computed r for Biocarta PKA activation (~ +0.47) and KEGG ATP synthesis (~ -0.43) must match Table 1 within about ±0.05 with matching signs
- Permutation p-values (10,000 permutations) must fall within sampling uncertainty of reported values: OxPhos p = 0.03, PKA activation p = 0.06, ATP synthesis p = 0.17
- Ordering of significance must be preserved: OxPhos most significant, then PKA activation, then ATP synthesis
- Gene standardization invariant: each gene's expression has zero mean and unit variance across the 43 samples before SVD

## Limitations

- KEGG and Biocarta pathways continue to expand and evolve; results depend on the pathway versions used, which are available at the PLAGE website but have no stated version numbers or accessions
- Permutation p-values carry sampling uncertainty, so exact reproduction of reported p-values (e.g., 0.03 vs 0.06 vs 0.17) is limited by the 10,000-permutation scheme
- The t-statistic formula used for group-comparison mode is referenced as an equation placeholder and is not fully rendered in the available text

## Evidence

- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
- [intro] Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD).: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [intro] We demonstrate the approach using the same expression data analyzed by Mootha et al. [9] in their study of type 2 diabetes: "We demonstrate the approach using the same expression data analyzed by Mootha et al. [9] in their study of type 2 diabetes"
- [discussion] the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons: "the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons"
- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11].: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11]."
- [discussion] For one, features defined by groups of genes will tend to be more robust in the face of variation at the level of individual genes.: "For one, features defined by groups of genes will tend to be more robust in the face of variation at the level of individual genes."
