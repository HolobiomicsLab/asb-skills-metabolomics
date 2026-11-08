---
name: rpkm-tpm-abundance-normalization
description: Use when raw read counts per gene cluster family must be converted to
  comparable abundance units (TPM/RPKM) and coverage from shotgun metagenomic or metatranscriptomic
  FASTQ data.
when_to_use_negative:
- The input is already a normalized abundance or feature table rather than raw reads/counts
- The data are amplicon reads rather than shotgun metagenomic/metatranscriptomic FASTQ,
  since mapping to full cluster references is not meaningful
- The goal is differential abundance/expression testing or visualization — that belongs
  to the bigmap-analyse module
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: BiG-MAP
  role: Pipeline whose bigmap-map module performs read counting, TPM/RPKM normalization,
    and coverage computation
  repo: github.com/medema-group/BiG-MAP
- name: Bowtie2
  role: Builds indices on representative cluster sequences and aligns FASTQ reads
    in fast mode
- name: samtools
  role: Converts SAM alignments to BAM sorted by leftmost coordinates before counting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:521d14b7335a9048e70bd35bc3560b971736763e12dbe5138fcfd26455b1eafa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - Bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rpkm-tpm-abundance-normalization

## Summary

Normalizes read counts mapping to representative metabolic gene clusters (MGCs/GCFs) into TPM and RPKM abundance tables, and derives per-cluster read coverage, as implemented in the BiG-MAP.map module for metagenomic/metatranscriptomic profiling.

## When to use

Use when raw read counts per gene cluster family must be converted to comparable abundance units (TPM/RPKM) and coverage from shotgun metagenomic or metatranscriptomic FASTQ data.

## When NOT to use

- The input is already a normalized abundance or feature table rather than raw reads/counts
- The data are amplicon reads rather than shotgun metagenomic/metatranscriptomic FASTQ, since mapping to full cluster references is not meaningful
- The goal is differential abundance/expression testing or visualization — that belongs to the bigmap-analyse module

## Inputs

- Metagenomic/metatranscriptomic shotgun FASTQ reads (optionally downloaded from SRA via bigmap-download)
- Representative gene cluster reference sequences (BiG-MAP.GCF.fna from bigmap-family)
- Core-region BED file (BiG-MAP.GCF.bed) for core-gene coverage
- GCF membership dictionaries (BiG-MAP.GCs.json / BiG-MAP.GCF.json)

## Outputs

- Raw read count table per gene cluster/GCF and sample
- TPM/RPKM abundance table (gene_cluster_rpkm_table.tsv) with sample-ID columns
- Per-cluster whole-gene and core-gene coverage files

## How to apply

Build Bowtie2 indices on the medoid-selected representative gene cluster nucleotide sequences (the BiG-MAP.GCF.fna file produced by bigmap-family) and align the FASTQ reads in fast mode. Convert the SAM output to BAM sorted by leftmost coordinates with samtools, then count aligned reads and sum them per gene cluster family (GCF), or average over GCF size. Convert the corrected raw counts first to TPM and then to RPKM so that abundance is comparable across samples with different library sizes. Compute per-cluster coverage from per-base coverage bedgraphs as (cl − ncb)/cl (covered length minus uncovered gaps over cluster length), computing this both for whole genes and for core genes extracted with the BiG-MAP.GCF.bed file. Write the raw count tables, gene_cluster_rpkm_table.tsv, and coverage files to the module output directory for downstream bigmap-analyse differential testing.

## Related tools

- **BiG-MAP** (Pipeline whose bigmap-map module performs read counting, TPM/RPKM normalization, and coverage computation) — github.com/medema-group/BiG-MAP
- **Bowtie2** (Builds indices on representative cluster sequences and aligns FASTQ reads in fast mode)
- **samtools** (Converts SAM alignments to BAM sorted by leftmost coordinates before counting)

## Evaluation signals

- gene_cluster_rpkm_table.tsv exists and is a parseable tabular file with sample-ID columns
- The raw count table and coverage files both exist and parse as tabular output
- Every representative cluster and GCF appears as a row in the count and RPKM/TPM tables
- Sorted BAM files exist (SAM converted to BAM by leftmost coordinates) for each aligned sample
- Coverage values fall in [0, 1] since they equal (cl − ncb)/cl, the fraction of the cluster length covered

## Limitations

- BiG-MAP has only been tested on Ubuntu
- Redundancy filtering (MASH at a default 0.8 similarity cut-off, optionally followed by BiG-SCAPE) is required upstream to avoid ambiguous mapping of reads to redundant clusters
- Validation of abundance/expression recapitulation was performed with simulated metagenomic data; real-data accuracy depends on cluster reference quality

## Evidence

- [intro] Mapping-based quantification of cluster abundance: "by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies"
- [readme] bigmap-map module role: [evidence span withheld — anchor status: external_doc]
- [readme] Reference file for mapping: [evidence span withheld — anchor status: external_doc]
- [readme] Core region extraction input: [evidence span withheld — anchor status: external_doc]
