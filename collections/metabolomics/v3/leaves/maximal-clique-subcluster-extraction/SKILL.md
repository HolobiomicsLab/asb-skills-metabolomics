---
name: maximal-clique-subcluster-extraction
description: Use when tokenised BGCs (each gene as a (sub)Pfam domain combination,
  original gene order retained) must be grouped into sub-clusters via gene co-localisation
  and adjacency statistics.
when_to_use_negative:
- When gene order is unavailable or a bag-of-words representation is preferred — iPRESTO's
  PRESTO-TOP LDA approach, which does not consider original gene order, is the intended
  alternative
- When the input BGCs have not been tokenised into (sub)Pfam domain combinations with
  non-biosynthetic Pfams removed
- 'When singletons are of interest: putative sub-clusters with fewer than three genes
  or occurring in only one BGC are deliberately filtered out as noise'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3473
tools:
- name: iPRESTO
  role: 'Command-line tool implementing PRESTO-STAT: hypergeometric interaction testing,
    FDR correction, graph construction, maximal clique extraction, and sub-cluster
    family/clan clustering'
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Supplies the prokaryotic BGCs from antiSMASH-DB whose tokenised genes are
    clustered by this skill
- name: scikit-learn
  role: K-means clustering of extracted sub-clusters into families and clans using
    the presence/absence matrix of tokenised genes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:4bdc767e2902ae949b46846536ff31de04805f9e2fa82c8da3ff5fa6ff468492
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - iPRESTO
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# maximal-clique-subcluster-extraction

## Summary

PRESTO-STAT procedure from iPRESTO that turns statistical co-occurrence of tokenised BGC genes into undirected graphs and extracts maximal cliques as biosynthetic sub-clusters. It is the statistical (co-localisation/adjacency) branch of iPRESTO's unsupervised sub-cluster detection.

## When to use

Use when tokenised BGCs (each gene as a (sub)Pfam domain combination, original gene order retained) must be grouped into sub-clusters via gene co-localisation and adjacency statistics.

## When NOT to use

- When gene order is unavailable or a bag-of-words representation is preferred — iPRESTO's PRESTO-TOP LDA approach, which does not consider original gene order, is the intended alternative
- When the input BGCs have not been tokenised into (sub)Pfam domain combinations with non-biosynthetic Pfams removed
- When singletons are of interest: putative sub-clusters with fewer than three genes or occurring in only one BGC are deliberately filtered out as noise

## Inputs

- Filtered, tokenised non-redundant antiSMASH-DB dataset of 60,028 prokaryotic BGCs represented as 10,539 (sub)Pfam domain combinations, with genes in their original order
- Gene adjacency and co-localisation interaction counts across the BGC collection

## Outputs

- presto_stat_subclusters.tsv containing 108,085 statistical sub-clusters (multi-gene, multi-BGC)
- Sub-cluster families and clans from K-means clustering (scikit-learn) on the presence/absence matrix of tokenised genes

## How to apply

First tokenise antiSMASH-predicted BGCs by detecting (sub)Pfams in each gene and removing non-biosynthetic Pfams, working on the non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain combinations). For each gene pair, count adjacency and co-localisation interactions and compute hypergeometric p-values by keeping one gene fixed while randomly distributing the other across the dataset, retaining only the larger of the two p-values per interaction for conservatism; correct both p-value sets with the Benjamini–Yekutieli false discovery rate. Build an undirected graph with genes as nodes, adding an edge when the adjacency or co-localisation p-value falls below the 0.1 threshold, then extract all maximal cliques as candidate sub-clusters, iteratively lowering the threshold to all p-values below 0.1. Filter out putative sub-clusters with fewer than three genes or those occurring in only one BGC (length-1 sub-clusters are almost definitely noise). Optionally group similar sub-clusters into families and clans by K-means clustering (scikit-learn) on a presence/absence matrix of tokenised genes, and write the full table to presto_stat_subclusters.tsv. On the filtered antiSMASH-DB v2 dataset this yields exactly 108,085 sub-clusters, over 80% of which contain fewer than ten genes and 17% of which occur in more than 10 BGCs.

## Related tools

- **iPRESTO** (Command-line tool implementing PRESTO-STAT: hypergeometric interaction testing, FDR correction, graph construction, maximal clique extraction, and sub-cluster family/clan clustering) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Supplies the prokaryotic BGCs from antiSMASH-DB whose tokenised genes are clustered by this skill)
- **scikit-learn** (K-means clustering of extracted sub-clusters into families and clans using the presence/absence matrix of tokenised genes)

## Evaluation signals

- Output TSV contains exactly 108,085 sub-clusters when run on the filtered, non-redundant antiSMASH-DB v2 dataset
- Over 80% of detected statistical sub-clusters contain fewer than ten genes
- 17% of sub-clusters occur in more than 10 BGCs
- No emitted sub-cluster has fewer than three genes or occurs in only one BGC
- Edge set derived from Benjamini–Yekutieli-corrected adjacency and co-localisation p-values below the 0.1 threshold, keeping the larger of the two p-values per interaction

## Limitations

- Sub-clusters of length one are treated as almost definitely noise and excluded, so single-gene co-occurrence signals are lost
- The previous COG-based statistical method suffered from poor scalability and overestimation of sub-clusters; this skill's conservative choices (larger of two p-values, BY correction, <3-gene and single-BGC filters) trade sensitivity for precision
- No version numbers are stated in the supplementary documentation for antiSMASH or iPRESTO, so exact reproducibility may require checking the repository

## Evidence

- [intro] iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP): "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
- [supplementary] Tokenised BGC input definition: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] Non-redundant dataset scale: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [supplementary] Noise filtering rationale: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [abstract] Scalable unsupervised sub-cluster detection motivation: "we constructed a scalable method for unsupervised sub-cluster detection, called iPRESTO"
