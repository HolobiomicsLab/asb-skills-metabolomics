---
name: subcluster-family-and-clan-grouping
description: Use when you have a large set of predicted sub-clusters (e.g. 108,085
  PRESTO-STAT sub-clusters from tens of thousands of BGCs) and need to organise them
  into families and clans for redundancy removal and relatedness assessment.
when_to_use_negative:
- Sub-clusters have not yet been predicted — this grouping step operates on PRESTO-STAT
  output, not directly on antiSMASH BGCs or tokenised genes
- The dataset is small or contains few sub-clusters, so a fixed 10,000/2,000 cluster
  count would be meaningless or over-fragment the data
- You need sub-clusters linked to Mass2Motif substructures (MS/MS correlation analysis)
  rather than hierarchical organisation of statistical sub-clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Source of PRESTO-STAT-predicted statistical sub-clusters and implementation
    of the K-means sub-cluster grouping procedure
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Provides the BGC predictions whose tokenised genes (domain combinations) form
    the presence/absence matrix input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:2292c2050e128f3f1be42e35ef7be49466d200d00219c159a81e1d9cc601b9c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# subcluster-family-and-clan-grouping

## Summary

Groups related PRESTO-STAT-predicted statistical sub-clusters into a two-level hierarchy — sub-cluster families (SCFs) and sub-cluster clans (SCCs) — via two rounds of K-means clustering on a presence/absence matrix of tokenised genes (domain combinations).

## When to use

Use when you have a large set of predicted sub-clusters (e.g. 108,085 PRESTO-STAT sub-clusters from tens of thousands of BGCs) and need to organise them into families and clans for redundancy removal and relatedness assessment.

## When NOT to use

- Sub-clusters have not yet been predicted — this grouping step operates on PRESTO-STAT output, not directly on antiSMASH BGCs or tokenised genes
- The dataset is small or contains few sub-clusters, so a fixed 10,000/2,000 cluster count would be meaningless or over-fragment the data
- You need sub-clusters linked to Mass2Motif substructures (MS/MS correlation analysis) rather than hierarchical organisation of statistical sub-clusters

## Inputs

- PRESTO-STAT-predicted statistical sub-clusters (e.g. 108,085 sub-clusters from the non-redundant antiSMASH-DB dataset)
- Presence/absence matrix of tokenised genes (domain combinations) per sub-cluster
- Number of target clusters for each K-means level (10,000 SCFs; 2,000 SCCs)

## Outputs

- 10,000 sub-cluster families (SCFs)
- 2,000 sub-cluster clans (SCCs) grouping related SCFs
- Relatedness QC metric: fraction of SCCs whose member sub-clusters share at least three genes (expected ~81%)

## How to apply

First load the predicted statistical sub-clusters (in the iPRESTO workflow: 108,085 PRESTO-STAT sub-clusters from the non-redundant antiSMASH-DB dataset of 60,028 BGCs, covering 10,539 domain combinations) and represent each sub-cluster as a row in a presence/absence matrix over the tokenised genes (domain combinations). Apply K-means clustering (scikit-learn) on this matrix to obtain 10,000 sub-cluster families (SCFs). Then run a second K-means clustering step on the SCFs to join similar families into 2,000 sub-cluster clans (SCCs), removing redundant sub-clusters during the grouping per the supplementary K-means grouping procedure. Finally compute the relatedness QC metric: the fraction of SCCs whose member sub-clusters share at least three genes, which the paper reports as 81%.

## Related tools

- **iPRESTO** (Source of PRESTO-STAT-predicted statistical sub-clusters and implementation of the K-means sub-cluster grouping procedure) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Provides the BGC predictions whose tokenised genes (domain combinations) form the presence/absence matrix input)

## Examples

```
from sklearn.cluster import KMeans; scfs = KMeans(n_clusters=10000).fit_predict(presence_absence_matrix); sccs = KMeans(n_clusters=2000).fit_predict(scf_centroids)
```

## Evaluation signals

- Output contains exactly 10,000 sub-cluster families (SCFs)
- Output contains exactly 2,000 sub-cluster clans (SCCs)
- Relatedness QC metric reproduces the reported 81%: 81% of SCCs group related sub-clusters sharing at least three genes
- Redundant sub-clusters are removed during grouping per the supplementary K-means grouping procedure
- Every sub-cluster in the input is assigned to exactly one SCF and one SCC (complete, non-overlapping partition)

## Limitations

- No version numbers are stated for antiSMASH or iPRESTO in the context, so exact reproducibility of cluster counts may depend on the software version used
- The cluster counts (10,000 SCFs, 2,000 SCCs) and the 81% relatedness metric were calibrated on the non-redundant antiSMASH-DB dataset (60,028 BGCs); other datasets may require different K values
- Grouping quality depends on the underlying PRESTO-STAT sub-cluster predictions and the tokenisation of BGCs by (sub)Pfam domain combinations

## Evidence

- [methods] K-means on presence/absence matrix: [evidence span withheld — anchor status: not_found]
- [methods] Second K-means level for SCCs: [evidence span withheld — anchor status: not_found]
- [supplementary] Input dataset definition: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [results] Relatedness QC metric: [evidence span withheld — anchor status: not_found]
