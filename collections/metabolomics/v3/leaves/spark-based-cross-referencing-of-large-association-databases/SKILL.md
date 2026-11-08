---
name: spark-based-cross-referencing-of-large-association-databases
description: Use when a genome-scale gene-trait association database (e.g. Open Targets
  Release 22.04) must be filtered by an evidence-score threshold and joined against
  a mapped gene list too large for in-memory R tools.
when_to_use_negative:
- When the gene-trait query set is small enough to fit comfortably in base R/data.frame
  joins — sparklyr overhead is unnecessary
- 'When indirect or low-evidence associations are wanted: this skill is restricted
  to Open Targets direct associations at evidence score > 0.8'
- When the trait associations come from a different release or database without a
  comparable evidence-score field, since the > 0.8 threshold and validation counts
  are release-specific
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: sparklyr
  role: R package used to load the Open Targets Release 22.04 snapshot into Spark
    and cross-reference orthologous human genes against gene-trait direct associations
    at evidence score > 0.8
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:69345803ce65e689cc4ef7f3824ae20049960ad18e27edb3ca3f8cdcc64d654d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - sparklyr
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Spark-based cross-referencing of large association databases

## Summary

Loads a full Open Targets gene-trait association snapshot into Spark via the R package sparklyr and cross-references it against ortholog-mapped differential-expression gene lists to extract high-evidence-score disease associations at tissue-specific and tissue-shared resolution.

## When to use

Use when a genome-scale gene-trait association database (e.g. Open Targets Release 22.04) must be filtered by an evidence-score threshold and joined against a mapped gene list too large for in-memory R tools.

## When NOT to use

- When the gene-trait query set is small enough to fit comfortably in base R/data.frame joins — sparklyr overhead is unnecessary
- When indirect or low-evidence associations are wanted: this skill is restricted to Open Targets direct associations at evidence score > 0.8
- When the trait associations come from a different release or database without a comparable evidence-score field, since the > 0.8 threshold and validation counts are release-specific

## Inputs

- MoTrPAC rat-to-human ortholog map (compiled from GENCODE and RGD resources)
- Sex-consistent 8-week DE gene lists: F1_M1 (up-regulated in both sexes) and F-1_M-1 (down-regulated in both sexes)
- Open Targets database Release 22.04 gene-trait direct associations (snapshot downloaded June 8th, 2022)

## Outputs

- Trait-tissue-gene association table (CSV) of high-scoring disease genes annotated with tissue sets
- Count summaries: 370 high-scoring disease genes from 251 traits overall; 178 genes in non-biopsied tissues across 143 traits; 101 traits with no easily-biopsied-tissue association
- Tissue-set assignments for exemplar genes LDLR, APOB, SLC6A8, FOXP3, and BRCA2

## How to apply

First map rat DE genes to human orthologs using the MoTrPAC rat-to-human ortholog map compiled from GENCODE and RGD resources, taking the sex-consistent 8-week states F1_M1 (up-regulated in both sexes) and F-1_M-1 (down-regulated in both sexes). Load the Open Targets database (Release 22.04, snapshot downloaded June 8th, 2022) into Spark via the R-package sparklyr so the full association table can be queried at scale. Cross-reference the orthologous human genes against all Open Targets gene-trait direct associations at an evidence-score threshold > 0.8, then quantify the resulting associations on tissue-specific and tissue-shared bases (genes differentially expressed in three or more tissues), excluding easily biopsied tissues (blood, skeletal muscle, adipose) from the non-biopsied set. The rationale is that the full direct-association snapshot is too large for base R, while Spark's distributed joins make thresholded cross-referencing with tissue-set bookkeeping tractable and reproducible.

## Related tools

- **sparklyr** (R package used to load the Open Targets Release 22.04 snapshot into Spark and cross-reference orthologous human genes against gene-trait direct associations at evidence score > 0.8)

## Examples

```
library(sparklyr); sc <- spark_connect(master = "local"); ot <- spark_read(sc, "open_targets", "open_targets_release_22.04"); ot %>% filter(evidence_score > 0.8) %>% inner_join(ortholog_map, by = c("target" = "human_ortholog"))
```

## Evaluation signals

- The trait/gene counts in the output table reproduce the reported figures: 370 genes/251 traits overall, 178 genes/143 traits in non-biopsied tissues, and 101 traits with no easily-biopsied-tissue association
- All five exemplar genes (LDLR, APOB, SLC6A8, FOXP3, BRCA2) appear in the output with their correct tissue assignments
- Every retained association has an Open Targets evidence score > 0.8 and corresponds to a human ortholog of a sex-consistent 8-week F1_M1 or F-1_M-1 rat DE gene
- Easily biopsied tissues (blood, skeletal muscle, adipose) are absent from the non-biopsied set, and tissue-shared genes are defined as differentially expressed in three or more tissues

## Limitations

- Rat-human orthology is a difficult problem; important biology in disease/exercise-responsive genes without easily established cross-species correspondence is likely missed
- Results are tied to a specific Open Targets snapshot (Release 22.04, downloaded June 8th, 2022) and will change with newer releases
- Excluding blood, skeletal muscle, and adipose removes well-studied, easily biopsied tissue associations from the non-biopsied count by design

## Evidence

- [other] Load the Open Targets database, Release 22.04 (snapshot downloaded June 8th, 2022) into Spark via the R-package sparklyr: [evidence span withheld — anchor status: not_found]
- [other] Cross-reference orthologous human genes against Open Targets direct associations at evidence score > 0.8: [evidence span withheld — anchor status: not_found]
- [methods] Open Targets Release 22.04 download date from methods: "The Open Targets50 database (Release 22.04) was downloaded on June 8th, 2022"
- [methods] Tissue-shared definition and exclusion of easily biopsied tissues: "comprising genes differentially expressed in three or more tissues."
