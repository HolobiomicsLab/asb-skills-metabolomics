---
name: wes-candidate-list-integration
description: Use when a patient has a DeepVariant/Exomiser candidate gene list and
  per-gene metabolomic enrichment scores, and you need to rank those candidates by
  network-propagated metabolomic evidence and benchmark against Exomiser's phenotype
  ranking.
when_to_use_negative:
- Input is not an Exomiser-filtered WES candidate list (e.g., no variant data, or
  variant calling/filtered candidate genes have not been produced by DeepVariant/Exomiser)
- The goal is metabolomic preprocessing itself (XCUS/median-scaling, 10%-of-samples
  feature filtering, HMDB annotation, DAM identification at |z| ≥ 2) — that upstream
  pipeline is a separate skill
- 'Feature-to-metabolite identity matters for the analysis: in the TIDEX setting the
  mapping was unavailable, so exact metabolite identities could not be determined
  and results rely on uncurated features'
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0654
tools:
- name: Exomiser
  role: Filters DeepVariant singleton VCFs via variant effect and population frequency
    filters to produce the 281–609-gene candidate list that is ranked; its hiPHIVE
    phenotype ranking provides the comparison baseline
- name: DeepVariant
  role: Creates the singleton VCFs per patient that are upstream of Exomiser filtering
- name: metPropagate
  role: Supplies the propagated Score.ID for every candidate gene after label propagation
    of ME/DME-scaled labels over the STRING v11 network
  repo: github.com/emmagraham/metPropagate
- name: STRING
  role: Protein functional linkage network (v11, functional_entire_db) over which
    metabolomic scores are propagated; node-ID mapping file links candidate gene names
    to network nodes
- name: yamaguchiyuto/label_propagation
  role: Adapted LGC (Local and Global Consistency) implementation run with alpha =
    0.99 to propagate the metabolomic labels across the STRING graph
  repo: https://github.com/yamaguchiyuto/label_propagation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:d3962589a552c7511bacb1712b6b574898bcb79737ddb834a9fcbb2f00a974e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepVariant
  - Exomiser
  - STRING
  - metPropagate
  - yamaguchiyuto/label_propagation
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# wes-candidate-list-integration

## Summary

Integrates Exomiser-filtered WES candidate gene lists (from DeepVariant singleton VCFs) with metPropagate propagated scores on the STRING v11 functional network, ranking each patient's 281–609 candidates to test whether at least one causative gene falls in the top 20th percentile (55–115 genes).

## When to use

Use when a patient has a DeepVariant/Exomiser candidate gene list and per-gene metabolomic enrichment scores, and you need to rank those candidates by network-propagated metabolomic evidence and benchmark against Exomiser's phenotype ranking.

## When NOT to use

- Input is not an Exomiser-filtered WES candidate list (e.g., no variant data, or variant calling/filtered candidate genes have not been produced by DeepVariant/Exomiser)
- The goal is metabolomic preprocessing itself (XCUS/median-scaling, 10%-of-samples feature filtering, HMDB annotation, DAM identification at |z| ≥ 2) — that upstream pipeline is a separate skill
- Feature-to-metabolite identity matters for the analysis: in the TIDEX setting the mapping was unavailable, so exact metabolite identities could not be determined and results rely on uncurated features

## Inputs

- DeepVariant singleton VCF per patient, filtered by Exomiser into a candidate gene list (281–609 genes per TIDEX patient)
- metabolomic enrichment file linear_msea_genebig_allpeaks_combined.csv from the metPropagate METABOLOMIC_PROCESSING_PIPELINE
- STRING_graph_file_v11_functional_entire_db network file (unzipped)
- STRING_graph_file_v11_gene_to_nodeid_mapping gene-to-node ID mapping file
- Known causative gene(s) per patient for top-20th-percentile evaluation

## Outputs

- Per-patient propagated-score file (Score.ID and Original.Score.ID per candidate gene) in integration/results/
- Ranked candidate gene list per patient ordered by propagated metPropagate score
- Summary table reporting, for each of the 11 patients, whether a causative gene falls within the top 20th percentile

## How to apply

