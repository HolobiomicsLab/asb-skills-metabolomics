---
name: sam-to-sorted-bam-processing
description: Use when metagenomic/metatranscriptomic FASTQ reads have been aligned
  to representative gene cluster (MGC/GCF) reference sequences with Bowtie2 and the
  resulting SAM output must be converted to leftmost-coordinate-sorted BAM for downstream
  counting and RPKM/coverage computation.
when_to_use_negative:
- Input reads target whole genomes or transcriptomes rather than representative metabolic
  gene cluster sequences — general-purpose RNA-seq/genome quantification pipelines
  are a better fit
- Alignment is against contigs from a metagenomic assembly without prior redundancy
  filtering (MinHash 0.8 similarity cut-off / BiG-SCAPE GCF grouping), since ambiguous
  mapping between redundant clusters would corrupt abundance estimates
- Only differential abundance/expression testing is needed and sorted BAMs already
  exist — use bigmap-analyse instead of re-aligning and re-sorting
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BiG-MAP
  role: bigmap-map module that builds the Bowtie2 indices on representative gene clusters,
    aligns reads, and consumes the sorted BAMs for counting, RPKM, and coverage computation
  repo: github.com/medema-group/BiG-MAP
- name: Bowtie2
  role: aligns metagenomic/metatranscriptomic FASTQ reads to representative MGC/GCF
    sequences in fast mode, producing the SAM input for conversion
- name: samtools
  role: converts SAM alignments to BAM and sorts them by leftmost coordinates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:f070d45464cac5411ea8a01771835471611850112e9f3461353c4be89c1bd460
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - Bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sam-to-sorted-bam-processing

## Summary

Converts Bowtie2 SAM alignments of metagenomic/metatranscriptomic reads to coordinate-sorted BAM files with samtools, as performed in the BiG-MAP.map module so that reads mapping to representative gene cluster (MGC/GCF) sequences can be counted and normalized to RPKM/TPM.

## When to use

Use when metagenomic/metatranscriptomic FASTQ reads have been aligned to representative gene cluster (MGC/GCF) reference sequences with Bowtie2 and the resulting SAM output must be converted to leftmost-coordinate-sorted BAM for downstream counting and RPKM/coverage computation.

## When NOT to use

- Input reads target whole genomes or transcriptomes rather than representative metabolic gene cluster sequences — general-purpose RNA-seq/genome quantification pipelines are a better fit
- Alignment is against contigs from a metagenomic assembly without prior redundancy filtering (MinHash 0.8 similarity cut-off / BiG-SCAPE GCF grouping), since ambiguous mapping between redundant clusters would corrupt abundance estimates
- Only differential abundance/expression testing is needed and sorted BAMs already exist — use bigmap-analyse instead of re-aligning and re-sorting

## Inputs

- SAM alignment files from Bowtie2 (reads aligned to representative MGC/GCF nucleotide sequences)
- FASTQ metagenomic/metatranscriptomic read files (optionally downloaded from SRA by bigmap-download)
- Representative gene cluster reference sequences (BiG-MAP.GCF.fna) and core-region BED file (BiG-MAP.GCF.bed) from bigmap-family

## Outputs

- Coordinate-sorted BAM files per sample
- Downstream raw count tables, TPM/RPKM abundance tables (gene_cluster_rpkm_table.tsv), and per-cluster coverage files derived from the sorted BAMs

## How to apply

In BiG-MAP.map, Bowtie2 indices are first built from medoid-selected representative gene cluster nucleotide sequences (the BiG-MAP.GCF.fna reference produced by bigmap-family), and metagenomic or metatranscriptomic reads are aligned to these representatives in fast mode. The resulting SAM file is then converted to BAM with samtools and sorted by leftmost coordinates so that positional indexing and per-base coverage extraction (bedgraphs) are possible. Sorted BAMs feed the counting step, where aligned reads are summed per gene cluster family (GCF), normalized to TPM and RPKM, and used to compute per-cluster whole-gene and core-gene coverage from the per-base coverage bedgraphs as (cl − ncb)/cl. Correct sorting is a prerequisite: unsorted BAMs would break the bedgraph extraction that underlies the coverage metric.

## Related tools

- **BiG-MAP** (bigmap-map module that builds the Bowtie2 indices on representative gene clusters, aligns reads, and consumes the sorted BAMs for counting, RPKM, and coverage computation) — github.com/medema-group/BiG-MAP
- **Bowtie2** (aligns metagenomic/metatranscriptomic FASTQ reads to representative MGC/GCF sequences in fast mode, producing the SAM input for conversion)
- **samtools** (converts SAM alignments to BAM and sorts them by leftmost coordinates)

## Evaluation signals

- The BAM files are sorted by leftmost coordinates (verifiable with samtools, e.g. index succeeds without error)
- The downstream raw count table, gene_cluster_rpkm_table.tsv, and coverage files all exist and are parseable tabular files with sample-ID columns
- Every representative cluster and GCF appears as a row in the count/RPKM/coverage tables
- Per-cluster coverage values derived from the bedgraphs follow the (cl − ncb)/cl formulation for both whole-gene and core-gene regions
- Read counts summed per GCF remain consistent after normalization to TPM and then RPKM

## Limitations

- BiG-MAP has only been tested on Ubuntu, per the repository README
- Redundant gene clusters must be filtered (MinHash 0.8 similarity by default; optional second BiG-SCAPE round) before mapping, otherwise reads map ambiguously to multiple representatives
- Optionally, for unpaired ancient read data, the additional filterBAM tool must be installed to filter BAMs
- Abundance/expression accuracy was validated only with simulated metagenomic data in the original study

## Evidence

- [intro] Map metagenomic or metatranscriptomic reads to representative gene clusters: [evidence span withheld — anchor status: not_found]
- [intro] Reads mapped to gene cluster sequences: "by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies"
- [readme] bigmap-map assesses abundance and expression: [evidence span withheld — anchor status: external_doc]
- [readme] GCF fasta as mapping reference: [evidence span withheld — anchor status: external_doc]
- [intro] MinHash redundancy filtering rationale: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
