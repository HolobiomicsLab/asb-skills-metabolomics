---
name: cross-tissue-differential-expression-overlap
description: Use when you have per-tissue differential expression results (e.g., MoTrPAC
  PASS1B 15-tissue rat EET data) and need to measure cross-tissue overlap or tissue-specificity
  of DE gene sets.
when_to_use_negative:
- Input is raw RNA-seq counts rather than completed per-tissue differential expression
  results — run DESeq2 DEA first (separately per sex and tissue, 5% IHW alpha).
- The goal is heritability or disease enrichment of exercise-responsive genes (use
  LDSC/MESC/S-PrediXcan workflows instead) rather than cross-tissue gene-set overlap.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: DESeq2
  role: Performs the per-sex, per-tissue differential expression analysis (likelihood
    ratio tests on filtered raw counts) that generates the gene sets whose cross-tissue
    overlap is computed
- name: data.table
  role: R package used to load the differential expression results across the 15 tissues
    from data/external/dea
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:dbcf5eda08056c721f2a3aa545d35af8698d8d9f799c65bf324a1a4e318cfc4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - data.table
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-tissue-differential-expression-overlap

## Summary

Quantifies how concordant or tissue-specific the adaptive transcriptional response to exercise is by computing pairwise Jaccard similarity between differential expression gene sets across the 15 rat tissues of the MoTrPAC PASS1B endurance exercise training study.

## When to use

Use when you have per-tissue differential expression results (e.g., MoTrPAC PASS1B 15-tissue rat EET data) and need to measure cross-tissue overlap or tissue-specificity of DE gene sets.

## When NOT to use

- Input is raw RNA-seq counts rather than completed per-tissue differential expression results — run DESeq2 DEA first (separately per sex and tissue, 5% IHW alpha).
- The goal is heritability or disease enrichment of exercise-responsive genes (use LDSC/MESC/S-PrediXcan workflows instead) rather than cross-tissue gene-set overlap.

## Inputs

- MoTrPAC PASS1B differential expression analysis results across 15 tissues (data/external/dea directory, R objects/tables)
- Rat-to-human ortholog map (RGD_ORTHOLOGS_20201001.txt, compiled GENCODE/RGD orthologs)
- Per-sex, per-tissue DESeq2 differential expression results (filtered raw counts input)

## Outputs

- jaccard_similarity_matrix.csv (15x15 pairwise Jaccard similarity between tissue DE gene sets)
- Summary statistics table (ortholog mapping fraction, tissue-specificity fraction, top-overlapping tissue pairs)

## How to apply

Load the MoTrPAC PASS1B differential expression analysis results for all 15 tissues from the repository's data/external/dea directory using R data.table. Map rat genes to human orthologs with the compiled GENCODE/RGD ortholog map (RGD_ORTHOLOGS_20201001.txt), then compute the fraction of unique DE genes with human orthologs and the fraction of DE genes unique to a single tissue. Compute pairwise Jaccard similarity between all tissue DE gene sets as |A∩B|/|A∪B| and save the resulting matrix as jaccard_similarity_matrix.csv along with a summary statistics table. DE gene sets derive from DESeq2 differential expression analysis performed separately in each sex and tissue, with training-differential genes selected at 5% IHW alpha.

## Related tools

- **DESeq2** (Performs the per-sex, per-tissue differential expression analysis (likelihood ratio tests on filtered raw counts) that generates the gene sets whose cross-tissue overlap is computed)
- **data.table** (R package used to load the differential expression results across the 15 tissues from data/external/dea)

## Examples

```
library(data.table); de <- lapply(Sys.glob('data/external/dea/*'), fread); jaccard <- function(a,b) length(intersect(a,b))/length(union(a,b)); J <- outer(de_genes, de_genes, Vectorize(function(x,y) jaccard(x,y))); write.csv(J, 'jaccard_similarity_matrix.csv')
```

## Evaluation signals

- Ortholog mapping fraction equals 94.5% of unique DE genes mapped to human orthologs via RGD_ORTHOLOGS_20201001.txt
- Tissue-specificity fraction is approximately 78% of DE genes unique to one tissue
- Gastrocnemius–vastus lateralis Jaccard similarity ≈ 0.21 and is the only tissue pair exceeding a Jaccard index of 0.1
- jaccard_similarity_matrix.csv is a symmetric 15x15 matrix with diagonal values of 1
- All DE gene sets trace back to training-differential genes selected at 5% IHW alpha from DESeq2 results

## Limitations

- Rat-human orthology identification is a difficult problem, so the ~94.5% mapping fraction means some exercise-responsive genes lack cross-species correspondence and their biology is likely missed
- Results are specific to the 8-week adaptive time point and sex-combined DE sets; qualitative sex-specificity of exercise adaptation fell outside the scope considered in the paper

## Evidence

- [methods] DEA was performed separately in each sex and tissue using filtered raw counts as input for DESeq2: "DEA was performed separately in each sex and tissue using filtered raw counts as input for DESeq2"
- [methods] Training-differential genes were selected at 5% IHW alpha.: "Training-differential genes were selected at 5% IHW α."
- [discussion] Identification of rat-human gene orthology is another difficult problem: "Identification of rat-human gene orthology is another difficult problem"
- [discussion] a unique aspect of the MoTrPAC rat exercise training data includes the availability of differential expression data across 15 distinct tissues: "a unique aspect of the MoTrPAC rat exercise training data includes the availability of differential expression data across 15 distinct tissues"
