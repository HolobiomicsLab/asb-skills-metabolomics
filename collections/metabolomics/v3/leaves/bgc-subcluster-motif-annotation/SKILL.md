---
name: bgc-subcluster-motif-annotation
description: Use when a trained PRESTO-TOP model with latent sub-cluster motifs exists
  and the goal is to assign substructure-level annotations (e.g. aminoglycoside, moenomycin
  motifs) by matching motif instances to validated sub-clusters in MIBiG and antiSMASH
  SubClusterBlast data.
when_to_use_negative:
- When the goal is de novo sub-cluster discovery rather than annotation of an existing
  trained PRESTO-TOP motif library
- When BGCs have not been tokenised into (sub)Pfam words — PRESTO-TOP requires tokenised
  gene/Pfam representations, not raw nucleotide or protein sequences
- When no experimentally characterised sub-clusters (SubClusterBlast or MIBiG) are
  available to ground the motif annotations
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0797
tools:
- name: iPRESTO
  role: Generates the PRESTO-TOP topic-model sub-cluster motifs being annotated and
    provides the matching/querying method with the probability and overlap cut-offs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: SubClusterBlast
  role: Source of the 90 experimentally validated sub-clusters used as ground truth
    for annotating motif substructures at the 0.6 overlap threshold
- name: antiSMASH
  role: Predicts the BGCs whose (sub)Pfam tokenisations are queried against the motifs;
    also supplies the SubClusterBlast characterised sub-clusters
- name: MIBiG
  role: Reference database of 1,819 experimentally characterised BGCs providing sub-cluster
    instances that annotate 22 additional motifs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:a5f5ac5bb52908e16f5ea3d198090920c591e86bb2b4effa50148838356e11d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG
  - SubClusterBlast
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# bgc-subcluster-motif-annotation

## Summary

Annotate PRESTO-TOP sub-cluster motifs with the substructures they encode by matching predicted sub-clusters against experimentally characterised sub-clusters from SubClusterBlast and sub-cluster instances in MIBiG BGCs, yielding a reusable motif-to-substructure library.

## When to use

Use when a trained PRESTO-TOP model with latent sub-cluster motifs exists and the goal is to assign substructure-level annotations (e.g. aminoglycoside, moenomycin motifs) by matching motif instances to validated sub-clusters in MIBiG and antiSMASH SubClusterBlast data.

## When NOT to use

- When the goal is de novo sub-cluster discovery rather than annotation of an existing trained PRESTO-TOP motif library
- When BGCs have not been tokenised into (sub)Pfam words — PRESTO-TOP requires tokenised gene/Pfam representations, not raw nucleotide or protein sequences
- When no experimentally characterised sub-clusters (SubClusterBlast or MIBiG) are available to ground the motif annotations

## Inputs

- Trained PRESTO-TOP model containing 1,000 sub-cluster motifs
- Tokenised MIBiG v1.4 reference dataset (1,819 BGCs tokenised by (sub)Pfam detection)
- antiSMASH-DB BGC dataset (non-redundant set of 60,028 BGCs)
- 90 experimentally validated antiSMASH SubClusterBlast sub-clusters

## Outputs

- Table of 45 annotated sub-cluster motif rows spanning 24 substructure types
- Per-motif occurrence counts and BGC class distributions in the non-redundant antiSMASH-DB dataset

## How to apply

Load the trained PRESTO-TOP model (1,000 sub-cluster motifs) and the tokenised MIBiG v1.4 reference dataset (1,819 BGCs), where BGCs were tokenised by detecting (sub)Pfams in each gene with non-biosynthetic Pfams removed. Query MIBiG and antiSMASH-DB BGCs against the motifs using the PRESTO-TOP cut-offs: gene-to-motif probability 0.3, summed gene-to-motif probability 1.1, and overlap score threshold 0.15 (motif probability vector truncated at cumulative 0.95). Match predicted sub-clusters to the 90 experimentally validated SubClusterBlast sub-clusters using an overlap of 0.6 as sufficient, and annotate each matched motif with the substructures encoded by the characterised sub-cluster (23 motifs). Then annotate an additional 22 motifs for which sub-cluster instances occur inside MIBiG BGCs, yielding 45 annotated motifs covering 24 substructure types. Finally, count occurrences of the 45 annotated motifs in the non-redundant antiSMASH-DB dataset (60,028 BGCs), ignoring length-1 matches as noise and counting hybrid-class BGCs for all classes they contain.

## Related tools

- **iPRESTO** (Generates the PRESTO-TOP topic-model sub-cluster motifs being annotated and provides the matching/querying method with the probability and overlap cut-offs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **SubClusterBlast** (Source of the 90 experimentally validated sub-clusters used as ground truth for annotating motif substructures at the 0.6 overlap threshold)
- **antiSMASH** (Predicts the BGCs whose (sub)Pfam tokenisations are queried against the motifs; also supplies the SubClusterBlast characterised sub-clusters)
- **MIBiG** (Reference database of 1,819 experimentally characterised BGCs providing sub-cluster instances that annotate 22 additional motifs)

## Examples

```
python iPRESTO/query.py --model presto_top_model.pkl --tokenized_mibig mibig_v1.4_tokens.json --gene_prob 0.3 --sum_prob 1.1 --overlap 0.15
```

## Evaluation signals

- The annotated motif table contains exactly 45 rows spanning 24 different substructure types (23 from SubClusterBlast matches plus 22 from MIBiG instances)
- Around 9,500 (16%) of the 60,028 non-redundant antiSMASH-DB BGCs contain at least one annotated motif
- Per-motif occurrence counts fall within the reported range of 19 BGCs (motif 190) to 873 BGCs (motif 220)
- All motif matches respect the PRESTO-TOP cut-offs: gene-to-motif probability 0.3, summed probability 1.1, overlap score 0.15, motif vector truncated at cumulative 0.95
- Length-1 motif matches are excluded from occurrence counting, and hybrid-class BGCs are counted for all classes they contain

## Limitations

- Length-1 sub-clusters/matches are almost definitely noise and must be ignored, but this also means very short genuine substructures may be missed
- Sub-cluster match validation relies on an overlap of 0.6, so partial or fragmented matches to characterised sub-clusters may be rejected or misassigned
- Annotation coverage is limited by what is characterised: only 45 of 1,000 motifs (16% of antiSMASH-DB BGCs) could be assigned substructures, so most motifs remain unannotated

## Evidence

- [results] A total of 45 PRESTO-TOP sub-cluster motifs (23 from SubClusterBlast sub-clusters plus 22 from MIBiG BGC instances) were annotated, constituting 24 different types of substructures at different levels of detail.: [evidence span withheld — anchor status: not_found]
- [methods] gene-to-motif probability cut-off 0.3, summed gene-to-motif probability cut-off 1.1, and overlap score threshold 0.15: [evidence span withheld — anchor status: not_found]
- [supplementary] We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text): "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] Matches of length 1 are ignored and hybrid class BGCs are counted for all classes they contain: "Matches of length 1 are ignored and hybrid class BGCs are counted for all classes they contain"
- [supplementary] The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
