---
name: bgc-redundancy-network-filtering
description: Use when a large tokenised BGC set (e.g. antiSMASH output with per-gene
  (sub)Pfam tokens) must be de-duplicated before motif/sub-cluster mining, and near-identical
  clusters (Adjacency Index > 0.95 or full containment) should be collapsed.
when_to_use_negative:
- The input BGCs are not yet tokenised into (sub)Pfam domains — tokenisation and non-biosynthetic
  Pfam removal must precede network construction
- The goal is novelty-oriented comparison against MIBiG rather than de-replication
  of a mining corpus, since removing near-identical BGCs discards redundant but potentially
  informative members
- BGCs flagged by antiSMASH as lying on a contig-edge are present, as their truncated
  domain content would distort Adjacency Index and containment edges
tools:
- name: iPRESTO
  role: Implements the preprocessing pipeline including graph-based redundancy filtering
    (maximal-clique pruning by AI > 0.95 or containment) and writes preprocessed_nonredundant_bgcs.tsv
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: hmmscan
  role: HMMER3 tool that tokenises each gene against the custom subPfam-augmented
    pHMM database, producing the domain tokens used to compute Adjacency Index edges
- name: build_subpfam
  role: Builds the clade-derived subPfam pHMMs (cutting an ultrametricized phylogenetic
    tree of each Pfam alignment) that replace the 112 most abundant biosynthetic Pfams
    before tokenisation
  repo: satriaphd/build_subpfam
- name: ECDomainMiner
  role: Source of the 1,839 curated biosynthetic Pfams retained during tokenisation
    filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:4fb5043f2020bd7aa196d1f1cdbfabb9c47060c19c53cb7c57246f1c93507398
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# BGC redundancy network filtering

## Summary

Dereplicates a tokenised biosynthetic gene cluster (BGC) dataset by building a BGC similarity network and pruning maximal cliques, keeping only the most domain-rich BGC per clique. Applied in iPRESTO to reduce antiSMASH-DB v2 (152,122 BGCs) to 60,028 non-redundant BGCs.

## When to use

Use when a large tokenised BGC set (e.g. antiSMASH output with per-gene (sub)Pfam tokens) must be de-duplicated before motif/sub-cluster mining, and near-identical clusters (Adjacency Index > 0.95 or full containment) should be collapsed.

## When NOT to use

- The input BGCs are not yet tokenised into (sub)Pfam domains — tokenisation and non-biosynthetic Pfam removal must precede network construction
- The goal is novelty-oriented comparison against MIBiG rather than de-replication of a mining corpus, since removing near-identical BGCs discards redundant but potentially informative members
- BGCs flagged by antiSMASH as lying on a contig-edge are present, as their truncated domain content would distort Adjacency Index and containment edges

## Inputs

- antiSMASH-DB v2 download (asdb_20180828_all_results.tar.xz; 152,122 BGCs from antiSMASH 4.0), MIBiG 1.4 (1,819 BGCs), or a Streptomyces/Salinispora dataset (5,927 BGCs), with contig-edge BGCs discarded
- Per-gene (sub)Pfam tokenisations from hmmscan against the subPfam-augmented Pfam 32.0 pHMM library
- Biosynthetic Pfam list (1,839 Pfams from ECDomainMiner)

## Outputs

- preprocessed_nonredundant_bgcs.tsv — tokenised, non-redundant BGCs (60,028 BGCs for antiSMASH-DB v2)
- Set of 10,539 distinct subPfam domain combinations (Table A of S1 Text)

## How to apply

First tokenise every BGC gene by scanning proteins with hmmscan against a custom pHMM database built from Pfam 32.0 in which the 112 most abundant biosynthetic Pfams are replaced by clade-derived subPfams (build_subpfam pipeline), retaining only the 1,839 biosynthetic Pfams curated via ECDomainMiner, dropping Pfams occurring fewer than three times and BGCs with fewer than two non-empty genes, and discarding contig-edge BGCs flagged by antiSMASH. Then build a BGC similarity network in which an edge connects two BGCs when their domain Adjacency Index (AI) exceeds 0.95 or when one BGC is fully contained within another. Decompose the network into maximal cliques and keep, from each clique, only the BGC with the most domains. Emit the surviving clusters as preprocessed_nonredundant_bgcs.tsv; the rationale is that clique members with AI > 0.95 encode essentially the same domain combinations, so keeping the largest representative preserves domain-combination diversity for sub-cluster mining.

## Related tools

- **iPRESTO** (Implements the preprocessing pipeline including graph-based redundancy filtering (maximal-clique pruning by AI > 0.95 or containment) and writes preprocessed_nonredundant_bgcs.tsv) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **hmmscan** (HMMER3 tool that tokenises each gene against the custom subPfam-augmented pHMM database, producing the domain tokens used to compute Adjacency Index edges)
- **build_subpfam** (Builds the clade-derived subPfam pHMMs (cutting an ultrametricized phylogenetic tree of each Pfam alignment) that replace the 112 most abundant biosynthetic Pfams before tokenisation) — satriaphd/build_subpfam
- **ECDomainMiner** (Source of the 1,839 curated biosynthetic Pfams retained during tokenisation filtering)

## Evaluation signals

- Count invariant: the processed antiSMASH-DB v2 dataset must contain exactly 60,028 BGCs, matching Table A in S1 Text
- Domain-combination invariant: exactly 10,539 distinct domain combinations remain after filtering
- No pair of surviving BGCs should share an Adjacency Index edge above 0.95 or a full-containment relation within the output
- Every surviving BGC must contain at least two non-empty genes and only domains from the 1,839 biosynthetic Pfams occurring at least three times
- Output file preprocessed_nonredundant_bgcs.tsv must be present and parsable for downstream PRESTO-STAT/PRESTO-TOP sub-cluster prediction

## Limitations

- The Adjacency Index 0.95 cut-off and 'keep the BGC with the most domains' clique rule are dataset-tuned choices; other corpora may need different thresholds
- BGCs flagged by antiSMASH as lying on a contig-edge are discarded, so partial clusters at assembly breaks are excluded from the non-redundant set
- Build_subpfam's README notes usage documentation is 'under construction', so reproducing the subPfam library may require inspecting the code directly

## Evidence

- [supplementary] an edge represents an adjacency index (AI) of 0.95 or higher: "an edge represents an adjacency index (AI) of 0.95 or higher"
- [supplementary] Non-redundant dataset size: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [supplementary] Tokenisation precondition: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [readme] SubPfam construction basis: [evidence span withheld — anchor status: external_doc]
