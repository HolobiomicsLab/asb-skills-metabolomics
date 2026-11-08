---
name: permutation-based-prioritization-benchmarking
description: Use when you have a ranked per-patient candidate gene list with a known
  causative gene and need to quantify whether the prioritization rate exceeds chance,
  e.g. reproducing metPropagate's 92% top-20th-percentile rate.
when_to_use_negative:
- When no known causative gene exists for a case (e.g. novel discovery patients like
  TIDEX), since the permutation requires seeding the causative gene in each random
  list
- When candidate lists are much smaller than the fixed 300-gene design (e.g. trio
  WES lists), where the top-20th-percentile cutoff of rank <= 60 no longer matches
  the article's 55-115 gene range per patient
tools:
- name: metPropagate
  role: Supplies the propagated scores that are re-ranked on each permuted candidate
    gene list
  repo: https://github.com/emmagraham/metPropagate
- name: STRING
  role: Defines the 19,354-gene universe from which the 300-gene random candidate
    lists are drawn
- name: NumPy
  role: Used for random gene sampling and rank aggregation in the Python benchmarking
    step
- name: tidyverse
  role: R package used to aggregate per-patient outcomes and write the rankings summary
    CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:dcb45702d1050683b13d3e52a054042f6919f076f8da310559dce19a2bd49438
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - HMDB
  - NumPy
  - STRING
  - metPropagate
  - scikit-learn
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# permutation-based-prioritization-benchmarking

## Summary

Benchmark a gene-prioritization method by generating random candidate gene lists that include the known causative gene, ranking it against random genes, and scoring how often it lands in the top 20th percentile — as done for metPropagate on the Miller 107-patient IEM dataset.

## When to use

Use when you have a ranked per-patient candidate gene list with a known causative gene and need to quantify whether the prioritization rate exceeds chance, e.g. reproducing metPropagate's 92% top-20th-percentile rate.

## When NOT to use

- When no known causative gene exists for a case (e.g. novel discovery patients like TIDEX), since the permutation requires seeding the causative gene in each random list
- When candidate lists are much smaller than the fixed 300-gene design (e.g. trio WES lists), where the top-20th-percentile cutoff of rank <= 60 no longer matches the article's 55-115 gene range per patient

## Inputs

- Per-patient ranked candidate gene lists with propagated scores (e.g. metPropagate output from label propagation over STRING v11)
- Mapping of each Miller patient to its IEM and causative gene(s) (21 IEMs, 107 patients)
- Gene universe list: 19,354 STRING v11 human genes (STRING_graph_file_v11_gene_list_functional_entire_db)
- Supplementary Data Set 1 of reported median ranks for comparison

## Outputs

- Per-patient causative gene rank statistics across 1000 permutations (median rank per IEM-associated gene, Supplementary Data Set 1)
- miller_metpropagate_rankings_summary.csv with per-patient top-20th-percentile prioritization calls (rank <= 60 of 300) and the aggregate prioritization rate

## How to apply

For each patient (or case), keep the causative gene(s) fixed and draw random genes to fill a candidate list of fixed size — in the metPropagate Miller benchmark, 300 genes drawn from the 19,354 STRING v11-annotated genes. Generate 1000 such permutations per patient, apply the prioritization score (here, the propagated metPropagate label-propagation score) to each permuted list, and record the rank of the causative gene in each permutation. Compute the median rank of each IEM-associated gene across permutations (Supplementary Data Set 1). Define success per patient as at least one causative gene falling in the top 20th percentile of the candidate list (rank <= 60 of 300 genes); aggregate the per-patient success rate and write rankings plus summary rates to a CSV such as miller_metpropagate_rankings_summary.csv. Accept the run only if the aggregate prioritization rate across the 107 Miller patients reproduces the reported ~91-92%.

## Related tools

- **metPropagate** (Supplies the propagated scores that are re-ranked on each permuted candidate gene list) — https://github.com/emmagraham/metPropagate
- **STRING** (Defines the 19,354-gene universe from which the 300-gene random candidate lists are drawn)
- **NumPy** (Used for random gene sampling and rank aggregation in the Python benchmarking step)
- **tidyverse** (R package used to aggregate per-patient outcomes and write the rankings summary CSV)

## Evaluation signals

- Aggregate prioritization rate across all 107 Miller patients falls in the reported 91-92% range (98/107 patients with at least one causative gene in the top 20th percentile)
- Per-patient prioritization is scored as rank <= 60 of 300 candidate genes (top 20th percentile)
- Each permutation list has exactly 300 genes, contains the causative gene, and all other genes are drawn from the 19,354-gene STRING set without replacement
- Number of permutations equals 1000 per patient and the median (not mean) rank per IEM-associated gene is recorded
- Output CSV (miller_metpropagate_rankings_summary.csv) contains per-patient rankings plus aggregate summary rates

## Limitations

- The 300-gene candidate list is a simulation; real Exomiser-filtered lists varied between 281 and 609 genes, so the top-20th-percentile cutoff (55-115 genes) differs from the fixed rank <= 60 benchmark
- Benchmarking is only possible for IEMs with known causative genes in OMIM; it cannot evaluate performance on novel disease genes
- Analyses used singleton exomes rather than trio WES, so real-world candidate lists would be shorter than modeled

## Evidence

- [results] 1000 permutations of random candidate gene lists that included the causative gene were generated: "1000 permutations of random candidate gene lists that included the causative gene were generated"
- [abstract] top 20th percentile prioritization rate: "metPropagate was able to prioritize at least one causative gene in the top 20th percentile of candidate genes for 92% of patients with known IEMs."
- [discussion] candidate list size context: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
- [readme] score files to be ranked and benchmarked: "Results of label propagation will populate in *integration/results/*. File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as"
