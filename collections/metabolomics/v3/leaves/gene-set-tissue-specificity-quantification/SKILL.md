---
name: gene-set-tissue-specificity-quantification
description: Use when a multi-tissue differential expression study (e.g., MoTrPAC
  PASS1B rat endurance training, 15 tissues) needs a quantitative measure of DE gene-set
  overlap and tissue-specificity across tissues.
when_to_use_negative:
- The input is a single-tissue DE result with no multi-tissue replicates — cross-tissue
  Jaccard overlap cannot be computed.
- The goal is trait-disease enrichment of exercise DE genes (LDSC/MESC/S-PrediXcan
  analyses) rather than measuring cross-tissue DE concordance.
- DE gene sets have not yet been thresholded (e.g., at 5% IHW α); raw DE outputs will
  inflate overlap estimates.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Produces the per-tissue differential expression results whose gene sets are
    compared for cross-tissue Jaccard overlap
- name: data.table (R)
  role: Used to load the per-tissue MoTrPAC DEA result files from data/external/dea
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:59c382c83f154746537b54309df9eb508df7ad20f13c9d99d2134a3437108a43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - data.table (R)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-set-tissue-specificity-quantification

## Summary

Quantifies how concordant or tissue-specific differential expression gene sets are across tissues by computing the fraction of genes unique to one tissue and all pairwise Jaccard similarities |A∩B|/|A∪B| between tissue DE sets.

## When to use

Use when a multi-tissue differential expression study (e.g., MoTrPAC PASS1B rat endurance training, 15 tissues) needs a quantitative measure of DE gene-set overlap and tissue-specificity across tissues.

## When NOT to use

- The input is a single-tissue DE result with no multi-tissue replicates — cross-tissue Jaccard overlap cannot be computed.
- The goal is trait-disease enrichment of exercise DE genes (LDSC/MESC/S-PrediXcan analyses) rather than measuring cross-tissue DE concordance.
- DE gene sets have not yet been thresholded (e.g., at 5% IHW α); raw DE outputs will inflate overlap estimates.

## Inputs

- MoTrPAC PASS1B differential expression analysis results for 15 rat tissues (files under data/external/dea)
- Rat-to-human ortholog map RGD_ORTHOLOGS_20201001.txt (GENCODE/RGD compiled orthologs)
- Training-differential gene sets per tissue selected at 5% IHW alpha

## Outputs

- jaccard_similarity_matrix.csv (pairwise Jaccard similarity between all 15 tissue DE gene sets)
- Summary statistics table (ortholog mapping fraction, fraction of DE genes unique to one tissue, top-overlapping tissue pairs)

## How to apply

Load MoTrPAC PASS1B differential expression analysis results for the 15 rat tissues from the repository's data/external/dea directory using R's data.table. Map rat DE genes to human orthologs with the compiled GENCODE/RGD ortholog map (RGD_ORTHOLOGS_20201001.txt) before cross-tissue comparison. Compute the fraction of unique DE genes with human orthologs and the fraction of DE genes unique to a single tissue, then compute pairwise Jaccard similarity |A∩B|/|A∪B| between all tissue DE gene sets. Save the 15x15 Jaccard similarity matrix as jaccard_similarity_matrix.csv along with a summary statistics table. In the reference analysis, training-differential genes were selected at 5% IHW α, and gastrocnemius and vastus lateralis was the only tissue pair exceeding a Jaccard index of 0.1 (~0.21).

## Related tools

- **DESeq2** (Produces the per-tissue differential expression results whose gene sets are compared for cross-tissue Jaccard overlap)
- **data.table (R)** (Used to load the per-tissue MoTrPAC DEA result files from data/external/dea)

## Examples

```
library(data.table); de <- lapply(tissues, function(t) fread(file.path('data/external/dea', t))); jac <- function(a, b) length(intersect(a, b)) / length(union(a, b))
```

## Evaluation signals

- Ortholog mapping fraction of unique DE genes with human orthologs equals 94.5%
- Fraction of DE genes unique to a single tissue is approximately 78%
- Gastrocnemius–vastus lateralis Jaccard similarity is approximately 0.21 and is the only tissue pair exceeding 0.1
- jaccard_similarity_matrix.csv is a symmetric 15x15 matrix with diagonal values of 1
- Every Jaccard value lies in [0, 1] and equals |A∩B|/|A∪B| for the corresponding DE gene sets

## Limitations

- Cross-tissue overlap is minimal: only one pair (gastrocnemius and vastus lateralis) exceeds Jaccard 0.1, so small gene-set differences can change rankings
- Identification of rat-human gene orthology is a difficult problem; important biology in disease/exercise-responsive genes without easily established cross-species correspondence is likely missed
- Rats are separated from humans by nearly 140 million years of evolution, complicating cross-species interpretation

## Evidence

- [results] Differentially expressed gene sets show minimal cross-tissue overlap: only one tissue pair (gastrocnemius and vastus lateralis) exceeds a Jaccard index of 0.1, with a Jaccard Similarity of approximately 0.21.: [evidence span withheld — anchor status: not_found]
- [methods] DEA was performed separately in each sex and tissue using filtered raw counts as input for DESeq2: "DEA was performed separately in each sex and tissue using filtered raw counts as input for DESeq2"
- [methods] Training-differential genes were selected at 5% IHW α.: "Training-differential genes were selected at 5% IHW α."
- [discussion] Identification of rat-human gene orthology is another difficult problem: "Identification of rat-human gene orthology is another difficult problem"
- [readme] It primarily hosts scripts written in the R programming language: "It primarily hosts scripts written in the R programming language, but interfacing with several other languages"
