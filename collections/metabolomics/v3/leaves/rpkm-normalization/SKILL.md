---
name: rpkm-normalization
description: Use when WGS/metatranscriptomic reads have been mapped to MGC/GCF reference
  sequences (BiG-MAP.GCF.fna) and per-sample cluster abundance must be normalized
  as RPKM before phenotype-level differential analysis.
when_to_use_negative:
- Input is still raw FASTQ that has not been mapped to gene cluster reference sequences
  — mapping (bigmap-map) must precede RPKM computation
- The goal is taxonomic/species-level profiling rather than MGC/GCF metabolic gene
  cluster abundance
- No gene cluster predictions exist yet — antiSMASH/gutSMASH must first be run and
  clusters deduplicated via bigmap-family before abundance can be profiled
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: BiG-MAP
  role: bigmap-map module maps reads to representative gene cluster sequences and
    computes the RPKM abundance profiles; bigmap-analyse performs the differential
    abundance testing on the normalized values
  repo: github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: Second redundancy-filtering pass in bigmap-family grouping clusters into GCFs
    so RPKM is aggregated at the family level and ambiguous mapping is avoided
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: ncbi/sra-tools
  role: fastq-dump conversion of SRA samples into FASTQ reads that become the input
    to mapping and RPKM quantification
  repo: github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:be0d094fce3a5b47e78d9792c174ccfc5163e3049f0b38c351e16e0a7c839ab7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - bigmap-analyse
  - bigmap-map
  - gutSMASH
  - ncbi/sra-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# rpkm-normalization

## Summary

Computes RPKM (Reads Per Kilobase per Million mapped reads) abundance profiles of metabolic gene clusters (MGCs) and gene cluster families (GCFs) from mapped metagenomic/metatranscriptomic reads using BiG-MAP, enabling cross-sample comparison and differential abundance testing (e.g. Kruskal-Wallis between healthy and caries-associated oral samples).

## When to use

Use when WGS/metatranscriptomic reads have been mapped to MGC/GCF reference sequences (BiG-MAP.GCF.fna) and per-sample cluster abundance must be normalized as RPKM before phenotype-level differential analysis.

## When NOT to use

- Input is still raw FASTQ that has not been mapped to gene cluster reference sequences — mapping (bigmap-map) must precede RPKM computation
- The goal is taxonomic/species-level profiling rather than MGC/GCF metabolic gene cluster abundance
- No gene cluster predictions exist yet — antiSMASH/gutSMASH must first be run and clusters deduplicated via bigmap-family before abundance can be profiled

## Inputs

- FASTQ shotgun metagenomic/metatranscriptomic reads downloaded from SRA (fastq-dump output)
- BiG-MAP.GCF.fna reference file of representative gene cluster sequences
- BiG-MAP.GCF.bed bedfile of core regions to extract in bigmap-map
- BiG-MAP.GCF.json / BiG-MAP.GCs.json GCF membership dictionaries from bigmap-family

## Outputs

- Per-sample RPKM abundance profiles of MGCs/GCFs across all samples (e.g. 96 oral metagenomes)
- Housekeeping gene family abundance profiles
- Differential abundance statistics (e.g. Kruskal-Wallis P values) with phenotype comparison
- Per-sample core coverage values for presence/absence calls (e.g. 0.77 mean coverage among healthy samples with coverage >= 0.5)

## How to apply

Map the downloaded shotgun reads (FASTQ, retrieved from SRA via bigmap-download) against the representative gene cluster sequences (BiG-MAP.GCF.fna) with bigmap-map, using BiG-MAP.GCF.bed to extract core regions. Normalize mapped read counts into RPKM abundance profiles per MGC/GCF per sample so clusters of different lengths are comparable; housekeeping gene families (HGFs, similarity threshold -th 0.1) can be tracked in parallel as internal profiles. Redundancy filtering (MinHash similarity cut-off 0.8 by default, optionally a second BiG-SCAPE-based pass) must precede mapping to avoid ambiguous read assignment inflating RPKM. Feed the RPKM matrix to bigmap-analyse for nonparametric differential testing (e.g. Kruskal-Wallis between healthy and diseased phenotypes) and interpret cluster presence alongside coverage (e.g. the 0.5 core-coverage threshold used for the Streptococcus sp. F0442 pdu operon: healthy mean 5.62 RPKM vs diseased 3.38 RPKM, P = 0.00049).

## Related tools

- **BiG-MAP** (bigmap-map module maps reads to representative gene cluster sequences and computes the RPKM abundance profiles; bigmap-analyse performs the differential abundance testing on the normalized values) — github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Second redundancy-filtering pass in bigmap-family grouping clusters into GCFs so RPKM is aggregated at the family level and ambiguous mapping is avoided) — https://github.com/medema-group/BiG-SCAPE/wiki
- **ncbi/sra-tools** (fastq-dump conversion of SRA samples into FASTQ reads that become the input to mapping and RPKM quantification) — github.com/ncbi/sra-tools

## Evaluation signals

- RPKM matrix must have one normalized value per MGC/GCF per sample (e.g. 96 oral samples) with no unnormalized read-count columns leaking into the analysis
- Reproducing the pdu operon benchmark: healthy samples average 5.62 RPKM vs 3.38 RPKM for diseased samples, Kruskal-Wallis P = 0.00049
- Cluster-level counts match the published scale: 246 differentially abundant clusters (whole set) and 173 (core); 252 gene clusters significantly more abundant in either phenotype in the article's oral analysis
- Core coverage check consistent with the pdu operon verification: mean core coverage 0.77 among healthy samples with coverage >= 0.5, and 13 of 34 healthy samples below the 0.5 threshold
- Mapping used deduplicated representative clusters only (MinHash 0.8 / BiG-SCAPE GCF representatives), confirmed by BiG-MAP.GCF.fna and the GCF json dictionaries

## Limitations

- BiG-MAP has only been tested on Ubuntu; other platforms are unsupported per the README
- Ambiguous reads between highly similar clusters can distort RPKM unless MinHash (0.8 default) and BiG-SCAPE redundancy filtering are applied first
- RPKM abundance alone does not guarantee full-cluster presence — the pdu operon analysis required an additional core-coverage threshold (0.5), with 13/34 healthy samples falling below it
- For unpaired ancient read data, an extra tool (filterBAM) must be installed separately

## Evidence

- [abstract] This report presents a new software tool called BiG-MAP, which allows assessing gene cluster abundance and expression in microbiome samples using metagenomic and metatranscriptomic data.: "This report presents a new software tool called BiG-MAP, which allows assessing gene cluster abundance and expression in microbiome samples using metagenomic and metatranscriptomic data."
- [intro] by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies: "by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies"
- [intro] Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests.: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [intro] BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19): "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] BiG-MAP.GCF.fna = Reference file to map the WGS reads to: [evidence span withheld — anchor status: external_doc]
- [readme] 3) Assess gene cluster (and HGF representatives) abundance and expression profiles using bigmap-map: "Assess gene cluster (and HGF representatives) abundance and expression profiles
   using bigmap-map"
