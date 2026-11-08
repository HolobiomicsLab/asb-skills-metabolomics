---
name: pathway-gene-set-mapping-kegg-biocarta
description: Use when a genes-by-samples expression matrix (e.g., microarray) must
  be decomposed into predefined KEGG/Biocarta pathway gene sets (~400 pathways and
  complexes) as input to SVD-based pathway activity analysis.
when_to_use_negative:
- You need de novo discovery of gene groups from the expression data itself — this
  skill only maps genes onto the predefined KEGG/Biocarta gene sets, not clusters
  derived from the data.
- Input is already pathway activity levels or a reduced pathway-level table — re-mapping
  genes to KEGG/Biocarta sets would redo work downstream of this step.
- Your gene identifiers do not overlap the KEGG/Biocarta namespace for the platform
  — intersections would silently shrink or empty the per-pathway matrices.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3308
tools:
- name: PLAGE
  role: Web program that performs the SVD-based pathway activity analysis on the KEGG/Biocarta
    gene sets mapped here; also archives the exact pathway versions used in the article
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:6b367bcba0d166d08d7fd5bc5a0070d1bb862b0cb9c9d8c1924ef97f41051604
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pathway-gene-set-mapping-kegg-biocarta

## Summary

Maps genes from a gene-expression dataset onto the ~400 predefined KEGG and Biocarta pathway/complex gene sets used by PLAGE, yielding one genes-by-samples matrix Y per pathway whose SVD yields pathway activity levels.

## When to use

Use when a genes-by-samples expression matrix (e.g., microarray) must be decomposed into predefined KEGG/Biocarta pathway gene sets (~400 pathways and complexes) as input to SVD-based pathway activity analysis.

## When NOT to use

- You need de novo discovery of gene groups from the expression data itself — this skill only maps genes onto the predefined KEGG/Biocarta gene sets, not clusters derived from the data.
- Input is already pathway activity levels or a reduced pathway-level table — re-mapping genes to KEGG/Biocarta sets would redo work downstream of this step.
- Your gene identifiers do not overlap the KEGG/Biocarta namespace for the platform — intersections would silently shrink or empty the per-pathway matrices.

## Inputs

- Gene expression matrix (genes x samples), e.g., the Mootha et al. type 2 diabetes microarray data with 43 samples
- Phenotype/label data paired to samples (e.g., 1-hour OGTT blood glucose concentrations, diabetic/non-diabetic labels)
- KEGG pathway gene sets (article-frozen versions archived at the PLAGE website)
- Biocarta pathway/complex gene sets (article-frozen versions archived at the PLAGE website)

## Outputs

- Gene-to-pathway membership mapping assigning dataset genes to the ~400 KEGG/Biocarta pathways and complexes
- One genes-by-samples matrix Y per pathway (gene rows standardized to zero mean/unit variance across samples)
- Per-pathway activity levels per sample (first-row coefficients cj of C from Y = WDC) for correlation or group-comparison analysis

## How to apply

Retrieve the KEGG and Biocarta pathway definitions (~400 pathways and complexes); because both resources 'continue to expand and evolve', use the article-frozen versions archived at the PLAGE website rather than a live pull. Intersect each pathway's member genes with the genes present in the expression study (e.g., the Mootha et al. type 2 diabetes dataset of 43 samples) and build one genes-by-samples matrix Y per pathway, excluding genes not in that set. Standardize each gene's expression to zero mean and unit variance across samples before forming/decomposing Y. Decompose each Y by singular value decomposition (Y = WDC) and take the first-row coefficients cj of C as that pathway's activity levels, per sample. Keep the gene-to-pathway mapping fixed across samples and permutations so downstream tests (Pearson correlation against 1-hour OGTT glucose, or t statistics with 10,000 label permutations) interrogate activity, not set membership. Validate by reproducing Table 1 targets, e.g., KEGG Oxidative Phosphorylation r = -0.52 (p = 0.03) versus 1-hour OGTT glucose.

## Related tools

- **PLAGE** (Web program that performs the SVD-based pathway activity analysis on the KEGG/Biocarta gene sets mapped here; also archives the exact pathway versions used in the article) — http://dulci.biostat.edu/pathways

## Examples

```
import numpy as np; activity = np.linalg.svd(Y)[2][0, :]  # first-row coefficients of C in Y = WDC (PLAGE activity levels)
```

## Evaluation signals

- Mapped gene sets cover about 400 KEGG/Biocarta pathways and complexes, matching the article's stated pathway universe.
- Each per-pathway matrix Y has that pathway's member genes as rows and the same 43 sample columns as the source diabetes expression matrix.
- Standardization invariant: every gene row in each Y has mean 0 and variance 1 across samples before SVD.
- Downstream PLAGE on the mapped sets reproduces Table 1: KEGG Oxidative Phosphorylation r = -0.52 (p = 0.03), Biocarta PKA activation r = +0.47 (p = 0.06), KEGG ATP synthesis r = -0.43 (p = 0.17), within ±0.05 on r with identical signs and ordering (OxPhos most significant).
- Permutation p-values from 10,000 glucose-label permutations fall within sampling uncertainty of the reported values.

## Limitations

- KEGG and Biocarta pathways continue to expand and evolve; no version numbers or accessions are stated in the article — the frozen versions used are only available at the PLAGE website.
- Coverage is limited to the ~400 predefined KEGG/Biocarta pathways and complexes; genes belonging to none of these sets contribute nothing at the pathway level.
- Pathway activity levels depend on SVD of the standardized genes-by-samples matrix, so very small intersections between a pathway and the arrayed genes make the resulting first-row coefficients unstable.

## Evidence

- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [intro] Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD).: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [discussion] the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons: "the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons"
- [discussion] The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11].: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11]."
- [intro] We demonstrate the approach using the same expression data analyzed by Mootha et al. [9] in their study of type 2 diabetes: "We demonstrate the approach using the same expression data analyzed by Mootha et al. [9] in their study of type 2 diabetes"
- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
