---
name: candidate-gene-prioritization-scoring
description: Use when you have both metPropagate (LPA) propagated scores and Exomiser-Phenotype
  scores for the same patient's candidate gene list and want a combined ranking that
  reduces metabolomic false-positive prioritization in suspected IEM patients.
when_to_use_negative:
- No per-patient untargeted metabolomics are available (no DAM z-scores to propagate),
  so metPropagate cannot contribute a term to the combination
- Feature-to-metabolite mapping for the metabolomic system is unavailable and the
  candidate gene set cannot be linked to HMDB-annotated metabolites (as in the TIDEX
  system)
- Trio WES data with parental filters already shortens candidate lists enough that
  re-ranking by metabolomic scores adds little, per the authors' note that trio WES
  simplifies prioritization
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3173
tools:
- name: metPropagate
  role: Provides the network-propagated metabolomic score (LPA output) that is one
    weighted term of the combined prioritization score
  repo: github.com/emmagraham/metPropagate
- name: Exomiser
  role: Supplies the phenotype (HPO/hiPHIVE) candidate gene scores and the filtered
    candidate gene lists (281-609 genes) that the combined score re-ranks
- name: STRING
  role: Functional linkage network (v11, 19,354 genes) over which label propagation
    runs and from which first-degree-neighbor HMDB percentages are computed for the
    p weighting
- name: yamaguchiyuto/label_propagation
  role: Adapted implementation of label propagation (e.g., Harmonic Function) used
    to produce metPropagate's propagated per-gene scores
  repo: https://github.com/yamaguchiyuto/label_propagation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:540005f4f28b392226cef199f9bdad88855d28b8df6bddac680d300ebcb6b8a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Exomiser
  - STRING
  - metPropagate
  - yamaguchiyuto/label_propagation
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# candidate-gene-prioritization-scoring

## Summary

A weighted additive scoring skill that combines metPropagate's network-propagated metabolomic scores with Exomiser's phenotype-based candidate gene scores to re-rank exome candidate lists, penalizing metPropagate's bias toward genes with many HMDB-annotated first-degree neighbors.

## When to use

Use when you have both metPropagate (LPA) propagated scores and Exomiser-Phenotype scores for the same patient's candidate gene list and want a combined ranking that reduces metabolomic false-positive prioritization in suspected IEM patients.

## When NOT to use

- No per-patient untargeted metabolomics are available (no DAM z-scores to propagate), so metPropagate cannot contribute a term to the combination
- Feature-to-metabolite mapping for the metabolomic system is unavailable and the candidate gene set cannot be linked to HMDB-annotated metabolites (as in the TIDEX system)
- Trio WES data with parental filters already shortens candidate lists enough that re-ranking by metabolomic scores adds little, per the authors' note that trio WES simplifies prioritization

## Inputs

- Per-patient metPropagate label-propagation (LPA) output scores over the STRING network
- Per-patient Exomiser-Phenotype candidate gene scores from Exomiser-filtered DeepVariant singleton VCFs
- STRING v11 functional linkage network (19,354 genes) with gene-to-node-ID mapping
- HMDB gene-metabolite annotations (HMDB_db_ver4.csv)
- Known causative gene per patient for validation

## Outputs

- Combined score C = p*Exomiser + (1-p)*metPropagate for every candidate gene of every patient
- Per-patient re-ranked candidate gene lists by combined score
- Per-patient summary counts of patients with a top-20th-percentile causative gene and improved ranks (written to combined_rankings.csv)

## How to apply

Load the per-patient metPropagate LPA output (propagated 'Score.ID' per candidate gene) and Exomiser-Phenotype scores, and scale both vectors between 0 and 1. Using the STRING v11 functional network and HMDB gene-metabolite annotations, compute for each of the 19,354 STRING genes the percentage of first-degree neighbors annotated in HMDB, and bin these percentages into ten 10%-interval bins. Within each bin compute p as the percentage of genes ranked in the top 20th percentile of all 19,354 genes, then form the combined score C = p*Exomiser + (1-p)*metPropagate, rescaling each term between 0 and 1 before summing. Rank each patient's candidate genes (281-609 genes per TIDEX singleton exome from Exomiser variant effect/frequency filters) by C and test whether at least one known causative gene falls in the top 20th percentile (rank <= 60 for 300-gene lists).

## Related tools

- **metPropagate** (Provides the network-propagated metabolomic score (LPA output) that is one weighted term of the combined prioritization score) — github.com/emmagraham/metPropagate
- **Exomiser** (Supplies the phenotype (HPO/hiPHIVE) candidate gene scores and the filtered candidate gene lists (281-609 genes) that the combined score re-ranks)
- **STRING** (Functional linkage network (v11, 19,354 genes) over which label propagation runs and from which first-degree-neighbor HMDB percentages are computed for the p weighting)
- **yamaguchiyuto/label_propagation** (Adapted implementation of label propagation (e.g., Harmonic Function) used to produce metPropagate's propagated per-gene scores) — https://github.com/yamaguchiyuto/label_propagation

## Examples

```
edit sample_name in label_propagation/run_label_prop_cluster.sh; bash label_propagation/run_label_prop_cluster.sh  # LPA output feeds the combined C = p*Exomiser + (1-p)*metPropagate scoring
```

## Evaluation signals

- At least one causative gene falls in the top 20th percentile of the combined ranking for 10/11 TIDEX patients
- The combined ranking places the causative gene higher than both Exomiser-alone and metPropagate-alone rankings in 8/11 patients
- All per-patient Exomiser, metPropagate, and combined score terms are rescaled to the [0, 1] interval before summation
- The ten HMDB-neighbor bins cover all 19,354 STRING genes at 10% intervals (0-10%, ..., 90-100%)
- LPA output files contain final propagated scores per candidate gene (Score.ID) alongside original scores (Original.Score.ID)

## Limitations

- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, so exact metabolite identities (and thus gene-metabolite links) could not be confirmed
- The TIDEX metabolic system could not measure large glycosaminoglycans, limiting ME analysis for the IDS patient
- Common IEM biomarkers such as acyl glycines and acyl carnitines are not annotated to IEM-associated genes in HMDB, weakening the metabolomic term for some diseases
- No software version reported for the STRING network or for Exomiser/hiPHIVE, complicating exact reproduction

## Evidence

- [abstract] ranking by a weighted combination of metPropagate and Exomiser scores resulted in improved prioritization: "ranking by a weighted combination of metPropagate and Exomiser scores resulted in improved prioritization"
- [abstract] metPropagate placed at least one causative gene in the top 20th percentile in 9/11 patients: "metPropagate placed at least one causative gene in the top 20th percentile in 9/11 patients"
- [results] we multiped the scaled enrichment p-value (differentially metabolic enrichment—DME) by the absolute value of each gene’s largest metabolite z-score: "we multiped the scaled enrichment p-value (differentially metabolic enrichment—DME) by the absolute value of each gene’s largest metabolite z-score"
- [discussion] candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
- [readme] File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID".: "File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID"."
