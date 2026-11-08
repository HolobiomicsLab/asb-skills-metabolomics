---
name: redundant-subcluster-removal
description: Use when a large collection of PRESTO-STAT statistical sub-clusters (e.g.
  108,085 from 60,028 non-redundant antiSMASH-DB BGCs) must be de-replicated and organised
  into hierarchical sub-cluster families and clans.
when_to_use_negative:
- Input consists of PRESTO-TOP topic-modelled sub-clusters, whose native representation
  is a bag-of-words LDA model rather than the presence/absence matrix of tokenised
  genes used here
- Input is a single BGC or a small BGC set — K-means grouping into SCFs/SCCs requires
  a large corpus of predicted sub-clusters to be meaningful
- The goal is clustering whole BGCs by gene content; this skill operates at the sub-cluster
  level within BGCs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Generates the PRESTO-STAT sub-clusters and implements the supplementary K-means
    grouping procedure for SCF/SCC construction and redundancy removal
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Provides the BGCs from antiSMASH-DB that are tokenised into (sub)Pfam domain
    combinations prior to sub-cluster prediction and grouping
- name: scikit-learn
  role: Supplies the K-means clustering used at both the SCF and SCC grouping levels
    on the presence/absence matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:f711af847a7974a586989a94b8666ed9c860f7f804936d0473b8f65de56a840b
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# redundant-subcluster-removal

## Summary

Groups related PRESTO-STAT-predicted biosynthetic sub-clusters into sub-cluster families (SCFs) and clans (SCCs) with K-means on a presence/absence matrix of tokenised genes, removing redundant sub-clusters during grouping.

## When to use

Use when a large collection of PRESTO-STAT statistical sub-clusters (e.g. 108,085 from 60,028 non-redundant antiSMASH-DB BGCs) must be de-replicated and organised into hierarchical sub-cluster families and clans.

## When NOT to use

- Input consists of PRESTO-TOP topic-modelled sub-clusters, whose native representation is a bag-of-words LDA model rather than the presence/absence matrix of tokenised genes used here
- Input is a single BGC or a small BGC set — K-means grouping into SCFs/SCCs requires a large corpus of predicted sub-clusters to be meaningful
- The goal is clustering whole BGCs by gene content; this skill operates at the sub-cluster level within BGCs

## Inputs

- 108,085 PRESTO-STAT-predicted statistical sub-clusters from the non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain combinations)
- Presence/absence matrix of tokenised genes (domain combinations) per sub-cluster

## Outputs

- 10,000 sub-cluster families (SCFs)
- 2,000 sub-cluster clans (SCCs) grouping related SCFs
- Relatedness QC metric: fraction of SCCs whose member sub-clusters share at least three genes (expected 81%)

## How to apply

Start from the PRESTO-STAT-predicted sub-clusters derived from BGCs tokenised by detecting (sub)Pfams in each gene (with non-biosynthetic Pfams removed). Represent each statistical sub-cluster as a presence/absence matrix over the tokenised genes (domain combinations), then apply scikit-learn K-means to group the 108,085 sub-clusters into 10,000 sub-cluster families (SCFs), dropping redundant sub-clusters during the grouping per the supplementary K-means grouping procedure. Run a second K-means step on the SCFs to merge similar families into 2,000 sub-cluster clans (SCCs). Sub-clusters of length one must not be counted, as these are almost definitely noise. Judge success by reproducing the reported counts (10,000 SCFs, 2,000 SCCs) and the relatedness QC metric: 81% of SCCs whose member sub-clusters share at least three genes; the 0.6 overlap cut-off is the article's threshold for considering a sub-cluster 'predicted' during validation.

## Related tools

- **iPRESTO** (Generates the PRESTO-STAT sub-clusters and implements the supplementary K-means grouping procedure for SCF/SCC construction and redundancy removal) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Provides the BGCs from antiSMASH-DB that are tokenised into (sub)Pfam domain combinations prior to sub-cluster prediction and grouping)
- **scikit-learn** (Supplies the K-means clustering used at both the SCF and SCC grouping levels on the presence/absence matrix)

## Examples

```
from sklearn.cluster import KMeans; scfs = KMeans(n_clusters=10000).fit_transform(subcluster_presence_absence_matrix); sccs = KMeans(n_clusters=2000).fit_transform(scfs)
```

## Evaluation signals

- K-means on the presence/absence matrix yields exactly 10,000 sub-cluster families (SCFs)
- The second K-means step over SCFs yields exactly 2,000 sub-cluster clans (SCCs)
- 81% of SCCs contain member sub-clusters sharing at least three genes (relatedness QC metric)
- Sub-clusters of length one are excluded as noise during grouping
- Grouping is run on the non-redundant 60,028-BGC antiSMASH-DB set, so redundant sub-clusters are removed during the procedure rather than after

## Limitations

- K-means requires pre-specified cluster counts (10,000 SCFs and 2,000 SCCs in the article); changing these targets changes the family/clan structure
- Sub-clusters of length one are almost definitely noise and must not be counted, so very short predicted sub-clusters cannot contribute to families
- Validation relies on the 0.6 overlap cut-off for calling a sub-cluster 'predicted', which is a chosen threshold rather than a ground-truth measure

## Evidence

- [supplementary] K-means grouping of PRESTO-STAT sub-clusters into SCFs and SCCs: [evidence span withheld — anchor status: not_found]
- [supplementary] Source of the sub-clusters being grouped: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [supplementary] Tokenisation upstream of the presence/absence matrix: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] Noise exclusion rule during grouping: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [supplementary] Validation overlap threshold for sub-cluster prediction: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [intro] Distinguishing PRESTO-STAT from PRESTO-TOP scope: "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
