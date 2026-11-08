---
name: gene-expression-standardization-and-matrix-construction
description: Use when expression data must be standardized per gene and pathway-restricted
  matrices built as input to an SVD-based pathway activity analysis such as PLAGE,
  or when reproducing metagene results.
when_to_use_negative:
- Expression values are already standardized or the analysis is not SVD-based pathway
  activity estimation — double standardization would distort metagene weights
- The goal is single-gene differential expression rather than pathway-level activity
  levels; per-gene statistics do not need pathway-restricted matrices
- Pathway membership files are missing; building Y for undefined gene sets has no
  basis
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Provides the web program implementing the standardization and pathway-matrix
    construction preceding SVD activity level computation
  repo: http://dulci.biostat.edu/pathways
- name: KEGG
  role: Source of pathway gene sets whose gene lists define which rows of the standardized
    expression matrix are assembled into each Y
- name: Biocarta
  role: Source of pathway gene sets whose gene lists define which rows of the standardized
    expression matrix are assembled into each Y
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:b3a77760967f46fd996ca7fc14bb29f07bee3c2e8837373d9ea8301ece0b0e94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biocarta
  - KEGG
  - PLAGE
  - PLAGE (Pathway Level Analysis of Gene Expression)
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-expression-standardization-and-matrix-construction

## Summary

Standardizes each gene's expression values to zero mean and unit variance across samples, then assembles the standardized values of a pre-defined pathway's genes into a genes-by-samples matrix Y. This preprocessing is the prerequisite for computing the pathway's singular value decomposition and first-metagene activity levels in PLAGE.

## When to use

Use when expression data must be standardized per gene and pathway-restricted matrices built as input to an SVD-based pathway activity analysis such as PLAGE, or when reproducing metagene results.

## When NOT to use

- Expression values are already standardized or the analysis is not SVD-based pathway activity estimation — double standardization would distort metagene weights
- The goal is single-gene differential expression rather than pathway-level activity levels; per-gene statistics do not need pathway-restricted matrices
- Pathway membership files are missing; building Y for undefined gene sets has no basis

## Inputs

- Gene expression matrix (genes x samples), e.g. type 2 diabetes muscle expression data from Whitehead Institute / GEO, or airway epithelium profiles (75 subjects)
- Pathway gene sets in KEGG or Biocarta format (~400 pathways and complexes)
- Phenotype/clinical variables (e.g. 1-hour OGTT blood glucose, smoking status) for downstream association

## Outputs

- Per-gene standardized expression matrix (zero mean, unit variance per gene)
- One genes-by-samples matrix Y per pathway, containing only that pathway's standardized expression levels
- Numeric table ready for SVD decomposition (Y = WDC)

## How to apply

For each gene, rescale its expression levels across the cohort to zero mean and unit variance (PLAGE standardization applied before any pathway SVD). For each pathway of interest (~400 KEGG and Biocarta gene sets in the paper), collect the rows of the standardized expression matrix corresponding to that pathway's genes, forming Y with rows as genes and columns as samples (e.g., 43 type 2 diabetes samples: 17 DM2, 17 NGT, 9 IGT; or 75 airway epithelium subjects: 34 current, 18 former, 23 never smokers). Decision point: pathways with very few genes yield unstable SVDs — verify orthonormality of the resulting metagenes before proceeding. The rationale is that standardized values remove gene-scale differences so the SVD metagene weights and activity levels are directly comparable across pathways. Write Y (or its per-pathway restriction) to a numeric table ready for the downstream SVD step.

## Related tools

- **PLAGE** (Provides the web program implementing the standardization and pathway-matrix construction preceding SVD activity level computation) — http://dulci.biostat.edu/pathways
- **KEGG** (Source of pathway gene sets whose gene lists define which rows of the standardized expression matrix are assembled into each Y)
- **Biocarta** (Source of pathway gene sets whose gene lists define which rows of the standardized expression matrix are assembled into each Y)

## Examples

```
Y = standardized_expr.loc[pathway_genes]  # rows=genes, cols=samples, per-gene z-scored
```

## Evaluation signals

- Verify the standardized expression matrix has per-gene mean ≈ 0 and standard deviation ≈ 1 across all samples (e.g. 43 diabetes samples or 75 airway epithelium subjects)
- Check that each pathway matrix Y contains exactly the genes listed in its KEGG/Biocarta gene set and no others
- Confirm Y's dimensions: rows = number of genes in the pathway, columns = number of samples (e.g. 43 for diabetes, 75 for airway)
- Spot-check that Y reproduces the standardized expression values for a random gene-sample pair
- Validate downstream orthonormality W^T W = I, C C^T = I as an indirect check that Y was constructed correctly

## Limitations

- KEGG and Biocarta pathways evolve; versions differ from those used in the article and results may shift accordingly
- Pathways with very few genes produce unstable SVDs and unreliable first-metagene activity levels
- PLAGE results are complementary to, not a replacement for, individual-gene-focused analyses

## Evidence

- [methods] Per-gene standardization before SVD: [evidence span withheld — anchor status: not_found]
- [methods] Pathway matrix construction: [evidence span withheld — anchor status: not_found]
- [discussion] Pathway sources: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [methods] Cohort description: [evidence span withheld — anchor status: not_found]
- [discussion] Role of pathway activity levels: "the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons"
