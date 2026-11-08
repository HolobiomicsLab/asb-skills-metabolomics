---
name: adjacency-index-maximal-clique-deduplication
description: Use when a large tokenised BGC dataset (e.g. antiSMASH-DB domain-tokenised
  clusters) contains near-duplicate clusters that must be reduced to a non-redundant
  set before motif/sub-cluster mining.
when_to_use_negative:
- The input set is already curated as non-redundant (e.g. a small reference set like
  MIBiG alone), so clique-based collapsing would remove intentionally similar reference
  clusters.
- BGCs lying on a contig edge must be retained — antiSMASH flags these as incomplete
  and they are discarded before this deduplication step, so results would be biased.
- Similarity should be judged on full gene order or class-level membership rather
  than domain composition; this skill explicitly ignores order and counts hybrid-class
  matches per class.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Provides the preprocessing pipeline that builds the similarity network, applies
    the AI > 0.95 / containment edges, and performs maximal-clique reduction before
    sub-cluster mining
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: hmmscan
  role: HMMER3 tool used to tokenise every gene against the subPfam-substituted pHMM
    database, producing the domain token sequences on which the Adjacency Index is
    computed
- name: antiSMASH
  role: Source of the BGC predictions (antiSMASH 4.0) and contig-edge flags that define
    the deduplication input
- name: build_subpfam
  role: Builds the clade-derived subPfam pHMMs used to tokenise BGCs prior to redundancy
    filtering
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:ddd977b03f3c6592e8dbcbf1d7d5fc95bd259aab3b469fa8e8c729dde4003160
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ECDomainMiner
  - antiSMASH
  - build_subpfam
  - hmmscan
  - iPRESTO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Adjacency-index maximal-clique BGC deduplication

## Summary

Graph-based redundancy removal for tokenised biosynthetic gene clusters: BGCs are connected when their domain Adjacency Index exceeds 0.95 or when one BGC fully contains another, and only the member with the most domains is kept from each maximal clique. This collapses near-identical clusters before downstream sub-cluster mining with iPRESTO.

## When to use

Use when a large tokenised BGC dataset (e.g. antiSMASH-DB domain-tokenised clusters) contains near-duplicate clusters that must be reduced to a non-redundant set before motif/sub-cluster mining.

## When NOT to use

- The input set is already curated as non-redundant (e.g. a small reference set like MIBiG alone), so clique-based collapsing would remove intentionally similar reference clusters.
- BGCs lying on a contig edge must be retained — antiSMASH flags these as incomplete and they are discarded before this deduplication step, so results would be biased.
- Similarity should be judged on full gene order or class-level membership rather than domain composition; this skill explicitly ignores order and counts hybrid-class matches per class.

## Inputs

- antiSMASH-DB v2 download (asdb_20180828_all_results.tar.xz; 152,122 BGCs detected with antiSMASH 4.0), excluding contig-edge BGCs
- MIBiG 1.4 (1,819 BGCs) and Streptomyces/Salinispora dataset (5,927 BGCs)
- Tokenised BGCs restricted to the 1,839 biosynthetic Pfams (subPfam-substituted), Pfams occurring fewer than three times removed, and BGCs with fewer than two non-empty genes removed

## Outputs

- preprocessed_nonredundant_bgcs.tsv: surviving tokenised, non-redundant BGCs
- Non-redundant antiSMASH-DB set of 60,028 BGCs spanning 10,539 distinct domain combinations (Table A, S1 Text)

## How to apply

After tokenising each gene with hmmscan against the custom pHMM database (Pfam 32.0 with the 112 most abundant biosynthetic Pfams replaced by clade-derived subPfams) and filtering to the 1,839 ECDomainMiner-curated biosynthetic Pfams, build a BGC similarity network in which an edge joins two BGCs when their domain Adjacency Index (AI) is 0.95 or higher, or when one BGC is fully contained within another. Enumerate the maximal cliques of this network and, from each clique, retain only the BGC carrying the most domains, discarding the rest as redundant. Emit the surviving tokenised clusters as preprocessed_nonredundant_bgcs.tsv. The rationale is that near-identical domain combinations add no information for statistical (PRESTO-STAT) or LDA-based (PRESTO-TOP) sub-cluster detection but inflate co-occurrence counts, so collapsing them stabilises motif discovery.

## Related tools

- **iPRESTO** (Provides the preprocessing pipeline that builds the similarity network, applies the AI > 0.95 / containment edges, and performs maximal-clique reduction before sub-cluster mining) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **hmmscan** (HMMER3 tool used to tokenise every gene against the subPfam-substituted pHMM database, producing the domain token sequences on which the Adjacency Index is computed)
- **antiSMASH** (Source of the BGC predictions (antiSMASH 4.0) and contig-edge flags that define the deduplication input)
- **build_subpfam** (Builds the clade-derived subPfam pHMMs used to tokenise BGCs prior to redundancy filtering) — satriaphd/build_subpfam

## Evaluation signals

- The processed antiSMASH-DB dataset must contain exactly 60,028 BGCs and 10,539 distinct domain combinations, matching Table A in S1 Text.
- Every retained edge in the similarity network must satisfy Adjacency Index ≥ 0.95 or full containment of one BGC within another.
- Each maximal clique in the network must contribute exactly one surviving BGC, and it must be the clique member with the most domains.
- No surviving BGC set should contain a pair whose domain token sequences would yield AI > 0.95 (spot-check recompute AI on random pairs).
- Output file preprocessed_nonredundant_bgcs.tsv must exclude any BGC flagged by antiSMASH as lying on a contig edge.

## Limitations

- Contig-edge BGCs are discarded upstream, so clusters truncated by assembly boundaries never enter the network and cannot be rescued.
- The AI threshold of 0.95 and the 'keep the BGC with the most domains' rule are fixed heuristics; they may discard a shorter but biologically distinct cluster fully contained in a larger one.
- Pfams occurring fewer than three times and BGCs with fewer than two non-empty genes are removed before deduplication, so rare domains cannot influence the network.
- The build_subpfam README states 'Usage under construction', so reproducing the subPfam basis for tokenisation may require contacting the authors.

## Evidence

- [supplementary] adjacency index edge definition: "an edge represents an adjacency index (AI) of 0.95 or higher"
- [supplementary] clique-based survivor selection: [evidence span withheld — anchor status: not_found]
- [supplementary] resulting non-redundant dataset size: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [intro] motivation for redundancy filtering at scale: "we used the antiSMASH-DB, which is one of the largest collections of BGCs that currently exists, and which has been scrutinized for underlying genome assembly quality"
