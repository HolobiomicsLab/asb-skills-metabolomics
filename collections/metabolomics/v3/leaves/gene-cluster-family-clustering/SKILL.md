---
name: gene-cluster-family-clustering
description: Use when antiSMASH v4 GenBank BGC predictions (optionally with MIBiG
  reference BGCs) need to be grouped into gene cluster families or clans, or when
  a BGC sequence similarity network must be built and clustered.
when_to_use_negative:
- Input BGCs are already pre-clustered into a GCF presence/absence matrix — downstream
  metabologenomic correlation, not clustering, is needed.
- The task is phylogenetic ordering of BGCs within an established family — CORASON's
  multi-locus syntenic core analysis, not BiG-SCAPE clustering, is the appropriate
  tool.
- BGC predictions are not in antiSMASH-processed GenBank format (e.g. raw FASTA or
  RAST annotations), since BiG-SCAPE expects antiSMASH outputs as direct inputs.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: BiG-SCAPE
  role: Core tool that builds the BGC similarity network, computes the three pairwise
    distance indices, and clusters gene clusters into GCFs/GCCs via Affinity Propagation.
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: antiSMASH v4
  role: Produces the GenBank BGC predictions that are consumed directly as BiG-SCAPE
    inputs (run with --minimal parameters).
- name: HMMER 3.1b2
  role: Searches BGC protein sequences for conserved domains against the user-supplied
    Pfam database; overlapping domains are filtered by hmmer score.
- name: hmmalign
  role: Aligns sequences of every predicted domain type with its corresponding HMM
    model, feeding the Domain Sequence Similarity index.
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:c0feeb12e303494bb195272d4c0dd298733ca6db37fb7185d1c59a9b6cf57604
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# gene-cluster-family-clustering

## Summary

Groups antiSMASH-predicted biosynthetic gene clusters into gene cluster families (GCFs) and clans (GCCs) using BiG-SCAPE's pairwise distance network (Jaccard, Domain Sequence Similarity, Adjacency indices) filtered at a distance cutoff followed by Affinity Propagation clustering.

## When to use

Use when antiSMASH v4 GenBank BGC predictions (optionally with MIBiG reference BGCs) need to be grouped into gene cluster families or clans, or when a BGC sequence similarity network must be built and clustered.

## When NOT to use

- Input BGCs are already pre-clustered into a GCF presence/absence matrix — downstream metabologenomic correlation, not clustering, is needed.
- The task is phylogenetic ordering of BGCs within an established family — CORASON's multi-locus syntenic core analysis, not BiG-SCAPE clustering, is the appropriate tool.
- BGC predictions are not in antiSMASH-processed GenBank format (e.g. raw FASTA or RAST annotations), since BiG-SCAPE expects antiSMASH outputs as direct inputs.

## Inputs

- antiSMASH-predicted BGC GenBank files (e.g. antiSMASH v4 run with --minimal)
- MIBiG reference BGC GenBank files (release 1.3, 1,393 BGCs)
- Pfam database version 31 (HMM file for HMMER)
- HMMER 3.1b2 installation for domain detection

## Outputs

- BGC sequence similarity network (pairwise distance table)
- Gene Cluster Family assignments (17,718 GCFs for 74,652 BGCs in the reference case)
- Gene Cluster Clan assignments (801 GCCs)
- SQLite database of all generated results
- HTML interactive visualization with the BGC similarity network and CORASON-like multi-locus phylogenies per GCF

## How to apply

Run BiG-SCAPE with HMMER 3.1b2 against Pfam version 31, taking antiSMASH-predicted GenBank BGC files (e.g. antiSMASH v4 --minimal outputs) plus MIBiG reference BGCs as input. BiG-SCAPE aligns every predicted domain type with hmmalign and computes three pairwise distance components — the Jaccard index over shared domain types, the Domain Sequence Similarity index, and the Adjacency index — filtering overlapping domains by their hmmer score. Construct the network using only edges at or below the empirical cutoff of 0.3 (glocal mode default) and assign GCFs with the Affinity Propagation algorithm; enable the Gene Cluster Clan layer for a second clustering round at cutoff 0.75, the value that maximized node count while minimizing impact on network structural integrity. Choose global vs glocal mode and the cutoff based on the intended metabologenomic validation: in this study glocal/0.30 recovered at least 6 of 9 known molecule-GCF pairs in the rightmost tail of the correlation distribution. Count the resulting GCFs and GCCs (e.g. 17,718 GCFs and 801 GCCs for 74,652 actinobacterial BGCs plus 1,393 MIBiG v1.3 clusters) and compare against reported values as a verification step.

## Related tools

- **BiG-SCAPE** (Core tool that builds the BGC similarity network, computes the three pairwise distance indices, and clusters gene clusters into GCFs/GCCs via Affinity Propagation.) — https://github.com/medema-group/BiG-SCAPE/wiki
- **antiSMASH v4** (Produces the GenBank BGC predictions that are consumed directly as BiG-SCAPE inputs (run with --minimal parameters).)
- **HMMER 3.1b2** (Searches BGC protein sequences for conserved domains against the user-supplied Pfam database; overlapping domains are filtered by hmmer score.)
- **hmmalign** (Aligns sequences of every predicted domain type with its corresponding HMM model, feeding the Domain Sequence Similarity index.)

## Examples

```
bigscape.py -i antismash_bGCs/ -o output/ -d Pfam-A.seed -c 0.3 --mode glocal --clans 0.75
```

## Evaluation signals

- GCF/GCC counts should match reported benchmarks (17,718 GCFs, 801 GCCs for 74,652 actinobacterial BGCs plus 1,393 MIBiG BGCs).
- Network edges must be restricted to distances ≤ 0.3 (or the chosen cutoff) before Affinity Propagation is applied.
- The Gene Cluster Clan layer should use the 0.75 cutoff, reported as the value that maximizes node count while preserving network structural integrity.
- Metabologenomic validation: with glocal/0.30 at least 6 of the 9 known molecule-GCF pairs should fall in the rightmost tail of the correlation distribution.
- GCF calls made with --mix should show high agreement across hmmalign, Muscle and MAFFT alignment methods, with hmmalign shortest at large input sizes.

## Limitations

- If the input BGCs are not closely related, the CORASON conserved core may collapse to only the query gene.
- Duplicate ORGANISM identifiers across assembly projects cause the interactive report to drop genomes (only 96 of 103 Streptomyces genomes displayed).
- Empty alignments may require manually reducing the score cutoff or the taxonomic scope of the genomic database.
- Alignment method choice (hmmalign vs Muscle vs MAFFT) trades runtime for GCF-calling agreement; hmmalign scales best but agreement was assessed only via --mix comparison.

## Evidence
