---
name: gcf-prevalence-distribution-analysis-across-genomes
description: Use when you have antiSMASH-detected BGCs across many genomes and BiG-SCAPE
  GCF assignments, and you need to quantify which GCFs are conserved vs rare to prioritize
  biosynthetic diversity.
when_to_use_negative:
- When the goal is to resolve evolutionary relationships or sort BGCs phylogenetically
  within a single GCF — use the CORASON multi-locus phylogeny workflow instead of
  prevalence binning.
- When the input is raw sequencing reads or metabolomics data rather than antiSMASH-processed
  GenBank files; BiG-SCAPE requires antiSMASH BGC predictions as input.
- When only a handful of genomes are available, since conserved-vs-rare prevalence
  bins are meaningful only across a large genome panel.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: antiSMASH v4
  role: Predicts biosynthetic gene clusters from the downloaded genomes (--minimal
    parameters), producing the GenBank files consumed downstream
- name: BiG-SCAPE
  role: Computes pairwise BGC distances (Jaccard, Domain Sequence Similarity, Adjacency),
    filters the network at cutoff 0.3, and clusters GCFs via Affinity Propagation
    whose output is parsed into the absence/presence matrix
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: HMMER 3.1b2
  role: Searches sequences against the user-supplied Pfam version 31 database for
    domain detection; hmmer scores drive overlapping-domain filtering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:849509e1145df31162177bda84aa9da5a073e8de3907e02fc6054dc65df089e6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# GCF prevalence distribution analysis across genomes

## Summary

Builds a genome-by-GCF absence/presence matrix from BiG-SCAPE clustering output and bins gene cluster families by how many genomes they occur in, separating a small conserved core (e.g. 27 GCFs across >10 of 103 Streptomyces genomes) from a large rare pool (1,564 GCFs in ≤3 genomes).

## When to use

Use when you have antiSMASH-detected BGCs across many genomes and BiG-SCAPE GCF assignments, and you need to quantify which GCFs are conserved vs rare to prioritize biosynthetic diversity.

## When NOT to use

- When the goal is to resolve evolutionary relationships or sort BGCs phylogenetically within a single GCF — use the CORASON multi-locus phylogeny workflow instead of prevalence binning.
- When the input is raw sequencing reads or metabolomics data rather than antiSMASH-processed GenBank files; BiG-SCAPE requires antiSMASH BGC predictions as input.
- When only a handful of genomes are available, since conserved-vs-rare prevalence bins are meaningful only across a large genome panel.

## Inputs

- antiSMASH v4 GenBank outputs for all analyzed genomes (e.g. 103 complete Streptomyces genomes plus outgroups CP001700 and CP000850)
- User-supplied Pfam database version 31 searched with HMMER 3.1b2
- BiG-SCAPE network/clustering output files (tab-delimited tables and SQLite database)

## Outputs

- Genome-by-GCF absence/presence matrix with per-genome entries for all analyzed genomes
- Counts of conserved GCFs (present in >10 genomes) and rare GCFs (present in ≤3 genomes)
- Prevalence distribution table of GCFs binned by number of genomes occupied

## How to apply

First process all genomes with antiSMASH v4 using the --minimal parameter setting to predict BGCs, then run BiG-SCAPE with a user-supplied Pfam database (version 31 searched with HMMER 3.1b2), applying overlapping-domain filtering by hmmer score and computing Jaccard, Domain Sequence Similarity, and Adjacency pairwise distances. Filter the similarity network at the empirical cutoff distance of 0.3 and apply Affinity Propagation clustering to assemble GCFs. Parse the BiG-SCAPE network/clustering output (tab-delimited files and SQLite database) into a genome-by-GCF absence/presence matrix across all analyzed genomes, including outgroups such as Catenulispora acidiphila (CP001700) and Salinispora arenicola (CP000850). Bin GCFs by genome prevalence — present in >10 genomes (highly conserved) versus ≤3 genomes (rare) — and count families per bin; the reported exemplar yielded 27 conserved (~2%) versus 1,564 rare (92%) GCFs across 103 Streptomyces genomes.

## Related tools

- **antiSMASH v4** (Predicts biosynthetic gene clusters from the downloaded genomes (--minimal parameters), producing the GenBank files consumed downstream)
- **BiG-SCAPE** (Computes pairwise BGC distances (Jaccard, Domain Sequence Similarity, Adjacency), filters the network at cutoff 0.3, and clusters GCFs via Affinity Propagation whose output is parsed into the absence/presence matrix) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **HMMER 3.1b2** (Searches sequences against the user-supplied Pfam version 31 database for domain detection; hmmer scores drive overlapping-domain filtering)

## Evaluation signals

- The absence/presence table exists with per-genome presence/absence entries for all analyzed genomes (e.g. 103 Streptomyces plus 2 outgroup genomes = 105 genomes).
- Prevalence binning reproduces the reported pattern: 27 GCFs (~2%) present across >10 genomes and 1,564 GCFs (92%) present in ≤3 genomes.
- Verify the BiG-SCAPE run used the empirical cutoff distance of 0.3 and Pfam version 31 with HMMER 3.1b2.
- Check for duplicate ORGANISM identifiers — BiG-SCAPE's interactive report may collapse genomes sharing an identifier (only 96 of 103 Streptomyces genomes reported in the exemplar), so the matrix should be built from the clustering files, not the HTML report alone.

## Limitations

- BiG-SCAPE's interactive report may cover fewer genomes than analyzed because scaffolds belonging to the same genome are grouped by ORGANISM identifier (only 96 of 103 Streptomyces genomes reported in the exemplar).
- The 0.3 cutoff and >10/≤3 genome prevalence bins are empirical choices from the exemplar study, not universal thresholds.
- CORASON conserved cores can collapse to only the query gene for divergent BGCs, and empty alignments may require manual parameter reduction — but this applies to downstream phylogenetic steps, not the prevalence binning itself.

## Evidence

- [readme] constructs sequence similarity networks of Biosynthetic Gene Clusters (BGCs) and groups them into Gene Cluster Families (GCFs): [evidence span withheld — anchor status: external_doc]
