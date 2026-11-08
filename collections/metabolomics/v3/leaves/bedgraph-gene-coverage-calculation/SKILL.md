---
name: bedgraph-gene-coverage-calculation
description: Reference for computing per-cluster read coverage from bedgraphs when
  profiling BGC/MGC abundance in metagenomic/metatranscriptomic samples aligned to
  representative gene cluster sequences.
when_to_use_negative:
- When only expression-level summaries are needed and per-base coverage is already
  summarized — RPKM/TPM tables suffice; coverage is a complementary completeness metric
- When reads have not been aligned to representative (medoid) gene cluster sequences
  — no bedgraph of cluster per-base coverage exists yet, so the formula cannot be
  applied
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BiG-MAP
  role: Implements bigmap-map, which converts sorted alignments to bedgraphs and computes
    whole-gene and core-gene coverage per cluster as (cl − ncb)/cl
  repo: github.com/medema-group/BiG-MAP
- name: Bowtie2
  role: Aligns metagenomic/metatranscriptomic FASTQ reads against representative gene
    cluster indices in fast mode, producing the SAM that feeds coverage computation
- name: samtools
  role: Converts SAM to BAM sorted by leftmost coordinates, the alignment state from
    which per-base coverage bedgraphs are derived
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:2fa1c80112d060e01d682f4e5676dacfffe99eba256d1651cc0338430893743a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - Bowtie2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# bedgraph-gene-coverage-calculation

## Summary

Computes per-gene-cluster read coverage (whole-gene and core-gene regions) in BiG-MAP's bigmap-map module by summarizing per-base coverage bedgraphs produced after Bowtie2/samtools alignment, complementing RPKM/TPM abundance tables.

## When to use

Reference for computing per-cluster read coverage from bedgraphs when profiling BGC/MGC abundance in metagenomic/metatranscriptomic samples aligned to representative gene cluster sequences.

## When NOT to use

- When only expression-level summaries are needed and per-base coverage is already summarized — RPKM/TPM tables suffice; coverage is a complementary completeness metric
- When reads have not been aligned to representative (medoid) gene cluster sequences — no bedgraph of cluster per-base coverage exists yet, so the formula cannot be applied

## Inputs

- Sorted BAM files of reads aligned to representative gene cluster sequences (from Bowtie2 + samtools sort)
- Per-base coverage bedgraph files derived from the BAM alignments
- BiG-MAP.GCF.bed core-region bedfile from the bigmap-family module
- BiG-MAP.GCF.fna reference file of representative gene cluster nucleotide sequences

## Outputs

- Per-cluster whole-gene coverage values per sample
- Per-cluster core-gene coverage values per sample
- Coverage files written to the bigmap-map output directory alongside raw count and RPKM/TPM tables (e.g. gene_cluster_rpkm_table.tsv)

## How to apply

Within bigmap-map, reads from metagenomic/metatranscriptomic FASTQ files are aligned with Bowtie2 to medoid-selected representative gene cluster nucleotide sequences (BiG-MAP.GCF.fna from the bigmap-family step), and SAM alignments are converted to coordinate-sorted BAM with samtools. Per-base coverage bedgraphs are then generated from the sorted BAMs, and per-cluster coverage is computed as (cl − ncb)/cl — the number of covered bases relative to the total cluster length — for both whole gene clusters and core regions. Core regions are defined by the BiG-MAP.GCF.bed file produced by bigmap-family, which is used specifically to extract core-gene regions in bigmap-map. The resulting coverage values are written alongside the raw count and RPKM/TPM abundance tables so abundance and coverage can be interpreted together per sample.

## Related tools

- **BiG-MAP** (Implements bigmap-map, which converts sorted alignments to bedgraphs and computes whole-gene and core-gene coverage per cluster as (cl − ncb)/cl) — github.com/medema-group/BiG-MAP
- **Bowtie2** (Aligns metagenomic/metatranscriptomic FASTQ reads against representative gene cluster indices in fast mode, producing the SAM that feeds coverage computation)
- **samtools** (Converts SAM to BAM sorted by leftmost coordinates, the alignment state from which per-base coverage bedgraphs are derived)

## Evaluation signals

- Coverage files exist in the bigmap-map output directory and are parseable tabular files with sample-ID columns
- Every representative cluster and GCF appears as a row in the coverage tables
- Coverage values fall in the valid range [0, 1] since (cl − ncb)/cl is the covered fraction of the cluster length
- Core-gene coverage rows correspond to the regions defined in BiG-MAP.GCF.bed and never exceed the whole-gene coverage for the same cluster and sample
- Coverage table sample columns match the FASTQ sample IDs supplied to the mapping step

## Limitations

- Coverage requires an initial redundancy filtering step (MASH at a 0.8 similarity threshold by default, optionally BiG-SCAPE) to avoid ambiguous multi-mapping to redundant gene clusters
- Core-region coverage depends on the quality of the BiG-MAP.GCF.bed file from bigmap-family; malformed or missing bedfiles will silently misdefine core regions
- BiG-MAP has only been tested on Ubuntu per the repository README

## Evidence

- [readme] BiG-MAP.GCF.bed = Bedfile to extract core regions in bigmap-map: [evidence span withheld — anchor status: external_doc]
- [readme] Reference FASTA used as the mapping target: [evidence span withheld — anchor status: external_doc]
- [intro] Reads are mapped to gene cluster sequences from references or assemblies: "by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies"
