---
name: gene-disease-association-intersection
description: Use when a list of differentially expressed genes (e.g. sex-consistent
  F1_M1 / F-1_M-1 states from an 8-week exercise time point) must be prioritized against
  curated gene-disease associations such as Open Targets.
when_to_use_negative:
- The gene list is not anchored to a defined differential-expression state (e.g. no
  sex-consistent F1_M1 / F-1_M-1 assignment) — the intersection loses its interpretive
  frame
- The goal is to estimate SNP-heritability or mediated heritability of trait associations
  (use LDSC or MESC instead) rather than count curated gene-disease targets
- Associations below evidence score 0.8 are acceptable — the counting logic and reported
  figures depend on this threshold
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: sparklyr
  role: R package used to load the Open Targets Release 22.04 database into Spark
    and cross-reference orthologous human genes against gene-trait associations at
    evidence score > 0.8
- name: Open Targets
  role: Source database of gene-trait direct associations, Release 22.04, thresholded
    at evidence score > 0.8
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:bcfde2a4501a5cbdc47108856a8586d15d5a2f4f90b8195121c6e12dd3e665a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Open Targets
  - sparklyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# gene-disease-association-intersection

## Summary

Cross-reference rat-to-human orthologs of sex-consistent 8-week exercise-responsive DE genes against Open Targets gene-trait direct associations at evidence score > 0.8, quantifying disease genes by tissue specificity (tissue-shared = differentially expressed in 3+ tissues) and excluding easily biopsied tissues.

## When to use

Use when a list of differentially expressed genes (e.g. sex-consistent F1_M1 / F-1_M-1 states from an 8-week exercise time point) must be prioritized against curated gene-disease associations such as Open Targets.

## When NOT to use

- The gene list is not anchored to a defined differential-expression state (e.g. no sex-consistent F1_M1 / F-1_M-1 assignment) — the intersection loses its interpretive frame
- The goal is to estimate SNP-heritability or mediated heritability of trait associations (use LDSC or MESC instead) rather than count curated gene-disease targets
- Associations below evidence score 0.8 are acceptable — the counting logic and reported figures depend on this threshold

## Inputs

- Sex-consistent 8-week DE gene sets (F1_M1 up-regulated in both sexes; F-1_M-1 down-regulated in both sexes) from the MoTrPAC rat endurance exercise training study
- Rat-to-human ortholog map compiled from GENCODE and RGD resources
- Open Targets database, Release 22.04 (snapshot downloaded June 8th, 2022), gene-trait direct associations with evidence scores

## Outputs

- Trait-tissue-gene table (CSV) of high-scoring disease genes per tissue, with tissue-specific vs tissue-shared (3+ tissues) annotations
- Counts of high-scoring disease genes and associated traits (370 genes / 251 traits overall; 178 genes / 143 traits in non-biopsied tissues; 101 traits with no easily-biopsied-tissue association)
- Annotated exemplar genes (LDLR, APOB, SLC6A8, FOXP3, BRCA2) with their tissue assignments

## How to apply

Map rat DE genes (sex-consistent F1_M1 up-regulated and F-1_M-1 down-regulated states from the 8-week MoTrPAC time point) to human orthologs using the GENCODE/RGD-derived orthology map. Load the Open Targets database, Release 22.04 (snapshot downloaded June 8th, 2022), into Spark via the R package sparklyr, and retain only direct gene-trait associations with evidence score > 0.8. Quantify associations on tissue-specific vs tissue-shared bases (genes differentially expressed in three or more tissues), and assemble the trait-tissue-gene table counting non-biopsied-tissue genes after excluding blood, skeletal muscle, and adipose. Exemplar genes to annotate with tissue sets: LDLR, APOB, SLC6A8, FOXP3, and BRCA2.

## Related tools

- **sparklyr** (R package used to load the Open Targets Release 22.04 database into Spark and cross-reference orthologous human genes against gene-trait associations at evidence score > 0.8)
- **Open Targets** (Source database of gene-trait direct associations, Release 22.04, thresholded at evidence score > 0.8)

## Examples

```
library(sparklyr); sc <- spark_connect(master = "local"); ot <- spark_read_parquet(sc, name = "open_targets", path = "open_targets_release_22.04")
```

## Evaluation signals

- Total high-scoring disease gene count reproduces 370 genes across 251 traits before tissue exclusion
- After excluding easily biopsied tissues (blood, skeletal muscle, adipose), the non-biopsied set contains 178 genes across 143 traits
- Exactly 101 traits show no association with easily biopsied tissues
- All five exemplar genes (LDLR, APOB, SLC6A8, FOXP3, BRCA2) appear in the output table with correct tissue assignments
- Tissue-shared genes are restricted to those differentially expressed in three or more tissues

## Limitations

- Rats are separated from humans by nearly 140 million years of evolution, complicating orthology-based interpretation of the intersected gene set
- Important biology in disease/exercise-responsive genes without easily established cross-species correspondence is likely missed
- Evidence scores come from a dated Open Targets snapshot (Release 22.04, June 8th, 2022); counts will drift with database releases

## Evidence

- [methods] Open Targets Release 22.04: "The Open Targets50 database (Release 22.04) was downloaded on June 8th, 2022"
- [methods] sex-consistent state selection: [evidence span withheld — anchor status: not_found]
- [methods] tissue-shared definition: "comprising genes differentially expressed in three or more tissues."
- [discussion] orthology limitation: "Identification of rat-human gene orthology is another difficult problem"
- [abstract] triplet-scale result context: "we identify a total of 5523 trait-tissue-gene triplets to serve as a valuable starting point for future investigations"
