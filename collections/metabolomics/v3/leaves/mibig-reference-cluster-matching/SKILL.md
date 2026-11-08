---
name: mibig-reference-cluster-matching
description: Use when you have predicted (tokenised) sub-clusters or topic motifs
  from a BGC mining method and need to assign known substructure functions by matching
  them against MIBiG reference BGCs and validated SubClusterBlast sub-clusters.
when_to_use_negative:
- When the BGCs have not been tokenised into (sub)Pfam representations or a motif
  library has not yet been trained — matching requires these prerequisites
- When the goal is de novo sub-cluster discovery rather than annotation of already-predicted
  motifs against reference databases
- When the query BGCs are expected to contain sub-clusters absent from MIBiG and SubClusterBlast
  — motifs with no reference occurrence cannot be annotated by this approach
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Provides the PRESTO-TOP topic-modelling motifs and sub-cluster matching cut-offs
    (probability 0.3, summed 1.1, overlap 0.15) used to query MIBiG and antiSMASH-DB
    BGCs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Predicts the BGCs that are tokenised by detecting (sub)Pfams per gene, and
    supplies the 90 validated SubClusterBlast sub-clusters used as annotation reference
- name: build_subpfam
  role: Builds per-clade subPfam pHMM models from aligned protein sequences, refining
    the tokenisation vocabulary used for matching
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:d745b424d96b1639ebf3ffaf00d38633e54f0ca5773d1c8047d93ecf08c04c53
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mibig-reference-cluster-matching

## Summary

Annotates de novo biosynthetic sub-cluster motifs (e.g. PRESTO-TOP topics) by matching predicted sub-cluster instances against experimentally characterised BGCs in the MIBiG reference database and the antiSMASH SubClusterBlast sub-clusters, yielding motif-to-substructure assignments.

## When to use

Use when you have predicted (tokenised) sub-clusters or topic motifs from a BGC mining method and need to assign known substructure functions by matching them against MIBiG reference BGCs and validated SubClusterBlast sub-clusters.

## When NOT to use

- When the BGCs have not been tokenised into (sub)Pfam representations or a motif library has not yet been trained — matching requires these prerequisites
- When the goal is de novo sub-cluster discovery rather than annotation of already-predicted motifs against reference databases
- When the query BGCs are expected to contain sub-clusters absent from MIBiG and SubClusterBlast — motifs with no reference occurrence cannot be annotated by this approach

## Inputs

- Trained PRESTO-TOP sub-cluster motif library (e.g. 1,000 motifs) with probability vectors
- Tokenised MIBiG v1.4 reference dataset (1,819 BGCs) with (sub)Pfam gene annotations
- Set of 90 experimentally validated SubClusterBlast sub-clusters
- Non-redundant antiSMASH-DB dataset (60,028 BGCs) for occurrence counting

## Outputs

- Table of annotated motif rows mapping motif IDs to substructure types (45 motifs / 24 substructure types in the reference study)
- Per-motif occurrence counts across the non-redundant antiSMASH-DB BGC set (range 19 to 873 BGCs; motif 190 to motif 220)
- Coverage statistic: fraction of BGCs containing at least one annotated motif (approximately 9,500 BGCs, i.e. 16%)

## How to apply

Tokenise the BGCs by detecting (sub)Pfams per gene and removing non-biosynthetic Pfams, then query MIBiG (e.g. v1.4, 1,819 BGCs) against the trained motif library using PRESTO-TOP matching cut-offs: gene-to-motif probability 0.3, summed gene-to-motif probability 1.1, and an overlap score threshold of 0.15 with the motif probability vector truncated at cumulative 0.95. Separately, match predicted sub-clusters against the 90 experimentally validated SubClusterBlast sub-clusters, considering an overlap of 0.6 sufficient to declare a prediction. Annotate motifs found in matched characterised sub-clusters with the substructures they encode, and additionally annotate motifs whose instances occur inside known MIBiG BGCs (in the iPRESTO study this yielded 23 + 22 = 45 annotated motifs covering 24 substructure types). Finally, count occurrences of the annotated motifs across the target dataset (non-redundant 60,028 antiSMASH-DB BGCs), ignoring length-1 matches since these are almost certainly noise.

## Related tools

- **iPRESTO** (Provides the PRESTO-TOP topic-modelling motifs and sub-cluster matching cut-offs (probability 0.3, summed 1.1, overlap 0.15) used to query MIBiG and antiSMASH-DB BGCs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Predicts the BGCs that are tokenised by detecting (sub)Pfams per gene, and supplies the 90 validated SubClusterBlast sub-clusters used as annotation reference)
- **build_subpfam** (Builds per-clade subPfam pHMM models from aligned protein sequences, refining the tokenisation vocabulary used for matching) — satriaphd/build_subpfam

## Evaluation signals

- The annotated motif table contains exactly 45 rows spanning 24 distinct substructure types
- Approximately 9,500 (16%) of the non-redundant antiSMASH-DB BGCs contain at least one annotated motif
- Per-motif occurrence counts fall within a plausible range, e.g. 19 BGCs (motif 190) to 873 BGCs (motif 220), with length-1 matches excluded
- The method confirms 83% of experimentally characterised sub-clusters in MIBiG reference BGCs, serving as a recall benchmark
- Matched predicted sub-clusters satisfy the overlap >= 0.6 criterion against validated SubClusterBlast sub-clusters

## Limitations

- Motifs without occurrences in MIBiG BGCs or similarity to the 90 SubClusterBlast sub-clusters remain unannotated, limiting coverage to the 45/1,000 motifs
- Length-1 sub-cluster matches are almost certainly noise and must be ignored; hybrid BGCs are counted for all classes they contain, which complicates per-class statistics
- No version numbers for antiSMASH or iPRESTO are stated in the supplementary material, hindering exact reproducibility

## Evidence

- [intro] 45 annotated motifs from MIBiG occurrences: "we were able to annotate 45 sub-cluster motifs based on occurrences in known BGCs from the MIBiG reference BGC database [16]."
- [abstract] 83% confirmation of characterised sub-clusters: "our method was able to confirm 83% of the experimentally characterised sub-clusters in MIBiG reference BGCs"
- [supplementary] overlap 0.6 validation cut-off: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] length-1 matches ignored: "Matches of length 1 are ignored and hybrid class BGCs are counted for all classes they contain"
- [supplementary] BGC tokenisation via (sub)Pfams: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene"
- [readme] subPfam clade construction method: [evidence span withheld — anchor status: external_doc]
