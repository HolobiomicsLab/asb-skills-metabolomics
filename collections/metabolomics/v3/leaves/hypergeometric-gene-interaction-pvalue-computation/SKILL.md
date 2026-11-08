---
name: hypergeometric-gene-interaction-pvalue-computation
description: Use when tokenised BGCs (genes as (sub)Pfam domain combinations) are
  available and gene-pair co-occurrence significance must be quantified to build sub-cluster
  interaction graphs.
when_to_use_negative:
- The analysis goal is topic-model-based sub-cluster detection (PRESTO-TOP), which
  uses an LDA bag-of-words model where original gene order is not considered and no
  pairwise p-values are computed.
- Only a small dataset (e.g. the Streptomyces/Salinispora dataset) is available, where
  graph-based filtering rather than genome-scale pairwise statistics was used for
  visualisation purposes.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0160
tools:
- name: iPRESTO
  role: Implements the PRESTO-STAT statistical sub-cluster prediction, including the
    hypergeometric p-value computation over gene-pair adjacency and co-localisation
    interactions.
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Provides the prokaryotic BGCs that are tokenised by (sub)Pfam detection prior
    to p-value computation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:8c78ee651741a70ef87b579904ca9e1276a4cc8c01a8d997f8005a6884884376
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Hypergeometric gene-interaction p-value computation

## Summary

A per-gene-pair statistical test in iPRESTO's PRESTO-STAT that quantifies whether two (sub)Pfam-tokenised genes co-occur within BGCs more often than expected by chance, using hypergeometric p-values for both adjacency and co-localisation interactions on the antiSMASH-DB dataset.

## When to use

Use when tokenised BGCs (genes as (sub)Pfam domain combinations) are available and gene-pair co-occurrence significance must be quantified to build sub-cluster interaction graphs.

## When NOT to use

- The analysis goal is topic-model-based sub-cluster detection (PRESTO-TOP), which uses an LDA bag-of-words model where original gene order is not considered and no pairwise p-values are computed.
- Only a small dataset (e.g. the Streptomyces/Salinispora dataset) is available, where graph-based filtering rather than genome-scale pairwise statistics was used for visualisation purposes.

## Inputs

- Filtered, tokenised non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain combinations), each gene represented as a combination of (sub)Pfam domains
- Per-gene (sub)Pfam token assignments with non-biosynthetic Pfams removed

## Outputs

- Per gene-pair hypergeometric p-values for adjacency and co-localisation interactions (conservative larger p-value retained)
- Benjamini–Yekutieli FDR-corrected co-localisation and adjacency p-values used to define edges (p < 0.1) in the gene interaction graphs

## How to apply

Load the filtered, tokenised non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain combinations) in which each gene is represented as a combination of (sub)Pfam domains. For each pair of genes, count adjacency and co-localisation interactions and compute hypergeometric p-values by keeping one gene fixed while randomly distributing the other across the dataset; retain only the larger of the two p-values per interaction for conservatism. Apply Benjamini–Yekutieli false discovery rate correction to both the co-localisation and adjacency p-values. The corrected p-values then feed graph construction, where an edge is added when the adjacency or co-localisation p-value falls below the 0.1 threshold and maximal cliques are extracted as candidate sub-clusters.

## Related tools

- **iPRESTO** (Implements the PRESTO-STAT statistical sub-cluster prediction, including the hypergeometric p-value computation over gene-pair adjacency and co-localisation interactions.) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Provides the prokaryotic BGCs that are tokenised by (sub)Pfam detection prior to p-value computation.)

## Evaluation signals

- The downstream sub-cluster table (presto_stat_subclusters.tsv) must contain exactly 108,085 sub-clusters on the filtered antiSMASH-DB dataset.
- Over 80% of the resulting statistical sub-clusters should contain fewer than ten genes.
- 17% of the resulting sub-clusters should occur in more than 10 BGCs.
- Retained per-interaction p-values must be the larger of the two hypergeometric p-values computed by fixing each gene in turn.
- All co-localisation and adjacency p-values must be Benjamini–Yekutieli FDR-corrected before graph edges are drawn at the p < 0.1 threshold.

## Limitations

- Computing pairwise hypergeometric p-values does not scale well; the article's objective was explicitly to address 'poor scalability and overestimation of sub-clusters by the previous COG-based statistical method', which motivated the complementary PRESTO-TOP LDA approach.
- Genes must be tokenised with (sub)Pfams first; non-biosynthetic Pfams must be removed, otherwise spurious interactions inflate p-value counts.
- Sub-clusters with fewer than three genes or occurring in only one BGC are filtered out as putative noise downstream, so p-value significance alone does not guarantee a retained sub-cluster.

## Evidence

- [other] For each pair of genes, count adjacency and co-localisation interactions and compute hypergeometric p-values by keeping one gene fixed while randomly distributing the other across the dataset: [evidence span withheld — anchor status: not_found]
- [other] Apply Benjamini–Yekutieli false discovery rate correction to both the co-localisation and adjacency p-values.: [evidence span withheld — anchor status: not_found]
- [other] Build undirected graphs with genes as nodes, adding an edge when the adjacency or co-localisation p-value is below the 0.1 threshold: [evidence span withheld — anchor status: not_found]
- [supplementary] the non-redundant 60,028 BGCs from the antiSMASH-DB dataset: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [supplementary] The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
