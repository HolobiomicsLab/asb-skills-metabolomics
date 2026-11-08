---
name: presence-absence-matrix-construction-from-tokenised-genes
description: Use when you have a large set of predicted statistical sub-clusters (e.g.
  PRESTO-STAT output from antiSMASH-DB BGCs) and need to organise related ones into
  hierarchical families/clans via a binary gene-content representation.
when_to_use_negative:
- Input is raw antiSMASH BGCs that have not yet been tokenised and processed by PRESTO-STAT
  — run BGC tokenisation and sub-cluster prediction first.
- The analysis requires gene-order information — a presence/absence matrix is a bag-of-genes
  representation (like the LDA 'bag of words' model) and discards original gene order
  that the statistical (Stat) method preserves.
- The data is already grouped into SCFs/SCCs and only needs annotation or query against
  sub-cluster families/clans.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3473
tools:
- name: iPRESTO
  role: Command-line tool providing PRESTO-STAT sub-cluster prediction and the K-means
    grouping of sub-clusters into SCFs and SCCs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Source of the predicted BGCs whose genes are tokenised by detecting (sub)Pfams
    before matrix construction
- name: satriaphd/build_subpfam
  role: Builds the per-clade subPfam pHMM models used to tokenise genes into domain
    combinations
  repo: https://github.com/satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:c0c458b5e9ebc7324d54603e3729e955f49c50a0609545d1e49fb910832e27c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - iPRESTO
  - satriaphd/build_subpfam
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# presence-absence-matrix-construction-from-tokenised-genes

## Summary

Represents PRESTO-STAT-predicted statistical sub-clusters as a presence/absence matrix over tokenised genes (domain combinations/subPfams) so that two rounds of K-means clustering can group 108,085 sub-clusters into 10,000 sub-cluster families (SCFs) and 2,000 sub-cluster clans (SCCs).

## When to use

Use when you have a large set of predicted statistical sub-clusters (e.g. PRESTO-STAT output from antiSMASH-DB BGCs) and need to organise related ones into hierarchical families/clans via a binary gene-content representation.

## When NOT to use

- Input is raw antiSMASH BGCs that have not yet been tokenised and processed by PRESTO-STAT — run BGC tokenisation and sub-cluster prediction first.
- The analysis requires gene-order information — a presence/absence matrix is a bag-of-genes representation (like the LDA 'bag of words' model) and discards original gene order that the statistical (Stat) method preserves.
- The data is already grouped into SCFs/SCCs and only needs annotation or query against sub-cluster families/clans.

## Inputs

- PRESTO-STAT-predicted statistical sub-clusters (108,085 sub-clusters from the non-redundant 60,028 BGC antiSMASH-DB dataset)
- Tokenised BGCs: (sub)Pfam domain combinations per gene, with non-biosynthetic Pfams removed
- subPfam pHMM models built from Pfam multiple sequence alignments (multifasta)

## Outputs

- Presence/absence matrix of tokenised genes (domain combinations) per statistical sub-cluster
- 10,000 sub-cluster families (SCFs) from the first K-means clustering
- 2,000 sub-cluster clans (SCCs) from the second K-means clustering
- Relatedness QC metric: fraction of SCCs whose member sub-clusters share at least three genes (expected 81%)

## How to apply

First tokenise each antiSMASH-predicted BGC by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed; subPfams are per-clade pHMMs built by cutting phylogenetic trees of Pfam alignments. Take the PRESTO-STAT-predicted sub-clusters (here 108,085 sub-clusters from 60,028 non-redundant antiSMASH-DB BGCs spanning 10,539 domain combinations) and encode each sub-cluster as a binary row indicating presence/absence of every tokenised gene (domain combination). Apply K-means clustering (scikit-learn) on this presence/absence matrix to obtain 10,000 sub-cluster families (SCFs), then run a second K-means step on the SCF representation to join similar families into 2,000 sub-cluster clans (SCCs). Remove redundant sub-clusters during grouping per the supplementary K-means grouping procedure, ignoring sub-clusters of length one since these are almost definitely noise. Judge success by the relatedness QC metric: the fraction of SCCs whose member sub-clusters share at least three genes, which should reproduce the reported 81%.

## Related tools

- **iPRESTO** (Command-line tool providing PRESTO-STAT sub-cluster prediction and the K-means grouping of sub-clusters into SCFs and SCCs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Source of the predicted BGCs whose genes are tokenised by detecting (sub)Pfams before matrix construction)
- **satriaphd/build_subpfam** (Builds the per-clade subPfam pHMM models used to tokenise genes into domain combinations) — https://github.com/satriaphd/build_subpfam

## Examples

```
from sklearn.cluster import KMeans; scfs = KMeans(n_clusters=10000).fit(presence_absence_matrix); sccs = KMeans(n_clusters=2000).fit(scfs.cluster_centers_)
```

## Evaluation signals

- The first K-means step yields exactly 10,000 sub-cluster families (SCFs).
- The second K-means step yields exactly 2,000 sub-cluster clans (SCCs).
- 81% of SCCs group related sub-clusters sharing at least three genes (the reported relatedness QC metric).
- The presence/absence matrix covers 108,085 sub-clusters over 10,539 domain combinations from the 60,028-BGC non-redundant antiSMASH-DB dataset.
- Sub-clusters of length one are excluded from grouping as noise, and redundant sub-clusters are removed during grouping.

## Limitations

- No version numbers are stated for antiSMASH or iPRESTO in the source material, so exact reproduction may require tool-version tracking.
- Sub-clusters of length one are treated as noise and ignored, which may discard very short real signals.
- The presence/absence representation discards original gene order, so grouping reflects gene-content relatedness only.

## Evidence

- [supplementary] Related statistical sub-clusters are grouped using K-means clustering, with the statistical sub-clusters represented as a presence/absence matrix: [evidence span withheld — anchor status: not_found]
- [supplementary] BGC tokenisation by (sub)Pfams: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] Length-one sub-clusters are noise: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [readme] subPfam construction from Pfam alignments: [evidence span withheld — anchor status: external_doc]
- [intro] iPRESTO availability: "iPRESTO is available as a command-line tool at https://git.wageningenur.nl/bioinformatics/iPRESTO/."
