---
name: pfam-domain-tokenisation
description: Use when BGCs predicted by antiSMASH must be converted into per-gene
  (sub)Pfam domain token lists as input for iPRESTO PRESTO-STAT/PRESTO-TOP sub-cluster
  mining or motif querying.
when_to_use_negative:
- The BGCs are already tokenised into a domain-combination table (skip straight to
  sub-cluster prediction or querying)
- The goal is querying a BGC against an existing annotated motif set rather than producing
  the tokenised input for training/prediction
- The dataset consists of BGCs flagged by antiSMASH as lying on a contig edge, which
  are discarded before tokenisation in this workflow
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0080
tools:
- name: iPRESTO
  role: Command-line tool whose preprocessing pipeline performs the (sub)Pfam tokenisation,
    biosynthetic-domain filtering, and redundancy removal
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: hmmscan
  role: HMMER3 tool used to scan every gene against the custom (sub)Pfam pHMM database
    during tokenisation
- name: antiSMASH
  role: Produces the predicted BGCs whose genes are tokenised; contig-edge BGCs are
    flagged and discarded
- name: build_subpfam
  role: Builds the per-clade subPfam pHMM models used to replace the 112 most frequent
    Pfam domains
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:1a83d4561c2e3262bc339c4ac7faa7f69fb2dc3a39d9c71474b4751c5c65a0d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ECDomainMiner
  - antiSMASH
  - build_subpfam
  - hmmscan
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# pfam-domain-tokenisation

## Summary

Convert antiSMASH-predicted BGCs into tokenised gene representations by detecting (sub)Pfam domains per gene with HMMER3 hmmscan against a custom pHMM database in which the 112 most abundant biosynthetic Pfams are replaced by clade-derived subPfams, then strip non-biosynthetic Pfams. This tokenisation is the prerequisite input for iPRESTO sub-cluster prediction.

## When to use

Use when BGCs predicted by antiSMASH must be converted into per-gene (sub)Pfam domain token lists as input for iPRESTO PRESTO-STAT/PRESTO-TOP sub-cluster mining or motif querying.

## When NOT to use

- The BGCs are already tokenised into a domain-combination table (skip straight to sub-cluster prediction or querying)
- The goal is querying a BGC against an existing annotated motif set rather than producing the tokenised input for training/prediction
- The dataset consists of BGCs flagged by antiSMASH as lying on a contig edge, which are discarded before tokenisation in this workflow

## Inputs

- antiSMASH BGC predictions (GenBank/region identifiers, e.g. QZWF01000007.1.region003)
- Whole-genome assemblies from the public nucleotide archive (e.g. contig QZWF01000007 of Streptomyces sp. F001)
- Custom pHMM database built from Pfam 32.0 with 112 clade-derived subPfams (build_subpfam output)
- List of 1,839 biosynthetic Pfams curated via ECDomainMiner

## Outputs

- Tokenised per-gene (sub)Pfam domain lists for each BGC
- preprocessed_nonredundant_bgcs.tsv (surviving tokenised, non-redundant BGCs)
- Per-region sub-cluster motif match table (e.g. akashin_bgc_subcluster_motifs.tsv) after downstream querying

## How to apply

First run antiSMASH on the genome assembly to obtain predicted BGC regions and discard BGCs flagged as lying on a contig edge. Tokenise every gene with HMMER3 hmmscan against the custom pHMM database built from Pfam 32.0, where the 112 most frequent Pfam domains in antiSMASH-DB have been replaced by clade-derived subPfams constructed with the build_subpfam pipeline (each Pfam multiple sequence alignment is divided into clades and each clade converted into a new pHMM). Retain only the 1,839 biosynthetic Pfams curated via ECDomainMiner, removing all non-biosynthetic domains, then drop Pfams occurring fewer than three times and drop BGCs containing fewer than two non-empty genes. Apply graph-based redundancy filtering (Adjacency Index > 0.95 or full containment) before sub-cluster prediction, keeping from each maximal clique the BGC with the most domains. Emit the surviving tokenised BGCs as a TSV (e.g. preprocessed_nonredundant_bgcs.tsv) for downstream PRESTO-STAT or PRESTO-TOP analysis.

## Related tools

- **iPRESTO** (Command-line tool whose preprocessing pipeline performs the (sub)Pfam tokenisation, biosynthetic-domain filtering, and redundancy removal) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **hmmscan** (HMMER3 tool used to scan every gene against the custom (sub)Pfam pHMM database during tokenisation)
- **antiSMASH** (Produces the predicted BGCs whose genes are tokenised; contig-edge BGCs are flagged and discarded)
- **build_subpfam** (Builds the per-clade subPfam pHMM models used to replace the 112 most frequent Pfam domains) — satriaphd/build_subpfam

## Evaluation signals

- After full preprocessing of antiSMASH-DB v2 (152,122 BGCs) plus MIBiG 1.4 and the Streptomyces/Salinispora dataset, the output must contain exactly 60,028 BGCs and 10,539 distinct domain combinations, matching Table A of S1 Text
- No non-biosynthetic Pfams remain: every retained domain must belong to the 1,839 ECDomainMiner-curated biosynthetic Pfams
- No Pfam occurring fewer than three times and no BGC with fewer than two non-empty genes appears in the tokenised output
- Redundancy filtering removed all pairs with Adjacency Index > 0.95 or full-containment relations within the tokenised set
- For a validation case (e.g. the akashin candidate QZWF01000007.1.region003, StrepF001_25985–StrepF001_26130), the tokenised region must yield matches to sub-cluster motifs 194, 607, and 680 under PRESTO-TOP thresholds (0.3 gene-to-motif probability per gene, 1.1 summed gene-to-motif probability, 0.15 overlap score)

## Limitations

- Only the 112 most frequent Pfam domains in antiSMASH-DB are split into clade-derived subPfams; all other Pfams are used at full-Pfam resolution
- BGCs flagged by antiSMASH as lying on a contig edge are discarded, so partial clusters are excluded from the tokenised dataset
- PRESTO-TOP treats tokenised genes as a bag of words in which original gene order is not considered, and sub-clusters of length one are treated as noise (not counted)
- The build_subpfam README states 'Usage: under construction', so subPfam construction may require manual adaptation

## Evidence

- [supplementary] The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] subPfam construction from Pfam clades: "dividing the multiple sequence alignment of a Pfam into clades and converting each clade into a new pHMM"
- [supplementary] subPfam scope limited to 112 most frequent Pfams: "subPfams are constructed for the 112 most frequent Pfam domains in the antiSMASH-DB"
- [readme] Clade definition method in build_subpfam: [evidence span withheld — anchor status: external_doc]
