---
name: tissue-biopsy-accessibility-classification
description: Use when intersecting exercise- or condition-responsive DE genes with
  gene-disease associations and you need to stratify results by whether the underlying
  tissue is routinely accessible in human biopsies (blood, skeletal muscle, adipose)
  or not.
when_to_use_negative:
- The input is a single-tissue gene list with no tissue labels, since the accessibility
  split (blood/skeletal muscle/adipose vs. other tissues) cannot be made
- The gene-trait evidence threshold differs from the article's > 0.8 Open Targets
  evidence score, since the reported counts (370/251, 178/143, 101) are specific to
  that cutoff
- Genes are not yet mapped to human orthologs, since Open Targets gene-trait associations
  are keyed to human genes
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: sparklyr
  role: R package used to load the Open Targets Release 22.04 database into Spark
    for cross-referencing orthologous human genes against direct gene-trait associations
    at evidence score > 0.8
  repo: github.com/NikVetr/MoTrPAC_Complex_Traits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:13345f2bbabef376aea87acd8bf2d37531498d0493627d62a1f9df187c557995
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - sparklyr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tissue-biopsy-accessibility-classification

## Summary

Classifies tissues from a multi-tissue differential-expression atlas as easily biopsied (blood, skeletal muscle, adipose) versus non-easily-biopsied, so that gene-trait associations from Open Targets (evidence score > 0.8) can be counted separately for accessible vs. inaccessible tissue sets.

## When to use

Use when intersecting exercise- or condition-responsive DE genes with gene-disease associations and you need to stratify results by whether the underlying tissue is routinely accessible in human biopsies (blood, skeletal muscle, adipose) or not.

## When NOT to use

- The input is a single-tissue gene list with no tissue labels, since the accessibility split (blood/skeletal muscle/adipose vs. other tissues) cannot be made
- The gene-trait evidence threshold differs from the article's > 0.8 Open Targets evidence score, since the reported counts (370/251, 178/143, 101) are specific to that cutoff
- Genes are not yet mapped to human orthologs, since Open Targets gene-trait associations are keyed to human genes

## Inputs

- Sex-consistent 8-week DE gene sets (F1_M1 up-regulated and F-1_M-1 down-regulated in both sexes) from the MoTrPAC rat endurance exercise training (EET) transcriptome atlas
- Rat-to-human ortholog map compiled from GENCODE and RGD resources
- Open Targets database Release 22.04 (June 8th, 2022 snapshot), direct gene-trait associations with evidence scores
- Tissue labels for the 15 MoTrPAC tissues

## Outputs

- Trait-tissue-gene table (CSV) of high-scoring (evidence score > 0.8) Open Targets associations split by biopsy accessibility
- Counts of high-scoring disease genes per tissue-accessibility class and per trait (e.g., 370 genes/251 traits total; 178 genes/143 traits non-biopsied; 101 traits with no easily-biopsied-tissue association)
- Annotated exemplar gene-tissue assignments (LDLR, APOB, SLC6A8, FOXP3, BRCA2)

## How to apply

Map the gene identifiers (e.g., rat DE genes from the MoTrPAC endurance exercise training study) to human orthologs first, then load Open Targets Release 22.04 (downloaded June 8th, 2022) into Spark via the R package sparklyr and retain direct gene-trait associations with evidence score > 0.8. Assign each tissue in the DE atlas to the easily-biopsied set (blood, skeletal muscle, adipose) or the non-biopsied set; classify genes as tissue-shared when differentially expressed in three or more tissues. Count high-scoring disease genes and traits separately for each accessibility class, and assemble a trait-tissue-gene table so counts are reproducible against reported figures (e.g., 370 genes across 251 traits overall; 178 genes across 143 traits in non-biopsied tissues). Annotate exemplar genes such as LDLR, APOB, SLC6A8, FOXP3, and BRCA2 with their tissue sets to spot-check the classification.

## Related tools

- **sparklyr** (R package used to load the Open Targets Release 22.04 database into Spark for cross-referencing orthologous human genes against direct gene-trait associations at evidence score > 0.8) — github.com/NikVetr/MoTrPAC_Complex_Traits

## Examples

```
library(sparklyr); sc <- spark_connect(master = "local"); ot <- spark_read_parquet(sc, "open_targets", "open_targets_release_22.04"); dplyr::filter(ot, score > 0.8, !tissue %in% c("blood", "skeletal_muscle", "adipose"))
```

## Evaluation signals

- The output trait-tissue-gene table reproduces the reported figures: 370 high-scoring genes across 251 traits; 178 genes across 143 traits in non-biopsied tissues; 101 traits with no easily-biopsied-tissue association
- All five exemplar genes (LDLR, APOB, SLC6A8, FOXP3, BRCA2) appear with their tissue assignments
- Every retained Open Targets association has evidence score strictly > 0.8 from the Release 22.04 snapshot
- Easily-biopsied set contains exactly blood, skeletal muscle, and adipose; tissue-shared genes are those differentially expressed in three or more tissues
- Gene identifiers in the result are human orthologs resolvable via the GENCODE/RGD-derived ortholog map

## Limitations

- Excluding blood, skeletal muscle, and adipose removes biology from tissues that are actually easy to sample in humans, so counts reflect a deliberately restrictive 'non-biopsied' view rather than total exercise-relevant disease associations
- Rat-to-human orthology is imperfect: important biology in disease/exercise-responsive genes without easily established cross-species correspondence is likely missed
- Results are tied to Open Targets Release 22.04 (June 8th, 2022); later releases with changed evidence scores will not reproduce the reported counts

## Evidence

- [other] Intersecting the sex-consistent 8-week exercise-responsive DE genes with Open Targets associations at evidence score > 0.8 and excluding easily biopsied tissues (blood, skeletal muscles, adipose) yielded 178 well-established disease genes associated with the adaptive exercise training response.: [evidence span withheld — anchor status: not_found]
- [methods] Open Targets Release 22.04 downloaded June 2022 into Spark: "The Open Targets50 database (Release 22.04) was downloaded on June 8th, 2022"
- [methods] Sex-consistent 8-week gene state selection: [evidence span withheld — anchor status: not_found]
- [methods] Tissue-shared gene definition: "comprising genes differentially expressed in three or more tissues."
- [other] Cross-referencing orthologous human genes against Open Targets: [evidence span withheld — anchor status: not_found]
