---
name: biosynthetic-gene-cluster-similarity-network-analysis
description: Use when antiSMASH-predicted BGC GenBank files (plus optional MIBiG reference
  BGCs) need to be organized into sequence similarity networks and GCFs/GCCs to map
  biosynthetic diversity across large genome sets.
when_to_use_negative:
- Input BGCs are not antiSMASH-processed GenBank files; BiG-SCAPE relies on antiSMASH
  annotations (e.g. 'Core Biosynthetic' gene markers) for adjacency and LCS rules
- Analysis goal is the phylogenetic resolution within one GCF rather than global GCF/GCC
  assignment across many BGCs; use CORASON instead for multi-locus phylogenies within
  a GCF
- A prior normalized or harmonized GCF/GCC clustering already exists and only counts
  are needed — recomputing the network would duplicate the step
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3174
tools:
- name: BiG-SCAPE
  role: Constructs BGC sequence similarity networks and assigns GCFs/GCCs via Affinity
    Propagation at the 0.3 distance cutoff
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH
  role: Produces the GenBank BGC predictions consumed directly as BiG-SCAPE inputs
  repo: https://antismash.secondarymetabolites.org
- name: HMMER 3.1b2
  role: Domain detection against the user-supplied Pfam database; provides the hmmer
    scores used to filter overlapping domains
- name: hmmalign
  role: Aligns sequences of every predicted domain type with the corresponding Pfam
    model for Domain Sequence Similarity calculations
- name: MIBiG
  role: Source of reference BGC GenBank files (release 1.3 in this study) included
    in the network alongside predicted BGCs
  repo: https://mibig.secondarymetabolites.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:59f52e2c77f92e508b4cbc15a942faae02b8222ae25f515ebfa5a58dc4f832b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - MIBiG
  - antiSMASH
  - hmmalign
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# biosynthetic-gene-cluster-similarity-network-analysis

## Summary

Constructs and clusters sequence similarity networks of biosynthetic gene clusters (BGCs) with BiG-SCAPE, computing pairwise distances from domain content, order, copy number and sequence identity, then grouping clusters into gene cluster families (GCFs) and gene cluster clans (GCCs).

## When to use

Use when antiSMASH-predicted BGC GenBank files (plus optional MIBiG reference BGCs) need to be organized into sequence similarity networks and GCFs/GCCs to map biosynthetic diversity across large genome sets.

## When NOT to use

- Input BGCs are not antiSMASH-processed GenBank files; BiG-SCAPE relies on antiSMASH annotations (e.g. 'Core Biosynthetic' gene markers) for adjacency and LCS rules
- Analysis goal is the phylogenetic resolution within one GCF rather than global GCF/GCC assignment across many BGCs; use CORASON instead for multi-locus phylogenies within a GCF
- A prior normalized or harmonized GCF/GCC clustering already exists and only counts are needed — recomputing the network would duplicate the step

## Inputs

- antiSMASH-processed GenBank files of predicted BGCs (e.g. antiSMASH v4 --minimal outputs)
- Reference BGC GenBank files (user-defined and/or MIBiG repository, e.g. MIBiG release 1.3)
- User-supplied external Pfam database (e.g. Pfam version 31 with HMMER 3.1b2)

## Outputs

- Gene cluster family (GCF) assignments from Affinity Propagation clustering at distance cutoff 0.3
- Gene cluster clan (GCC) assignments from the second-layer clustering on GCFs
- BGC sequence similarity network
- Tab-delimited output files with pairwise distance results

## How to apply

