---
name: pathway-coexpression-coordination-quantification
description: Use when you have expression profiles plus predefined KEGG/Biocarta gene-sets
  and need to measure pathway-wide expression coordination, or to test whether GSEA's
  top-ranked sets are merely coordinately expressed rather than differentially active
  between groups.
when_to_use_negative:
- When the goal is to test whether a pathway's mean activity level differs between
  groups (the PLAGE pairwise-comparison use case with permuted t-statistics) rather
  than to quantify how coordinately the pathway's genes are expressed
- When genes are not organized into predefined gene-sets (no KEGG/Biocarta membership
  lists available), since each score requires a restricted gene-by-sample matrix per
  named pathway
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway analysis used to derive pathway
    activity/coordination scores from the expression matrices
  repo: http://dulci.biostat.edu/pathways
- name: GSEA
  role: 'Benchmark used for comparison: enrichment scores and ranks (signal-to-noise
    gene ranking, 1000 random permutations) are joined to the eigenvalue fractions
    to test what GSEA''s top-ranked sets represent'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:a09c6a54b2a300d67263377a7ad3a331691241a8fe1f628412e64553c3c14598
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSEA
  - PLAGE
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pathway-coexpression-coordination-quantification

## Summary

Quantifies how strongly a predefined pathway's genes are coordinately expressed by computing the fraction of total variation captured by the first singular value (λ²/(ng·ns)) of the restricted gene-by-sample matrix, and relates this coordination score to gene-set enrichment (GSEA) rankings.

## When to use

Use when you have expression profiles plus predefined KEGG/Biocarta gene-sets and need to measure pathway-wide expression coordination, or to test whether GSEA's top-ranked sets are merely coordinately expressed rather than differentially active between groups.

## When NOT to use

- When the goal is to test whether a pathway's mean activity level differs between groups (the PLAGE pairwise-comparison use case with permuted t-statistics) rather than to quantify how coordinately the pathway's genes are expressed
- When genes are not organized into predefined gene-sets (no KEGG/Biocarta membership lists available), since each score requires a restricted gene-by-sample matrix per named pathway

## Inputs

- Gene expression matrix standardized to zero mean and unit variance over samples (e.g., Mootha et al. type 2 diabetes profiles: 43 samples — 17 DM2, 17 NGT, 9 IGT; or airway epithelium profiles from AGED: 75 subjects — 34 current, 18 former, 23 never smokers)
- Predefined pathway gene-sets (~400 from KEGG and Biocarta) giving gene membership per pathway

## Outputs

- Per-pathway first-eigenvalue fraction λ²/(ng·ns) quantifying pathway expression coordination
- Joined table (eigenvalue_fraction_vs_gsea_rank.csv) of each pathway's first-eigenvalue fraction against its GSEA enrichment score and rank in each two-group comparison

## How to apply

Standardize all gene expression levels to zero mean and unit variance over samples. For each of the ~400 KEGG/Biocarta pathway gene-sets, form the restricted gene-by-sample matrix Y (rows = pathway genes ng, columns = samples ns) and compute its singular value decomposition Y = WDC, recording the first eigenvalue λ. Compute the first-eigenvalue fraction λ²/(ng·ns) as the pathway coordination score; expected landmarks are ribosome ≈0.53 in the smoking data, Oxidative Phosphorylation ≈0.23, and ≈0.05 for uncorrelated Gaussian noise of ribosome size averaged over 1000 simulated sets. To relate coordination to enrichment, run GSEA on the same pathway collection for two-group comparisons, ranking genes by signal-to-noise ratio with 1000 random permutations, then join each pathway's first-eigenvalue fraction to its GSEA ES and rank. The rationale (PLAGE) is that the leading singular vector captures a pathway's dominant coordinated expression pattern, and pathway-level features defined by groups of genes are more robust to individual-gene variation than gene-by-gene statistics.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway analysis used to derive pathway activity/coordination scores from the expression matrices) — http://dulci.biostat.edu/pathways
- **GSEA** (Benchmark used for comparison: enrichment scores and ranks (signal-to-noise gene ranking, 1000 random permutations) are joined to the eigenvalue fractions to test what GSEA's top-ranked sets represent)

## Examples

```
W, d, C = np.linalg.svd(Y_standardized, full_matrices=False); first_eigenvalue_fraction = d[0]**2 / (ng * ns)
```

## Evaluation signals

- The ribosome gene-set's first metagene accounts for ~0.53 (over fifty percent) of total variation in the smoking data set
- Oxidative Phosphorylation scores ~0.23 and uncorrelated Gaussian noise of ribosome size (averaged over 1000 simulated sets) scores ~0.05, bracketing the expected range
- The ribosome gene-set has the highest enrichment score in three of the four GSEA comparisons (e.g., ribosome ES=304 p=0.047 former-vs-never), confirming top-ranked sets are strongly coordinately expressed
- The output table joins every pathway's first-eigenvalue fraction to its GSEA ES and rank in each of the four comparisons with no missing pathway rows
- Fraction values fall well below ~0.05 for pathways whose genes show no coordinated expression, matching the size-matched Gaussian noise baseline

## Limitations

- KEGG and Biocarta pathways continue to expand and evolve; the versions used are available at the PLAGE website but no specific version numbers or accessions are stated
- A high enrichment score does not guarantee statistical significance (e.g., ribosome ES=162 p=0.17 current-vs-never; Lectin Induced Complement Pathway ES=128 p=0.52 current-vs-former), so rankings must be interpreted alongside p-values
- GSEA does not effectively distinguish sets with consistent group differences from sets whose genes are merely coordinately expressed — the very ambiguity this coordination score is meant to expose

## Evidence

- [intro] Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD).: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] For one, features defined by groups of genes will tend to be more robust in the face of variation at the level of individual genes.: "For one, features defined by groups of genes will tend to be more robust in the face of variation at the level of individual genes."
