---
name: bowtie2-index-construction
description: Use when a FASTA of dereplicated representative gene cluster sequences
  (e.g., BiG-MAP.GCF.fna from bigmap-family) must be indexed with Bowtie2 so FASTQ
  shotgun reads can be mapped for MGC/GCF abundance and coverage profiling.
when_to_use_negative:
- Gene clusters have not been redundancy-filtered first — indexing raw antiSMASH/gutSMASH
  output without the MASH (0.8 similarity) or BiG-SCAPE step will cause ambiguous
  mapping of reads across near-identical clusters.
- The goal is functional pathway quantification against reference databases (e.g.,
  KEGG/MetaCyc) rather than gene cluster abundance — BiG-MAP deliberately does not
  depend on such pathway databases.
- Input data are amplicon reads rather than shotgun metagenomic/metatranscriptomic
  reads, which the mapping-to-reference-clusters design does not address.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: BiG-MAP
  role: Pipeline whose bigmap-map module orchestrates index construction, read mapping,
    counting, TPM/RPKM normalization, and coverage computation
  repo: https://github.com/medema-group/BiG-MAP
- name: Bowtie2
  role: Builds indices from representative gene cluster FASTA and aligns FASTQ reads
    in fast mode
- name: samtools
  role: Converts SAM alignments to BAM sorted by leftmost coordinates prior to counting
- name: BiG-SCAPE
  role: Optional second redundancy-filtering step in bigmap-family that determines
    the GCF grouping of the representative sequences to be indexed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:57dbf31350e360bc8f804f2e44357c10df87f7c4ef4eaeb169763fac9b919990
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - Bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# bowtie2-index-construction

## Summary

Constructs Bowtie2 indices from medoid-selected representative metabolic gene cluster (MGC/GCF) nucleotide sequences so metagenomic/metatranscriptomic reads can be mapped to quantify gene cluster abundance (counts, TPM/RPKM) and coverage in BiG-MAP.

## When to use

Use when a FASTA of dereplicated representative gene cluster sequences (e.g., BiG-MAP.GCF.fna from bigmap-family) must be indexed with Bowtie2 so FASTQ shotgun reads can be mapped for MGC/GCF abundance and coverage profiling.

## When NOT to use

- Gene clusters have not been redundancy-filtered first — indexing raw antiSMASH/gutSMASH output without the MASH (0.8 similarity) or BiG-SCAPE step will cause ambiguous mapping of reads across near-identical clusters.
- The goal is functional pathway quantification against reference databases (e.g., KEGG/MetaCyc) rather than gene cluster abundance — BiG-MAP deliberately does not depend on such pathway databases.
- Input data are amplicon reads rather than shotgun metagenomic/metatranscriptomic reads, which the mapping-to-reference-clusters design does not address.

## Inputs

- Representative gene cluster nucleotide sequences from bigmap-family (BiG-MAP.GCF.fna)
- BiG-MAP.GCF.bed (bedfile for extracting core regions during coverage computation)
- Metagenomic/metatranscriptomic shotgun reads in FASTQ format (optionally downloaded via bigmap-download from SRA accessions)

## Outputs

- Bowtie2 index files built from the representative gene cluster sequences
- SAM alignments converted to BAM sorted by leftmost coordinates
- Raw count table with reads summed per representative cluster and per GCF
- TPM and RPKM abundance tables (e.g., gene_cluster_rpkm_table.tsv)
- Per-cluster whole-gene and core-gene coverage files derived from per-base coverage bedgraphs

## How to apply

Take the nucleotide FASTA of representative gene clusters emitted by bigmap-family (BiG-MAP.GCF.fna, produced after MASH redundancy filtering at a default 0.8 similarity threshold and optional BiG-SCAPE GCF grouping) and build a Bowtie2 index from it. Align metagenomic or metatranscriptomic FASTQ reads to this index with Bowtie2 in fast mode, then convert the SAM output to BAM sorted by leftmost coordinates using samtools. Count reads mapping to each representative gene cluster, sum counts per gene cluster family (GCF), convert corrected raw counts to TPM and then RPKM, and compute per-cluster whole-gene and core-gene coverage from bedgraphs as (cl − ncb)/cl. The rationale for indexing only medoid representatives is to avoid ambiguous mapping from redundant clusters (the MinHash/BiG-SCAPE filtering step) while still reporting abundance at both MGC and GCF levels.

## Related tools

- **BiG-MAP** (Pipeline whose bigmap-map module orchestrates index construction, read mapping, counting, TPM/RPKM normalization, and coverage computation) — https://github.com/medema-group/BiG-MAP
- **Bowtie2** (Builds indices from representative gene cluster FASTA and aligns FASTQ reads in fast mode)
- **samtools** (Converts SAM alignments to BAM sorted by leftmost coordinates prior to counting)
- **BiG-SCAPE** (Optional second redundancy-filtering step in bigmap-family that determines the GCF grouping of the representative sequences to be indexed)

## Examples

```
bowtie2-build BiG-MAP.GCF.fna BiG-MAP.GCF && bowtie2 --very-fast -x BiG-MAP.GCF -1 sample_1.fastq -2 sample_2.fastq | samtools view -bS - | samtools sort -o sample.sorted.bam
```

## Evaluation signals

- BiG-MAP.GCF.fna exists and is parseable FASTA, containing one medoid representative per MGC/GCF after redundancy filtering.
- Bowtie2 index files are generated for the representative FASTA, and Bowtie2 reports a non-zero overall alignment rate for each sample.
- The raw count table, gene_cluster_rpkm_table.tsv, and coverage files are all present, parseable tabular files with sample-ID columns.
- Every representative cluster and every GCF appears as a row in the output count/RPKM tables.
- Per-cluster coverage values computed from bedgraphs as (cl − ncb)/cl fall in [0, 1], and TPM values per sample are on a comparable normalized scale.

## Limitations

- BiG-MAP has only been tested on Ubuntu, per the repository README.
- For unpaired ancient read data, an additional tool (filterBAM / bam-filter) must be installed separately to filter the BAM output.
- Mapping is limited to representative (medoid) sequences; abundance estimates depend on the upstream similarity thresholds (default -tg 0.8 for gene clusters, -th 0.1 for housekeeping genes), and reads divergent from all representatives are not captured.

## Evidence

- [intro] Map reads to representative gene clusters: "by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies"
- [readme] Representative FASTA from bigmap-family: [evidence span withheld — anchor status: external_doc]
- [readme] Core-region bedfile: [evidence span withheld — anchor status: external_doc]
- [readme] Redundancy filtering rationale: [evidence span withheld — anchor status: external_doc]
- [readme] Ancient read filtering dependency: [evidence span withheld — anchor status: external_doc]
