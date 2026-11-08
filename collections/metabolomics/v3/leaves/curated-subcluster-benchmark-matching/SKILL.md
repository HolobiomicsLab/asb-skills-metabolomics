---
name: curated-subcluster-benchmark-matching
description: Use when a sub-cluster prediction method must be benchmarked against
  curated/verified sub-clusters (e.g. the 109 SubClusterBlast entries) to quantify
  how many known sub-clusters are recovered and whether two methods agree.
when_to_use_negative:
- When assessing recovery of full BGCs or BGC classes rather than sub-clusters — the
  0.6 overlap rule applies to sub-cluster gene sets, not whole clusters
- When no curated ground-truth sub-cluster set is available; in that case use MIBiG-based
  characterised sub-clusters or qualitative evaluation instead of a fixed overlap
  benchmark
- When sub-clusters of length one are the object of study — these are almost certainly
  noise and are excluded per the paper's filtering rules
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: iPRESTO
  role: Source of the predicted sub-clusters (PRESTO-STAT statistical co-occurrence
    and PRESTO-TOP LDA topic-modelling outputs) being benchmarked
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Provides the SubClusterBlast module holding the 109 experimentally verified
    sub-clusters used as the benchmark ground truth
- name: SubClusterBlast
  role: Curated benchmark of experimentally verified sub-clusters whose genes are
    compared to predicted sub-clusters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:e0704fd7694c8b90e10eea408e71b06be643a624f545d5e190c46a79332ce251
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# curated-subcluster-benchmark-matching

## Summary

Validates computationally predicted biosynthetic sub-clusters (e.g. from PRESTO-STAT or PRESTO-TOP) against experimentally verified sub-clusters curated in the antiSMASH SubClusterBlast module by computing per-sub-cluster gene-overlap fractions and applying a 0.6 detection cut-off.

## When to use

Use when a sub-cluster prediction method must be benchmarked against curated/verified sub-clusters (e.g. the 109 SubClusterBlast entries) to quantify how many known sub-clusters are recovered and whether two methods agree.

## When NOT to use

- When assessing recovery of full BGCs or BGC classes rather than sub-clusters — the 0.6 overlap rule applies to sub-cluster gene sets, not whole clusters
- When no curated ground-truth sub-cluster set is available; in that case use MIBiG-based characterised sub-clusters or qualitative evaluation instead of a fixed overlap benchmark
- When sub-clusters of length one are the object of study — these are almost certainly noise and are excluded per the paper's filtering rules

## Inputs

- Set of predicted sub-clusters (e.g. PRESTO-STAT sub-clusters from iPRESTO run on the non-redundant 60,028-BGC antiSMASH-DB dataset)
- Set of predicted sub-cluster motif matches from a second method (e.g. PRESTO-TOP LDA topic-modelling motifs)
- Curated set of experimentally verified sub-clusters from the antiSMASH SubClusterBlast module (109 validated sub-clusters)

## Outputs

- Per-sub-cluster table of overlap fractions (validated vs predicted sub-clusters) for each method
- Summary counts of detected validated sub-clusters per method (expected: 78 of 109 for PRESTO-STAT)
- Cross-method overlap count of detections shared between PRESTO-STAT and PRESTO-TOP motifs (expected: 75 of the 78 PRESTO-STAT detections)

## How to apply

Load the predicted sub-clusters (e.g. PRESTO-STAT output generated on the non-redundant antiSMASH-DB dataset of 60,028 BGCs) alongside the set of 109 experimentally verified sub-clusters stored in the antiSMASH SubClusterBlast module. For each validated sub-cluster, compute the overlap as the fraction of genes from the original SubClusterBlast sub-cluster that are found in an iPRESTO-predicted sub-cluster. Declare the validated sub-cluster detected when the overlap fraction is at least 0.6, the cut-off established in the paper's supplementary methods (S1 Text). Repeat the identical overlap computation and 0.6 cut-off for a second method such as PRESTO-TOP sub-cluster motif matches, then cross-tabulate how many detections are shared between the two methods. The rationale is that a fixed fractional-overlap threshold provides a reproducible, per-sub-cluster detection criterion that supports both absolute recovery counts and inter-method consistency checks.

## Related tools

- **iPRESTO** (Source of the predicted sub-clusters (PRESTO-STAT statistical co-occurrence and PRESTO-TOP LDA topic-modelling outputs) being benchmarked) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Provides the SubClusterBlast module holding the 109 experimentally verified sub-clusters used as the benchmark ground truth)
- **SubClusterBlast** (Curated benchmark of experimentally verified sub-clusters whose genes are compared to predicted sub-clusters)

## Evaluation signals

- Each validated sub-cluster is counted as detected only when the fraction of its original SubClusterBlast genes found in a predicted sub-cluster is ≥ 0.6
- The PRESTO-STAT run should reproduce 78 of 109 validated sub-clusters detected
- The cross-method comparison should reproduce 75 of those 78 PRESTO-STAT detections also covered by PRESTO-TOP motif matches
- Per-sub-cluster overlap fractions are tabulated so individual benchmark entries can be inspected, not just aggregate counts
- Sub-clusters of length one are not counted, as they are treated as noise

## Limitations

- The 0.6 overlap cut-off is a fixed heuristic from the supplementary methods (S1 Text); sub-clusters detected at slightly lower overlap are counted as missed
- The benchmark depends on the completeness of the antiSMASH SubClusterBlast curated set (109 entries) and cannot validate sub-clusters for which no experimental evidence exists
- No version numbers for antiSMASH or iPRESTO are stated in the supplementary section, so exact reproduction may require tool-version resolution

## Evidence

- [supplementary] Overlap cut-off definition: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] Benchmark source and scale: "Overlap between predicted SubClusterBlast sub-clusters and output of both sub-cluster prediction methods"
- [supplementary] Dataset used for prediction: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [intro] Complementary prediction approaches being cross-compared: "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
- [supplementary] Noise exclusion rule: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
