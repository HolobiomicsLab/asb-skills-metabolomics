---
name: scc-gene-sharing-relatedness-validation
description: Use when validating hierarchical K-means grouping of predicted biosynthetic
  sub-clusters into sub-cluster families (SCFs) and clans (SCCs), where correctness
  is judged by gene-sharing relatedness within clans.
when_to_use_negative:
- The sub-clusters come from PRESTO-TOP (LDA topic modelling) rather than the PRESTO-STAT
  statistical co-occurrence method, whose output distribution differs from the 108,085
  statistical sub-clusters this QC was calibrated on
- The goal is de novo sub-cluster prediction from BGCs rather than validating an already-computed
  hierarchical grouping of sub-clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0736
tools:
- name: iPRESTO
  role: Source of the PRESTO-STAT sub-clusters and the K-means grouping procedure
    (SCFs into SCCs) whose relatedness is validated by this QC metric
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH-DB
  role: Provides the non-redundant set of 60,028 BGCs whose tokenised (sub)Pfam domain
    combinations underlie the presence/absence matrix used for K-means clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:7d7fde6e043f253d48c9863ebfee9f57855b4521f2b79619ead34f5fd44be00a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH-DB
  - iPRESTO
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0736
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# scc-gene-sharing-relatedness-validation

## Summary

Quality-control validation for iPRESTO's PRESTO-STAT sub-cluster grouping, which checks that sub-cluster clans (SCCs) produced by two-stage K-means clustering are biologically coherent by measuring the fraction of SCCs whose member sub-clusters share at least three tokenised genes.

## When to use

Use when validating hierarchical K-means grouping of predicted biosynthetic sub-clusters into sub-cluster families (SCFs) and clans (SCCs), where correctness is judged by gene-sharing relatedness within clans.

## When NOT to use

- The sub-clusters come from PRESTO-TOP (LDA topic modelling) rather than the PRESTO-STAT statistical co-occurrence method, whose output distribution differs from the 108,085 statistical sub-clusters this QC was calibrated on
- The goal is de novo sub-cluster prediction from BGCs rather than validating an already-computed hierarchical grouping of sub-clusters

## Inputs

- PRESTO-STAT-predicted sub-clusters from the non-redundant antiSMASH-DB dataset (108,085 sub-clusters from 60,028 BGCs, 10,539 domain combinations)
- Presence/absence matrix of tokenised genes (sub)Pfam domain combinations per sub-cluster

## Outputs

- 10,000 sub-cluster families (SCFs) from the first K-means clustering
- 2,000 sub-cluster clans (SCCs) from the second K-means clustering
- Relatedness QC metric: fraction of SCCs whose member sub-clusters share at least three genes (expected 81%)

## How to apply

After loading PRESTO-STAT-predicted sub-clusters from the non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain combinations, 108,085 predicted sub-clusters), represent each sub-cluster as a presence/absence matrix of tokenised genes (sub)Pfam domain combinations. Apply K-means clustering (scikit-learn) to group sub-clusters into 10,000 sub-cluster families (SCFs), then a second K-means step on the SCFs to join similar families into 2,000 sub-cluster clans (SCCs), removing redundant sub-clusters per the supplementary K-means grouping procedure. Then compute the relatedness QC metric: the fraction of SCCs whose member sub-clusters share at least three genes, and verify it reproduces the reported 81%. Correctness is confirmed only if the output contains exactly 10,000 SCFs and 2,000 SCCs and the relatedness metric matches 81%.

## Related tools

- **iPRESTO** (Source of the PRESTO-STAT sub-clusters and the K-means grouping procedure (SCFs into SCCs) whose relatedness is validated by this QC metric) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH-DB** (Provides the non-redundant set of 60,028 BGCs whose tokenised (sub)Pfam domain combinations underlie the presence/absence matrix used for K-means clustering)

## Examples

```
from sklearn.cluster import KMeans; scf_labels = KMeans(n_clusters=10000).fit_predict(presence_absence_matrix); scc_labels = KMeans(n_clusters=2000).fit_predict(scf_centroids)
```

## Evaluation signals

- The K-means grouping yields exactly 10,000 sub-cluster families (SCFs)
- The second K-means step on SCFs yields exactly 2,000 sub-cluster clans (SCCs)
- 81% of SCCs contain member sub-clusters sharing at least three genes (the reported relatedness QC metric)
- The number of tokenised genes (domain combinations) in the presence/absence matrix matches the expected 10,539 domain combinations from the non-redundant antiSMASH-DB dataset
- Sub-clusters of length one are excluded as noise, consistent with the supplementary filtering rule

## Limitations

- The 81% relatedness metric implies roughly one in five SCCs does not group sub-clusters sharing at least three genes, so clan membership should not be treated as universally gene-coherent
- Redundant sub-clusters are removed during grouping per a supplementary K-means procedure whose exact parameters are only documented in supplementary methods
- No version numbers are stated for iPRESTO or antiSMASH in the supplementary section, so exact reproduction may require checking the repository

## Evidence

- [supplementary] Related statistical sub-clusters are grouped using K-means clustering, with the statistical sub-clusters represented as a presence/absence matrix: [evidence span withheld — anchor status: not_found]
- [supplementary] the non-redundant 60,028 BGCs from the antiSMASH-DB dataset: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [supplementary] The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene"
- [supplementary] not counting sub-clusters of length one as these are almost definitely noise (see Methods): "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [intro] iPRESTO is available as a command-line tool: "iPRESTO is available as a command-line tool at https://git.wageningenur.nl/bioinformatics/iPRESTO/."