Take antiSMASH GenBank outputs (e.g. antiSMASH v4 run with --minimal) and optionally MIBiG reference BGCs as inputs. Run BiG-SCAPE with a user-supplied Pfam database (version 31 with HMMER 3.1b2 in this study); each domain type's sequences are aligned with hmmalign, and overlapping domains are filtered by the hmmer score. BiG-SCAPE computes pairwise distances as three indices — Jaccard (shared domain types), Domain Sequence Similarity, and Adjacency — with the longest common segment required to be either three genes long or contain at least one gene marked as 'Core Biosynthetic'. Build the network from all distances at or below the default empirical cutoff of 0.3, assign GCFs with Affinity Propagation clustering, and enable the Gene Cluster Clan (GCC) mode for a second clustering layer on the GCFs; in this study, a GCC cutoff of 0.75 maximized node count while minimizing impact on network structural integrity. Apply the --mix option to combine distance types and compare GCF calling across alternative alignment methods (hmmalign, Muscle, MAFFT), where high agreement and shorter runtimes with hmmalign as input BGC count increases justify its default use.

## Related tools

- **BiG-SCAPE** (Constructs BGC sequence similarity networks and assigns GCFs/GCCs via Affinity Propagation at the 0.3 distance cutoff) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH** (Produces the GenBank BGC predictions consumed directly as BiG-SCAPE inputs) — https://antismash.secondarymetabolites.org
- **HMMER 3.1b2** (Domain detection against the user-supplied Pfam database; provides the hmmer scores used to filter overlapping domains)
- **hmmalign** (Aligns sequences of every predicted domain type with the corresponding Pfam model for Domain Sequence Similarity calculations)
- **MIBiG** (Source of reference BGC GenBank files (release 1.3 in this study) included in the network alongside predicted BGCs) — https://mibig.secondarymetabolites.org

## Examples

```
python bigscape.py -i bgc_gbk/ -o output --mix -c 1 --family_cutoff 0.3 --clans_cutoff 0.75
```

## Evaluation signals

- Total GCF and GCC counts reproduce expected numbers when run on a known benchmark; in this study the 74,652 BGCs (3,080 actinobacterial genomes plus 1,393 MIBiG release 1.3 references) yielded 17,718 GCFs and 801 GCCs
- Pairwise distances satisfy the internal BiG-SCAPE rule: only distances ≤ 0.3 enter the network, and LCS must be three genes long or contain a 'Core Biosynthetic' gene
- Overlapping domains are consistently filtered by hmmer score, and using --mix with hmmalign yields GCF calls in high agreement with Muscle and MAFFT alternatives
- The GCC second-layer clustering uses a cutoff of 0.75, which maximizes node count while minimizing impact on network structural integrity
- AntiSMASH domain annotation coverage is complete for every BGC in the input set before distance calculation

## Limitations

- CORASON/EvoMining analyses in this study used a reduced unpublished set of 1,668 actinobacterial draft genomes, so BiG-SCAPE's global GCF/GCC counts are not directly transferable to that subset
- CORASON conserved core can collapse to only the query gene if BGCs are not closely related, and if the alignment is empty it is recommended to reduce the hmmer score cutoff or the scope of taxonomic diversity
- BiG-SCAPE's interactive report groups genomic scaffolds by ORGANISM identifier, so only 96 of 103 Streptomyces genomes appeared when duplicate ORGANISM identifiers existed across assembly projects
- Alignment methods differ in runtime; Muscle and MAFFT scale poorly compared with hmmalign as the number of input BGCs increases
- GC content and BGC fragmentation (as in metagenome-assembled genomes) can affect distance calculation; BiG-SCAPE accounts for fragmented BGCs via the glocal alignment mode, but very short or heavily fragmented clusters may still distort network structure

## Evidence

- [readme] BiG-SCAPE is a software package, written in Python, that constructs sequence similarity networks of Biosynthetic Gene Clusters (BGCs) and groups them into Gene Cluster Families (GCFs).: [evidence span withheld — anchor status: not_found]
- [readme] BiG-SCAPE uses antiSMASH processed GenBank files, i.e. BGC predictions, as well as reference BGC GenBank files (user-defined and/or MIBiG repository).: [evidence span withheld — anchor status: not_found]
