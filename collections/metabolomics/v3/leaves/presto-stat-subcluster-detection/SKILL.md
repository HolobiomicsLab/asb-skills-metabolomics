---
name: presto-stat-subcluster-detection
description: Use when a biosynthetic gene cluster dataset (e.g. tokenised antiSMASH
  BGCs) needs statistical sub-cluster prediction whose performance must be benchmarked
  against experimentally verified SubClusterBlast sub-clusters using an overlap-fraction
  criterion.
when_to_use_negative:
- The goal is topic-model-based sub-cluster discovery rather than validating the statistical
  co-occurrence method — use PRESTO-TOP (LDA, bag-of-words, gene order ignored) instead
  of the order-aware PRESTO-STAT approach.
- The BGCs have not been tokenised into (sub)Pfams with non-biosynthetic Pfams removed,
  so no comparable gene-token representation exists for overlap computation.
- No experimentally verified reference sub-clusters (e.g. SubClusterBlast or MIBiG-annotated
  sub-clusters) are available to compute overlap against.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0797
tools:
- name: iPRESTO
  role: Runs the PRESTO-STAT statistical sub-cluster prediction and PRESTO-TOP topic-modelling
    predictions on tokenised BGCs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Source of predicted BGCs and of the SubClusterBlast module holding the 109
    experimentally verified sub-clusters used as validation ground truth
- name: SubClusterBlast
  role: Provides the experimentally verified sub-clusters against which PRESTO-STAT
    predictions are benchmarked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:38ba3ed9166e8380807d66cda0090353fa32a21a3b1810cfbdeac5a74205c3c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# presto-stat-subcluster-detection

## Summary

Validate PRESTO-STAT statistical sub-cluster predictions from iPRESTO by computing gene-overlap fractions against experimentally verified SubClusterBlast sub-clusters, using a 0.6 overlap cut-off to declare detection. Confirms the statistical co-occurrence method recovers most known sub-clusters and is largely consistent with PRESTO-TOP topic-model motifs.

## When to use

Use when a biosynthetic gene cluster dataset (e.g. tokenised antiSMASH BGCs) needs statistical sub-cluster prediction whose performance must be benchmarked against experimentally verified SubClusterBlast sub-clusters using an overlap-fraction criterion.

## When NOT to use

- The goal is topic-model-based sub-cluster discovery rather than validating the statistical co-occurrence method — use PRESTO-TOP (LDA, bag-of-words, gene order ignored) instead of the order-aware PRESTO-STAT approach.
- The BGCs have not been tokenised into (sub)Pfams with non-biosynthetic Pfams removed, so no comparable gene-token representation exists for overlap computation.
- No experimentally verified reference sub-clusters (e.g. SubClusterBlast or MIBiG-annotated sub-clusters) are available to compute overlap against.

## Inputs

- PRESTO-STAT-predicted sub-clusters generated on the non-redundant antiSMASH-DB dataset (60,028 BGCs)
- Experimentally verified sub-clusters from the antiSMASH SubClusterBlast module (109 validated sub-clusters)
- PRESTO-TOP sub-cluster motif matches (for the consistency comparison)
- Tokenised BGCs with (sub)Pfam domains per gene, non-biosynthetic Pfams removed

## Outputs

- Per-sub-cluster table of overlap fractions between validated SubClusterBlast sub-clusters and predicted PRESTO-STAT sub-clusters
- Summary counts: validated sub-clusters detected by PRESTO-STAT at the 0.6 cut-off (78/109)
- Count of PRESTO-STAT-detected sub-clusters also covered by PRESTO-TOP motifs (75/78)

## How to apply

First tokenise antiSMASH-predicted BGCs by detecting (sub)Pfams in each gene and removing non-biosynthetic Pfams, then run the PRESTO-STAT statistical method, which represents tokenised genes in their original order. Load the experimentally verified sub-clusters stored in the antiSMASH SubClusterBlast module (109 validated sub-clusters in this study). For each validated sub-cluster, compute the overlap as the fraction of its genes found in an iPRESTO-predicted PRESTO-STAT sub-cluster, and apply the established cut-off of 0.6 overlap to declare the validated sub-cluster detected. Repeat the same computation and 0.6 cut-off for PRESTO-TOP sub-cluster motif matches, then determine how many PRESTO-STAT detections are also covered by PRESTO-TOP motifs. Tabulate per-sub-cluster overlap fractions and summary counts: the reference result is 78 of 109 validated sub-clusters detected by PRESTO-STAT, with 75 of those 78 also overlapping PRESTO-TOP motif matches.

## Related tools

- **iPRESTO** (Runs the PRESTO-STAT statistical sub-cluster prediction and PRESTO-TOP topic-modelling predictions on tokenised BGCs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Source of predicted BGCs and of the SubClusterBlast module holding the 109 experimentally verified sub-clusters used as validation ground truth)
- **SubClusterBlast** (Provides the experimentally verified sub-clusters against which PRESTO-STAT predictions are benchmarked)

## Evaluation signals

- Overlap analysis with the 0.6 cut-off reproduces 78 of 109 validated SubClusterBlast sub-clusters detected by PRESTO-STAT.
- 75 of the 78 PRESTO-STAT-detected sub-clusters overlap with PRESTO-TOP motif matches.
- Overlap is computed as the fraction of genes from the original SubClusterBlast sub-cluster found in an iPRESTO-predicted sub-cluster, matching the S1 Text supplementary methods definition.
- Per-sub-cluster overlap fractions are tabulated and each detected sub-cluster meets or exceeds the 0.6 threshold.
- Predictions are generated on the non-redundant set of 60,028 antiSMASH-DB BGCs.

## Limitations

- Overlap of 0.6 is a pragmatic cut-off ('We considered an overlap of 0.6 sufficient for having predicted a sub-cluster'), so borderline sub-clusters below the threshold are counted as undetected even with partial gene overlap.
- Sub-clusters of length one are treated as noise ('not counting sub-clusters of length one as these are almost definitely noise'), limiting detection of single-gene sub-clusters.
- Hybrid-class BGCs complicate counting since matches of length 1 are ignored and hybrid class BGCs are counted for all classes they contain.

## Evidence

- [supplementary] PRESTO-STAT detected 78 of the 109 validated sub-clusters, and 75 of those 78 detections overlap with the sub-cluster motifs matched by PRESTO-TOP: [evidence span withheld — anchor status: not_found]
- [supplementary] Overlap cut-off definition: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] Tokenisation of BGCs: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] PRESTO-STAT uses gene order: "sub-cluster can be predicted with the statistical method (Stat), where the tokenised genes are represented in their original order"
- [supplementary] Non-redundant dataset: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
