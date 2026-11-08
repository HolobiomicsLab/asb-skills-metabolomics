---
name: kmeans-subcluster-family-clustering
description: Use when a large set of predicted sub-clusters (e.g. 108,085 PRESTO-STAT
  sub-clusters from 60,028 antiSMASH-DB BGCs) must be organised into related families
  and higher-level clans.
when_to_use_negative:
- Sub-clusters have not yet been filtered (fewer than three genes or occurring in
  only one BGC must already be removed by PRESTO-STAT)
- Input is raw gene/domain token data rather than a per-sub-cluster presence/absence
  matrix over tokenised genes
- The goal is de novo sub-cluster detection itself (use PRESTO-STAT hypergeometric
  interaction graphs or PRESTO-TOP LDA) rather than grouping existing sub-clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0736
tools:
- name: scikit-learn K-means
  role: 'Performs both clustering levels: sub-clusters into 10,000 SCFs, then SCFs
    into 2,000 SCCs, on the presence/absence matrix of tokenised genes'
- name: iPRESTO
  role: Provides the PRESTO-STAT sub-clusters and implements the supplementary K-means
    grouping procedure that families and clans are derived from
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:436de56664681404b675be4ecdcd51dfc30c77a4f6f60234b9ddd3573bf83191
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - iPRESTO
  - scikit-learn K-means
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0736
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# kmeans-subcluster-family-clustering

## Summary

Groups predicted biosynthetic sub-clusters into a two-level hierarchy of sub-cluster families (SCFs) and sub-cluster clans (SCCs) by K-means clustering (scikit-learn) on a presence/absence matrix of tokenised genes, as used by iPRESTO's PRESTO-STAT post-processing step.

## When to use

Use when a large set of predicted sub-clusters (e.g. 108,085 PRESTO-STAT sub-clusters from 60,028 antiSMASH-DB BGCs) must be organised into related families and higher-level clans.

## When NOT to use

- Sub-clusters have not yet been filtered (fewer than three genes or occurring in only one BGC must already be removed by PRESTO-STAT)
- Input is raw gene/domain token data rather than a per-sub-cluster presence/absence matrix over tokenised genes
- The goal is de novo sub-cluster detection itself (use PRESTO-STAT hypergeometric interaction graphs or PRESTO-TOP LDA) rather than grouping existing sub-clusters

## Inputs

- Table of predicted statistical sub-clusters (e.g. presto_stat_subclusters.tsv with 108,085 PRESTO-STAT sub-clusters)
- Tokenised gene definitions: each gene represented as a combination of (sub)Pfam domains from the non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain combinations)
- Fixed cluster counts for the two-level hierarchy: 10,000 SCFs and 2,000 SCCs

## Outputs

- Sub-cluster to sub-cluster family (SCF) assignment for 10,000 SCFs
- SCF to sub-cluster clan (SCF-to-SCC) assignment for 2,000 SCCs
- Relatedness QC metric: fraction of SCCs whose member sub-clusters share at least three genes (expected 81%)

## How to apply

Load the predicted sub-clusters and represent each one as a presence/absence binary vector over the tokenised genes, i.e. the (sub)Pfam domain combinations used to tokenise each BGC gene. Apply K-means clustering from scikit-learn to this matrix to partition the sub-clusters into 10,000 sub-cluster families (SCFs). Then apply a second K-means clustering step on the SCF representations to join similar families into 2,000 sub-cluster clans (SCCs), removing redundant sub-clusters during grouping per the supplementary K-means grouping procedure. Rationale: sub-clusters detected independently by the statistical hypergeometric interaction method often share tokenised genes, so binary co-presence vectors capture gene-content relatedness and K-means collapses them into reusable family/clan units. Validate the hierarchy by computing the fraction of SCCs whose member sub-clusters share at least three genes, which should reproduce the reported 81%.

## Related tools

- **scikit-learn K-means** (Performs both clustering levels: sub-clusters into 10,000 SCFs, then SCFs into 2,000 SCCs, on the presence/absence matrix of tokenised genes)
- **iPRESTO** (Provides the PRESTO-STAT sub-clusters and implements the supplementary K-means grouping procedure that families and clans are derived from) — https://git.wageningenur.nl/bioinformatics/iPRESTO/

## Examples

```
from sklearn.cluster import KMeans; scf = KMeans(n_clusters=10000).fit(presence_absence_matrix); scc = KMeans(n_clusters=2000).fit(scf_centers(scf))
```

## Evaluation signals

- Output contains exactly 10,000 sub-cluster families (SCFs)
- Second-level output contains exactly 2,000 sub-cluster clans (SCCs)
- 81% of SCCs group related sub-clusters sharing at least three genes (relatedness QC metric)
- Every sub-cluster in the input table is assigned to exactly one SCF (no unassigned members)
- Redundant sub-clusters are removed during grouping per the supplementary procedure (no duplicate sub-cluster content within families)

## Limitations

- Cluster counts (10,000 SCFs and 2,000 SCCs) are fixed hyperparameters from the antiSMASH-DB analysis; other datasets may require different numbers of clusters
- Bag-of-genes representation ignores gene order, so grouping reflects domain-combination content only
- Sub-clusters of length one or fewer than three genes, or those occurring in a single BGC, must be filtered out beforehand as they are treated as noise

## Evidence

- [results] Related statistical sub-clusters are grouped using K-means clustering, with the statistical sub-clusters represented as a presence/absence matrix: [evidence span withheld — anchor status: not_found]
- [methods] Apply K-means clustering (scikit-learn) on the presence/absence matrix to group sub-clusters into 10,000 sub-cluster families (SCFs): [evidence span withheld — anchor status: not_found]
- [methods] Apply a second K-means clustering step on the SCFs to join similar families into 2,000 sub-cluster clans (SCCs): [evidence span withheld — anchor status: not_found]
- [results] the fraction of SCCs whose member sub-clusters share at least three genes, and verify it reproduces the reported 81%: [evidence span withheld — anchor status: not_found]
- [supplementary] BGC tokenisation by detecting (sub)Pfams in each gene: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
