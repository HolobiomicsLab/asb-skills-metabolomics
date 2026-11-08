---
name: biosynthetic-gene-cluster-pairwise-distance-computation
description: Use when you have antiSMASH-processed GenBank files (or MIBiG reference
  BGCs) and need pairwise BGC distances to build a similarity network or call GCFs,
  especially across large, mixed complete/fragmented BGC sets.
when_to_use_negative:
- The input is not domain-annotated BGC GenBank data (e.g., raw genome contigs or
  a plain sequence FASTA without antiSMASH BGC prediction) — run BGC prediction and
  domain detection first.
- You need full-length gene/protein sequence phylogeny rather than domain-content-based
  BGC comparison (e.g., single-gene phylogenetics — use CORASON-style multi-locus
  phylogenies or a standard aligner instead).
- Only one BGC is available, so no pairwise comparison or network can be constructed.
edam_operation: http://edamontology.org/operation_0496
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: BiG-SCAPE
  role: 'Core engine: calculates pairwise BGC distances (Jaccard, DSS, Adjacency indices),
    builds the similarity network, and calls GCFs; ''glocal'' mode handles fragmented
    BGCs'
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH v4
  role: Produces the BGC GenBank predictions (--minimal parameters) used directly
    as pairwise-distance inputs
- name: hmmalign
  role: Aligns every predicted domain sequence to its corresponding Pfam profile HMM
    so DSS sequence identity can be scored efficiently (shorter runtimes than Muscle/MAFFT
    as BGC count increases)
- name: HMMER 3.1b2
  role: Domain detection against the user-supplied Pfam database (v31) and scoring
    of overlapping-domain filtering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:3781e08bc1e21568f8de1f011512c33c0d6dfa7ebe1d7372821c49763acf7181
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_0496
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# biosynthetic-gene-cluster-pairwise-distance-computation

## Summary

Compute pairwise distances between biosynthetic gene clusters (BGCs) using BiG-SCAPE's three-component metric — Jaccard domain-type index, Domain Sequence Similarity (DSS) index, and Adjacency index — so that BGCs can be placed in sequence similarity networks and grouped into gene cluster families (GCFs).

## When to use

Use when you have antiSMASH-processed GenBank files (or MIBiG reference BGCs) and need pairwise BGC distances to build a similarity network or call GCFs, especially across large, mixed complete/fragmented BGC sets.

## When NOT to use

- The input is not domain-annotated BGC GenBank data (e.g., raw genome contigs or a plain sequence FASTA without antiSMASH BGC prediction) — run BGC prediction and domain detection first.
- You need full-length gene/protein sequence phylogeny rather than domain-content-based BGC comparison (e.g., single-gene phylogenetics — use CORASON-style multi-locus phylogenies or a standard aligner instead).
- Only one BGC is available, so no pairwise comparison or network can be constructed.

## Inputs

- antiSMASH v4 (--minimal) GenBank output files containing predicted BGCs
- Reference BGC GenBank files (user-defined and/or MIBiG repository)
- User-supplied external Pfam database (version 31 used in the study)
- Predicted domain sequences for every BGC

## Outputs

- Pairwise BGC distance values combining Jaccard, Domain Sequence Similarity, and Adjacency indices
- Sequence similarity network of BGCs with edges at distances <= the chosen cutoff
- GCF assignments (via network filtering plus Affinity Propagation clustering) and optional Gene Cluster Clan second-layer clustering
- Tab-delimited output files, SQLite results database, and HTML visualization

## How to apply

Process input genomes with antiSMASH v4 (--minimal) and feed the GenBank output directly to BiG-SCAPE together with user-supplied or MIBiG reference BGC GenBank files. BiG-SCAPE searches each file's sequences for conserved domains against a user-supplied Pfam database (version 31 with HMMER 3.1b2 in this study), filtering overlapping domains based on the hmmer score. For each BGC pair, the distance combines three values: the percentage of shared domain types (Jaccard index), the DSS index (Munkres/Hungarian one-to-one assignment of same-type domains, scored on copy-number differences and sequence identity of domains aligned via hmmalign to their Pfam profile HMMs), and the Adjacency index capturing gene order. The 'glocal' alignment mode handles similarity between complete and fragmented BGCs, and class-specific distance parameters account for different modes of evolution between BGC classes. Distances at or below a cutoff build the network edges; the study filtered GCF networks at an empirical cutoff of 0.3, and found 0.75 the value maximizing node count while minimizing impact on network structural integrity for the broader network analysis.

## Related tools

- **BiG-SCAPE** (Core engine: calculates pairwise BGC distances (Jaccard, DSS, Adjacency indices), builds the similarity network, and calls GCFs; 'glocal' mode handles fragmented BGCs) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH v4** (Produces the BGC GenBank predictions (--minimal parameters) used directly as pairwise-distance inputs)
- **hmmalign** (Aligns every predicted domain sequence to its corresponding Pfam profile HMM so DSS sequence identity can be scored efficiently (shorter runtimes than Muscle/MAFFT as BGC count increases))
- **HMMER 3.1b2** (Domain detection against the user-supplied Pfam database (v31) and scoring of overlapping-domain filtering)

## Evaluation signals

- Each of the three distance components (Jaccard, DSS, Adjacency) is reported per BGC pair and the combined distance falls in [0, 1]
- DSS reflects both Pfam domain copy-number differences and sequence identity of Munkres-matched domains aligned via hmmalign
- Overlapping domains are removed based on hmmer score before distance computation
- GCF calling using hmmalign-based alignment shows high agreement with Muscle- and MAFFT-based variants (--mix comparison)
- Network edges are restricted to distances <= the cutoff (0.3 for GCF assignment; 0.75 evaluated as maximizing node count while preserving network structural integrity)

## Limitations

- Cutoff choice strongly affects the network: the empirical 0.3 cutoff governs GCF assignment, while 0.75 was chosen to maximize node count while minimizing impact on network structural integrity — different cutoffs yield different families.
- BiG-SCAPE's interactive report grouped genomic scaffolds by ORGANISM identifier, causing only 96 of 103 Streptomyces genomes to be reported when identifiers duplicated across assembly projects.
- hmmalign is the fast profile-based DSS alignment choice; Muscle and MAFFT variants were tested and largely agree, but alignment method affects runtime as BGC count grows.
- Results depend on the Pfam database version supplied (version 31 with HMMER 3.1b2 here).

## Evidence

- [readme] BiG-SCAPE README: distance basis: [evidence span withheld — anchor status: external_doc]