Take the candidate gene list produced by Exomiser's variant effect and population frequency filters (281–609 genes per singleton-exome patient in the TIDEX cohort). Map the metabolomic enrichment file (linear_msea_genebig_allpeaks_combined.csv) onto STRING v11 node IDs using STRING_graph_file_v11_gene_to_nodeid_mapping, scale ME/DME scores between 0 and 1 as initial labels y, and run label propagation (Zhou et al. LGC formulation, alpha/lambda = 0.99) over STRING_graph_file_v11_functional_entire_db until convergence via the yamaguchiyuto/label_propagation code. Extract each candidate gene's propagated score (Score.ID) alongside its original pre-propagation score (Original.Score.ID), rank the candidate list by Score.ID, and check whether at least one known causative gene falls within the top 20th percentile (rank ≤ ~60 for a 300-gene list). Repeat per patient — the target replication result is ≥9/11 TIDEX patients with a causative gene in the top 20th percentile. Optionally compare ranks against Exomiser's hiPHIVE ordering (metPropagate wins in 6/11 patients) and test a weighted combination of metPropagate and Exomiser scores, which reportedly improves prioritization.

## Related tools

- **Exomiser** (Filters DeepVariant singleton VCFs via variant effect and population frequency filters to produce the 281–609-gene candidate list that is ranked; its hiPHIVE phenotype ranking provides the comparison baseline)
- **DeepVariant** (Creates the singleton VCFs per patient that are upstream of Exomiser filtering)
- **metPropagate** (Supplies the propagated Score.ID for every candidate gene after label propagation of ME/DME-scaled labels over the STRING v11 network) — github.com/emmagraham/metPropagate
- **STRING** (Protein functional linkage network (v11, functional_entire_db) over which metabolomic scores are propagated; node-ID mapping file links candidate gene names to network nodes)
- **yamaguchiyuto/label_propagation** (Adapted LGC (Local and Global Consistency) implementation run with alpha = 0.99 to propagate the metabolomic labels across the STRING graph) — https://github.com/yamaguchiyuto/label_propagation

## Examples

```
bash label_propagation/run_label_prop_cluster.sh
```

## Evaluation signals

- Success criterion: at least one known causative gene ranks in the top 20th percentile of the candidate list (rank ≤ ~60 for a 300-gene list; 55–115 genes for the 281–609-gene TIDEX lists), replicated in ≥9/11 TIDEX patients
- Output file schema check: results contain both Score.ID (propagated) and Original.Score.ID (pre-propagation) columns for each candidate gene
- Per-candidate-gene count check: every patient's ranked list should contain between 281 and 609 genes after Exomiser filtering
- Propagation sanity check: propagated Score.ID for genes with high initial labels should remain elevated relative to Original.Score.ID, with labels initialized by scaling ME scores between 0 and 1
- Benchmark comparison: metPropagate should rank the causative gene more highly than Exomiser's phenotype-based ranking in ~6/11 patients; a weighted combination of the two scores should improve prioritization

## Limitations

- Analyses used singleton exomes rather than trio WES; the authors note trio parental data would simplify prioritization and shorten candidate gene lists, so candidate-list sizes may not reflect current practice
- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, so the exact metabolic identity of each feature could not be determined
- The metabolic system could not measure large glycosaminoglycans, limiting ME analysis for the IDS patient
- No software version reported for the STRING network or Exomiser/hiPHIVE, hindering exact replication
- DAMs from the linear vs. log scaled intensity matrices are treated differently — the ranking workflow uses the linear-scaled enrichment file (linear_msea_genebig_allpeaks_combined.csv)

## Evidence

- [results] Exomiser variant filtering of DeepVariant singleton VCFs to generate candidate gene lists: "Singleton VCFs for all patient were created by DeepVariant, which were then filtered by Exomiser to generate a list of candidate genes"
- [discussion] Top 20th percentile corresponds to 55–115 genes on 281–609-gene candidate lists: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
- [readme] README documents the per-gene Score.ID / Original.Score.ID output consumed by the ranking step: "Results of label propagation will populate in *integration/results/*. File contains final propagated scores for each candidate gene as "Score.ID""
- [abstract] Benchmark against Exomiser's phenotype-based ranking: "ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients"
- [results] ME scores as labels over the STRING functional network before candidate ranking: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
