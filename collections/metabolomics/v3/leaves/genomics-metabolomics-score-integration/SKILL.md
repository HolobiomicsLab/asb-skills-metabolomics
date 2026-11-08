---
name: genomics-metabolomics-score-integration
description: Use when two per-gene prioritization score vectors (e.g., Exomiser phenotype
  scores and metPropagate LPA-propagated scores) exist for the same candidate gene
  lists and a combined ranking should down-weight the metabolomics score for highly
  connected metabolic genes.
when_to_use_negative:
- Only one score source is available (e.g., no Exomiser phenotype ranking or no LPA
  output) — the weighted combination requires both score vectors per patient.
- The network is not STRING (v11) or gene-metabolite annotations are not from HMDB
  — the p weights are calibrated from HMDB-annotated first-degree STRING neighbor
  fractions across 19,354 genes and will not transfer.
- Candidate gene lists are trio-WES-derived and much shorter than the 281–609 gene
  singleton-exome lists — top-20th-percentile thresholds (rank <= 60) were calibrated
  on the singleton lists.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: metPropagate
  role: Provides the metabolomics-derived propagated gene scores (LPA output) used
    as the (1-p) term of the combination
  repo: github.com/emmagraham/metPropagate
- name: Exomiser
  role: Provides the phenotype-based candidate gene scores (hiPHIVE-driven ranking)
    used as the p term of the combination and its variant-effect/population-frequency
    filtering generates candidate gene lists
- name: STRING
  role: Functional linkage network (v11) whose gene-degree/HMDB-annotation statistics
    define the per-bin weight p
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:398141ef837b254a589659b8f1317cbeca9b4c7743c4dc86eb3b3a69cb11c41c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Exomiser
  - STRING
  - metPropagate
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# genomics-metabolomics-score-integration

## Summary

Combines Exomiser phenotype-based rankings with metPropagate network-propagated metabolomic scores via a weighted additive model whose weights penalize metPropagate's bias toward genes with many HMDB-annotated first-degree STRING neighbors, improving causative gene prioritization in TIDEX patients.

## When to use

Use when two per-gene prioritization score vectors (e.g., Exomiser phenotype scores and metPropagate LPA-propagated scores) exist for the same candidate gene lists and a combined ranking should down-weight the metabolomics score for highly connected metabolic genes.

## When NOT to use

- Only one score source is available (e.g., no Exomiser phenotype ranking or no LPA output) — the weighted combination requires both score vectors per patient.
- The network is not STRING (v11) or gene-metabolite annotations are not from HMDB — the p weights are calibrated from HMDB-annotated first-degree STRING neighbor fractions across 19,354 genes and will not transfer.
- Candidate gene lists are trio-WES-derived and much shorter than the 281–609 gene singleton-exome lists — top-20th-percentile thresholds (rank <= 60) were calibrated on the singleton lists.

## Inputs

- Per-patient metPropagate propagated scores (LPA output files with Score.ID / Original.Score.ID)
- Exomiser-Phenotype candidate gene scores per patient (from Exomiser filtering of DeepVariant singleton VCFs)
- STRING v11 functional network graph file (STRING_graph_file_v11_gene_list_functional_entire_db)
- HMDB gene-metabolite annotations (HMDB_db_ver4.csv)
- Known causative gene labels per patient for validation

## Outputs

- combined_rankings.csv with combined per-gene scores, ranks, and per-patient summary counts
- Per-patient comparison table of combined vs. Exomiser-alone vs. metPropagate-alone causative gene ranks

## How to apply

Scale both the metPropagate LPA output scores and the Exomiser-Phenotype candidate gene scores between 0 and 1 per patient. Using STRING v11 and HMDB gene-metabolite annotations, compute for each of the 19,354 STRING genes the percentage of first-degree neighbors annotated in HMDB, bin these percentages into ten 10% intervals (0–10%, 10–20%, etc.), and within each bin calculate p as the percentage of genes ranked in the top 20th percentile of all genes — bins full of highly connected metabolic genes get low p, penalizing metPropagate where it is biased. Compute the combined score C = p*Exomiser + (1-p)*metPropagate for every candidate gene, rescaling each term between 0 and 1 before summing, then rank each patient's candidate genes by C. Evaluate whether at least one known causative gene falls in the top 20th percentile (rank <= 60 for ~300-gene candidate lists) and compare per patient against Exomiser-alone and metPropagate-alone rankings.

## Related tools

- **metPropagate** (Provides the metabolomics-derived propagated gene scores (LPA output) used as the (1-p) term of the combination) — github.com/emmagraham/metPropagate
- **Exomiser** (Provides the phenotype-based candidate gene scores (hiPHIVE-driven ranking) used as the p term of the combination and its variant-effect/population-frequency filtering generates candidate gene lists)
- **STRING** (Functional linkage network (v11) whose gene-degree/HMDB-annotation statistics define the per-bin weight p)

## Examples

```
python main.py hmn -g STRING_graph_file_v11_functional_entire_db.txt -l intermediate_label_file_sampleA -o LPA_output/sampleA  # then combine: C = p*rescale(exomiser) + (1-p)*rescale(lpa_scores)
```

## Evaluation signals

- Combined ranking places at least one causative gene in the top 20th percentile in 10/11 TIDEX patients.
- Combined ranking improves the causative gene's rank relative to either single algorithm in 8/11 patients.
- The ten p-value bins partition all 19,354 STRING genes with 10% intervals of HMDB-annotated first-degree neighbor percentage; each bin sums to its gene count.
- Both score terms are rescaled to [0,1] before summing, so combined scores C fall within [0,2] and per-gene ranks are monotone in C.
- Sanity baseline: metPropagate alone reaches 9/11 and Exomiser alone fewer than 8/11 top-20th-percentile hits, so the combination must not underperform either alone.

## Limitations

- Singleton (non-trio) exomes were used, producing candidate lists of 281–609 genes; trio parental data would shorten lists and change top-20th-percentile thresholds.
- The metabolomic system could not measure large glycosaminoglycans, limiting ME analysis for the IDS patient, and common IEM biomarker metabolites (acyl glycines, acyl carnitines) are not annotated to IEM genes in HMDB.
- Analyses are insensitive to diseases associated with metabolites unmeasurable by the chromatographic/MS platform combination used.
- No software versions reported for STRING network and Exomiser/hiPHIVE, complicating exact reproduction.

## Evidence

- [abstract] ranking by a weighted combination of metPropagate and Exomiser scores resulted in improved prioritization: "ranking by a weighted combination of metPropagate and Exomiser scores resulted in improved prioritization"
- [results] STRING v11 network degree-bin weighting: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
- [discussion] Candidate list size range defining top-20th-percentile thresholds: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
- [readme] README description of LPA output files feeding integration: "LPA_output contains the post-propagation node scores (automatically generated)"
- [readme] STRING graph files used in integration step: "graph_files contains input STRING network files (manually placed)"
