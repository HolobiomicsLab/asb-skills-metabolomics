---
name: microarray-expression-standardization
description: Use when a microarray expression matrix (e.g. muscle biopsies from DM2/NGT/IGT
  subjects or airway epithelia from smokers and non-smokers) must be gene-wise normalized
  to zero mean and unit variance across samples before translating gene expression
  levels into SVD-derived pathway 'activity' levels.
when_to_use_negative:
- Input is already a normalized feature or activity table (e.g. per-sample pathway
  activity scores); re-standardizing genes is unnecessary and could distort the metagene.
- The analysis goal is single-gene differential expression rather than pathway-level
  aggregation; the article states a pathway-based analysis 'can be complementary to
  those obtained from one more focused on individual genes', not a replacement.
- Expression data are high-throughput sequencing counts rather than microarray intensities;
  the method was demonstrated on microarray expression data and no count-model guidance
  is given here.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3524
tools:
- name: PLAGE
  role: Web program that implements the pathway-level SVD analysis; it consumes the
    gene-standardized matrix and produces pathway activity levels and pathway statistics.
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:7929acf3780a279bcd1d177b8bb4062a7b36c2bd0cb5028b28d9525450b65402
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# microarray-expression-standardization

## Summary

Standardize each gene's microarray expression profile to zero mean and unit variance across samples before computing PLAGE (Pathway Level Analysis of Gene Expression) pathway activity levels via SVD, so that pathway-level features are on a comparable scale for downstream t-statistic ranking.

## When to use

Use when a microarray expression matrix (e.g. muscle biopsies from DM2/NGT/IGT subjects or airway epithelia from smokers and non-smokers) must be gene-wise normalized to zero mean and unit variance across samples before translating gene expression levels into SVD-derived pathway 'activity' levels.

## When NOT to use

- Input is already a normalized feature or activity table (e.g. per-sample pathway activity scores); re-standardizing genes is unnecessary and could distort the metagene.
- The analysis goal is single-gene differential expression rather than pathway-level aggregation; the article states a pathway-based analysis 'can be complementary to those obtained from one more focused on individual genes', not a replacement.
- Expression data are high-throughput sequencing counts rather than microarray intensities; the method was demonstrated on microarray expression data and no count-model guidance is given here.

## Inputs

- Microarray expression matrix (genes × samples), e.g. the Mootha et al. muscle expression data with 17 DM2, 17 NGT, and 9 IGT profiles
- Gene annotation linking rows to genes used for pathway membership
- List of pathway gene sets (e.g. ~400 KEGG and Biocarta pathways)

## Outputs

- Standardized expression matrix with each gene at zero mean and unit variance across samples
- Per-pathway activity level vector (first SVD metagene coefficient) for each sample, suitable for t-statistic comparisons
- Ranked pathway results (e.g. dm2_ngt_pathway_pvalues.csv with pathway names, t statistics, and p-values)

## How to apply

For each gene in the expression matrix, rescale its expression values over the sample set to have zero mean and unit variance. This gene-wise standardization is a prerequisite of the PLAGE workflow: after standardization, each of the roughly 400 KEGG/Biocarta pathways is analyzed by forming its restricted matrix and computing the first SVD metagene, whose first-row coefficient becomes the pathway activity level per sample. The decision to standardize is grounded in the article's goal of translating gene expression levels into pathway 'activity' levels derived from singular value decompositions; without it, gene-scale differences would distort the SVD metagene. Group structure (e.g. 17 DM2, 17 NGT, 9 IGT profiles) is preserved — standardization is applied over all samples, not per group. Validation is done downstream via permutation tests (10,000 label permutations recording the maximum t-statistic) rather than by direct inspection of the normalized matrix.

## Related tools

- **PLAGE** (Web program that implements the pathway-level SVD analysis; it consumes the gene-standardized matrix and produces pathway activity levels and pathway statistics.) — http://dulci.biostat.edu/pathways

## Examples

```
python plage_pipeline.py --expression muscle_expr.csv --groups dm2_ngt_igt_labels.csv --standardize-genes --pathways kegg_biocarta.gmt --permutations 10000 --out dm2_ngt_pathway_pvalues.csv
```

## Evaluation signals

- Each gene's standardized expression vector over all samples should have mean ≈ 0 and variance ≈ 1.
- The restricted pathway matrix for each of the ~400 KEGG/Biocarta pathways yields a first SVD metagene whose first-row coefficient is used as the per-sample pathway activity level.
- Downstream t-statistic ranking (e.g. DM2 vs NGT) should identify the Biocarta 'Activation of cAMP-dependent protein kinase, PKA (protein kinase A)' pathway as the top hit with p ≈ 0.4.
- 10,000 label permutations should yield no pathway reaching genome-wide significance (p ≈ 0.4 for the top pathway), matching the paper's reported null result.
- Comparisons against gene-set enrichment analysis (GSEA) results on the same data should show qualitative agreement for pathways such as nicotine metabolism, mucus production, and glutathione metabolism in smoker/non-smoker comparisons.

## Limitations

- The KEGG and Biocarta pathway versions evolve; the article notes 'The versions used for this article are available at the PLAGE website' without specifying version numbers, so reproducing exact p-values depends on using the same pathway versions.
- The t-statistic formula is referenced as an equation placeholder and is not fully rendered in the available text, making exact reimplementation of the statistic dependent on external documentation.
- The method relies on SVD metagenes whose interpretation can be sensitive to gene set composition and to the standardization choice; no guidance is provided for data types beyond microarray expression.

## Evidence

- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
- [intro] Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD).: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [results] 1. Load the Mootha et al. muscle expression matrix (17 DM2, 17 NGT, 9 IGT; all 43 profiles used) and standardize each gene's expression to zero mean and unit variance over samples.: [evidence span withheld — anchor status: not_found]
- [discussion] The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11].: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11]."
