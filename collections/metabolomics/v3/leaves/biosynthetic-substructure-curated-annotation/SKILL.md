---
name: biosynthetic-substructure-curated-annotation
description: Use when you have predicted sub-cluster motifs (e.g. PRESTO-TOP LDA topics
  over tokenised BGCs) and a characterised reference set (SubClusterBlast sub-clusters,
  MIBiG) and need to link motifs to chemical substructures for genome-based structure
  prediction.
when_to_use_negative:
- When no experimentally characterised reference sub-clusters (SubClusterBlast or
  MIBiG) are available to anchor motif annotations — the method depends on matching
  to validated sub-clusters
- When the goal is de novo sub-cluster discovery rather than annotation of existing
  motifs — use iPRESTO's PRESTO-STAT/PRESTO-TOP prediction first
- When single-gene (length-1) motif matches must be trusted — these are treated as
  noise and ignored
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: iPRESTO
  role: Provides PRESTO-TOP sub-cluster motif model and query/matching cut-offs used
    to find sub-cluster instances in MIBiG and antiSMASH-DB BGCs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Source of predicted BGCs and the SubClusterBlast characterised sub-clusters
    used as annotation reference
- name: build_subpfam
  role: Builds per-clade subPfam pHMMs used to tokenise BGC genes before motif querying
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:3a2986ab0af00687bc4be52e99b13e99fd7dc47b066c67bf51471384b710e164
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# biosynthetic-substructure-curated-annotation

## Summary

Annotates unsupervised sub-cluster motifs (e.g. the 1,000 PRESTO-TOP motifs from iPRESTO) with the natural product substructures they encode by matching predicted sub-clusters against 90 experimentally validated SubClusterBlast sub-clusters and sub-cluster instances in MIBiG BGCs, yielding a curated motif library usable to predict substructures genome-wide.

## When to use

Use when you have predicted sub-cluster motifs (e.g. PRESTO-TOP LDA topics over tokenised BGCs) and a characterised reference set (SubClusterBlast sub-clusters, MIBiG) and need to link motifs to chemical substructures for genome-based structure prediction.

## When NOT to use

- When no experimentally characterised reference sub-clusters (SubClusterBlast or MIBiG) are available to anchor motif annotations — the method depends on matching to validated sub-clusters
- When the goal is de novo sub-cluster discovery rather than annotation of existing motifs — use iPRESTO's PRESTO-STAT/PRESTO-TOP prediction first
- When single-gene (length-1) motif matches must be trusted — these are treated as noise and ignored

## Inputs

- Trained PRESTO-TOP model with 1,000 sub-cluster motifs
- Tokenised MIBiG v1.4 reference dataset (1,819 BGCs)
- antiSMASH-DB BGC predictions (non-redundant set of 60,028 BGCs) tokenised by (sub)Pfam detection
- 90 experimentally validated antiSMASH SubClusterBlast sub-clusters

## Outputs

- Curated library of 45 annotated sub-cluster motifs spanning 24 substructure types
- Per-motif occurrence counts and degrees in antiSMASH-DB (ranging from 19 BGCs for motif 190 to 873 BGCs for motif 220)
- Per-motif BGC class distributions (hybrid-class BGCs counted for all classes they contain)

## How to apply

Load the trained PRESTO-TOP model with its 1,000 sub-cluster motifs and the tokenised MIBiG v1.4 reference dataset (1,819 BGCs). Query MIBiG and antiSMASH-DB BGCs against the motifs using PRESTO-TOP cut-offs: gene-to-motif probability 0.3, summed gene-to-motif probability 1.1, overlap score threshold 0.15 (motif probability vector truncated at cumulative 0.95). Match predicted sub-clusters to the 90 experimentally validated SubClusterBlast sub-clusters using an overlap of 0.6 as sufficient, and annotate matched motifs with the substructures they encode (23 motifs); then annotate additional motifs (22) whose sub-cluster instances occur inside MIBiG BGCs. Count occurrences of the resulting 45 annotated motifs (24 substructure types) in the non-redundant antiSMASH-DB dataset (60,028 BGCs), ignoring length-1 matches, which should flag roughly 16% of BGCs as carrying at least one annotated substructure.

## Related tools

- **iPRESTO** (Provides PRESTO-TOP sub-cluster motif model and query/matching cut-offs used to find sub-cluster instances in MIBiG and antiSMASH-DB BGCs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Source of predicted BGCs and the SubClusterBlast characterised sub-clusters used as annotation reference)
- **build_subpfam** (Builds per-clade subPfam pHMMs used to tokenise BGC genes before motif querying) — satriaphd/build_subpfam

## Examples

```
python iPRESTO_preprocessing.py --pfams pfam2subpfam.tsv --bgos mibig_v1.4 --model presto_top_1000_motifs --g2m_prob 0.3 --sum_prob 1.1 --overlap 0.15
```

## Evaluation signals

- The annotated motif table contains exactly 45 rows spanning 24 different substructure types (23 from SubClusterBlast matches plus 22 from MIBiG instances)
- Approximately 9,500 of the 60,028 non-redundant antiSMASH-DB BGCs (~16%) contain at least one annotated motif
- Per-motif occurrence counts fall within the reported range of 19 BGCs (motif 190) to 873 BGCs (motif 220)
- Matches respect the PRESTO-TOP cut-offs (0.3 gene-to-motif probability, 1.1 summed probability, 0.15 overlap score, motif vector truncated at cumulative 0.95)
- Sub-cluster validation matches use overlap of 0.6, and length-1 matches are excluded

## Limitations

- Only 16% of antiSMASH-DB BGCs could be assigned a substructure, so most BGCs remain unannotated by this motif library
- Length-1 sub-cluster matches are almost certainly noise and must be ignored
- Hybrid-class BGCs are counted for all classes they contain, so per-class counts overlap

## Evidence

- [results] A total of 45 PRESTO-TOP sub-cluster motifs (23 from SubClusterBlast sub-clusters plus 22 from MIBiG BGC instances) were annotated, constituting 24 different types of substructures: [evidence span withheld — anchor status: not_found]
- [methods] gene-to-motif probability cut-off 0.3, summed gene-to-motif probability cut-off 1.1, and overlap score threshold 0.15 (motif probability vector truncated at cumulative 0.95): [evidence span withheld — anchor status: not_found]
- [supplementary] we considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text): "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [abstract] we could predict a substructure for 16% of the antiSMASH-DB BGCs: "we could predict a substructure for 16% of the antiSMASH-DB BGCs"
- [supplementary] Matches of length 1 are ignored and hybrid class BGCs are counted for all classes they contain: "Matches of length 1 are ignored and hybrid class BGCs are counted for all classes they contain"
