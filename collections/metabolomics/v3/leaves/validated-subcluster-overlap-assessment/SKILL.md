---
name: validated-subcluster-overlap-assessment
description: Use when predicted biosynthetic sub-clusters or sub-cluster motifs must
  be validated/annotated against experimentally characterised sub-clusters (SubClusterBlast,
  MIBiG) and quantified genome-wide.
when_to_use_negative:
- When sub-clusters of length one are expected to be biologically meaningful — the
  article treats these as 'almost definitely noise' and excludes them
- When characterised reference sub-clusters (SubClusterBlast / MIBiG) are unavailable,
  since the overlap-based validation and substructure annotation depend entirely on
  them
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: iPRESTO
  role: Generates the PRESTO-TOP sub-cluster motifs and applies the matching cut-offs
    (0.3 / 1.1 / 0.15) used for validation against characterised sub-clusters
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Provides SubClusterBlast characterised sub-clusters and the antiSMASH-DB BGC
    corpus counted for per-motif coverage
- name: build_subpfam
  role: Builds the per-clade subPfam pHMMs used to tokenise genes prior to sub-cluster
    prediction and validation
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:1404b266951a2d7cf5874d630464895ed9333084434ece0b94d5de69bd4e42ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Validated sub-cluster overlap assessment

## Summary

Anchors unsupervised PRESTO-TOP sub-cluster predictions to experimentally validated sub-clusters by requiring a 0.6 overlap with SubClusterBlast sub-clusters, then annotates motifs with substructures via MIBiG instances — yielding 45 annotated motifs across 24 substructure types and 83% confirmation of known sub-clusters.

## When to use

Use when predicted biosynthetic sub-clusters or sub-cluster motifs must be validated/annotated against experimentally characterised sub-clusters (SubClusterBlast, MIBiG) and quantified genome-wide.

## When NOT to use

- When sub-clusters of length one are expected to be biologically meaningful — the article treats these as 'almost definitely noise' and excludes them
- When characterised reference sub-clusters (SubClusterBlast / MIBiG) are unavailable, since the overlap-based validation and substructure annotation depend entirely on them

## Inputs

- Trained PRESTO-TOP model with 1,000 sub-cluster motifs
- Tokenised MIBiG v1.4 reference dataset (1,819 BGCs)
- 90 experimentally validated antiSMASH SubClusterBlast sub-clusters
- Non-redundant antiSMASH-DB dataset (60,028 BGCs)

## Outputs

- Table of 45 annotated sub-cluster motif rows spanning 24 substructure types
- Per-motif occurrence counts and BGC class distributions across antiSMASH-DB
- Fraction of BGCs with at least one annotated motif (≈9,500, i.e. 16%)
- Validation overlap score between predicted and characterised sub-clusters (83% of MIBiG sub-clusters confirmed)

## How to apply

First query tokenised BGCs (MIBiG v1.4, 1,819 BGCs) against the trained PRESTO-TOP motif library (1,000 motifs) using the iPRESTO matching cut-offs: gene-to-motif probability ≥ 0.3, summed gene-to-motif probability ≥ 1.1, overlap score ≥ 0.15, with motif probability vectors truncated at cumulative 0.95. Then match predicted sub-clusters against the 90 experimentally validated SubClusterBlast sub-clusters, treating an overlap of 0.6 as sufficient for a predicted sub-cluster, and assign the substructures encoded by matched characterised sub-clusters to the motifs. Annotate additional motifs whose sub-cluster instances occur inside MIBiG BGCs (22 motifs in this case, totalling 45 motifs / 24 substructure types). Finally, count occurrences of the annotated motifs in the non-redundant antiSMASH-DB dataset (60,028 BGCs), ignoring length-1 matches as noise and counting hybrid BGCs for all classes they contain.

## Related tools

- **iPRESTO** (Generates the PRESTO-TOP sub-cluster motifs and applies the matching cut-offs (0.3 / 1.1 / 0.15) used for validation against characterised sub-clusters) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Provides SubClusterBlast characterised sub-clusters and the antiSMASH-DB BGC corpus counted for per-motif coverage)
- **build_subpfam** (Builds the per-clade subPfam pHMMs used to tokenise genes prior to sub-cluster prediction and validation) — satriaphd/build_subpfam

## Evaluation signals

- The annotation table contains exactly 45 annotated motif rows spanning 24 substructure types (23 from SubClusterBlast sub-clusters plus 22 from MIBiG BGC instances)
- Around 9,500 BGCs (16%) of the non-redundant antiSMASH-DB dataset contain at least one annotated motif
- Per-motif occurrence in antiSMASH-DB falls between 19 BGCs (motif 190) and 873 BGCs (motif 220)
- 83% of experimentally characterised sub-clusters in MIBiG reference BGCs are confirmed by the predicted sub-clusters
- Length-1 matches are excluded from the occurrence counts

## Limitations

- Matches of length 1 are ignored because they are considered noise; sub-clusters relying on single-gene signals cannot be validated this way
- Hybrid BGCs are counted for all classes they contain, which inflates per-class counts for hybrids
- Only a subset of motifs can be annotated — 45 of 1,000 PRESTO-TOP motifs — so most motifs remain unvalidated
- The 0.6 overlap threshold for 'sufficient' sub-cluster prediction is a declared convention (Supplementary methods, S1 Text), not a derived statistic

## Evidence

- [supplementary] Overlap threshold for validated sub-clusters: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] Length-1 matches excluded: "Matches of length 1 are ignored and hybrid class BGCs are counted for all classes they contain"
- [intro] 45 motifs annotated via MIBiG: "we were able to annotate 45 sub-cluster motifs based on occurrences in known BGCs from the MIBiG reference BGC database [16]."
- [abstract] 83% validation rate: "our method was able to confirm 83% of the experimentally characterised sub-clusters in MIBiG reference BGCs"
