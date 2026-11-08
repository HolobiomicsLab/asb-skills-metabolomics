---
name: sequence-similarity-network-construction-for-gcfs
description: Use when antiSMASH GenBank BGC predictions must be clustered into gene
  cluster families across many genomes, e.g. to build absence/presence maps of conserved
  vs rare GCFs over a genus-scale genome set.
when_to_use_negative:
- The goal is to resolve evolutionary relationships and order BGCs by a multi-locus
  phylogeny within a GCF — use the CORASON conserved-core workflow instead of plain
  network clustering.
- The input is raw genome assemblies or reads rather than annotated BGCs — antiSMASH
  BGC prediction (and, for drafts, SPAdes assembly) must be run first.
- Single-cluster chemical structure prediction rather than cross-genome family-level
  comparison is needed — antiSMASH/PRISM annotation alone suffices.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: antiSMASH v4
  role: Predicts BGCs with the --minimal parameter; its GenBank outputs are used directly
    as BiG-SCAPE input
  repo: https://antismash.secondarymetabolites.org
- name: BiG-SCAPE
  role: Computes pairwise BGC distances (Jaccard, Domain Sequence Similarity, Adjacency),
    builds and filters the network at cutoff 0.3, and clusters GCFs via Affinity Propagation
  repo: https://git.wur.nl/medema-group/BiG-SCAPE
- name: HMMER 3.1b2
  role: Searches sequences for conserved domains against the user-supplied Pfam v31
    database; hmmer scores drive overlapping-domain filtering
- name: hmmalign
  role: Aligns each predicted domain type with its corresponding model; high agreement
    in GCF calling with Muscle and MAFFT under BiG-SCAPE's --mix parameter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:168b4ee555ff872d70878273a44e7cb6b41bd06661f314ead2786c0fd8f8fa49
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Sequence similarity network construction for GCFs

## Summary

Constructs biosynthetic gene cluster (BGC) sequence similarity networks and groups them into gene cluster families (GCFs) by computing Jaccard, Domain Sequence Similarity and Adjacency pairwise distances over Pfam domain content and filtering at an empirical cutoff of 0.3, as implemented by BiG-SCAPE in the Navarro-Muñoz et al. 2019 framework.

## When to use

Use when antiSMASH GenBank BGC predictions must be clustered into gene cluster families across many genomes, e.g. to build absence/presence maps of conserved vs rare GCFs over a genus-scale genome set.

## When NOT to use

- The goal is to resolve evolutionary relationships and order BGCs by a multi-locus phylogeny within a GCF — use the CORASON conserved-core workflow instead of plain network clustering.
- The input is raw genome assemblies or reads rather than annotated BGCs — antiSMASH BGC prediction (and, for drafts, SPAdes assembly) must be run first.
- Single-cluster chemical structure prediction rather than cross-genome family-level comparison is needed — antiSMASH/PRISM annotation alone suffices.

## Inputs

- antiSMASH GenBank output files (BGC predictions), e.g. produced with antiSMASH v4 --minimal
- User-supplied Pfam database version 31 (HMM profiles searched with HMMER 3.1b2)
- Optional MIBiG reference BGC GenBank records (e.g. release 1.3)

## Outputs

- BGC sequence similarity network with pairwise Jaccard / Domain Sequence Similarity / Adjacency distances
- GCF assignments from Affinity Propagation clustering at cutoff 0.3, plus optional Gene Cluster Clan layer
- Tab-delimited output files and SQLite database with an interactive HTML network visualization
- Genome-by-GCF absence/presence matrix across all analyzed genomes

## How to apply

Take antiSMASH GenBank outputs (here produced with antiSMASH v4 --minimal) directly as input to BiG-SCAPE, optionally together with MIBiG reference BGCs. Detect conserved domains against a user-supplied Pfam database (version 31 searched with HMMER 3.1b2), filtering overlapping domains by hmmer score. Compute three pairwise distances — Jaccard index (shared domain types), Domain Sequence Similarity, and Adjacency — using the 'glocal' alignment mode so complete and fragmented clusters compare correctly. Filter the network to distances ≤ 0.3 and apply Affinity Propagation clustering to assemble GCFs; the Gene Cluster Clan (GCC) setting enabled by default adds a second, coarser clustering layer. Parse the network/clustering output into a genome-by-GCF absence/presence matrix for downstream prevalence analyses.

## Related tools

- **antiSMASH v4** (Predicts BGCs with the --minimal parameter; its GenBank outputs are used directly as BiG-SCAPE input) — https://antismash.secondarymetabolites.org
- **BiG-SCAPE** (Computes pairwise BGC distances (Jaccard, Domain Sequence Similarity, Adjacency), builds and filters the network at cutoff 0.3, and clusters GCFs via Affinity Propagation) — https://git.wur.nl/medema-group/BiG-SCAPE
- **HMMER 3.1b2** (Searches sequences for conserved domains against the user-supplied Pfam v31 database; hmmer scores drive overlapping-domain filtering)
- **hmmalign** (Aligns each predicted domain type with its corresponding model; high agreement in GCF calling with Muscle and MAFFT under BiG-SCAPE's --mix parameter)

## Examples

```
python bigscape.py -i ./antiSMASH_gbk_output --cutoff 0.3 --mix
```

## Evaluation signals

- The genome-by-GCF matrix covers all analyzed genomes (e.g. 105 genomes for the 103 Streptomyces + C. acidiphila CP001700 + S. arenicola CP000850 outgroup case)
- Prevalence binning reproduces the reported pattern: 27 GCFs (~2%) present in >10 genomes and 1,564 GCFs (92%) present in ≤3 genomes
- Network edges retain only pairwise distances ≤ the 0.3 cutoff before Affinity Propagation clustering is applied
- Overlapping domain calls are reconciled by hmmer score before distance calculation
- GCF calling is robust to the domain-alignment method — --mix results agree across hmmalign, Muscle and MAFFT

## Limitations

- The BiG-SCAPE interactive report can undercount genomes: it reported only 96 of 103 Streptomyces genomes because scaffolds from the same genome are grouped by ORGANISM identifier across duplicate assembly projects
- Fragmented/complete BGC similarity is sensitive to the comparison mode; BiG-SCAPE's 'glocal' mode exists specifically because earlier methods failed to compare complete and fragmented gene clusters correctly
- The 0.3 cutoff is empirical and predefined; other cutoff values trade off network connectivity and GCF granularity (a 0.75 cutoff was the value maximizing node count while minimizing structural integrity impact in a related analysis)

## Evidence

- [readme] BiG-SCAPE ... constructs sequence similarity networks of Biosynthetic Gene Clusters (BGCs) and groups them into Gene Cluster Families (GCFs): [evidence span withheld — anchor status: not_found]
- [readme] BiG-SCAPE uses antiSMASH processed GenBank files, i.e. BGC predictions, as well as reference BGC GenBank files (user-defined and/or MIBiG repository): [evidence span withheld — anchor status: not_found]
