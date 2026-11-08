---
name: biosynthetic-gene-domain-tokenisation
description: Use when raw antiSMASH BGC annotations must be converted into per-gene
  protein-family tokens before statistical or topic-model sub-cluster mining, and
  clade-level subPfam resolution is needed to differentiate related domains.
when_to_use_negative:
- The BGCs are already tokenised into (sub)Pfam gene content — re-tokenisation would
  be redundant
- The goal is linking MS/MS-derived Mass2Motifs to existing sub-clusters rather than
  preparing gene content for sub-cluster prediction
- Input consists of single genes or non-biosynthetic loci rather than antiSMASH-predicted
  BGCs, since tokenisation is defined per BGC gene
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: antiSMASH
  role: Predicts the BGCs whose genes are subsequently tokenised by detecting (sub)Pfams
- name: iPRESTO
  role: Consumes the tokenised BGCs for sub-cluster prediction via PRESTO-STAT and
    PRESTO-TOP (LDA)
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: build_subpfam
  role: Builds per-clade subPfam pHMM models from aligned protein sequences used as
    finer-grained tokens
  repo: satriaphd/build_subpfam
- name: Pfam
  role: Source protein-family definitions and multiple sequence alignments from which
    subPfams are derived
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:3aa481ab273f65ae46c2c97e95c8452d3838dcbf413e33c9fd007a7e74b05159
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pfam
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# biosynthetic-gene-domain-tokenisation

## Summary

Converts antiSMASH-predicted biosynthetic gene clusters (BGCs) into token sequences of (sub)Pfam protein-family domains — including clade-resolved subPfams built with build_subpfam — after removing non-biosynthetic Pfams, so that sub-cluster prediction methods (PRESTO-STAT, PRESTO-TOP/LDA) can operate on gene content as words.

## When to use

Use when raw antiSMASH BGC annotations must be converted into per-gene protein-family tokens before statistical or topic-model sub-cluster mining, and clade-level subPfam resolution is needed to differentiate related domains.

## When NOT to use

- The BGCs are already tokenised into (sub)Pfam gene content — re-tokenisation would be redundant
- The goal is linking MS/MS-derived Mass2Motifs to existing sub-clusters rather than preparing gene content for sub-cluster prediction
- Input consists of single genes or non-biosynthetic loci rather than antiSMASH-predicted BGCs, since tokenisation is defined per BGC gene

## Inputs

- antiSMASH-predicted BGC annotations (e.g. from the non-redundant antiSMASH-DB dataset of 60,028 prokaryotic BGCs)
- Pfam multiple sequence alignments for frequent domains (FASTA/multifasta of aligned protein sequences)
- subPfam pHMM definitions built per clade (112 most frequent Pfam domains in antiSMASH-DB)

## Outputs

- Per-gene (sub)Pfam token sequences for each BGC (non-biosynthetic Pfams removed)
- Tokenised BGC corpus suitable for PRESTO-STAT (order-aware) and PRESTO-TOP/LDA (bag-of-words) sub-cluster prediction

## How to apply

First predict BGCs with antiSMASH (e.g. on the antiSMASH-DB non-redundant set of 60,028 BGCs) and detect (sub)Pfams in each gene of each cluster, explicitly removing non-biosynthetic Pfams so they do not pollute the token vocabulary. Where finer resolution is required, construct subPfams for the most frequent Pfam domains in the dataset (the paper built them for the 112 most frequent Pfam domains in antiSMASH-DB) using build_subpfam: divide the multiple sequence alignment of a Pfam into clades by cutting a mid-rooted, laderized, ultrametricized phylogenetic tree of the aligned sequences at a specified length point, and convert each clade into a new pHMM. Tokenise every gene by the (sub)Pfams it contains, then supply the tokenised BGCs to iPRESTO in two modes: the statistical method (PRESTO-STAT), which uses tokens in their original gene order, or LDA topic modelling (PRESTO-TOP), which assumes a bag of words where original gene order is not considered. Sub-clusters of length one are treated as noise and discarded since they are almost definitely spurious.

## Related tools

- **antiSMASH** (Predicts the BGCs whose genes are subsequently tokenised by detecting (sub)Pfams)
- **iPRESTO** (Consumes the tokenised BGCs for sub-cluster prediction via PRESTO-STAT and PRESTO-TOP (LDA)) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **build_subpfam** (Builds per-clade subPfam pHMM models from aligned protein sequences used as finer-grained tokens) — satriaphd/build_subpfam
- **Pfam** (Source protein-family definitions and multiple sequence alignments from which subPfams are derived)

## Evaluation signals

- Every token in the corpus corresponds to a biosynthetic (sub)Pfam — non-biosynthetic Pfams have been filtered out of all gene tokens
- subPfam models exist only for the most frequent Pfam domains in the dataset (paper: the 112 most frequent in antiSMASH-DB), each built as a clade-specific pHMM
- Tokenised BGCs remain ordered for PRESTO-STAT runs and bag-of-words for PRESTO-TOP runs, matching each method's model assumption
- No predicted sub-cluster has length one; single-token sub-clusters are discarded as noise
- Downstream validation against SubClusterBlast sub-clusters uses the established 0.6 overlap cut-off, reproducing 78 of 109 validated sub-clusters detected by PRESTO-STAT

## Limitations

- build_subpfam's README states 'Usage under construction', so clade-cutting parameters must be configured manually
- Non-biosynthetic Pfams must be reliably removed; retaining them would corrupt the token vocabulary used by both sub-cluster prediction methods
- Bag-of-words (PRESTO-TOP) discards gene order, so sub-clusters whose signal depends on co-localisation adjacency may only be recovered by the order-aware statistical method

## Evidence

- [supplementary] Tokenisation output feeds both prediction modes: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] SubPfam construction scope: "subPfams are constructed for the 112 most frequent Pfam domains in the antiSMASH-DB"
- [supplementary] subPfam method: "dividing the multiple sequence alignment of a Pfam into clades and converting each clade into a new pHMM"
- [readme] Clade definition in build_subpfam: [evidence span withheld — anchor status: external_doc]
- [supplementary] Order-awareness decision point: "or by LDA, which assumes a bag of words model where original gene order is not considered"
