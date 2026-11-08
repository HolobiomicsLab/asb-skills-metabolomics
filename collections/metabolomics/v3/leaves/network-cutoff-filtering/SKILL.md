---
name: network-cutoff-filtering
description: Use when a BGC sequence similarity network with pairwise Jaccard/Domain
  Sequence Similarity/Adjacency distances must be filtered at a hard cutoff to define
  GCFs, and optionally a second-layer clan cutoff for GCCs.
when_to_use_negative:
- Input is raw unannotated sequence (no antiSMASH GenBank domain annotations) — BiG-SCAPE
  requires antiSMASH-predicted BGC GenBank files as input
- The goal is evolutionary ordering of clusters within a family rather than network-based
  family/clan assignment — CORASON's multi-locus phylogenies are the appropriate tool
- Distances between complete and fragmented clusters must be handled by an alternative
  scheme — the article notes current methods (including the cutoff approach's scope)
  fail to correctly measure similarity between complete and fragmented gene clusters
  in some settings
edam_operation: http://edamontology.org/operation_3432
tools:
- name: BiG-SCAPE
  role: Constructs the BGC similarity network, filters it at the distance cutoff,
    and assigns GCFs/GCCs via Affinity Propagation
  repo: https://github.com/medema-group/BiG-SCAPE
- name: HMMER 3.1b2
  role: Scores conserved domains against Pfam v31; scores drive overlapping-domain
    filtering before distance calculation
- name: hmmalign
  role: Aligns each predicted domain type against its model to support Domain Sequence
    Similarity distances
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:64721b514381775b86aa93c337337cd7f9c44926b8c58089d141198079bbed56
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/medema-group/BiG-SCAPE
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - hmmalign
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# network-cutoff-filtering

## Summary

Select and apply a distance cutoff to a biosynthetic gene cluster (BGC) similarity network so that edges below the cutoff are retained and Gene Cluster Families (GCFs) — and, in a second layer, Gene Cluster Clans (GCCs) — are assigned with Affinity Propagation.

## When to use

Use when a BGC sequence similarity network with pairwise Jaccard/Domain Sequence Similarity/Adjacency distances must be filtered at a hard cutoff to define GCFs, and optionally a second-layer clan cutoff for GCCs.

## When NOT to use

- Input is raw unannotated sequence (no antiSMASH GenBank domain annotations) — BiG-SCAPE requires antiSMASH-predicted BGC GenBank files as input
- The goal is evolutionary ordering of clusters within a family rather than network-based family/clan assignment — CORASON's multi-locus phylogenies are the appropriate tool
- Distances between complete and fragmented clusters must be handled by an alternative scheme — the article notes current methods (including the cutoff approach's scope) fail to correctly measure similarity between complete and fragmented gene clusters in some settings

## Inputs

- antiSMASH GenBank BGC output files (e.g. antiSMASH v4 --minimal outputs for actinobacterial genomes from Zenodo DOI 10.5281/zenodo.1532752)
- Reference BGC GenBank files from MIBiG (release 1.3, 1,393 clusters)
- Pfam database version 31 for conserved-domain detection with HMMER 3.1b2
- BiG-SCAPE pairwise distance network (raw distance values per BGC pair)

## Outputs

- Filtered similarity network containing only edges with raw distance ≤ 0.3 (GCF layer) and ≤ 0.75 (clan layer)
- Gene Cluster Family assignments of input BGCs (e.g. 17,718 GCFs for 74,652 BGCs)
- Gene Cluster Clan assignments (second-layer clustering of GCFs, e.g. 801 GCCs)
- Verification summary table (e.g. gcf_gcc_counts.tsv)

## How to apply

Compute pairwise distances between antiSMASH GenBank BGCs using BiG-SCAPE's three indices — Jaccard index (shared domain types), Domain Sequence Similarity, and Adjacency — first filtering overlapping domains by the hmmer score and aligning each domain type with hmmalign. Filter the resulting network keeping only edges with distance ≤ the empirical cutoff of 0.3, then run Affinity Propagation clustering on the filtered network to assign GCFs. For the second Gene Cluster Clan layer (enabled by default), re-cluster the GCFs at a looser cutoff of 0.75, the value that maximized node count while minimizing impact on network structural integrity. Count the resulting families and clans and compare against reference values; for the 74,652 antiSMASH v4 (--minimal) actinobacterial BGCs plus 1,393 MIBiG release 1.3 references with Pfam v31/HMMER 3.1b2, the expected result is 17,718 GCFs and 801 GCCs.

## Related tools

- **BiG-SCAPE** (Constructs the BGC similarity network, filters it at the distance cutoff, and assigns GCFs/GCCs via Affinity Propagation) — https://github.com/medema-group/BiG-SCAPE
- **HMMER 3.1b2** (Scores conserved domains against Pfam v31; scores drive overlapping-domain filtering before distance calculation)
- **hmmalign** (Aligns each predicted domain type against its model to support Domain Sequence Similarity distances)

## Examples

```
python bigscape.py -i bgcs/ -o out --pfam-dir pfam/ --mode global --mix
```

## Evaluation signals

- All retained network edges have raw distance ≤ 0.3 for the GCF layer and ≤ 0.75 for the clan layer
- Reproduced GCF/GCC counts match the reference values: 17,718 GCFs and 801 GCCs from 74,652 BGCs
- GCF calling shows high agreement when domain alignment method is swapped between hmmalign, Muscle, and MAFFT
- GCFs are assigned only after Affinity Propagation runs on the cutoff-filtered network, not on the full distance matrix
- Counts written to gcf_gcc_counts.tsv are consistent with the BiG-SCAPE network/cluster output tables

## Limitations

- Cutoffs are empirical: 0.3 is a predefined cutoff, and 0.75 was chosen because it maximized node count while minimizing impact on network structural integrity — both may need revalidation for other BGC sets
- Overlapping-domain filtering depends on the hmmer score, so domain annotation quality directly affects distance values
- The BiG-SCAPE interactive report may omit genomes whose scaffolds share duplicate ORGANISM identifiers (e.g. only 96 of 103 Streptomyces genomes were reported)
- The 0.75 clan cutoff was validated on the 74,652-BGC actinobacterial + MIBiG dataset; other datasets may require recalculation

## Evidence

- [readme] README: BiG-SCAPE purpose: [evidence span withheld — anchor status: external_doc]
