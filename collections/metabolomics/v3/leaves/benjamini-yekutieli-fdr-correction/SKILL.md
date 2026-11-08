---
name: benjamini-yekutieli-fdr-correction
description: Use when a large, dependent family of p-values (e.g. hypergeometric tests
  for gene-pair adjacency or co-localisation in BGCs) must be FDR-corrected before
  thresholding, as in PRESTO-STAT sub-cluster detection.
when_to_use_negative:
- When the p-value family is small and mutually independent, where a less conservative
  Benjamini–Hochberg correction would retain more true interactions
- When testing a single pre-specified hypothesis (e.g. one gene pair) rather than
  a genome-scale interaction family — no multiple-testing correction is warranted
- When the analysis goal is PRESTO-TOP topic-modelling sub-cluster prediction, which
  uses LDA on a bag-of-words gene model and does not involve p-value correction
tools:
- name: iPRESTO
  role: Command-line tool whose PRESTO-STAT module computes the hypergeometric interaction
    p-values to which the Benjamini–Yekutieli correction is applied
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:12849ab8c26a8ff34e44b3ed8328ab5d3322cccdd3a4f4e6b054e7b13e7db3a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - iPRESTO
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Benjamini–Yekutieli FDR correction

## Summary

Applies the Benjamini–Yekutieli false discovery rate correction to the hypergeometric p-values from iPRESTO's PRESTO-STAT gene-pair interaction analysis, controlling false discovery under dependency between the many correlated gene co-occurrence tests across the tokenised antiSMASH-DB dataset.

## When to use

Use when a large, dependent family of p-values (e.g. hypergeometric tests for gene-pair adjacency or co-localisation in BGCs) must be FDR-corrected before thresholding, as in PRESTO-STAT sub-cluster detection.

## When NOT to use

- When the p-value family is small and mutually independent, where a less conservative Benjamini–Hochberg correction would retain more true interactions
- When testing a single pre-specified hypothesis (e.g. one gene pair) rather than a genome-scale interaction family — no multiple-testing correction is warranted
- When the analysis goal is PRESTO-TOP topic-modelling sub-cluster prediction, which uses LDA on a bag-of-words gene model and does not involve p-value correction

## Inputs

- Family of hypergeometric p-values for gene-pair adjacency interactions (tokenised (sub)Pfam genes in original BGC order)
- Family of hypergeometric p-values for gene-pair co-localisation interactions
- Filtered, non-redundant antiSMASH-DB tokenised dataset (60,028 BGCs, 10,539 domain combinations)

## Outputs

- FDR-corrected (Benjamini–Yekutieli-adjusted) p-values for adjacency interactions
- FDR-corrected (Benjamini–Yekutieli-adjusted) p-values for co-localisation interactions
- Corrected p-value sets feeding the 0.1-threshold undirected gene graphs used for maximal-clique sub-cluster extraction

## How to apply

First compute one hypergeometric p-value per gene pair by counting adjacency and co-localisation interactions while keeping one gene fixed and randomly distributing the other across the non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain-combination tokens); retain the larger of the two p-values per interaction for conservatism. Then apply the Benjamini–Yekutieli FDR correction separately to the adjacency p-value family and the co-localisation p-value family — BY is chosen over Benjamini–Hochberg because gene-pair tests are dependent (shared genes and shared BGCs violate independence). Corrected p-values feed directly into graph construction: an undirected edge is added between two genes when the corrected p-value is below the 0.1 threshold, and maximal cliques of that graph are extracted as candidate sub-clusters. Because BY is conservative, sub-clusters with fewer than three genes or occurring in only one BGC are discarded downstream.

## Related tools

- **iPRESTO** (Command-line tool whose PRESTO-STAT module computes the hypergeometric interaction p-values to which the Benjamini–Yekutieli correction is applied) — https://git.wageningenur.nl/bioinformatics/iPRESTO/

## Evaluation signals

- Corrected p-values must be monotonically ≥ the raw hypergeometric p-values for every tested gene pair
- Adjacency and co-localisation p-value families are each corrected independently, and the larger of the two raw p-values per interaction is retained before correction
- After adding edges only where corrected p-values < 0.1 and filtering sub-clusters with <3 genes or occurring in only one BGC, the output table contains exactly 108,085 sub-clusters
- Sanity distribution check: over 80% of recovered statistical sub-clusters contain fewer than ten genes, and 17% occur in more than 10 BGCs

## Limitations

- Benjamini–Yekutieli is more conservative than Benjamini–Hochberg under independence, so some weakly significant gene-pair interactions are lost; iterative threshold lowering (all p-values below 0.1) is used to compensate
- Retention of only the larger of the two p-values per interaction adds further conservatism, reducing sensitivity for rare gene pairs
- Sub-clusters of length one are treated as almost definitely noise and are excluded, so single-gene signals cannot be recovered

## Evidence

- [supplementary] Apply Benjamini–Yekutieli false discovery rate correction to both the co-localisation and adjacency p-values.: [evidence span withheld — anchor status: not_found]
- [other] compute hypergeometric p-values by keeping one gene fixed while randomly distributing the other across the dataset, retaining only the larger of the two p-values per interaction for conservatism: [evidence span withheld — anchor status: not_found]
- [supplementary] adding an edge when the adjacency or co-localisation p-value is below the 0.1 threshold, and extract all maximal cliques as sub-clusters: [evidence span withheld — anchor status: not_found]
- [intro] iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP): "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
