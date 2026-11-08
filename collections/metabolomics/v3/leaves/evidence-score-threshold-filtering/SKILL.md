---
name: evidence-score-threshold-filtering
description: Use when prioritizing disease-relevant genes from a candidate gene list
  by requiring strong Open Targets association evidence (score > 0.8), e.g. after
  mapping rat DE genes to human orthologs.
when_to_use_negative:
- When the goal is directional mediation of trait risk (e.g. S-PrediXcan effect-direction
  analysis) rather than cataloging high-confidence gene-trait associations — evidence-score
  filtering discards directionality.
- When the candidate gene set lacks reliable rat-to-human orthology (e.g. genes without
  well-established cross-species correspondence), since the article notes important
  biology may be missed there.
- When a lower evidence threshold is scientifically justified — the > 0.8 cutoff is
  intentionally stringent and will drop moderately supported associations.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0625
tools:
- name: sparklyr
  role: R interface used to load the Open Targets Release 22.04 snapshot into Apache
    Spark for scalable cross-referencing of orthologous genes against gene-trait associations
- name: MoTrPAC_Complex_Traits
  role: Publication repository hosting the R scripts implementing the ortholog mapping,
    Open Targets filtering, and trait-tissue-gene table assembly
  repo: https://github.com/NikVetr/MoTrPAC_Complex_Traits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:81a3427da865bf8aeec6a9b242f4b2da037af72d81a1b087b3fb092a70416e44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NikVetr/MoTrPAC_Complex_Traits
  tools:
  - MoTrPAC_Complex_Traits
  - sparklyr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# evidence-score-threshold-filtering

## Summary

Filters gene-disease associations by a high Open Targets overall association evidence score (> 0.8) to retain only well-supported trait-gene links, then intersects them with sex-consistent 8-week exercise-responsive DE genes mapped to human orthologs, excluding easily biopsied tissues.

## When to use

Use when prioritizing disease-relevant genes from a candidate gene list by requiring strong Open Targets association evidence (score > 0.8), e.g. after mapping rat DE genes to human orthologs.

## When NOT to use

- When the goal is directional mediation of trait risk (e.g. S-PrediXcan effect-direction analysis) rather than cataloging high-confidence gene-trait associations — evidence-score filtering discards directionality.
- When the candidate gene set lacks reliable rat-to-human orthology (e.g. genes without well-established cross-species correspondence), since the article notes important biology may be missed there.
- When a lower evidence threshold is scientifically justified — the > 0.8 cutoff is intentionally stringent and will drop moderately supported associations.

## Inputs

- MoTrPAC rat-to-human ortholog map (compiled from GENCODE and RGD resources)
- 8-week sex-consistent rat DE gene sets in F1_M1 (up in both sexes) and F-1_M-1 (down in both sexes) states
- Open Targets database Release 22.04 gene-trait direct association records with evidence scores

## Outputs

- Trait-tissue-gene table (CSV) of high-scoring (> 0.8) disease-gene associations, with 370 genes across 251 traits overall and 178 genes across 143 traits in non-biopsied tissues
- Per-gene tissue-set annotations (tissue-specific vs. tissue-shared, DE in >= 3 tissues) for exemplar genes such as LDLR, APOB, SLC6A8, FOXP3, BRCA2

## How to apply

Load the MoTrPAC rat-to-human ortholog map (compiled from GENCODE and RGD) and map 8-week sex-consistent DE genes — F1_M1 (up-regulated in both sexes) and F-1_M-1 (down-regulated in both sexes) — to human orthologs. Load the Open Targets database (Release 22.04, snapshot downloaded June 8th, 2022) into Apache Spark via the R package sparklyr, then cross-reference orthologous human genes against all gene-trait direct associations at an evidence-score threshold > 0.8. Quantify the resulting associations on tissue-specific and tissue-shared bases (genes differentially expressed in three or more tissues), excluding easily biopsied tissues (blood, skeletal muscle, adipose) from the non-biopsied set. Assemble a trait-tissue-gene table so counts reproduce the reported figures and annotate exemplar genes (LDLR, APOB, SLC6A8, FOXP3, BRCA2) with their tissue sets.

## Related tools

- **sparklyr** (R interface used to load the Open Targets Release 22.04 snapshot into Apache Spark for scalable cross-referencing of orthologous genes against gene-trait associations)
- **MoTrPAC_Complex_Traits** (Publication repository hosting the R scripts implementing the ortholog mapping, Open Targets filtering, and trait-tissue-gene table assembly) — https://github.com/NikVetr/MoTrPAC_Complex_Traits

## Examples

```
library(sparklyr); sc <- spark_connect(master = "local"); ot <- spark_read_parquet(sc, "opentargets", "ot_release_22_04/"); high <- ot %>% filter(overallScore > 0.8) %>% inner_join(ortholog_map, by = c("targetId" = "ensembl_gene_id"))
```

## Evaluation signals

- Output table contains exactly 370 high-scoring disease genes spanning 251 traits before tissue exclusion.
- After excluding blood, skeletal muscle, and adipose, the table yields 178 genes across 143 traits, with 101 traits having no easily-biopsied-tissue association.
- All five exemplar genes (LDLR, APOB, SLC6A8, FOXP3, BRCA2) appear in the output with their correct tissue assignments.
- Every retained gene-trait pair has Open Targets overall association score strictly > 0.8, and every retained gene maps to a valid human ortholog from the GENCODE/RGD-based map.
- Tissue-shared gene counts are consistent with the 'differentially expressed in three or more tissues' definition.

## Limitations

- Rat-to-human orthology is a hard problem; the article notes 'a notable limitation of this study may be that, despite their well-established use as an exercise model, rats are separated from humans by nearly 140 million years of evolution', and important biology in genes without easy cross-species correspondence is likely missed.
- The filter depends on a dated database snapshot (Open Targets Release 22.04, downloaded June 8th, 2022); scores and associations will drift in newer releases.
- The tissue-exclusion step is a curation decision (blood, skeletal muscle, adipose deemed easily biopsied) that may not suit other accessibility contexts.

## Evidence

- [methods] Load the Open Targets database, Release 22.04 (snapshot downloaded June 8th, 2022) into Spark via the R-package sparklyr.: "The Open Targets50 database (Release 22.04) was downloaded on June 8th, 2022"
- [methods] Sex-consistent DE gene states define the candidate gene input before evidence-score filtering.: [evidence span withheld — anchor status: not_found]
- [methods] Tissue-shared vs tissue-specific classification criterion used when quantifying filtered associations.: "comprising genes differentially expressed in three or more tissues."
- [discussion] Orthology identification is an acknowledged difficulty for this cross-species filtering step.: "Identification of rat-human gene orthology is another difficult problem"
