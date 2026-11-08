---
name: presto-top-motif-matching
description: Use when tokenised BGCs (antiSMASH-predicted clusters with detected (sub)Pfams)
  need unsupervised sub-cluster motif discovery, or when validating predicted sub-clusters
  against curated SubClusterBlast/MIBiG references via overlap fractions.
when_to_use_negative:
- When gene order within the BGC matters for the sub-cluster signal — PRESTO-TOP uses
  a bag-of-words model that discards original gene order; use PRESTO-STAT instead
- When BGCs have not been tokenised (sub)Pfam-annotated — raw GenBank/antiSMASH output
  without Pfam detection cannot be fed to the topic model
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Runs PRESTO-TOP LDA topic modelling and PRESTO-STAT on tokenised BGCs; used
    to predict sub-cluster motifs and query BGCs against them
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Provides the BGC predictions from antiSMASH-DB that are tokenised by (sub)Pfam
    detection, and hosts the SubClusterBlast validated sub-clusters used as reference
- name: SubClusterBlast
  role: Source of the 109 experimentally verified sub-clusters used as validation
    reference for overlap computation
- name: build_subpfam
  role: Builds per-clade pHMM (subPfam) models from aligned protein sequences by cutting
    a mid-rooted phylogenetic tree
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:0cf90950d9906acbb0e93df0696d19b0fea141dd63f07f12b19731851eebe094
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# presto-top-motif-matching

## Summary

Apply iPRESTO's PRESTO-TOP Latent Dirichlet Allocation topic-modelling approach to match sub-cluster motifs (bag-of-(sub)Pfam-words) across BGCs, and validate detected sub-clusters against experimentally verified antiSMASH SubClusterBlast sub-clusters using a 0.6 overlap cut-off.

## When to use

Use when tokenised BGCs (antiSMASH-predicted clusters with detected (sub)Pfams) need unsupervised sub-cluster motif discovery, or when validating predicted sub-clusters against curated SubClusterBlast/MIBiG references via overlap fractions.

## When NOT to use

- When gene order within the BGC matters for the sub-cluster signal — PRESTO-TOP uses a bag-of-words model that discards original gene order; use PRESTO-STAT instead
- When BGCs have not been tokenised (sub)Pfam-annotated — raw GenBank/antiSMASH output without Pfam detection cannot be fed to the topic model

## Inputs

- antiSMASH-predicted BGCs (e.g. non-redundant 60,028 BGC antiSMASH-DB dataset), tokenised by (sub)Pfam content with non-biosynthetic Pfams removed
- subPfam pHMM models built from Pfam multiple sequence alignments (build_subpfam, clade-based pHMMs)
- Set of experimentally verified sub-clusters from the antiSMASH SubClusterBlast module (109 validated sub-clusters) or MIBiG reference BGCs

## Outputs

- PRESTO-TOP sub-cluster motifs (LDA topics over (sub)Pfam tokens) and per-BGC motif matches
- Per-sub-cluster overlap fraction table (validated SubClusterBlast sub-cluster vs predicted sub-cluster) with detection calls at the 0.6 cut-off
- Summary counts of detected validated sub-clusters and overlap with PRESTO-STAT detections

## How to apply

Tokenise antiSMASH-predicted BGCs by detecting (sub)Pfams in each gene, removing non-biosynthetic Pfams; subPfams are built by dividing Pfam multiple sequence alignments into clades and converting each clade into a new pHMM (e.g. for the 112 most frequent Pfam domains in antiSMASH-DB). Run PRESTO-TOP, which assumes a bag-of-words model where original gene order is NOT considered, in contrast to PRESTO-STAT which preserves token order. Optionally query individual BGCs (e.g. rifamycin BGC0000373) against the generated PRESTO-TOP sub-cluster motifs. For validation, compute for each experimentally verified SubClusterBlast sub-cluster the fraction of its genes found in a predicted sub-cluster, and apply the established cut-off of 0.6 overlap to declare it detected; repeat the same computation for PRESTO-STAT and report how many PRESTO-STAT detections are covered by PRESTO-TOP motifs (expected: 78 of 109 detected, 75 shared).

## Related tools

- **iPRESTO** (Runs PRESTO-TOP LDA topic modelling and PRESTO-STAT on tokenised BGCs; used to predict sub-cluster motifs and query BGCs against them) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Provides the BGC predictions from antiSMASH-DB that are tokenised by (sub)Pfam detection, and hosts the SubClusterBlast validated sub-clusters used as reference)
- **SubClusterBlast** (Source of the 109 experimentally verified sub-clusters used as validation reference for overlap computation)
- **build_subpfam** (Builds per-clade pHMM (subPfam) models from aligned protein sequences by cutting a mid-rooted phylogenetic tree) — satriaphd/build_subpfam

## Examples

```
python build_subpfam.py --aln pfam.msa.fasta --cut-length <L>  # then iPRESTO: predict sub-clusters via LDA on tokenised BGCs and query e.g. BGC0000373 (rifamycin) against PRESTO-TOP motifs
```

## Evaluation signals

- Overlap analysis with the 0.6 cut-off reproduces 78 of 109 validated SubClusterBlast sub-clusters detected by PRESTO-STAT
- 75 of those 78 PRESTO-STAT detections overlap with PRESTO-TOP motif matches
- iPRESTO overall confirms 83% of experimentally characterised sub-clusters in MIBiG reference BGCs
- Per-sub-cluster overlap fractions are tabulated as the fraction of genes from the original SubClusterBlast sub-cluster found in a predicted sub-cluster
- Querying a known BGC (e.g. rifamycin BGC0000373) returns the expected sub-cluster motifs in both PRESTO-TOP and PRESTO-STAT outputs

## Limitations

- PRESTO-TOP discards original gene order (bag-of-words assumption), so it cannot capture order-dependent sub-cluster structure that PRESTO-STAT can
- Sub-clusters of length one are almost certainly noise and are not counted as detections
- Matches of length 1 are ignored; hybrid-class BGCs are counted for all classes they contain
- Version numbers for antiSMASH and iPRESTO are not stated in the supplementary section, complicating exact reproduction

## Evidence

- [supplementary] Bag-of-words vs gene-order models: "or by LDA, which assumes a bag of words model where original gene order is not considered"
- [supplementary] 0.6 overlap validation cut-off: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] BGC tokenisation by subPfams: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [abstract] MIBiG validation rate: "our method was able to confirm 83% of the experimentally characterised sub-clusters in MIBiG reference BGCs"
- [supplementary] Length-one noise filter: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
