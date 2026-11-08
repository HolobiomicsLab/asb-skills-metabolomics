---
name: subcluster-overlap-calculation
description: Use when validating predicted biosynthetic sub-clusters against a curated
  set of experimentally verified sub-clusters by quantifying per-sub-cluster gene
  overlap fractions against a defined cut-off.
when_to_use_negative:
- When the goal is de novo sub-cluster discovery or tokenisation of BGCs rather than
  benchmarking predicted sub-clusters against known ones
- When comparing sub-clusters by gene order or adjacency rather than gene-set membership
  — the overlap metric treats sub-clusters as sets of genes and ignores the adjacency-index
  graph filtering used elsewhere in iPRESTO
edam_operation: http://edamontology.org/operation_3763
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Source of the predicted PRESTO-STAT and PRESTO-TOP sub-clusters being validated
    by the overlap calculation
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Provides the SubClusterBlast module storing the 109 experimentally verified
    sub-clusters used as the validation reference set
- name: SubClusterBlast
  role: Reference database of experimentally verified sub-clusters whose gene content
    is compared to predicted sub-clusters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:37d868090c84aeda619b16683e787a09864ae04b99cfafa9a3312c6d3f4d995a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_3763
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# subcluster-overlap-calculation

## Summary

Computes the fraction of genes from an experimentally verified sub-cluster (e.g. from the antiSMASH SubClusterBlast module) that are found in a predicted sub-cluster (PRESTO-STAT or PRESTO-TOP), using a 0.6 overlap cut-off to declare a validated sub-cluster as detected.

## When to use

Use when validating predicted biosynthetic sub-clusters against a curated set of experimentally verified sub-clusters by quantifying per-sub-cluster gene overlap fractions against a defined cut-off.

## When NOT to use

- When the goal is de novo sub-cluster discovery or tokenisation of BGCs rather than benchmarking predicted sub-clusters against known ones
- When comparing sub-clusters by gene order or adjacency rather than gene-set membership — the overlap metric treats sub-clusters as sets of genes and ignores the adjacency-index graph filtering used elsewhere in iPRESTO

## Inputs

- Set of predicted sub-clusters from iPRESTO PRESTO-STAT (tokenised (sub)Pfam gene sets from the non-redundant antiSMASH-DB 60,028 BGC dataset)
- Set of predicted sub-cluster motif matches from iPRESTO PRESTO-TOP (LDA topic-modelling output)
- Set of experimentally verified sub-clusters from the antiSMASH SubClusterBlast module (109 validated sub-clusters)

## Outputs

- Per-sub-cluster table of overlap fractions (fraction of validated SubClusterBlast sub-cluster genes found in a predicted sub-cluster)
- Summary counts of validated sub-clusters detected by PRESTO-STAT at the 0.6 cut-off
- Summary counts of PRESTO-STAT-detected sub-clusters also covered by PRESTO-TOP motif matches

## How to apply

Load the predicted sub-clusters (e.g. PRESTO-STAT output generated on the non-redundant 60,028 BGC antiSMASH-DB dataset) together with the set of 109 experimentally verified sub-clusters stored in the antiSMASH SubClusterBlast module. For each validated SubClusterBlast sub-cluster, compute the overlap as the fraction of genes from the original SubClusterBlast sub-cluster found in an iPRESTO-predicted sub-cluster. Apply the established cut-off of 0.6 overlap to declare the validated sub-cluster as detected. Optionally repeat the identical computation for the PRESTO-TOP sub-cluster motif matches to assess consistency between the statistical (PRESTO-STAT) and topic-modelling (PRESTO-TOP) approaches, and tabulate per-sub-cluster overlap fractions plus summary counts.

## Related tools

- **iPRESTO** (Source of the predicted PRESTO-STAT and PRESTO-TOP sub-clusters being validated by the overlap calculation) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Provides the SubClusterBlast module storing the 109 experimentally verified sub-clusters used as the validation reference set)
- **SubClusterBlast** (Reference database of experimentally verified sub-clusters whose gene content is compared to predicted sub-clusters)

## Evaluation signals

- At the 0.6 overlap cut-off, PRESTO-STAT should detect 78 of the 109 validated SubClusterBlast sub-clusters
- Of those 78 PRESTO-STAT detections, 75 should also overlap with PRESTO-TOP sub-cluster motif matches
- Every per-sub-cluster overlap fraction falls in [0, 1], since it is a fraction of validated sub-cluster genes found in a predicted sub-cluster
- Each validated sub-cluster is evaluated against the full prediction set — no validated sub-cluster is dropped before the overlap computation
- Tabulated per-sub-cluster overlap fractions must be recomputable and consistent with the summary counts reported (78 detected, 75 shared)

## Limitations

- Sub-clusters of length one are treated as almost certainly noise, so single-gene sub-clusters are excluded from consideration
- The 0.6 cut-off is a heuristic threshold; sub-clusters with overlap just below 0.6 are counted as not detected even if partially recovered
- Overlap is defined as the fraction of validated-sub-cluster genes recovered, not symmetric — large predicted sub-clusters can absorb a validated sub-cluster without penalty

## Evidence

- [supplementary] Overlap between predicted SubClusterBlast sub-clusters and output of both sub-cluster prediction methods: "Overlap between predicted SubClusterBlast sub-clusters and output of both sub-cluster prediction methods"
- [supplementary] 0.6 overlap cut-off: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] non-redundant antiSMASH-DB dataset: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [supplementary] length-one sub-clusters are noise: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [intro] two complementary approaches: "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
