---
name: jaccard-similarity-matrix-computation
description: Use when you have binary gene membership sets (e.g., per-tissue DE gene
  lists from the MoTrPAC rat EET study) and need to measure how concordant or tissue-specific
  the sets are across conditions or tissues.
when_to_use_negative:
- You need signed concordance of direction-of-effect (up vs down regulation) rather
  than binary gene-set overlap — Jaccard ignores log2FC sign
- Your DE sets are not ortholog-mapped across species; comparing raw rat gene IDs
  to human gene IDs will inflate/deflate overlap spuriously
- The input is continuous gene-level statistics (p-values, z-scores) rather than discrete
  membership sets — use correlation-based methods instead
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Produces the per-tissue differential expression results (filtered raw counts
    as input, run separately per sex and tissue) whose gene sets feed the Jaccard
    computation
- name: data.table
  role: Used in R to load the 15-tissue DEA results from data/external/dea
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:6a1ffddd52c12d2cb8c8eefee64331c091bc07b3942dc0dfa12ec3deac842b0b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - data.table
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# jaccard-similarity-matrix-computation

## Summary

Quantifies cross-tissue concordance of exercise-responsive transcription by computing pairwise Jaccard similarity (|A∩B|/|A∪B|) between differentially expressed gene sets from the 15 tissues of the MoTrPAC PASS1B rat endurance exercise training (EET) study, after mapping rat genes to human orthologs.

## When to use

Use when you have binary gene membership sets (e.g., per-tissue DE gene lists from the MoTrPAC rat EET study) and need to measure how concordant or tissue-specific the sets are across conditions or tissues.

## When NOT to use

- You need signed concordance of direction-of-effect (up vs down regulation) rather than binary gene-set overlap — Jaccard ignores log2FC sign
- Your DE sets are not ortholog-mapped across species; comparing raw rat gene IDs to human gene IDs will inflate/deflate overlap spuriously
- The input is continuous gene-level statistics (p-values, z-scores) rather than discrete membership sets — use correlation-based methods instead

## Inputs

- MoTrPAC PASS1B per-tissue differential expression analysis results across 15 tissues (data/external/dea, read with R data.table)
- Rat-to-human ortholog map RGD_ORTHOLOGS_20201001.txt (compiled GENCODE/RGD orthologs)
- Training-differential gene sets selected at 5% IHW alpha

## Outputs

- jaccard_similarity_matrix.csv (15x15 pairwise Jaccard similarity matrix)
- Summary table with fraction of unique DE genes with human orthologs and fraction of DE genes unique to a single tissue

## How to apply

Load the MoTrPAC PASS1B differential expression results for all 15 tissues from data/external/dea using R data.table; DE genes derive from DESeq2 differential expression analysis performed separately in each sex and tissue. Map rat genes to human orthologs using the compiled GENCODE/RGD ortholog map (RGD_ORTHOLOGS_20201001.txt) before comparing sets, and record the ortholog mapping fraction. For every tissue pair, compute Jaccard similarity as |A∩B|/|A∪B| of the ortholog-mapped DE gene sets, and additionally compute the fraction of DE genes unique to one tissue. Emit the 15x15 symmetric matrix as jaccard_similarity_matrix.csv plus a summary table. Decision point: a pair exceeding Jaccard 0.1 indicates meaningful shared response; in this dataset only gastrocnemius–vastus lateralis (~0.21) passes, supporting a largely tissue-specific adaptive transcriptional response.

## Related tools

- **DESeq2** (Produces the per-tissue differential expression results (filtered raw counts as input, run separately per sex and tissue) whose gene sets feed the Jaccard computation)
- **data.table** (Used in R to load the 15-tissue DEA results from data/external/dea)

## Examples

```
library(data.table); de <- lapply(tissues, function(t) fread(sprintf('data/external/dea/%s_de.csv', t))); jac <- function(a,b) length(intersect(a,b))/length(union(a,b)); J <- outer(genesets, genesets, Vectorize(jac)); fwrite(as.data.table(J), 'jaccard_similarity_matrix.csv')
```

## Evaluation signals

- Ortholog mapping fraction of unique DE genes should equal 94.5%
- Fraction of DE genes unique to a single tissue should be approximately 78%
- Gastrocnemius–vastus lateralis Jaccard should be approximately 0.21 and the only tissue pair exceeding 0.1
- The output matrix must be 15x15, symmetric, with a diagonal of exactly 1 and all off-diagonal values in [0,1]

## Limitations

- Cross-species comparison is limited: rat-human orthology identification is a difficult problem, and important biology in responsive genes without easily established cross-species correspondence is likely missed
- Jaccard treats all DE genes equally and discards effect direction and magnitude, so opposite-direction changes in shared genes count as full overlap

## Evidence

- [methods] DEA was performed separately in each sex and tissue using filtered raw counts as input for DESeq2: "DEA was performed separately in each sex and tissue using filtered raw counts as input for DESeq2"
- [discussion] a unique aspect of the MoTrPAC rat exercise training data includes the availability of differential expression data across 15 distinct tissues: "a unique aspect of the MoTrPAC rat exercise training data includes the availability of differential expression data across 15 distinct tissues"
- [discussion] Identification of rat-human gene orthology is another difficult problem: "Identification of rat-human gene orthology is another difficult problem"
- [readme] It primarily hosts scripts written in the R programming language, but interfacing with several other languages: "It primarily hosts scripts written in the R programming language, but interfacing with several other languages"
